# Chapter 2 — Defining Nonfunctional Requirements

## Before reading

### Why this chapter matters

Functional requirements say what a system does; nonfunctional requirements determine whether it remains useful under real workloads, faults, growth, and years of change. This chapter supplies the vocabulary and measurement habits needed to turn vague goals such as “fast,” “reliable,” and “scalable” into design questions you can actually reason about.

### Mental map

1. **Case Study: Social Network Home Timelines** makes the trade-offs concrete through competing ways to assemble a user’s timeline.
2. **Describing Performance** separates throughput from response time and introduces distributions, percentiles, and service objectives.
3. **Reliability and Fault Tolerance** asks how faults in components become—or are prevented from becoming—failures of the whole system.
4. **Scalability** connects explicit load parameters to resource growth and architectural choices.
5. **Maintainability** widens the lens to the people who operate, understand, and change a system over its lifetime.

### Terms to notice

- Fan-out and materialized views
- Throughput, response time, service time, latency, and queueing delay
- Median and tail percentiles (p50, p95, p99, and beyond)
- Service level objectives (SLOs) and service level agreements (SLAs)
- Faults, failures, fault tolerance, and single points of failure
- Load parameters and linear scalability
- Vertical, shared-disk, and shared-nothing architectures
- Operability, simplicity, and evolvability

### Watch for

- **Read-time work vs. write-time work:** precomputation can make reads cheap by moving cost and complexity elsewhere.
- **Typical behavior vs. tail behavior:** an average can conceal the requests and users most affected by poor performance.
- **Fault vs. failure:** a broken component need not mean that the system stops providing its required service.
- **Scaling up vs. scaling out:** adding capacity changes cost, coordination, and fault-tolerance properties—not just raw performance.

### Useful prior knowledge

Working familiarity with web services, databases, caches, and basic statistics is enough. It helps to know what a database join, queue, replication, and horizontal scaling mean, but the chapter introduces its important terminology as it goes.

### Reading strategy

Treat the social-network example as a reusable lens: for every design, ask where work happens, which load parameter dominates, and what happens in an extreme case. In later sections, translate every adjective into evidence: “fast” into a response-time distribution, “reliable” into tolerated faults and an SLO, and “scalable” into a relationship among load, resources, performance, and cost.

### Diagnostic question

If a service is fast for the median request, survives a single machine crash, and can add servers as traffic grows, what important ways could it still fail to meet its nonfunctional requirements?

#### My answer (revised)

The service demostrates good fault tolerance and scalability. Nevertheless, fast median response is usually not enough to meet the SLO. A small amount of users can still be affected by the poor performance, and sometimes they can be the valuable clients with more complex requests. That being said, it makes sense to consider metrics like P95, P99, etc.

#### Review

**Assessment:** Mostly right. You correctly distinguish median from tail experience. Also consider incorrect results, unavailability, security, and poor operability/evolvability: reliability is meeting the required service despite faults, not only tolerating a crash. See [Reliability and Fault Tolerance](/workspaces/ddia/content/en/ch2.md:363).

## Focus questions

Answer in your own words; uncertainty is useful—mark anything you’re unsure about.

### 1. ★ **[Case Study: Social Network Home Timelines]** What workload makes computing a home timeline on every read expensive, and which parts of that workload would you measure before choosing another design?


#### My answer (revised)
This method is called polling. It is expensive because we need to fetch the posts from all those they follow. Even though users may not follow a lot of users on average, there are still a lot of requests to make given the huge number of users using the platform.
Before choosing another design, it is important to measure polling interval and online-user count, post rate, the distribution of followees (especially extremes).

#### Review

**Assessment:** Mostly right. You identify polling and the expensive fan-in query. Also measure polling interval and online-user count, post rate, the distribution of followees (especially extremes), and posts per followee; overlap is less directly important to one timeline query. See [Representing Users, Posts, and Follows](/workspaces/ddia/content/en/ch2.md:58).


### 2. **[Materializing and Updating Timelines]** Predict how materializing timelines changes the location and timing of work. Under what user or traffic patterns might that trade-off become unattractive?

