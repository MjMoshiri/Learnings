---
title: "Microservices Architecture"
tags: [ "distributed-systems", "architecture", "microservices" ]
prerequisites:
  - "concepts/distributed-systems/sidecar.md"
  - "concepts/distributed-systems/ambassador.md"
status: "complete"
last_updated: "2025-09-07"
todos:
  - "Add a diagram comparing monolithic and microservices architectures."
---

## Summary

Microservices is an architectural style that structures an application as a collection of loosely coupled, independently deployable services. Each service is responsible for a specific business capability and communicates with other services over a well-defined API. This approach, championed by figures like Brendan Burns in the context of container-based systems, offers increased modularity, scalability, and flexibility compared to traditional monolithic architectures.

## Core Concepts

-   **Single Responsibility Principle:** Each microservice is designed around a specific business capability and has a single responsibility. This makes the service easier to understand, develop, and maintain.
-   **Decentralization:** Microservices are decentralized, both in terms of technology and data management. Teams can choose the best technology stack for their service, and each service manages its own data.
-   **Independent Deployability:** Each microservice can be deployed independently of other services. This allows for faster and more frequent deployments, as changes to one service do not require rebuilding and redeploying the entire application.
-   **Containerization:** In modern distributed systems, microservices are often packaged and deployed as containers. This provides a consistent environment for the services and simplifies deployment and management.

## Use Cases

-   **Good for:**
    -   **Large, Complex Applications:** Breaking down a large application into smaller, manageable services.
    -   **Scalability:** Scaling individual services independently based on their specific needs.
    -   **Technology Diversity:** Allowing teams to use different technology stacks for different services.
-   **Not for:**
    -   **Small, Simple Applications:** The overhead of managing a distributed system can be significant, and a monolithic architecture might be simpler for small applications.
    -   **Applications Requiring Strong Consistency:** Maintaining data consistency across multiple services can be challenging.

## References

-   [Designing Distributed Systems by Brendan Burns](https://www.oreilly.com/library/view/designing-distributed-systems/9781491983638/)
