---
topic: book-summaries
status: wip
tags: [systems, data, distributed-systems]
---

# Ch 1 — Trade-offs in data systems

Most architecture questions have no single right answer, just options with different costs.

### OLTP vs OLAP

- **Operational (OLTP).** Transaction processing. Small reads and writes, low latency, serves the running app.
- **Analytical (OLAP).** Big scans for reports and dashboards. Serves analysts and decisions, not users.
- They manage different data, different access patterns, different audiences. Ch 4 shows they end up with different internal layouts because of the queries they serve.

### Data warehouse vs data lake

Both pull from operational systems through ETL. The split is when you impose structure.

- **Warehouse** is schema-on-write. Clean and model the data going in. Query is fast and predictable, but you decide the shape up front.
- **Lake** is schema-on-read. Dump raw data, figure out structure when you read it. Flexible, keeps everything, pushes the work downstream.

**Sushi principle: raw data is better.** Keep it raw as long as you can. Transforming early throws away information you didn't know you'd want. A lake holds the raw fish; you slice it per query.

### Cloud vs self-hosted

Newer split against the old self-hosted default. Which is cheaper depends entirely on your situation. The real shift is architectural: cloud native pulls **storage and compute apart** so you scale them on their own.

### Why go distributed

Cloud systems are distributed by nature. Don't rush there if one machine still works, but the usual drivers are scale, elasticity, latency, and fault tolerance. Two more worth keeping:

- **Inherently distributed.** The problem already spans locations. Multiple datacenters, devices in the field, data that legally can't move.
- **Sustainability.** You can place compute in a cheaper region that runs on renewable energy. Where the work runs is a lever, not a given.

### Privacy as architecture

A system's shape is set by the business *and* by privacy regulation protecting the people in the data. Engineers skip this. Turning legal requirements into technical design isn't formalized yet, but it's a real constraint, not an afterthought.
