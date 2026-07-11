# Chapter 1 — Trade-offs in Data Systems Architecture

## Before reading

### Why this chapter matters

This chapter establishes the book’s core habit: evaluate data systems against a particular workload and set of human needs, rather than searching for a universally best technology. It also introduces distinctions—operational versus analytical systems, source data versus derived data, cloud versus self-hosting, and single-node versus distributed—that recur throughout the book.

### Mental map

1. **Analytical versus Operational Systems** — begin with who uses data, what they do with it, and why those access patterns pull systems in different directions.
2. **Data Warehousing** and **Systems of Record and Derived Data** — follow data as it moves from authoritative sources into specialized representations.
3. **Cloud versus Self-Hosting** — examine what is really being outsourced and how that changes cost, control, architecture, and operational responsibility.
4. **Distributed versus Single-Node Systems** — identify the pressures that justify distribution and the failure modes and complexity it introduces.
5. **Data Systems, Law, and Society** — widen the design boundary to include privacy, safety, regulation, and the people represented by the data.

### Terms to notice

- Operational systems / online transaction processing (OLTP)
- Analytical systems / online analytical processing (OLAP)
- Data warehouses, data lakes, and data lakehouses
- ETL, ELT, data pipelines, and reverse ETL
- Systems of record and derived data systems
- Cloud services, self-hosting, and cloud-native architecture
- Single-node and distributed systems
- Data minimization

### Watch for

- **Workload versus product label:** a technology’s role depends on how it is used, not merely what category its vendor assigns it.
- **Authoritative versus reproducible data:** distinguish the place where a fact is first recorded from representations that can be rebuilt from it.
- **Outsourcing work versus eliminating work:** track which responsibilities move to a provider and which remain with the engineering team.
- **Technical capability versus justified complexity:** notice when scale, geography, organizational structure, or regulation makes a more complex architecture worthwhile.

### Useful prior knowledge

Working familiarity with application backends, databases, and basic SQL concepts will help, but no distributed-systems expertise is assumed. If terms such as API, cache, database index, virtual machine, or aggregate query are new, a quick conceptual understanding is sufficient; later chapters develop the mechanisms in depth.

### Reading strategy

For each architecture described, jot down four things: its workload, its main beneficiary, the problem it solves, and the new cost or risk it creates. Pay special attention to the chapter’s concrete boundary cases—such as HTAP, microservices, serverless systems, and privacy-driven deletion—because they test whether the headline distinctions actually hold.

### Diagnostic question

What specific evidence would convince you that a data system should become more specialized, outsourced, or distributed despite the additional complexity?

#### My answer

## Focus questions

Answer in your own words; uncertainty is useful—mark anything you’re unsure about.

### 1. ★ **[Analytical versus Operational Systems]** What differences in users, read/write patterns, and time horizons make operational and analytical workloads pull a data system toward different designs?

#### My answer

### 2. **[Data Warehousing]** Why might an organization copy data out of its operational databases instead of allowing analysts to query those databases directly?

#### My answer

### 3. ★ **[Data Warehousing / From data warehouse to data lake]** Given a relational warehouse, a data lake, and a lakehouse, what kinds of consumers and transformations would lead you to choose—or combine—these forms of analytical storage?

#### My answer

### 4. ★ **[Systems of Record and Derived Data]** How can you determine whether a dataset is a system of record or derived data, and why does that classification matter when data is lost, stale, or contradictory?

#### My answer

### 5. **[Cloud versus Self-Hosting / Pros and Cons of Cloud Services]** For a steady, predictable workload run by an experienced operations team, which factors would you examine before deciding whether a managed cloud service is actually cheaper or easier than self-hosting?

#### My answer

### 6. ★ **[Cloud-Native System Architecture]** Why does separating compute from storage make elasticity possible, and what new dependencies or trade-offs might that separation introduce?

#### My answer

### 7. **[Distributed versus Single-Node Systems]** A service is nearing the limits of one machine. Which requirement—scalability, fault tolerance, low latency, data residency, or something else—actually forces distribution, and how would changing that requirement alter the design?

#### My answer

### 8. ★ **[Problems with Distributed Systems]** Why can a timed-out network request leave a client uncertain about what happened, and what does that uncertainty imply for retries and application behavior?

#### My answer

### 9. **[Microservices and Serverless]** A small team proposes splitting a working monolith into twelve services so each component can scale independently. What organizational and technical evidence would justify the change, and what new costs should the team expect?

#### My answer

### 10. **[Data Systems, Law, and Society]** Suppose a company retains detailed user-location events “in case they become useful.” How would data minimization, user safety, deletion rights, and the existence of derived datasets change the architecture and retention decision?

#### My answer

### 11. **[Synthesis]** Across workload specialization, cloud adoption, distribution, and privacy compliance, when does “more capability” stop being an improvement and become unjustified complexity or risk?

#### My answer

## Closed-book recall

Close the chapter before answering these prompts.

### Three most important ideas

What are the three most important ideas in this chapter?

### A surprising trade-off

Which trade-off surprised you, and why?

### What I can explain confidently

What can you now explain confidently without the book?

### What remains unclear

What remains unclear?

## Concepts

## Concept explanations

## Review

## Application challenge

## Spaced review
