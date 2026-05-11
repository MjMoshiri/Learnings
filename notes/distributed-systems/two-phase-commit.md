---
title: Two-Phase Commit (2PC)
summary: An explanation of the Two-Phase Commit protocol, a distributed algorithm for achieving atomic commitment across multiple nodes to ensure data consistency.
---

# Two-Phase Commit (2PC)

The **Two-Phase Commit (2PC)** protocol is a distributed algorithm that ensures all participants in a distributed transaction either all commit or all abort (roll back). It is a foundational protocol for achieving **atomic commitment** across multiple nodes, which is critical for maintaining data consistency in systems like databases, financial transaction platforms, and messaging systems.

The protocol guarantees that a transaction is never left in a partially completed state, where some systems have committed the changes and others have not. This "all-or-nothing" outcome is essential for operations that must be indivisible, such as transferring funds between two different bank accounts.

### Key Roles

The protocol involves two types of nodes:

1.  **Coordinator:** The single node responsible for managing the transaction and making the final decision to commit or abort. It orchestrates the process by communicating with all participants.
2.  **Participants (or Cohorts):** The nodes that execute the transaction and vote on whether they are ready to commit. Each participant manages its own local resources (e.g., a database).

---

### The Two Phases

The protocol operates in two distinct phases, initiated by the coordinator.

#### Phase 1: The Prepare Phase (or Voting Phase)

In this phase, the coordinator asks every participant if it is ready to commit the transaction.

1.  **Prepare Request:** The coordinator sends a `PREPARE` message to all participants.
2.  **Participant Vote:** Upon receiving the `PREPARE` message, each participant determines if it can successfully commit the transaction.
    *   It performs all necessary work, such as validating data, acquiring locks, and writing the transaction's effects to a durable write-ahead log (WAL). This ensures the participant can guarantee the commit if asked later.
    *   If the participant is ready, it responds with a `VOTE-COMMIT` message to the coordinator.
    *   If the participant cannot commit (e.g., due to a constraint violation, disk failure, or optimistic locking conflict), it responds with a `VOTE-ABORT` message.

Crucially, by voting `VOTE-COMMIT`, a participant makes a promise: it is now locked in and cannot unilaterally abort the transaction. It must wait for the coordinator's final decision.

#### Phase 2: The Commit Phase (or Completion Phase)

In this phase, the coordinator makes the final decision based on the votes received from the participants.

The decision rule is as follows:

*   If **all** participants voted `VOTE-COMMIT`, the coordinator decides to **commit**.
*   If **at least one** participant voted `VOTE-ABORT` (or failed to respond), the coordinator decides to **abort**.

The coordinator then communicates this decision to all participants:

1.  **Decision Broadcast:**
    *   The coordinator writes its final decision (`COMMIT` or `ABORT`) to its own log to ensure it can recover from a failure.
    *   It then sends a `GLOBAL-COMMIT` or `GLOBAL-ABORT` message to all participants.
2.  **Participant Action:**
    *   Participants receiving `GLOBAL-COMMIT` proceed to finalize the transaction (e.g., apply the changes from the WAL to the database) and release any locks.
    *   Participants receiving `GLOBAL-ABORT` roll back the transaction (e.g., by discarding the WAL entries) and release locks.
3.  **Acknowledgment:** After completing the action, each participant sends an `ACK` (acknowledgment) message back to the coordinator. The transaction is considered complete once the coordinator has received acknowledgments from all participants.

---

### Failure Handling and Limitations

2PC is effective but has significant drawbacks, primarily related to failure scenarios.

#### Participant Failure

*   **Failure before voting:** If a participant fails before voting, the coordinator will time out and decide to abort the transaction.
*   **Failure after voting `VOTE-COMMIT`:** If a participant fails after promising to commit but before receiving the final decision, it must recover and ask the coordinator for the final outcome before it can proceed.

#### Coordinator Failure

This is the most problematic scenario and leads to the protocol's biggest limitation: **blocking**.

*   If the coordinator fails after sending `PREPARE` messages but before broadcasting its decision, all participants that voted `VOTE-COMMIT` are **blocked**. They cannot proceed because they do not know the final outcome. They must hold their locks and wait for the coordinator to recover, which could be an indefinite amount of time.
*   While blocked, participants hold resources, which can bring the entire system to a standstill.

Because of this blocking nature, 2PC is often considered a "pessimistic" or heavyweight protocol. Modern architectures often prefer alternative patterns like the [**Saga Pattern**](../../concepts/distributed-systems/saga-pattern.md), especially in high-availability, low-latency microservices environments. However, 2PC is still widely used in traditional relational databases (e.g., PostgreSQL, MySQL Cluster) and XA (eXtended Architecture) transactions where strong consistency is an absolute requirement.
