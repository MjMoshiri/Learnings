---
topic: book-summaries
status: wip
tags: [systems, data, distributed-systems]
---

# Ch 2 — Non-functional requirements

The qualities a system needs beyond what it does. Reliability, performance, maintainability.

### Response time and percentiles

Don't average response times. Averages hide the tail. Use percentiles: p50 is the median, p99 and p99.9 are the slow requests that hit your heaviest users.

> The right way of aggregating response time data is to add the histograms.

> Open source percentile estimation libraries include HdrHistogram, t-digest, OpenHistogram, and DDSketch.

### Reliability vs cost

> In some situations we may choose to sacrifice reliability in order to reduce development cost (e.g., when developing a prototype product for an unproven market)—but we should be very conscious of when we are cutting corners and keep in mind the potential consequences.

### Humans and reliability

> One of their strengths is being creative and adaptive in getting their jobs done. However, this characteristic also leads to unpredictability, and sometimes to mistakes that can lead to failures, despite best intentions. For example, one study of large internet services found that configuration changes by operators were the leading cause of outages, whereas hardware faults (servers or network) played a role in only 10%–25% of cases.
>
> It is tempting to label such problems as "human error" and to wish that they could be solved by better controlling human behavior through tighter procedures and compliance with rules. However, blaming people for mistakes is counterproductive. What we call "human error" is not really the cause of an incident, but rather a symptom of a problem with the sociotechnical system in which people are trying their best to do their jobs. Often complex systems have emergent behavior, in which unexpected interactions between components may also lead to failures.
>
> Various technical measures can help minimize the impact of human mistakes: thorough testing, rollback mechanisms for quickly reverting configuration changes, gradual rollouts, clear monitoring and observability, and well-designed interfaces that encourage "the right thing" and discourage "the wrong thing." But these require an investment of time and money, and organizations often prioritize revenue-generating features over resilience. When a preventable mistake inevitably occurs, blaming the person who made it does not make sense; the problem is the organization's priorities.
>
> Increasingly, organizations are adopting a culture of blameless postmortems: after an incident, the people involved share full details about what happened, without fear of punishment, so others can learn how to prevent similar problems in the future.
>
> As a general principle, when investigating an incident, you should be suspicious of simplistic answers. "Bob should have been more careful when deploying that change" is not productive, but neither is "We must rewrite the backend in Haskell." Instead, management should learn the details of how the sociotechnical system works from the point of view of the people who work with it every day, and take steps to improve it based on this feedback.

### Principles for scalability

> There is no such thing as a generic, one-size-fits-all scalable architecture (informally known as magic scaling sauce). For example, a system designed to handle 100,000 requests per second, each 1 kB in size, looks very different from a system designed for 3 requests per minute, each 2 GB in size—even though the two systems have the same data throughput (100 MB/second).
>
> An architecture that is appropriate for one level of load is unlikely to cope with 10 times that load. If you are working on a fast-growing service, you will probably need to rethink your architecture on every order of magnitude load increase. Since the needs of the application are likely to evolve, it is usually not worth planning future scaling needs more than one order of magnitude in advance.
>
> A good general principle for scalability is to break a system into smaller components that can operate largely independently from one another. This is the underlying principle behind microservices, sharding, stream processing, and shared-nothing architectures. The challenge lies in knowing where to draw the line between things that should be together and things that should be apart.
>
> Another good principle is not to make things more complicated than necessary. If a single-machine database will do the job, it's probably preferable to a complicated distributed setup. Autoscaling systems are cool, but if your load is fairly predictable, a manually scaled system may have fewer operational surprises. A system with 5 services is simpler than one with 50. Good architectures usually involve a pragmatic mixture of approaches.

### Complexity: essential vs accidental

> One attempt at reasoning about complexity breaks it into two categories: essential and accidental. The idea is that essential complexity is inherent in the problem domain of the application, while accidental complexity arises only because of limitations of our tooling. Unfortunately, this distinction is also flawed, because boundaries between the essential and the accidental shift as our tooling evolves.
>
> One of the best tools we have for managing complexity is abstraction. A good abstraction can hide a great deal of implementation detail behind a clean, simple-to-understand façade. A good abstraction can also be used for a wide range of applications, and quality improvements in the abstracted component benefit all applications that use it.

back to [[trade-offs]] for Ch 1.
