---
topic: book-summaries
status: wip
tags: [systems, data, distributed-systems]
---

# Ch 1 — Trade-offs in data systems

Architecture questions have no single right answer, just options with different costs.

### OLTP vs OLAP

- **Operational (OLTP).** Transaction processing. Small reads and writes, low latency, serves the running app.
- **Analytical (OLAP).** Big scans for reports and dashboards. Serves analysts and decisions, not users.

Ch 4: they end up with different internal layouts because of the queries they serve.

### Data warehouse vs data lake

Both pull from operational systems through ETL. The split is when you impose structure.

- **Warehouse** is schema-on-write. Clean and model the data going in. Query is fast and predictable, but you decide the shape up front.
- **Lake** is schema-on-read. Dump raw data, figure out structure when you read it. Flexible, keeps everything, pushes the work downstream.

**Sushi principle: raw data is better.** Keep it raw as long as you can. Transforming early throws away information you didn't know you'd want.

### Cloud vs self-hosted

Which is cheaper depends on your situation. Cloud native pulls storage and compute apart so you scale them separately.

### Why go distributed

Usual drivers: scale, elasticity, latency, fault tolerance. Don't go there if one machine still works.

- **Inherently distributed.** The problem already spans locations. Multiple datacenters, devices in the field, data that legally can't move.
- **Sustainability.** You can place compute in a cheaper region that runs on renewable energy. Where the work runs is a lever, not a given.

### Privacy as architecture

A system's shape is set by the business and by privacy regulation protecting the people in the data. Engineers skip this. Turning legal requirements into technical design isn't formalized yet, but it's a real constraint.
