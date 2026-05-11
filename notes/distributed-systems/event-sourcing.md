---
title: "Event Sourcing"
tags: [ "architecture-pattern", "distributed-systems", "data-management" ]
prerequisites:
  - "concepts/distributed-systems/cqrs.md"
status: "learning"
last_updated: "2025-02-14"
todos:
  - "Add a sequence diagram showing command handling and event replay."
  - "Include guidance for idempotent event handlers in distributed systems."
  - "Document migration strategies from legacy CRUD models to event sourcing."
---

## Summary

Event Sourcing is an architectural pattern where application state is captured as an append-only log of immutable events rather than as the latest snapshot of data. Each event records a fact about something that happened in the past (e.g., `OrderPlaced`, `PaymentCaptured`). The current state of an aggregate is derived by replaying all relevant events in order. This approach provides a complete audit log, enables temporal queries, facilitates complex recovery scenarios, and supports flexible read models that can be rebuilt or reshaped as requirements evolve.

## Core Concepts

-   **Event Store:** A database optimized for appending and retrieving ordered event streams. It guarantees immutability, supports optimistic concurrency, and often provides stream-specific metadata (version numbers, timestamps, causation/correlation identifiers).
-   **Events:** Domain-specific records that describe something that happened. Events are immutable, versioned, and carry enough information to reconstruct state changes. They should be modeled as facts in the past tense and designed with backward compatibility in mind.
-   **Aggregates:** Consistency boundaries from Domain-Driven Design that enforce invariants during command handling. An aggregate processes a command, validates business rules, and emits new events when changes are valid.
-   **Event Replay:** The process of rebuilding current state by folding (reducing) an aggregate's event stream. Replaying also supports rebuilding projections when schemas change or when new read models are introduced.
-   **Projections / Read Models:** Materialized views built by consuming event streams and shaping them for query workloads. Projections can be rebuilt from scratch, making schema evolution and bug fixes easier.
-   **Eventual Consistency:** Since projections are updated asynchronously, consumers must tolerate lag between the write-side event emission and read-side availability. Techniques like versioning, message acknowledgment, and idempotency help manage consistency.

## Implementation Patterns & Example

Consider a banking application that manages account balances.

-   **Command Handling:**
    -   A customer initiates a `DepositFunds` command via an API.
    -   The command handler loads the `BankAccount` aggregate by replaying its historical events (`AccountOpened`, previous deposits/withdrawals) from the event store.
    -   The aggregate validates the deposit (e.g., ensuring the account is active) and emits a `FundsDeposited` event.
    -   The event store appends the new event with a monotonically increasing version to guard against concurrent updates.

-   **Projection Updates:**
    -   A projection service subscribes to the `FundsDeposited` event stream via a message broker or event store subscription API.
    -   When it receives the event, it updates a read-optimized data store (e.g., a key-value cache or relational table) with the new account balance.
    -   If the projection falls behind or encounters an error, it can rewind to the last acknowledged event and replay forward without data loss.

-   **Temporal Queries and Auditing:**
    -   Operations teams can run a replay to inspect the account state at a specific point in time or reconstruct how a dispute arose.
    -   Because the event log is immutable, the system has an authoritative audit trail for compliance investigations.

## Relationship to CQRS

Event Sourcing and CQRS are often paired but can be adopted independently.

-   **Complementary Fit:** Event sourcing provides a natural history of state changes that the CQRS read side can consume to build projections. CQRS, in turn, offers a clean separation between command processing and read workloads.
-   **Independent Adoption:** You can implement event sourcing without CQRS by exposing queries through the same service that handles commands, and you can implement CQRS with a traditional state persistence model. The combination is most compelling when you need both granular auditability and independently scalable read models.

## Benefits and Trade-Offs

-   **Pros:**
    -   **Auditability & Traceability:** Every state change is captured, enabling forensic analysis and temporal debugging.
    -   **Flexibility:** Projections can evolve, be rebuilt, or be tailored to new consumers without touching the write model.
    -   **Resilience:** Event replay simplifies recovery after failures or schema changes and supports compensating workflows.

-   **Cons:**
    -   **Complexity:** Tooling, storage, and operational maturity are required to manage large event logs and projection rebuilds.
    -   **Learning Curve:** Teams must adopt new patterns for modeling events, handling eventual consistency, and versioning schemas.
    -   **Data Volume:** High event throughput can lead to large logs; compaction or snapshotting strategies may be necessary.

## Use Cases

-   **Good for:**
    -   **Financial Systems:** Ledger-style domains that demand complete audit trails and reversible histories.
    -   **Distributed Workflows:** Microservices coordinating via asynchronous events where replay aids in debugging and recovery.
    -   **Regulated Industries:** Scenarios requiring strict compliance evidence, such as healthcare or insurance claims processing.

-   **Not for:**
    -   **Simple CRUD Apps:** Applications without strong auditing or replay needs may find the overhead unjustified.
    -   **Strong Consistency Requirements:** Real-time transactional consistency across multiple aggregates is harder to guarantee and can negate ES benefits.
    -   **Limited Operational Capacity:** Teams without experience in messaging infrastructure, versioning, and monitoring may struggle to run ES reliably.

## References

-   [Martin Fowler: Event Sourcing](https://martinfowler.com/eaaDev/EventSourcing.html)
-   [Greg Young: A Decade of DDD, CQRS, Event Sourcing](https://leanpub.com/ddd-cqrs-event-sourcing)
-   [Microsoft Docs: Event Sourcing pattern](https://learn.microsoft.com/azure/architecture/patterns/event-sourcing)