#### My answer (revised)
Materializing kicks in when one user makes a post. It will add this new post in the view cached for their followers. Then, when their followers log in or referesh, the posts will be loaded for them from cache.
It becomes unattractive in certain extreme cases. For example:
a) If one user follows a huge number of accounts, most of which also post a lot, we will need to add a lot of posts to their materialized view, but it's not likely for them to read all of them.
b) If one user is followed by a huge number of users, we will have to update the view for a lot of users. This can be very expensive.

#### Review

**Assessment:** Solid. You accurately move work from reads to writes and identify both pathological cases. For celebrities, a hybrid often helps: merge their posts at read time instead of fanning them out to every timeline.


### 3. ★ **[Describing Performance]** Why must performance be described using both throughput and a distribution of response times rather than one “speed” number?

#### My answer (revised)
Throughput describes capacity/work per second; response-time percentiles describe user experience. A distribution summarizes many requests rather than showing each individual user, and exposes the tail an average hides.

#### Review

**Assessment:** Mostly right. Throughput describes capacity/work per second; response-time percentiles describe user experience. A distribution summarizes many requests rather than showing each individual user, and exposes the tail an average hides.


### 4. **[Latency and Response Time / Average, Median, and Percentiles]** Reconstruct the relationship among service time, queueing delay, network latency, and client-observed response time. Why can a request with little processing still be slow?

#### My answer (revised)
Client-observed response time consists of service time, queue delay, and network latency. A request with little processing may have a short "service time", but it can be slow because there are many requests in the queue or the internet connection is slow.

#### Review

**Assessment:** Solid. Queueing/head-of-line blocking can dominate even when the request itself is cheap. See [Latency and Response Time](/workspaces/ddia/content/en/ch2.md:210).

### 5. ★ **[Use of Response Time Metrics]** A page makes many backend calls in parallel. Explain why acceptable p99 latency for each backend may still produce poor end-user latency, and identify what you would measure

#### My answer (revised)
If the page makes many backend calls in parallel, it may have to wait until all the responses are in place. If any single service takes too long to respond, it will slow down the whole request.
This is *tail-latency amplification*. We can measure end-to-end page percentiles, per-backend percentiles, number of dependencies, and timeouts/errors.

#### Review

**Assessment:** Mostly right. This is *tail-latency amplification*. Measure end-to-end page percentiles, per-backend percentiles, number of dependencies, and timeouts/errors—not only the single slowest call.

### 6. ★ **[Reliability and Fault Tolerance]** Explain the difference between a fault and a failure using one component at two different system boundaries. What does that imply about the meaning of “fault-tolerant”?


#### My answer (revised)
A fault means part of the system stops working correctly, whereas a failure means the system as a whole stops processing users' requests correctly.
A failure in a small boundary can only be a fault in a bigger boundary. For example, when a hard drive crashes, it is a failure for itself. However, if other drives in the same cluster can take over its work, it will only be a fault for this hard drive cluster.
Then, being "fault-tolerent" does not mean nothing breaks. It means in a large scale (maybe the system as a whole) it still meets the service object, even if some parts are faultly.

#### Review

**Assessment:** Solid. Your boundary-based hard-drive example is correct. Specify the types and number of faults tolerated: fault tolerance is always bounded, and it means continuing to meet the required service/SLO.

### 7. **[Hardware and Software Faults / Humans and Reliability]** Why can redundancy handle some hardware faults more readily than correlated software faults or operational mistakes? What practices would help prevent faults from cascading?

#### My answer (revised)
Hardware faults are often not correleated, and they often can be fixed by replacing the faulty part.
Software faults are often correlated. Many nodes can be affected by the same bug, and a systematic failure is usually hard to recover. As for operational mistakes, it's largely because the unpredictibility of humans.
To prevent faults from cascading, we can add staged rollouts, safe defaults/permissions, monitoring, and backoff/load shedding.

#### Review

**Assessment:** Mostly right. Hardware faults are often, not always, independent: a rack, availability-zone, firmware, or power event can correlate them. Add staged rollouts, safe defaults/permissions, monitoring, and backoff/load shedding to contain operational and retry cascades. See [Hardware and Software Faults](/workspaces/ddia/content/en/ch2.md:428).

