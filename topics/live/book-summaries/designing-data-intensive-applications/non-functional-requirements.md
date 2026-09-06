---
topic: book-summaries
status: wip
tags: [systems, data, distributed-systems]
---

# Ch 2 — Non-functional requirements

The qualities a system needs beyond what it does. Reliability, performance, maintainability.

### Response time and percentiles

Don't average response times. Averages hide the tail. p50 is the median. p99 and p99.9 are the slow requests that hit your heaviest users.

Aggregate by adding histograms. Libraries: HdrHistogram, t-digest, OpenHistogram, DDSketch.

### Reliability vs cost

You can cut reliability to ship a prototype cheaper. Know when you're doing it.

### Humans and reliability

Config changes by operators are the leading cause of outages. Hardware is 10–25%.

"Human error" is a symptom of the system people are working in, not the cause. Blaming the person who deployed doesn't fix it. Testing, rollback, gradual rollouts, monitoring, and interfaces that make the right thing easy do. Those cost time and money, so orgs ship features instead.

Blameless postmortems: people say what happened without getting punished, so the next person doesn't repeat it.

"Bob should have been more careful" isn't useful. Neither is "rewrite the backend in Haskell." Ask the people who work with the system every day how it actually works.

### Principles for scalability

No magic scaling sauce. 100,000 requests/sec of 1 kB looks nothing like 3 requests/min of 2 GB, even though both move 100 MB/s.

An architecture that handles today's load probably won't handle 10x. Rethink on every order of magnitude. Don't plan more than one order of magnitude ahead.

Break the system into pieces that can run independently: microservices, sharding, stream processing, shared-nothing. The hard part is where to cut.

If a single-machine database works, use it. Predictable load doesn't need autoscaling. 5 services is simpler than 50.

### Complexity: essential vs accidental

Essential complexity is in the problem. Accidental complexity is in the tools. The split moves as tools get better, so the labels aren't that useful.

Abstraction is the actual tool. Hide the implementation, reuse the interface. A better implementation then helps every caller.

back to [[trade-offs]] for Ch 1.
