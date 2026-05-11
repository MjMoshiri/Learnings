---
title: "Sidecar Pattern"
tags: [ "distributed-systems", "design-patterns", "containers", "kubernetes" ]
prerequisites:
  - "concepts/messaging/pub-sub.md"
status: "complete"
last_updated: "2025-09-07"
todos:
  - "Add a diagram illustrating the sidecar pattern."
---

## Summary

The Sidecar pattern is a container-based design pattern for distributed systems that involves deploying a secondary "sidecar" container alongside a primary application container. This pattern promotes modularity and separation of concerns by offloading auxiliary tasks like logging, monitoring, and security to the sidecar, allowing the main application to focus on its core logic. The two containers share the same lifecycle and resources, enabling efficient communication.

## Core Concepts

-   **Separation of Concerns:** The primary benefit of the sidecar pattern is the separation of concerns. The main application container runs the core business logic, while the sidecar handles cross-cutting concerns like observability, security, and network services. This leads to smaller, more focused, and more maintainable containers.
-   **Shared Resources:** Sidecar containers are deployed in the same unit (e.g., a Kubernetes Pod) as the main application container. They share the same network namespace, storage volumes, and lifecycle. This allows for low-latency communication between the containers, often over `localhost` or through a shared file system.
-   **Reusability:** Sidecar containers can be developed and maintained independently and can be reused across multiple applications. For example, a standardized logging sidecar can be deployed with different microservices to ensure consistent log formatting and shipping.

## Use Cases

-   **Good for:**
    -   **Modernizing Legacy Applications:** Adding functionality like SSL/TLS termination to legacy applications without modifying their code.
    -   **Centralizing Cross-Cutting Concerns:** Managing logging, monitoring, and security in a consistent way across multiple microservices.
    -   **Platform Abstraction:** Abstracting away platform-specific details like service discovery or circuit breaking from the application.
-   **Not for:**
    -   **Tightly Coupled Logic:** If the logic in the sidecar is tightly coupled to the main application's logic, it might be better to include it in the main application itself.
    -   **Performance-Critical Applications:** The additional container and network hop (even over localhost) can introduce a small amount of latency, which might not be acceptable for highly performance-critical applications.

## References

-   [Designing Distributed Systems by Brendan Burns](https://www.oreilly.com/library/view/designing-distributed-systems/9781491983638/)
