---
topic: book-summaries
status: wip
tags: [systems, data, storage-engines, indexes]
---

# Ch 4 — Storage and retrieval

### Two kinds of storage engines

OLTP engines serve transactional workloads, lots of small reads and writes. OLAP engines serve analytics, big scans over many rows.

### Log-structured vs update-in-place

Log-structured engines only append. They never overwrite a value in place. SSTables, LSM-trees, RocksDB, Cassandra, HBase, ScyllaDB, and Lucene all work this way. Update-in-place engines use B-trees: fixed-size pages overwritten on disk, the design behind every major relational DB. Rule of thumb: B-trees are better for reads, log-structured is better for write throughput.

### SSTable

Sorted String Table. A key-value segment file sorted by key. Sorting buys you three things. You can merge segments like mergesort. You only need a sparse in-memory index, one key every few KB, instead of indexing every key. And range scans come for free.

### Tombstone

You can't delete in an append-only log. A delete writes a special marker called a tombstone. During compaction the tombstone shadows older values for that key, and the data drops once it merges past.

### Bloom filter

A probabilistic set membership check. It tells you "definitely not here" or "maybe here". That saves the LSM-tree from hitting disk when you read a key that doesn't exist. False positives happen, false negatives never.

### Column-oriented storage

Row stores keep a record's fields together. Column stores keep all values of one column together. Analytics scans a few columns across millions of rows, so the column layout reads only what the query touches. Compression also works well because the values in one column look alike.

### Bitmap encoding

A compression trick for low-cardinality columns. One bitmap per distinct value, one bit per row. Sparse bitmaps get run-length encoded. A `WHERE col IN (...)` turns into a bitwise OR across the bitmaps, which is cheap.

### Query engine and query plan

The optimizer takes a declarative query and turns it into an execution plan. It picks the access paths and the join order.

### Compilation vs vectorization

Two ways to make analytical queries fast on the CPU. Compilation generates machine code for the specific query with a JIT. Vectorization processes columns in batches through tight loops that stay in CPU cache and use SIMD, instead of handling one row at a time.

### Concatenated index

An index on several columns joined into one key, like (lastname, firstname). Order matters. It's good when the query fixes the leading columns. It's useless if you only filter on a trailing column.

### Multidimensional index

Concatenated indexes fall apart when you query several dimensions at once, like lat AND lng. R-trees and space-filling curves index multiple dimensions together, so you can run range queries over a map or a 2D area.

### Inverted index

Maps each term to the list of documents that contain it. This is the core of full-text search like Lucene. Searching for multiple keywords means intersecting the posting lists.

### IVF index for vectors

Inverted file index. Partition the vector space into clusters with centroids. At query time you only search the few clusters nearest the query vector instead of every vector. You trade recall for speed.

### HNSW

Hierarchical Navigable Small World, a graph-based approximate nearest-neighbor index. It's a multi-layer graph: sparse long-range links on top, dense local links at the bottom. Search walks greedily from the top layer down toward the nearest neighbor. Fast approximate search for vector and semantic similarity.

back to [[data-models-and-query-languages]] for Ch 3.
