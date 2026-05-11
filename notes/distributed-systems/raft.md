
---
title: Raft Consensus Algorithm
summary: An overview of the Raft consensus algorithm, designed for understandability and fault tolerance in distributed systems.
tags:
  - distributed-systems
  - consensus
  - fault-tolerance
---

# Raft Consensus Algorithm

The Raft consensus algorithm is a protocol for managing a replicated log, ensuring that a cluster of machines can work as a coherent group, agree on a single, ordered sequence of operations, and maintain a consistent state, even in the face of server failures. It was designed to be more understandable than its predecessor, Paxos.

## Core Concepts

Raft's functionality is built upon three key pillars:

1.  **Leader Election**: One server is elected as the leader, responsible for managing the replicated log.
2.  **Log Replication**: The leader ensures that the logs of all servers are kept in sync.
3.  **Safety**: A set of rules and guarantees that ensure the correctness and consistency of the system.

### Server States

In a Raft cluster, each server can be in one of three states:

*   **Follower**: Passively responds to requests from the leader.
*   **Leader**: Handles all client requests and manages log replication.
*   **Candidate**: A follower that has initiated an election to become the new leader.

### Terms

Raft uses a system of **terms**, which are monotonically increasing numbers that act as a logical clock. Each term begins with a leader election.

## Leader Election

The leader election process is as follows:

1.  **Timeout**: If a follower doesn't receive a "heartbeat" message from the leader within a randomized **election timeout**, it assumes the leader has failed and becomes a candidate.
2.  **Candidacy**: The candidate increments the current term, votes for itself, and sends `RequestVote` RPCs to all other servers.
3.  **Voting**: Servers vote for a candidate if they haven't already voted in the current term and if the candidate's log is at least as up-to-date as their own.
4.  **Winning**: A candidate becomes the new leader if it receives votes from a majority of the servers.
5.  **Split Votes**: If no candidate receives a majority, a new election is started with randomized timeouts to prevent repeated split votes.

## Log Replication

Once a leader is elected, it manages the log replication:

1.  **Append**: The leader appends new client commands to its own log.
2.  **Replicate**: The leader sends `AppendEntries` RPCs with the new log entry to its followers.
3.  **Acknowledge**: Followers append the entry to their logs and acknowledge the leader.
4.  **Commit**: An entry is committed once the leader has replicated it to a majority of servers. The leader then applies the command to its state machine.
5.  **Inform**: The leader notifies followers of the committed entry in subsequent `AppendEntries` RPCs, and they apply it to their state machines.

## Safety Guarantees

Raft provides several safety guarantees:

*   **Election Safety**: At most one leader per term.
*   **Leader Append-Only**: Leaders only append to their logs.
*   **Log Matching**: If two logs have an entry with the same index and term, they are identical up to that index.
*   **Leader Completeness**: A committed entry will be in the logs of leaders for all subsequent terms.
*   **State Machine Safety**: If a server applies a log entry at a given index, no other server will apply a different entry for the same index.

## Benefits and Drawbacks

**Benefits**:

*   **Understandability**: Easier to understand and implement than Paxos.
*   **Strong Consistency**: Ensures all nodes agree on the same state.
*   **Fault Tolerance**: Tolerates server failures as long as a majority are operational.
*   **Dynamic Membership**: Allows for changes in cluster membership without downtime.

**Drawbacks**:

*   **Single Leader Bottleneck**: Can be a performance bottleneck under high write loads.
*   **No Byzantine Fault Tolerance**: Does not handle malicious or arbitrary node behavior.
*   **Latency Sensitivity**: Performance can be sensitive to network latency and timeout configurations.

## Real-World Implementations

*   **etcd**: Used by Kubernetes for cluster data.
*   **Consul**: For leader election and service state management.
*   **CockroachDB**: For replicating data and ensuring transactional consistency.
*   **TiKV**: For data replication.
*   **Apache Kafka**: The KRaft protocol is based on Raft.