### 8. ★ **[Describing Load]** Choose a familiar service and define the smallest useful set of load parameters for it. How would you test whether doubling one parameter preserves performance at a reasonable resource cost?

#### My answer (revised)
Add concurrency, read/write mix, cache-hit rate, and a response-time SLO when they matter. First double the chosen parameter with fixed resources and observe p50/p99/error rate; then add resources until the same SLO is restored and compare resource/cost increase.

#### Review

**Assessment:** Needs revision. Add concurrency, read/write mix, cache-hit rate, and a response-time SLO when they matter. First double the chosen parameter with fixed resources and observe p50/p99/error rate; then add resources until the same SLO is restored and compare resource/cost increase. That tests scalability, not only the bottleneck. See [Describing Load](/workspaces/ddia/content/en/ch2.md:633).

### 9. **[Shared-Memory, Shared-Disk, and Shared-Nothing Architecture]** A single-node database is nearing its capacity. What evidence would justify scaling up, and what changed requirements would justify accepting the complexity of scaling out?

#### My answer (revised)
It is important to identify the bottleneck and vertical headroom. Evidence includes projected load, SLO breaches, price/performance of a bigger node, and its ceiling.
Scaling out will introduce more complexity to the service, but we should do so when capacity, elasticity, or fault-domain availability justifies sharding and distributed-system complexity.

#### Review

**Assessment:** Mostly right. Checking the bottleneck and vertical headroom is sound. Make the evidence explicit: projected load, SLO breaches, price/performance of a bigger node, and its ceiling. Scale out when capacity, elasticity, or fault-domain availability justifies sharding and distributed-system complexity; several regions alone do not necessarily require it. See [Shared-Memory, Shared-Disk, and Shared-Nothing Architecture](/workspaces/ddia/content/en/ch2.md:673).

### 10. **[Maintainability]** Imagine an auto-scaling, self-healing service that operators struggle to understand during incidents. Evaluate it in terms of operability, simplicity, and evolvability, and propose one design change that improves one quality without quietly damaging the others

#### My answer (revised)
Operability: Operability is weak. Although the service can handle routine load changes and failures automatically, operators struggle to understand its behavior during unusual incidents, when human intervention is most important.

Simplicity: Simplicity is also weak. The interactions between auto-scaling, self-healing, and changing system conditions create complexity that is difficult for operators and new engineers to understand. Automation makes routine operation easier, but does not necessarily make the system itself simpler.

Evolvability: Evolvability is uncertain. Handling faults and load changes demonstrates reliability and scalability, not evolvability. Evolvability depends on whether engineers can safely understand, test, and modify the service and its automation policies.

One improvement would be to make the automation observable and controllable. The service should record why it made each scaling or recovery decision, expose the inputs and thresholds involved, and provide a manual override. This improves operability while preserving the benefits of automation. It can also support simplicity and evolvability, provided the added controls remain small,
clear, and well documented.

#### Review

**Assessment:** Needs revision. You correctly diagnose weak operability and suggest a useful operational model. Resilience to faults/load is reliability or scalability, not evolvability; evolvability is how easily engineers can safely change the system. A stronger change is a documented, observable autoscaling policy with manual override and reversible changes. See [Operability](/workspaces/ddia/content/en/ch2.md:780) and [Evolvability](/workspaces/ddia/content/en/ch2.md:855).

### 11. **[Summary — synthesis]** How can an optimization for one nonfunctional requirement undermine another? Trace one concrete design choice through performance, reliability, scalability, and maintainability, and state where your reasoning might stop applying

#### My answer (revised)
For example:
a) If we are to adopt auto-scaling, it can undermine the maintainability since the addition and reduction in resouces is not under our control, making it difficult for the operation team to monitor and troubleshoot. The reasoning stops applying if the system's load is always stable or well below the capicity.
b) Improving the performance can also bring trouble to maintainablity, like additional resources requested will increase the workload of the operation team, new technology or components introduced also require the operation team to understand and train. The reasoning stops apply if we have scale-up options, which is less likely to increase the workload for the operation team.

