---
title: "Time Series Databases"
tags: [ "time-series", "databases", "observability" ]
prerequisites: []
status: "learning"
last_updated: "2025-02-15"
todos:
  - "Capture a diagram showing ingest, storage, and query paths in a reference TSDB."
  - "Compare retention and schema management strategies across InfluxDB, TimescaleDB, and ClickHouse."
  - "Add notes on budgeting for storage tiers (hot vs. cold) in large deployments."
---

## Summary

Time Series Databases (TSDBs) are purpose-built data stores optimized for handling sequences of timestamped measurements, events, and metrics at scale. They focus on fast ingest of append-only data, efficient compression of time-ordered records, and query primitives tuned for temporal analysis (aggregation, downsampling, rollups). Their architecture balances write-heavy workloads, high-cardinality metadata, and tiered retention policies to power observability, IoT telemetry, financial tick data, and any domain where the passage of time is a first-class dimension.

## Core Concepts

-   **Data Model:** TSDBs treat time as the primary index. Records are typically modeled as `(timestamp, series identifier/tags, fields)`. Tags (or labels) provide dimensions for filtering and grouping, while fields hold numeric or alphanumeric measurements. Schemas can be flexible (key-value style) or table-like (SQL extensions) depending on the product.
-   **Ingestion Pipeline:** Writes are append-only and often buffered in memory (Write-Ahead Logs, in-memory queues) before being compacted into immutable segments. Batching, deduplication, and ordering guarantees ensure chronological consistency even under high ingestion rates.
-   **Storage Engine & Compression:** Data is chunked by time into segments (e.g., shards, partitions, hypertables). Column-oriented layouts, delta-of-delta encoding, run-length encoding, and Gorilla-style compression exploit temporal locality to reduce storage costs while keeping sequential reads fast.
-   **Indexing & Cardinality Management:** Tag/label indexes map series identifiers to stored chunks. To avoid unbounded metadata growth, TSDBs apply cardinality limits, inverted indexes, bitmap compression, or hierarchical label stores. Some systems pre-compute materialized views or leverage bloom filters to skip irrelevant shards.
-   **Query Layer:** Query languages emphasize time-bounded scans, resampling (downsampling), aggregation, joins across series, and anomaly detection functions. Interfaces range from SQL extensions (TimescaleDB), to purpose-built DSLs (PromQL, Flux), to REST/gRPC APIs for analytic workloads.
-   **Retention & Tiering:** Hot data lives on fast storage for interactive queries, while older data is compacted, downsampled, or offloaded to cheaper tiers (object storage, columnar warehouses). Retention policies can delete, summarize, or archive data automatically based on age or storage budget.
-   **Observability Integrations:** Native support for metrics scrapers, agent SDKs, and alerting engines (e.g., Prometheus remote-write, OpenTelemetry) provide ecosystem glue. Stream processors often sit upstream to enrich or normalize telemetry before it lands in the TSDB.

## Use Cases

-   **Good for:** Platform observability (metrics, traces, logs), IoT device telemetry, industrial sensor monitoring, algorithmic trading tick streams, energy grid instrumentation, smart city data hubs.
-   **Not for:** Write-heavy transactional workloads requiring ACID row-level updates, ad-hoc relational analytics with complex joins, datasets where time is not a dominant query dimension, or workloads with highly mutable records.

## References

-   [TimescaleDB Documentation](https://docs.timescale.com/)
-   [InfluxDB Design Principles](https://docs.influxdata.com/influxdb/v2/reference/key-concepts/design-principles/)
-   [Prometheus TSDB Architecture](https://prometheus.io/docs/prometheus/latest/storage/)
-   [Gorilla: Facebook's Time Series Database](https://www.vldb.org/pvldb/vol8/p1816-teller.pdf)
