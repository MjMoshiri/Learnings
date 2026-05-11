---
title: "The Publish-Subscribe (Pub/Sub) Pattern"
tags: [ "messaging", "architecture-pattern", "decoupling" ]
status: "completed"
---

## Summary

The Publish-Subscribe (Pub/Sub) pattern is a messaging pattern where senders of messages, called **publishers**, do not programmatically send their messages directly to specific receivers, called **subscribers**. Instead, publishers categorize published messages into classes without knowledge of which subscribers, if any, there may be. Similarly, subscribers express interest in one or more classes and only receive messages that are of interest, without knowledge of which publishers, if any, there are.

This decoupling of publishers and subscribers can allow for greater scalability and a more dynamic network topology.

### Core Components

-   **Publisher:** The application that creates and sends messages.
-   **Subscriber:** The application that receives messages.
-   **Message Broker / Event Bus:** The intermediary that manages the distribution of messages from publishers to subscribers.