#### Review

**Assessment:** Needs revision. You identify a real maintainability cost, but trace one choice through all four qualities. For materialized timelines: precomputed reads improve performance; asynchronous fan-out needs fault tolerance to avoid delayed/missing delivery; fan-out scales poorly for celebrities; and queues, retries, reconciliation, and hybrid read paths increase operational/change complexity. This changes if follower distribution and freshness needs make simple fan-out cheap.

## Closed-book recall

Close the chapter before answering these prompts.

### Three most important ideas

- What are the three most important ideas in this chapter?

### A surprising trade-off

- Which trade-off surprised you, and why?

### What I can explain confidently

- What can you now explain confidently without the book?

### What remains unclear

- What remains unclear?

## Concepts

### System qualities

**Nonfunctional requirements** — Qualities that describe how well a system should operate rather than which features it provides. They include performance, reliability, scalability, and maintainability, and often involve trade-offs.

**Performance** — How quickly and efficiently a system handles its current workload, commonly measured with response-time distributions and throughput.

**Scalability** — How performance and cost change as a particular load parameter grows. A system is not simply “scalable”; it may scale well for reads but poorly for writes, large records, or extreme users.

**Reliability** — The ability to continue providing the required service even when faults or unexpected conditions occur.

**Maintainability** — How easy the system is to operate, understand, and change over its lifetime. Its three useful dimensions are:

- **Operability:** making routine operation and incident response easy and predictable.
- **Simplicity:** minimizing unnecessary complexity so engineers can understand and reason about the system.
- **Evolvability:** making safe, reversible changes as requirements evolve.

### Fetching and precomputing data

**Polling** — Repeatedly asking whether new data or a state change is available. It is simple, but frequent polling wastes resources while infrequent polling increases delay.

**Materialization** — Computing and storing a result ahead of time so later reads are faster, at the cost of additional write work, storage, and update complexity.

**Materialized view** — A stored result derived from other data, such as a precomputed home timeline. Unlike a normal view computed at read time, it must be refreshed when its source data changes.

**Fan-out** — One event causing work for many downstream recipients or operations. Writing a celebrity's post into millions of follower timelines is fan-out on write; fetching many sources when a user opens the feed is fan-out on read.

### Measuring performance

**Response time** — The total time from sending a request until receiving its response, including service time plus queueing, network, and scheduling delays.

**Service time** — The time a request spends actively being processed. A short service time can still produce a long response time if the request waits in a queue.

**Throughput** — The amount of work completed per unit of time, such as requests per second or gigabytes processed per hour.

**Head-of-line blocking** — A slow item at the front of a queue delays faster items behind it, increasing their response times even when they require little work.

**Tail latency (tail latencies)** — The unusually slow end of a response-time distribution, often described by high percentiles such as p95 or p99. It matters because a small fraction of very slow requests can still affect many users.

**Tail latency amplification** — When one user request depends on many backend calls, the chance that at least one call is slow increases. Since the overall request waits for the slowest dependency, its tail can be worse than each backend's tail.

**Service level objective (SLO)** — A measurable reliability or performance target, such as “99.9% of valid requests succeed” or “p99 response time stays below one second.”

**Service level agreement (SLA)** — A contract describing service commitments and the consequences of missing them. An SLO is the target; an SLA adds a promise to another party and remedies such as service credits.

### Controlling overload and feedback loops

**Retry storm** — Failed or slow requests trigger many retries, which add load to an already struggling service and cause still more failures.

**Metastable failure** — A temporary trigger pushes a system into a degraded state that sustains itself even after the original trigger disappears—for example, a retry backlog that keeps the service overloaded.

**Exponential backoff** — Increasing the delay after each failed retry, usually up to a limit, to reduce pressure on a recovering dependency.

**Jitter** — Random variation added to retry or scheduling delays so many clients do not act simultaneously and create synchronized traffic spikes.

**Token bucket** — A rate-limiting mechanism in which operations consume tokens that refill at a fixed rate. Saved tokens permit bounded bursts while the refill rate limits sustained traffic.

**Load shedding** — Deliberately rejecting or dropping some work when overloaded so the system can preserve useful service for the requests it accepts.

