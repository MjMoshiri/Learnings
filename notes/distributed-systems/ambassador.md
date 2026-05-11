---
title: "Ambassador Pattern"
tags: [ "distributed-systems", "design-patterns", "containers", "kubernetes" ]
prerequisites:
  - "concepts/distributed-systems/sidecar.md"
status: "complete"
last_updated: "2025-09-07"
todos:
  - "Add a diagram illustrating the ambassador pattern."
---

## Summary

The Ambassador pattern is a container-based design pattern for distributed systems where a separate "ambassador" container proxies communication between an application container and the outside world. This pattern simplifies the application code by offloading complex network-related tasks like service discovery, routing, and circuit breaking to the ambassador. The application communicates with the ambassador as if it were a local service, and the ambassador handles the complexities of connecting to external services.

## Core Concepts

-   **Proxying and Offloading:** The ambassador container acts as a proxy, handling tasks like service discovery, routing, circuit breaking, and monitoring. This offloads complex network-related logic from the main application container, allowing developers to focus on the core business logic.
-   **Simplified Communication:** From the application's perspective, it communicates with the ambassador as if it were a local service running on the same machine. The ambassador then handles the complexities of connecting to external services, which might be sharded, replicated, or dynamically located within the cluster.
-   **Language and Library Independence:** By abstracting away communication details, the ambassador pattern allows different services to be written in different languages and use different libraries without needing to share common communication code.

## Use Cases

-   **Good for:**
    -   **Service Discovery:** Simplifying how an application connects to other services, especially in a dynamic environment like Kubernetes.
    -   **Resiliency:** Implementing patterns like circuit breaking and retries in a language-agnostic way.
    -   **Sharding:** Connecting to a sharded data store without the application needing to know the location of all the shards.
-   **Not for:**
    -   **High-Performance, Low-Latency Applications:** The extra network hop to the ambassador can add latency.
    -   **Simple Applications:** For simple applications with a small number of external dependencies, the complexity of an ambassador might be overkill.

## References

-   [Designing Distributed Systems by Brendan Burns](https://www.oreilly.com/library/view/designing-distributed-systems/9781491983638/)
