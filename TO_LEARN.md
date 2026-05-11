# To Learn

Backlog of topics that surfaced while writing notes. When a topic is covered, add the note under the right `notes/<topic>/` folder, hyperlink the source document to it, and remove the row from the table.

| Topic | Source | Notes |
| :--- | :--- | :--- |
| Idempotency | Suggested | Ensures repeated API requests do not result in duplicate transactions. |
| ISO 8583 | Suggested | International standard for financial transaction card messages. |
| Payment Tokenization | Suggested | Replaces sensitive card data with a non-sensitive equivalent token. |
| Strangler Fig Pattern | Suggested | Strategy for incrementally migrating legacy systems to modern architectures. |
| Zero-Copy | [`kafka.md`](./products/kafka.md) | Key performance optimization in Kafka's log architecture. |
| Sequential vs. Random I/O | [`kafka.md`](./products/kafka.md) | Explains Kafka's high write throughput. |
| Hadoop | [`netflix-kafka.md`](./case-studies/netflix-kafka.md) | |
| Mantis | [`netflix-kafka.md`](./case-studies/netflix-kafka.md) | |
| Spark | [`netflix-kafka.md`](./case-studies/netflix-kafka.md) | |
| Delta-of-Delta Compression | [`time-series-databases.md`](./concepts/databases/time-series-databases.md) | Time-series encoding that compresses numeric streams by storing differences of differences. |
| Run-Length Encoding | [`time-series-databases.md`](./concepts/databases/time-series-databases.md) | Technique for compressing repeated values within time buckets. |
| Gorilla Compression | [`time-series-databases.md`](./concepts/databases/time-series-databases.md) | Facebook metric format combining XOR and adaptive encoding for doubles. |