**Backpressure** — A downstream component signals or forces upstream producers to slow down when it cannot keep up, preventing unbounded queues and overload.

### Fault tolerance and operational learning

**Single point of failure (SPOF)** — A component whose failure causes the whole system to fail because no redundancy or fallback can take over.

**Exactly-once semantics** — The intended effect of an operation appears once despite crashes, retries, or duplicate messages. Systems usually achieve this through coordination or deduplication rather than literally executing the operation only once.

**Fault injection** — Deliberately causing a specific fault, such as killing a process or adding network delay, to verify that recovery mechanisms work.

**Chaos engineering** — A broader experimental discipline that uses controlled failures and observations to build confidence in system resilience. Fault injection is one technique within it.

**Sociotechnical system** — The combined system of software, infrastructure, people, procedures, incentives, and organizational decisions. Incidents usually emerge from interactions across this whole system rather than from one person's isolated mistake.

### Scaling and decomposition

**Scaling up (vertical scaling)** — Giving one machine more CPU, memory, or storage. It is operationally simple but eventually reaches hardware and price-performance limits.

**Scaling out (horizontal scaling)** — Adding more independent machines and distributing work among them. It offers greater capacity and fault tolerance but introduces coordination, partitioning, and distributed-system complexity.

**Sharding** — Splitting a dataset into partitions and assigning them to different nodes. It enables scaling out, but shard selection, rebalancing, skew, and cross-shard operations become design concerns.

**Microservices** — Dividing an application into independently deployable services around clear responsibilities. This can support independent scaling and change, but too many or poorly chosen boundaries increase operational and network complexity.

**Serverless** — Running code or managed services without directly provisioning servers, often with automatic scaling and usage-based billing. It reduces some operational work but brings platform limits, variable latency, and provider dependence.

## Review

### Pattern across your answers

- You consistently locate the practical bottleneck and recognize that extreme users/workloads dominate design choices.
- Your strongest mental model is moving work between reads and writes, and containing component faults.
- Sharpen the distinction between adjacent qualities: resilience is reliability, capacity growth is scalability, and ease of changing the system is evolvability.

### Review next

1. Load parameters as a measurable relationship among workload, SLO, resources, and cost.
2. Tail-latency amplification and end-to-end percentile measurement.
3. Maintainability: especially the difference among operability, simplicity, and evolvability.

### Follow-up

1. A page makes 20 parallel backend calls, each with a 1% chance of exceeding its p99 threshold. Why can the page have a much larger slow-request fraction than 1%, and what would you dashboard?
2. For an image-upload API, choose three load parameters and define an experiment that tests whether it scales linearly while holding a p99 SLO.
3. A self-healing deployment system automatically rolls back bad releases but is hard to modify for a new rollout policy. Which maintainability quality is weak, and what change would improve it reversibly?

## Application challenge

Your team runs a social-news feed. The current design materializes every new post into every follower’s cached home timeline. A small group of creators has tens of millions of followers, and during breaking news the feed must show ordinary posts within 5 seconds at p99 while avoiding a large rise in operating cost.

State the decision you would make; the Chapter 2 concepts that support it; your assumptions; how the choice could fail; and what requirement change would make you choose an alternative.

## Spaced review

### 2026-07-13

- [ ] Why is an average response time insufficient for a user-facing SLO? Interpret p50 and p99 in your own words.
- [ ] In the timeline example, what work moves from read time to write time when timelines are materialized?
- [ ] At what system boundary can one failed disk be a fault rather than a failure?

### 2026-07-18

- [ ] A request has short service time but a poor p99. Name two sources of delay that can explain this.
- [ ] How would you test whether doubling API request rate preserves an existing p99 target at reasonable cost?
- [ ] Why does a shared-nothing design offer both useful scaling potential and additional complexity?

### 2026-08-11

- [ ] Explain tail-latency amplification for a page that calls many services in parallel.
- [ ] Contrast operability, simplicity, and evolvability with one sentence each.
- [ ] For a celebrity post, when might a hybrid read-time/write-time feed be preferable to full fan-out?
