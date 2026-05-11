---
title: "Apache Kafka"
tags: [ "message-queue", "streaming", "distributed-system", "pub-sub" ]
prerequisites:
  - "concepts/messaging/pub-sub.md"
status: "learning"
last_updated: "2025-08-22"
todos:
  - "Explain the role of Zookeeper/KRaft in managing the cluster."
  - "Detail the lifecycle of a message from producer to consumer."
  - "Add a section on Kafka's internal log-based architecture."
  - "Create and link prerequisite doc on the pub/sub pattern."
---

## Summary

Apache Kafka is a distributed streaming platform used for building real-time data pipelines and streaming applications. It is designed to be high-throughput, scalable, and fault-tolerant, functioning as a publish-subscribe messaging system where applications can send (produce) and receive (consume) streams of data records. Its core capability is handling continuous streams of data in a reliable and durable manner.

## Core Concepts

-   **Broker:** A single Kafka server. Brokers are responsible for receiving messages from producers, assigning them offsets, and storing them on disk.
-   **Cluster:** A group of brokers working together to form the Kafka messaging system. A cluster provides load balancing, scalability, and fault tolerance.
-   **Producer:** A client application that writes (publishes) streams of records to one or more Kafka topics.
-   **Consumer:** A client application that reads (subscribes to) streams of records from one or more Kafka topics.
-   **Topic:** A named category or feed to which records are published. Topics in Kafka are partitioned and spread across multiple brokers.
-   **Partition:** A topic is divided into one or more partitions. Each partition is an ordered, immutable sequence of records that is continually appended to. Partitions allow for parallelism, as multiple consumers can read from different partitions of the same topic simultaneously.
-   **Offset:** A unique, sequential ID assigned to each record within a partition. It guarantees the order of messages within that specific partition.

## Producer & Consumer Mechanics

### Writing Messages with Producers

-   **Message Keys:** Producers can send messages with a key (e.g., a user ID, a device ID). If a key is present, Kafka's default partitioner guarantees that all messages with the same key will always be sent to the same partition. This is critical for ensuring the order of messages for a specific entity. If the key is null, messages are distributed in a round-robin fashion across partitions.
-   **Acknowledgments (`acks`):** This producer setting controls the durability of a write. It's a trade-off between performance and reliability.
    -   `acks=0`: The producer doesn't wait for any acknowledgment. This offers the highest throughput but messages can be lost.
    -   `acks=1`: (Default) The producer waits for the leader broker to acknowledge the write. This is a balanced approach.
    -   `acks=all`: The producer waits for the leader and all in-sync replicas to acknowledge the write. This provides the strongest durability guarantee but has higher latency.

### Reading Messages with Consumers

-   **Consumer Groups:** A consumer group is a set of consumers that cooperate to consume data from a topic. Each partition of a topic is consumed by exactly one consumer within the group. This is Kafka's mechanism for scaling consumption and processing data in parallel. If you have more consumers in a group than partitions, the excess consumers will be idle.
-   **Offset Management:** Kafka tracks the last record fetched by a consumer group for each partition. This position is called the "offset." The process of updating this position is called an "offset commit." This is a critical concept for message delivery guarantees.
    -   **At-most-once:** Commit offsets before processing the message. If the consumer crashes after committing but before processing, the message is lost.
    -   **At-least-once:** Commit offsets after processing the message. If the consumer crashes after processing but before committing, the message will be re-processed by another consumer. This is the most common guarantee.
    -   **Exactly-once:** Achieved through more complex transactional capabilities in Kafka, ensuring a message is processed once and only once.

## Topics, Partitions, and Replication

### The Role of Topics

A **topic** is the fundamental unit of organization in Kafka. It's a logical channel to which producers write data and from which consumers read data. You can think of it like a table in a database or a folder in a filesystem. Topics are identified by a unique name within the Kafka cluster.

### Scalability Through Partitions

Topics are broken down into smaller, manageable segments called **partitions**. When a topic is created, you define the number of partitions it should have. This is the primary mechanism for scalability and parallelism in Kafka.

-   **Ordering:** Kafka only guarantees the order of messages *within a single partition*. A message's position in this ordered sequence is identified by its offset.
-   **Parallelism:** Each partition can be consumed by a different consumer within a consumer group. If a topic has 10 partitions, you can have up to 10 active consumers in a single group, each processing a fraction of the data in parallel. This allows for massive horizontal scaling of data consumption.
-   **Immutability:** Partitions are append-only logs. Once a message is written to a partition, it cannot be changed.

### Fault Tolerance Through Replication

To ensure data durability and high availability, Kafka replicates partitions across multiple brokers. The **replication factor** determines how many copies of a partition are stored in the cluster.

-   **Leader and Followers:** For each partition, one broker is elected as the **leader**, and the other brokers holding copies are called **followers**.
    -   The leader handles all read and write requests for that partition.
    -   Followers passively replicate the data from the leader. Their primary purpose is to be available to take over if the leader fails.
-   **In-Sync Replicas (ISR):** A follower is considered "in-sync" if it is fully caught up with the leader. The `acks=all` setting for producers ensures that a message is only considered committed when it has been successfully written to the leader and all in-sync replicas.
-   **Leader Election:** If a leader broker fails, Kafka's controller automatically elects a new leader from the pool of in-sync replicas. This process ensures that the system remains available for producers and consumers with minimal disruption.

## Cluster Management: Zookeeper and KRaft

For a Kafka cluster to function, it needs to manage the state of all its brokers, topics, partitions, and consumer groups. Historically, this was the job of Apache Zookeeper. More recently, Kafka has introduced its own built-in consensus mechanism called KRaft, which removes the Zookeeper dependency.

