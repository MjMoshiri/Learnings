---
topic: distributed-systems
status: done
tags: [batch-processing, message-queues]
---

## Summary

A work queue system is a pattern used for distributing time-consuming tasks among multiple workers. Tasks are sent as messages to a queue, and worker processes consume messages from the queue and perform the tasks. This pattern allows for decoupling of the task producer from the task consumer, and enables asynchronous processing, load balancing, and improved fault tolerance.

## Core Concepts

-   **Task:** The unit of work to be performed.
-   **Queue:** A message queue that stores tasks waiting to be processed.
-   **Producer:** The component that creates tasks and adds them to the queue.
-   **Consumer/Worker:** The component that takes tasks from the queue and executes them.

## Use Cases

-   **Good for:** Offloading long-running tasks from a web request-response cycle, processing large amounts of data in the background, and distributing work across a pool of machines.
-   **Not for:** Real-time, low-latency tasks where the overhead of a queue is not acceptable.

## References

-   [Designing Distributed Systems by Brendan Burns](https://www.oreilly.com/library/view/designing-distributed-systems/9781491983638/)
