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

#### My answer

The service demostrates good fault tolerance and scalability. Nevertheless, fast median response is usually not enough to meet the SLO. A small amount of users can still be affected by the poor performance, and sometimes they can be the valuable clients with more complex requests. That being said, it makes sense to consider metrics like P95, P99, etc.

## Focus questions

Answer in your own words; uncertainty is useful—mark anything you’re unsure about.

### 1. ★ **[Case Study: Social Network Home Timelines]** What workload makes computing a home timeline on every read expensive, and which parts of that workload would you measure before choosing another design?


#### My answer
This method is called polling. It is expensive because we need to fetch the posts from all those they follow. Even though users may not follow a lot of users on average, there are still a lot of requests to make given the huge number of users using the platform.
Before choosing another design, I think it's important to understand the average of people that one follows, and to what extent they overlap.


### 2. **[Materializing and Updating Timelines]** Predict how materializing timelines changes the location and timing of work. Under what user or traffic patterns might that trade-off become unattractive?

#### My answer
Materializing kicks in when one user makes a post. It will add this new post in the view cached for their followers. Then, when their followers log in or referesh, the posts will be loaded for them from cache.
It becomes unattractive in certain extreme cases. For example:
a) If one user follows a huge number of accounts, most of which also post a lot, we will need to add a lot of posts to their materialized view, but it's not likely for them to read all of them.
b) If one user is followed by a huge number of users, we will have to update the view for a lot of users. This can be very expensive.


### 3. ★ **[Describing Performance]** Why must performance be described using both throughput and a distribution of response times rather than one “speed” number?

#### My answer
Throughput measures the capacity for our system to deal with requests. It is important for us to decide the resources to allocate. It will affect the response time - how long it takes to process user requests.
The distrbution of response times tells us how the service performs for different users. It tells us how the service performs for each individual user, so we can make more targeted adjustments.


### 4. **[Latency and Response Time / Average, Median, and Percentiles]** Reconstruct the relationship among service time, queueing delay, network latency, and client-observed response time. Why can a request with little processing still be slow?

#### My answer
Client-observed response time consists of service time, queue delay, and network latency. A request with little processing may have a short "service time", but it can be slow because there are many requests in the queue or the internet connection is slow.

### 5. ★ **[Use of Response Time Metrics]** A page makes many backend calls in parallel. Explain why acceptable p99 latency for each backend may still produce poor end-user latency, and identify what you would measure

#### My answer
If the page makes many backend calls in parallel, it may have to wait until all the responses are in place. If any single service takes too long to respond, it will slow down the whole request.
In this case, it may be important to identify the bottleneck, i.e. which calls tend to be the slowest and affect the whole response.

### 6. ★ **[Reliability and Fault Tolerance]** Explain the difference between a fault and a failure using one component at two different system boundaries. What does that imply about the meaning of “fault-tolerant”?


#### My answer
A fault means part of the system stops working correctly, whereas a failure means the system as a whole stops processing users' requests correctly.
A failure in a small boundary can only be a fault in a bigger boundary. For example, when a hard drive crashes, it is a failure for itself. However, if other drives in the same cluster can take over its work, it will only be a fault for this hard drive cluster.
Then, being "fault-tolerent" does not mean nothing breaks. It means in a large scale (maybe the system as a whole) it still meets the service object, even if some parts are faultly.

### 7. **[Hardware and Software Faults / Humans and Reliability]** Why can redundancy handle some hardware faults more readily than correlated software faults or operational mistakes? What practices would help prevent faults from cascading?

#### My answer
Hardware faults tend not to be correleated, and they often can be fixed by replacing the faulty part.
Software faults are often correlated. Many nodes can be affected by the same bug, and a systematic failure is usually hard to recover. As for operational mistakes, it's largely because the unpredictibility of humans.
To prevent faults from cascading,
a) Through testing, especially the assumpions that the software is based on
b) Properly error handling
c) Isolated processes
d) Allow the process to crash and resdtart

### 8. ★ **[Describing Load]** Choose a familiar service and define the smallest useful set of load parameters for it. How would you test whether doubling one parameter preserves performance at a reasonable resource cost?

#### My answer
For example, a typical RESTful API can have load parameters like request per second and payload size. For testing, we can try increasing the load in a ceartain way while keeping all resources unchanged. Then, we try to identify the bottlenecks of the system and see what the additional resources are needed.

### 9. **[Shared-Memory, Shared-Disk, and Shared-Nothing Architecture]** A single-node database is nearing its capacity. What evidence would justify scaling up, and what changed requirements would justify accepting the complexity of scaling out?

#### My answer
I think it is important to identify the bottleneck of the current database. For example, high CPU and RAM utlization, low free diskspace. If there are still options for better CPUs, larger RAM, or more disk space, so it still makes sense to scale up.
Scaling out will introduce more complexity to the service, but there are a few considerations:
a) When we alrady hit limit of recourses
b) When it already gets too slow for read or write operations
c) When there are users from different regions
d) When there is a need for high availability

### 10. **[Maintainability]** Imagine an auto-scaling, self-healing service that operators struggle to understand during incidents. Evaluate it in terms of operability, simplicity, and evolvability, and propose one design change that improves one quality without quietly damaging the others

#### My answer
Operability: Since the operators have a hard time understanding during incidents. The operability is not really good especially for extreme cases, even though it can run smoothly most of the time by itself.
Simplicity: Such automation can make it hard for new staff to understand, so the simplicity is poor.
Evolvability: It has good evolvability since it is resilient to errors and loads, even if the service requirement or user patterns can change.
By including a clear audit logs and using a readable model for auto-scale, we can have better operability without damanging the other properties.

### 11. **[Summary — synthesis]** How can an optimization for one nonfunctional requirement undermine another? Trace one concrete design choice through performance, reliability, scalability, and maintainability, and state where your reasoning might stop applying

#### My answer
For example:
a) If we are to adopt auto-scaling, it can undermine the maintainability since the addition and reduction in resouces is not under our control, making it difficult for the operation team to monitor and troubleshoot. The reasoning stops applying if the system's load is always stable or well below the capicity.
b) Improving the performance can also bring trouble to maintainablity, like additional resources requested will increase the workload of the operation team, new technology or components introduced also require the operation team to understand and train. The reasoning stops apply if we have scale-up options, which is less likely to increase the workload for the operation team.

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

## Concept explanations

## Review

## Application challenge

## Spaced review
