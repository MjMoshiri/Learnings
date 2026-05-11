---
title: "Command Query Responsibility Segregation (CQRS)"
tags: [ "architecture-pattern", "distributed-systems", "data-management" ]
prerequisites:
  - "concepts/distributed-systems/microservices.md"
status: "learning"
last_updated: "2025-09-14"
todos:
  - "Add a diagram illustrating the separation of commands and queries."
  - "Include a code example of a simple CQRS implementation."
  - "Contrast CQRS with the traditional CRUD model in more detail."
---

## Summary

CQRS, or Command Query Responsibility Segregation, is an architectural pattern that separates the models for reading (Query) and writing (Command) data. Instead of using a single, unified model for both operations (as is common in CRUD-based systems), CQRS establishes two distinct models: a write model optimized for business logic and data consistency, and a read model optimized for efficient querying. This separation allows for independent scaling, optimization, and security of read and write operations, making it particularly useful for complex systems with high performance requirements or imbalanced read/write loads.

## Core Concepts

-   **Commands:** A command is an instruction to change the state of the system (e.g., `CreateUser`, `UpdateOrderStatus`). Commands are imperative, task-based, and should not return data. They are handled by the write model, which contains the business logic to validate and process the command. The focus of the write model is to enforce business rules and ensure transactional consistency.
-   **Queries:** A query is a request for data that does not alter the state of the system (e.g., `GetUserById`, `GetRecentOrders`). Queries are handled by the read model and return Data Transfer Objects (DTOs). The read model can be a denormalized view of the data, specifically tailored for a particular UI component or client, which simplifies querying and improves performance. It contains no business logic.
-   **Data Synchronization:** The read and write models often operate on separate data stores. When the write model processes a command, it must eventually update the read model. This is typically done asynchronously using events, which leads to **eventual consistency**. For example, after a `UserCreated` event is processed by the write side, a handler updates the read database to include the new user.

## Implementation Patterns & Example

Consider an e-commerce application. The process of placing an order is complex (checking inventory, validating payment, etc.), but viewing order history is a simple read operation.

-   **Write Model (The Command Side):**
    -   A user action triggers an `PlaceOrderCommand`.
    -   This command is sent to a **Command Handler**, which orchestrates the process.
    -   The handler might load an `Order` aggregate (a concept from Domain-Driven Design) from a relational database (e.g., PostgreSQL). This database is normalized and optimized for transactional integrity (OLTP).
    -   It executes business logic: validates the order, checks inventory, processes payment.
    -   If successful, it saves the new state and publishes an `OrderPlaced` event to a message broker (like RabbitMQ or Kafka).

-   **Read Model (The Query Side):**
    -   An **Event Handler** subscribes to `OrderPlaced` events from the message broker.
    -   When it receives an event, it updates a separate read database. This database could be a NoSQL document store (e.g., Elasticsearch or MongoDB) and contain a denormalized "view" of the order, optimized for fast reads. For example, it might store the order details along with the customer's name and product information in a single document, avoiding complex joins.
    -   When a user views their order history, the UI sends a query to a **Query Handler**, which reads directly from this denormalized read store and returns a simple DTO.

## CQRS and Event Sourcing

CQRS is often used with **Event Sourcing (ES)**, though they are separate patterns.

-   **How they work together:** In an ES system, the state of an entity is not stored directly. Instead, all changes are stored as a sequence of immutable events.
-   **Write Side:** The write model's only job is to validate commands and append new events to the event store (e.g., `OrderCreated`, `OrderItemAdded`, `OrderShipped`). The current state of an order is derived by replaying these events.
-   **Read Side:** The read model is built by subscribing to the stream of events and creating projections (materialized views) from them. This makes the read side highly decoupled and allows for the creation of multiple, diverse read models from the same event stream.

## Use Cases

-   **Good for:**
    -   **High-Performance Applications:** Systems where read and write workloads have different performance and scaling requirements. For example, an e-commerce site with many more product views (reads) than purchases (writes).
    -   **Complex Domains:** Applications with complex business logic that is difficult to manage in a single, unified model.
    -   **Collaborative Environments:** Systems where multiple users are operating on the same data, as the task-based nature of commands can reduce merge conflicts.
    -   **Event-Sourced Systems:** CQRS is a natural fit for event sourcing.

-   **Not for:**
    -   **Simple CRUD Applications:** The added complexity of CQRS is often overkill for simple applications where a single model is sufficient.
    -   **Systems Requiring Strong Consistency:** While not impossible, implementing strong, synchronous consistency between the read and write models adds significant complexity and can negate many of the benefits of the pattern.

## References

-   [Microsoft Docs: CQRS Pattern](https://docs.microsoft.com/en-us/azure/architecture/patterns/cqrs)
-   [Martin Fowler: CQRS](https://martinfowler.com/bliki/CQRS.html)
