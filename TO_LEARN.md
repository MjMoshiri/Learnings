# To Learn

**Workflow Guide:** When a topic from this list is learned and a document is created, please do the following:
1.  **Hyperlink the Concept:** In the source document where the topic was mentioned, update the text to hyperlink to the newly created local document.
2.  **Remove from List:** Remove the corresponding row from the table below.
3.  **Index the New Document:** Run `python3 knowledge_base.py add <path_to_new_doc.md>` to add it to the knowledge base.

This file tracks concepts and topics that have emerged during our discussions and are candidates for deeper exploration.

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