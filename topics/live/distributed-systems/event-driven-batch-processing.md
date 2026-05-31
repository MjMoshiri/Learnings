---
topic: distributed-systems
status: done
tags: [batch-processing, event-driven-architecture]
---

## Summary

Event-driven batch processing is a pattern where batch processing jobs are initiated in response to events. Instead of running on a fixed schedule, batch jobs are triggered by events such as the arrival of new data, the completion of a previous step in a workflow, or a specific time-based event. This allows for more flexible and efficient use of resources, as processing only occurs when it is needed.

## Core Concepts

-   **Event Trigger:** An event that signals the start of a batch job.
-   **Batch Job:** A process that reads a large amount of input data, performs a computation, and writes the output data.
-   **Workflow Orchestration:** A system that manages the dependencies and execution order of batch jobs in response to events.

## Use Cases

-   **Good for:** Data processing pipelines where the arrival of data is unpredictable, workflows that need to be executed in response to specific events, and optimizing resource usage by only running jobs when necessary.
--   **Not for:** Batch jobs that need to run on a strict, time-based schedule.

## References

-   [Designing Distributed Systems by Brendan Burns](https://www.oreilly.com/library/view/designing-distributed-systems/9781491983638/)