### The Traditional Role of Zookeeper

Zookeeper is a distributed coordination service that Kafka has historically relied on for:

-   **Controller Election:** Zookeeper is responsible for electing one broker in the cluster to act as the "controller." The controller manages the state of partitions and replicas and performs administrative tasks, such as reassigning partitions.
-   **Cluster Membership:** Zookeeper keeps track of which brokers are currently alive and part of the cluster.
-   **Topic Configuration:** It stores all metadata about topics, including the number of partitions, replication factor, and any other configuration overrides.
-   **Access Control Lists (ACLs):** Zookeeper stores the ACLs that determine which users can read from or write to which topics.

While reliable, the dependency on Zookeeper added significant operational complexity to managing a Kafka cluster. It was another complex distributed system to learn, configure, and monitor.

### The Rise of KRaft

Starting in version 2.8 and becoming production-ready in version 3.3, Kafka introduced **Kafka Raft Metadata mode (KRaft)**. KRaft is a consensus protocol based on the [Raft algorithm](../concepts/distributed-systems/raft.md) that allows Kafka to manage its own metadata without an external dependency on Zookeeper.

-   **Simplified Architecture:** With KRaft, the cluster metadata is no longer stored in Zookeeper but in a dedicated internal topic within Kafka itself.
-   **Controller Quorum:** A subset of brokers are designated as "controllers," and they use the KRaft protocol to elect a leader among themselves. This leader is responsible for managing the cluster's metadata.
-   **Improved Performance and Scalability:** By removing the Zookeeper dependency, KRaft significantly simplifies Kafka's architecture, improves performance, and allows a single cluster to support a much larger number of topics and partitions.

For new Kafka deployments, using KRaft is the recommended approach. Zookeeper is still supported for existing setups but is expected to be fully deprecated in future versions.

## Lifecycle of a Message

Understanding the end-to-end journey of a message is key to grasping how Kafka works in practice.

1.  **Producer Creates a Message:** A producer application creates a message, which consists of a topic name, a value (the data payload), and an optional key.

2.  **Producer Determines the Partition:** The producer fetches metadata from the Kafka cluster to learn about the topic's partitions. It then determines which partition to send the message to.
    *   If a key is provided, the partition is determined by a hash of the key. This ensures all messages with the same key land in the same partition.
    *   If the key is null, the producer defaults to a round-robin distribution across all available partitions for the topic.

3.  **Producer Sends the Message:** The producer sends the message to the broker that is the leader for the target partition. The producer can batch multiple messages together into a single request to improve efficiency.

4.  **Broker Receives and Appends:** The leader broker receives the message and appends it to the end of the partition's log file on its local disk. It assigns the message a unique, sequential offset within that partition.

5.  **Replication to Followers:** The leader broker's followers for that partition then issue fetch requests to the leader. The leader sends the new message to the followers, who append it to their own local logs.

6.  **Acknowledgment to Producer:** Once the write has been acknowledged by the required number of brokers (as configured by the `acks` setting), the leader sends a confirmation back to the producer. The message is now considered "committed."

7.  **Consumer Fetches the Message:** A consumer application, as part of a consumer group, polls the leader broker for new messages in the partitions it is assigned to. The consumer specifies the offset it wants to read from.

8.  **Broker Sends the Message:** The broker reads the message from the partition's log on disk and sends it to the consumer.

9.  **Consumer Processes the Message:** The consumer receives the message and executes its processing logic.

10. **Consumer Commits the Offset:** After successfully processing the message, the consumer commits the offset of the next message it needs to read. This offset is stored in an internal Kafka topic (`__consumer_offsets`). If the consumer crashes and restarts, it will resume reading from this last committed offset, ensuring it doesn't miss messages.

## The Log-Based Architecture

At its heart, Kafka is a distributed, replicated, append-only log. This simple data structure is the foundation for its performance, durability, and scalability.

### What is a Log?

A log is one of the simplest possible storage structures. It's an append-only, totally-ordered sequence of records. In Kafka, each partition of a topic is a structured commit log. When a producer sends a message to a partition, it is appended to the end of the log.

### Performance Through Sequential I/O

Kafka's performance is heavily reliant on its interaction with the filesystem.

-   **Sequential Writes:** Writing to a log is a sequential operation. On modern disk hardware, sequential writes are extremely fast, often faster than random writes to memory. By avoiding the random I/O patterns of traditional message brokers, Kafka can achieve very high write throughput.
-   **Zero-Copy:** When a consumer requests data, Kafka uses a "zero-copy" optimization. This allows the operating system to transfer data directly from the filesystem cache to the network socket without having to copy it into the Kafka broker's application memory. This is highly efficient and dramatically reduces CPU and memory overhead.

### Data Retention and Log Compaction

Unlike traditional message queues that often delete messages after they are consumed, Kafka retains messages for a configurable period of time (e.g., 7 days). This is known as the **retention policy**.

-   **Time-Based Retention:** The most common policy is to discard messages after a certain amount of time has passed.
-   **Size-Based Retention:** You can also configure a topic to start deleting old messages once the log reaches a certain size.
-   **Log Compaction:** For certain use cases, like storing the latest state of a changing entity, you can use log compaction. In a compacted topic, Kafka guarantees to retain at least the last known value for each message key. This is useful for change data capture (CDC) and other stateful applications.

This log-based design is what makes Kafka so versatile. It can function as a traditional message queue, but it also has the characteristics of a distributed database, allowing you to store and query streams of data.
