---
title: "Scatter-Gather Pattern"
tags: [ "distributed-systems", "design-patterns", "performance" ]
prerequisites:
  - "concepts/distributed-systems/microservices.md"
status: "complete"
last_updated: "2025-09-07"
todos:
  - "Add a diagram illustrating the scatter-gather pattern."
---

## Summary

The Scatter-Gather pattern is a design pattern for distributed systems that improves performance by parallelizing requests. A central "root" node receives a request, "scatters" it to multiple "leaf" nodes, and then "gathers" the partial results from the leaves to form a single, comprehensive response. This pattern is particularly effective when a request can be broken down into independent sub-tasks that can be processed in parallel.

## Core Concepts

-   **Parallelization:** The core idea of the scatter-gather pattern is to parallelize the processing of a request by distributing it to multiple nodes. This can significantly reduce the overall response time compared to processing the request sequentially.
-   **Root and Leaf Nodes:** The pattern consists of a root node that receives the initial request and distributes it, and multiple leaf nodes that perform the actual processing. The root node is also responsible for aggregating the results from the leaf nodes.
-   **Computational Sharding:** The scatter-gather pattern can be seen as a form of computational sharding, where the processing required to fulfill a request is divided among multiple nodes, rather than sharding the data itself.

## Use Cases

-   **Good for:**
    -   **Search Engines:** A search query can be sent to multiple search indexes in parallel, and the results can be aggregated to provide a comprehensive search result.
    -   **Data Aggregation:** Aggregating data from multiple sources, such as different microservices or databases.
    -   **Real-time Bidding:** In advertising technology, a bid request can be sent to multiple bidders in parallel, and the highest bid is selected.
-   **Not for:**
    -   **Sequential Tasks:** If the sub-tasks of a request are not independent and need to be processed sequentially, the scatter-gather pattern is not a good fit.
    -   **High Overhead:** The overhead of scattering the request, gathering the results, and managing the network communication can be significant. For simple requests, the overhead might outweigh the benefits of parallelization.

## References

-   [Designing Distributed Systems by Brendan Burns](https://www.oreilly.com/library/view/designing-distributed-systems/9781491983638/)
