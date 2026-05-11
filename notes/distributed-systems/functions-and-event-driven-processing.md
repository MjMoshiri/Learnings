---
title: "Functions and Event-Driven Processing"
tags: [ "distributed-systems", "serverless", "event-driven-architecture" ]
prerequisites:
  - "concepts/messaging/pub-sub.md"
status: "draft"
last_updated: "2025-09-10"
todos:
  - "Explore different FaaS (Function-as-a-Service) platforms."
  - "Add a sequence diagram showing the flow of an event-driven process."
---

## Summary

Event-driven processing is a paradigm where the flow of the application is determined by events. An event is a significant change in state. In a distributed system, this often involves using functions (as in FaaS or serverless computing) that are triggered by these events. This creates a loosely coupled system where services can react to changes in other parts of the system without being directly called.

## Core Concepts

-   **Events:** Immutable facts about something that has happened. They are the trigger for computation.
-   **Functions:** Small, single-purpose pieces of code that execute in response to an event. They are often stateless.
-   **Event Bus/Stream:** A central channel where events are published and from which functions can subscribe to be triggered.

## Use Cases

-   **Good for:** Asynchronous workflows, real-time data processing, and building highly scalable and resilient systems. Ideal for tasks that can be broken down into small, independent steps.
-   **Not for:** Long-running, stateful computations. Workflows that require complex coordination between different steps.

## References

-   [Designing Distributed Systems by Brendan Burns](https://www.oreilly.com/library/view/designing-distributed-systems/9781491983638/)
