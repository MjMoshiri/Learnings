---
topic: book-summaries
status: wip
tags: [systems, data, transactions, isolation]
---

# Ch 8 — Transactions

### Why transactions exist

A transaction groups several reads and writes into one logical unit that either fully happens or doesn't happen at all. The point is to keep the application from dealing with partial failure, a crash mid-write, and other people mutating the same rows at the same time. Without that grouping, every multi-step write becomes an error-handling problem in application code.

### ACID, and what the letters actually mean

ACID is a marketing bundle more than a spec.

- Atomicity: abortability. If you fail halfway, the database rolls back as if the transaction never ran.
- Consistency: the application-defined invariants hold (balances don't go negative, foreign keys exist, a meeting room has one booking).
- Isolation: concurrently running transactions don't step on each other.
- Durability: once the database says committed, the write survives a crash.

BASE (Basically Available, Soft state, Eventual consistency) was the NoSQL slogan.

### Isolation is a spectrum

Weaker isolation runs faster and allows more anomalies. Stronger isolation aborts or blocks more.

### Read committed

Default in Postgres, Oracle, SQL Server, and many others.

- No dirty reads: you only see committed data. A transaction that later aborts never leaked its writes.
- No dirty writes: you only overwrite committed data. The second writer waits for the first to commit or abort.

Writes take row locks. Reads either wait or, more commonly, see the old value so they never block. Read committed does not stop read skew, lost updates, write skew, or phantoms.

Read skew: Alice has $1000 split across two accounts. A transfer moves $100 from one to the other. Her read sees one account before the transfer and the other after, so the total is $900 or $1100. Each read was committed. Together they weren't a consistent snapshot.

### Snapshot isolation and MVCC

Each transaction reads from a consistent snapshot taken at its start. Later commits don't appear in its reads. Readers never block writers, writers never block readers.

Implemented with multi-version concurrency control (MVCC). Updates write a new version tagged with the transaction id that created it. A reader sees versions committed before its snapshot started, and ignores in-flight ones.

Naming is a mess. SQL's "repeatable read" is not snapshot isolation. Postgres REPEATABLE READ is snapshot isolation. Oracle and MySQL call snapshot isolation SERIALIZABLE or REPEATABLE READ and it isn't serializable.

Snapshot isolation stops dirty reads, dirty writes, and read skew. It does not stop lost updates or write skew.

### Lost updates

Two transactions read the same value, both modify it, both write back. The second write drops the first. Two people increment a counter, two users edit a wiki page.

Fixes:

- Atomic writes: `UPDATE counters SET n = n + 1`. The increment happens inside the database, no read-modify-write in the client.
- Explicit locking: `SELECT ... FOR UPDATE`.
- Automatic detection: abort at commit if the value you read changed. Postgres, Oracle, and SQL Server snapshot isolation do this. MySQL InnoDB's repeatable read does not.
- Compare-and-set: write only if the value is still what you read.
- Replicated databases: merge concurrent writes (CRDTs) or last-write-wins, which drops data.

Atomic operations and CAS only help when the conflict is on the same object.

### Write skew and phantoms

Write skew: two transactions read the same snapshot, both decide their write is safe, and they write different objects, so lost-update detection never fires. Together they break an invariant.

On-call doctors: at least one doctor must stay on call. Two doctors both see two on call, both go off duty, zero remain. Snapshot isolation lets both commit.

Same bug: two people book the same meeting room, two users take the last username, a spend that checks a sum of rows then inserts another.

Phantoms: a write in one transaction changes the result of a query in another. Snapshot isolation won't show you the new rows, but the writes still land.

Materializing conflicts: invent a row both transactions lock, like a lock row per meeting room, when you don't have serializability.

### Serializability

The result is identical to some serial order of the transactions. 

### Actual serial execution

Run every transaction on a single thread, one after another. Only viable if transactions are short, the dataset fits in memory, and one core is enough. VoltDB, Redis, Datomic.

Interactive transactions (BEGIN, app thinks, more queries, COMMIT) kill this: the thread sits idle while the app decides. Stored procedures ship the whole transaction as one function and run it to completion.

Partitioning scales this if each transaction stays on one partition. Cross-partition transactions have to coordinate across threads.

### Two-phase locking (2PL)

Readers take a shared lock, writers take an exclusive lock. A writer waits for readers; a reader waits for a writer. Locks are held until commit or abort (the second phase). First phase acquires, second releases. Not the same as two-phase commit.

Deadlocks happen. The database detects a cycle, aborts one transaction, that one retries.

Predicate locks (or cheaper index-range / next-key locks) cover rows that don't exist yet. That's how 2PL stops phantoms: "any row matching this query" is locked, so an insert that would appear in someone else's search waits.

Under contention, transactions queue. 2PL assumes conflict and blocks.

### Serializable snapshot isolation (SSI)

Transactions run as under snapshot isolation, no extra blocking. The database aborts at commit if it detects a conflict that would violate serializability.

Two detections:

- Stale MVCC read: you read a snapshot, then another transaction committed a write that would have changed what you read. If you also wrote, your premise is outdated.
- Writes that affect prior reads: another in-flight transaction read data you're now writing. One of you has to abort.

Postgres serializable (9.1+) is SSI. FoundationDB too. Abort-and-retry is cheaper than lock-waiting when conflicts are rare. Under high contention you retry a lot and 2PL can win.

### General notes

Snapshot isolation is the usual default, and write skew is the bug it allows. If an invariant spans several objects, you need serializability or you lock those objects yourself. The SQL names lie; read the product docs for the actual anomalies.

back to [[sharding]] for Ch 7.
