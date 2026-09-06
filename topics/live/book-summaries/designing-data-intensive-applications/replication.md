---
topic: book-summaries
status: wip
tags: [systems, data, replication, distributed-systems]
---

# Ch 6 — Replication

### Why replicate at all

Keep copies of the same data on multiple machines. You get lower latency (data near users), availability (survive a node or datacenter dying), read throughput (more machines serving reads), and offline operation (device keeps a local copy). None of this is hard for static data. All the difficulty is in handling changes to replicated data.

### Single-leader replication

One replica is the leader; writes go only to it. It appends changes to a replication log; followers apply the log in the same order. Reads can go to any replica. This is the default in Postgres, MySQL, and most managed databases.

Sync vs async is the core knob:

- Synchronous: leader waits for a follower to confirm before acking the client. The follower has the write, but one slow or dead follower blocks all writes.
- Asynchronous: leader acks immediately. Fast, but if the leader dies before followers catch up, acked writes are gone.
- Semi-synchronous: exactly one follower is sync, the rest async. You always have an up-to-date copy on two nodes without waiting on everyone. Usual compromise.

Fully sync across all followers is impractical. Fully async is what most setups actually run, trading durability for latency.

### New followers and failed nodes

Adding a follower needs no downtime. Take a consistent snapshot of the leader, copy it over, then replay the replication log from the snapshot's position until caught up. A crashed follower recovers the same way, from its own last log position.

Leader failure means failover: detect the leader is dead (usually a timeout), promote a replica (ideally the most up-to-date one), repoint clients and followers. Every step can go wrong:

- With async replication the new leader may be missing acked writes. Discarding them violates durability promises.
- Discarding writes is dangerous when other systems saw them. GitHub once promoted a stale MySQL follower. It reused auto-increment primary keys the old leader had already handed out, and a Redis cache keyed by those ids served private data to the wrong users.
- Split brain: two nodes both think they're leader, both accept writes, data diverges.
- The detection timeout is a trade-off. Too short and you fail over on a load spike, making things worse. Too long and you eat a longer outage.

Failover is fragile enough that some teams keep it manual.

### How the replication log is built

- Statement-based: ship the SQL text. Breaks on nondeterminism (NOW(), RAND(), auto-increments, triggers). Mostly historical.
- WAL shipping: ship the storage engine's write-ahead log. Works, but the log describes disk blocks, so leader and follower must run the same storage engine version. That blocks zero-downtime upgrades, since you can't run a newer follower against an older leader.
- Logical (row-based): a log of row insert/update/delete, decoupled from storage internals. Different versions, even different engines, interoperate. Also what change data capture reads to feed external systems.
- Trigger-based: application triggers write changes to a table an external process reads. Maximum flexibility, maximum overhead and bugs.

### Replication lag and its anomalies

With async followers a read can hit a replica that hasn't caught up. Eventually all replicas converge ("eventual consistency"), but "eventually" is unbounded, and the anomalies in between have names:

- Read-your-own-writes: user writes, refreshes, and their write is gone because the read hit a stale follower. Fix: read from the leader for anything the user may have modified, or remember the client's last-write timestamp and only read from replicas caught up to it. Cross-device makes this harder, since the timestamp lives on one device.
- Monotonic reads: user reads from a fresh replica, then a staler one, and sees data move backward in time. Fix: pin each user to one replica (hash of user id).
- Consistent prefix reads: an observer sees an answer before the question it responds to, because causally related writes landed on different shards with different lag. Guarantee that writes are read in the order they happened. Easy in a single shard, hard across shards.

If the app can't live with those, you need stronger guarantees (transactions, or consensus), not hand-tuned workarounds.

### Multi-leader replication

More than one node accepts writes; each leader is also a follower of the others. Rarely worth it inside one datacenter.

- Multi-datacenter: one leader per DC. Writes ack locally (lower latency), each DC survives the other's outage, and inter-DC replication is async in the background.
- Offline clients: your phone's calendar is a leader with a very long replication lag.
- Collaborative editing: every editor's local copy is a leader.

The price is write conflicts: two leaders accept concurrent writes to the same record and only notice when they replicate. Options:

- Avoid conflicts: route all writes for a given record to the same leader. Solves most of the problem. Usual recommendation.
- Last write wins: keep the write with the biggest timestamp, drop the rest. Simple, silently loses data.
- Keep both versions and let the application or user merge.
- CRDTs: data structures that merge concurrent updates automatically with guaranteed convergence.
- Operational transformation: the collaborative-text-editing algorithm family (Google Docs).

Circular and star topologies have single points of failure in the forwarding path. All-to-all handles failures better, but messages can arrive out of causal order, which needs version vectors to fix.

### Leaderless replication

Dynamo-style: Cassandra, ScyllaDB, Riak. No leader. The client or a coordinator sends writes to all n replicas in parallel and calls the write successful after w acks. Reads query several replicas and take r responses. If w + r > n, every read overlaps at least one replica with the newest write (a quorum). Typical: n=3, w=2, r=2.

Stale replicas catch up two ways:

- Read repair: a read that sees both new and old values writes the new value back to the stale replica.
- Anti-entropy: a background process diffs replicas and copies missing data.

Sloppy quorum and hinted handoff: during a partition, accept writes on any reachable nodes (not the designated n "home" nodes) and hand them off when the home nodes return. Buys write availability, but breaks the quorum overlap, so w + r > n no longer implies you read the latest value.

Even strict quorums aren't airtight. Concurrent writes, writes that succeeded on fewer than w nodes but weren't rolled back, and timing edge cases all allow stale reads. Quorums give you "probably fresh," not linearizability.

### Detecting concurrent writes

Two writes are concurrent when neither knew about the other, regardless of wall-clock time. What matters is the happens-before relation: if B builds on A, B supersedes A; if neither does, they conflict and both survive as siblings until something merges them.

Mechanics: each key carries a version number per replica, and the set of them is a version vector. The client sends back the version vector it read, so the server can tell "overwrite" from "concurrent." Merging siblings is the application's problem, or a CRDT's. LWW is the lazy option, and it drops writes. Cassandra defaults to it.

### General notes

Most decisions here reduce to two questions: who can accept a write (one node, several, any), and what you're willing to lose or expose when replication is behind (durability on failover, staleness on reads, conflicts on merge). Pick the anomaly you can live with. There's no option with none.

back to [[encoding-and-evolution]] for Ch 5.
