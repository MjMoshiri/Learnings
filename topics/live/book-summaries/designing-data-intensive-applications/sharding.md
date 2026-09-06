---
topic: book-summaries
status: wip
tags: [systems, data, sharding, distributed-systems]
---

# Ch 7 — Sharding

### Why shard

Shard when one machine can't hold the data or take the write throughput: split the dataset and give each node a subset. Same idea under many names: partition, tablet (Bigtable), region (HBase), vnode (Cassandra). Sharding sits on top of replication. Each record lives in exactly one shard; each shard is copied across several nodes. It complicates routing, rebalancing, and indexes. The other reason to shard is multitenancy: one shard per tenant gives you isolation, per-tenant deletes (nice for GDPR), and per-tenant cost accounting.

### Skew and hot spots

An unfair split is skew. A shard taking disproportionate load is a hot spot. Worst case, every write lands on one shard and sharding bought you nothing.

### Key-range sharding

Each shard owns a contiguous sorted range of keys. You always know which shard holds a key, and range scans are cheap. HBase and MongoDB in range mode work this way. The weakness is the access pattern: with timestamp keys the newest range takes every write. Prefix the key with something that spreads writes, sensor name before timestamp, and pay one scan per prefix. Rebalancing is dynamic here: split shards as they grow, merge them as they shrink, and pre-split a fresh database so it doesn't start life on a single node.

### Hash sharding

Hash the key and shard the hash space instead. A decent hash spreads skewed keys evenly and kills most hot spots, but range scans over the original keys are gone, since neighboring keys land far apart. Cassandra's compromise is compound keys: hash the first column to place the row, sort by the remaining columns inside the shard, so single-partition range scans still work.

Three ways to carve up the hash space:

- Fixed shard count: create far more shards than nodes up front, say 1,000 for 10 nodes, and give each node several. Rebalancing moves whole shards and the key-to-shard mapping never changes. Simple, but the count is frozen at setup and hard to guess before you know how big the dataset gets.
- Hash-range splitting: the same dynamic split and merge as key-range sharding, applied to hash values.
- Consistent hashing: a family of algorithms whose assignments barely move when nodes come and go. The ring version puts nodes at positions on a hash circle and each key belongs to the next node clockwise; many positions per node (vnodes) even out the load, Cassandra-style. The table-free versions are smarter: rendezvous hashing scores hash(node, key) for every node and the highest score wins, and jump consistent hash computes the shard straight from the key and the shard count in a few lines of arithmetic, no lookup table. Either way, a node joining moves only about 1/n of the keys.

Never hash mod N. One new node reshuffles nearly every key.

### Hot keys

Hashing does nothing for a single hot key, a celebrity user or a viral post, since all its writes hash to the same shard anyway. Fix: append a random two-digit suffix so writes spread over 100 keys on different shards; reads then fan out over all 100 and merge. Worth it only for keys you already know are hot, and something has to track which keys got the treatment.

### Rebalancing operations

Moving shards saturates disk and network while production traffic keeps arriving. Fully automatic rebalancing plus automatic failure detection is a cascade waiting to happen: an overloaded node answers slowly, gets declared dead, and the rebalancer dumps its data on neighbors that are already struggling. The common stance is that automation proposes and a human approves.

### Request routing

Who knows which node owns key X? Three answers. Send to any node and let it forward when it doesn't own the key (Cassandra, over gossip). Put a routing tier in front that tracks the assignment (MongoDB's mongos, backed by config servers). Or let clients hold the mapping themselves. Keeping that mapping authoritative while shards move is a consensus problem, usually handed to ZooKeeper or etcd (HBase, Kafka, SolrCloud) or spread by gossip.

### Secondary indexes

The hard part. A local index (document-partitioned) covers only the records on its own shard, so a write touches one shard, but a read scatters to every shard and merges, which wrecks tail latency. It's still the default in MongoDB, Cassandra, and Elasticsearch. A global index (term-partitioned) is sharded by term, so a term read hits one index shard, but writing one record touches an index shard per term, which in practice means async updates and an index that lags the data. Local favors writes, global favors reads; pick by workload.

### General notes

Every scheme juggles three things: even load, cheap rebalancing, and key order for range scans. Nothing gets all three.

back to [[replication]] for Ch 6.
