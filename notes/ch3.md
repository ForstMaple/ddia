# Chapter 3 — Data Models and Query Languages

## Before reading

### Why this chapter matters

Choosing a data model shapes not only storage, but also which questions are natural to ask and how developers think about the domain. This chapter develops a practical basis for choosing among relational, document, graph, event-based, and analytical representations by connecting each model to its relationships, query patterns, and trade-offs.

### Mental map

1. **Relational Model versus Document Model** — compare tables and nested documents through object mapping, one-to-many data, normalization, joins, schema flexibility, and locality.
2. **Graph-Like Data Models** — move from tree-shaped records to highly connected data, then compare property graphs, RDF triples, and recursive query languages.
3. **Event Sourcing and CQRS** — represent changes as an ordered log of events and derive read-oriented views from that history.
4. **Dataframes, Matrices, and Arrays** — examine representations designed for interactive analysis, numerical computation, and multidimensional scientific data.
5. **Summary** — return to the central question: which model makes the required data and queries easiest to express without creating unacceptable costs elsewhere?

### Terms to notice

- Object-relational impedance mismatch and ORM
- Normalization, denormalization, references, and joins
- Schema-on-read and schema-on-write
- Data locality
- Property graphs, vertices, edges, and traversal
- RDF triples, SPARQL, Cypher, and Datalog
- Event sourcing, commands, events, CQRS, and materialized views
- Dataframes, sparse matrices, and multidimensional arrays

### Watch for

- **Containment versus connection:** nested documents fit bounded one-to-many structures, while many-to-many relationships often favor references and joins or graph traversal.
- **Read convenience versus update consistency:** duplicating data can simplify or accelerate reads but creates more copies that must remain consistent.
- **Flexible storage versus absent structure:** a database may not enforce a document schema, while application code still assumes one.
- **Source history versus derived state:** in event sourcing, distinguish immutable facts about what happened from views rebuilt to answer current queries.

### Useful prior knowledge

Basic familiarity with tables, rows, primary keys, foreign keys, JSON objects, and SQL queries will help. Knowing the broad difference between transactional and analytical workloads from Chapter 1 is useful; prior experience with graph databases, event sourcing, or numerical computing is not required.

### Reading strategy

For every model, write down its natural unit of data, the relationships it represents cleanly, the queries it makes easy, and the cost it moves onto reads, writes, or application code. Use the résumé/profile example as a running comparison, and treat query-language syntax as evidence of what a model can express rather than as syntax to memorize.

### Diagnostic question

If the same application data can be represented as tables, nested documents, a graph, or a log of events, what evidence would you use to decide which representation should be authoritative and which should be derived?

#### My answer

## Focus questions

Answer in your own words; uncertainty is useful—mark anything you’re unsure about.

### 1. ★ **[Introduction]** Why is choosing a data model more than a storage decision? Explain how it affects both the questions an application can express easily and the way developers frame the problem.

#### My answer

Choosing a data model is important because it has a profound effect on how the software is designed and written and on how we think about the problem we are solving.

- A data model determines how information and relationships are stored. Therefore, it makes a certain type of queries faster and easier and determines which relationships and operations are natural or awkward to express, even before performance is considered.
  - For example, a document data model specializes in stroging and retrieving a complete nested records, a relational data model support storing highly structured data and querying them by joins, and graph models support well the information on different entities and their relationships, using nodes and links.
- Usually the same data can be represented in different data models. It is up to the developers to decided based on what kind of problems they are solving and what functionalities the application should support.

#### Review

- **Assessment:** Solid
- **What works:** You connect representation to both query convenience and the developer's mental framing, and your examples correctly distinguish document locality, relational joins, and graph relationships.
- **Clarification:** A model does not merely make some queries faster; it also determines which relationships and operations are natural or awkward to express, even before performance is considered.

### 2. **[Introduction — Declarative Query Languages]** Contrast a declarative query with a hand-written imperative algorithm. Why can the same declarative query benefit from a later database-engine improvement without application changes?

#### My answer

In a declarative query, the user just specifies how they want the data without caring about how the database executes it. The abstraction allows the database engine to choose the most efficient way to execute the query, as long as the final format is what the user desired.
Even if there are improvements in the query engine, there is no need to adjsut the original query, provided that the resposne format remains the same.

#### Review

- **Assessment:** Solid
- **What works:** You correctly separate specifying the desired result from prescribing an execution algorithm, leaving indexes, join order, algorithms, and parallelism to the optimizer.
- **Clarification:** The stable contract is the query's meaning—not just its response format. The engine may change the execution plan as long as it returns the same semantically correct result.

### 3. ★ **[The Object-Relational Mismatch / When to Use Which Model]** A user profile contains a small, bounded set of addresses, education entries, and preferences that are almost always fetched together. Why might a document representation fit well? What change to the access pattern or relationships would make a relational design more attractive?

#### My answer

A document datamodel has good locality. It stores all the relevant in one place, so it suits the said purposes well since the informaiton is always retrieved together. By contrast, many joins are needed to gather all the information needed in a relational database.
A document fits because the profile is a bounded one-to-many tree that is normally read as a whole, giving a natural object mapping and good locality. Relational tables become more attractive if components need independent identity or updates, only small portions are usually fetched, or education/employer/address data become shared many-to-one or many-to-many relationships queried in both directions.

#### Review

- **Assessment:** Mostly right
- **What works:** You identify the document model's key advantage here: the bounded tree can be stored and fetched as one unit, with good locality and no reconstruction across tables. Accessing only small pieces is also a valid reason for reconsidering it.
- **Clarification:** ACID transactions are not inherently relational, and frequent updates alone do not imply a relational model. Likewise, table size and homogeneous data do not by themselves call for normalization. The stronger signals are independently addressed nested items, frequent small updates to a large document, and shared or many-to-many entities that benefit from references and joins.
- **Stronger answer:** A document fits because the profile is a bounded one-to-many tree that is normally read as a whole, giving a natural object mapping and good locality. Relational tables become more attractive if components need independent identity or updates, only small portions are usually fetched, or education/employer/address data become shared many-to-one or many-to-many relationships queried in both directions.

### 4. ★ **[Normalization, Denormalization, and Joins]** A company name and logo appear on millions of user profiles but occasionally change. Compare storing copies on each profile with storing one organization record referenced by ID. Trace the read, write, consistency, and storage consequences.

#### My answer

- Read: Since the name and logo appear in millions of user profiles, we can assume they are read very frequently as the user profiles are accessed. Using denormalizaiton can make such information self-contained and easily retrieved, while normalization will incur addtional cost and overhead to perform a join to get the information.
- Write: The name and logo are updated occasionally. While the write operation may not be often, we need to do so for millions of users. Normalization enables us to only update at one place - where the entity is stored, but for denomalization, we will have to find the fields to update for the millions of users, which can be very expensive and inefficient.
- Consistency: Similar to that mentioned in "Write", normalization yields good consistency since we only need to update at one place, whereas we can easily miss out some records to update if using denormalization and have to consider cases where the update fails halfway.
- Storage Consequence: Normalization provides very good storage efficiency since we only store one entity and reference it by ids in all the user profiles; denomalization can cause waste of storage since we need to store the relevant information in each relevant profile.

#### Review

- **Assessment:** Solid
- **What works:** You accurately trace all four consequences: copies favor self-contained reads, while an ID plus organization record favors cheap writes, lower storage, and a single consistent value.
- **Clarification:** Atomic multi-record updates can reduce the risk of partially updated copies, but they do not remove the write amplification or the application logic needed to find and maintain every duplicate.

### 5. **[Many-to-One and Many-to-Many Relationships]** A platform must answer both “which organizations has this person worked for?” and “which people worked for this organization?” How would a normalized relational design represent this relationship, and why is copying references on both sides risky?

#### My answer

A normalized design would represent this relationship in a so-called "associative table" or "join table". Each row associate one id with another - one person id with an organization id in this given case.
A major risk of copying the references on both sides is inconsisteny, as the references must be updated on both sides - the person as well as the organization.

#### Review

- **Assessment:** Solid
- **What works:** You put the relationship in one join table and correctly identify two-sided copies as denormalized state that can diverge.
- **Clarification:** Index both foreign-key columns (`person_id` and `organization_id`) so the one stored relationship can still be queried efficiently in either direction. An employment row may also carry relationship attributes such as title and dates.

### 6. **[Stars and Snowflakes: Schemas for Analytics]** For a retailer’s historical sales analysis, distinguish a fact table from dimension tables. Why might analysts prefer a star schema to a more normalized snowflake schema, and when can denormalization be relatively safe here?

#### My answer

In this case, one row in the fact table is likely to represent one transaction with other information of the transaction (attributes). The dimension tables stores other information that is referenced in the fact able by foreign keys, maybe details of the products, persons handling the transactions, informaiton on the transaction date, etc.
Star schema is usually prefered to the snowflake schema because it is simpler and more intuitive, as the snowflake schema further breaks down a dimension into subdimensions.
Denormalization is relatively safe if the historical data are not updated once generated, except for error corrections.

#### Review

- **Assessment:** Solid
- **What works:** You correctly treat the fact row as a sales event, dimensions as descriptive context, the star as simpler for analysts, and immutable historical data as reducing denormalization's update and consistency risks.
- **Clarification:** A fact table usually contains both measures (such as sale price or quantity) and foreign keys to dimensions, which describe the event's who, what, where, when, how, and why.

### 7. ★ **[Graph-Like Data Models]** You are building a fraud-investigation tool that must find several-hop links among people, devices, accounts, and transactions. Why is a graph model a natural fit, and what must a query language express to support this task well?

#### My answer

A graph naturally represents people, devices, accounts, and transactions as typed vertices connected by typed, property-bearing edges. Its query language must match relationship direction and labels, filter properties, and express multi-hop or variable-length paths so an investigator can ask for suspicious connection patterns without fixing the number of hops in advance.

#### Review

- **Assessment:** Mostly right
- **What works:** You correctly recognize a heterogeneous, highly connected many-to-many domain and the need for variable-length traversal rather than a fixed number of joins.
- **Clarification:** The query language should declaratively match vertex and edge types, directions, properties, and path patterns, including bounded or arbitrary-length paths. SQL can express recursion with `WITH RECURSIVE`; graph languages mainly make such patterns more direct and concise.
- **Stronger answer:** A graph naturally represents people, devices, accounts, and transactions as typed vertices connected by typed, property-bearing edges. Its query language must match relationship direction and labels, filter properties, and express multi-hop or variable-length paths so an investigator can ask for suspicious connection patterns without fixing the number of hops in advance.

### 8. ★ **[Event Sourcing and CQRS]** A conference system records registrations, cancellations, and room-capacity changes. Explain the respective roles of commands, immutable events, and materialized read models. Why must rebuilding a view be deterministic and process events in log order?

#### My answer

Command: the request made by the user. It will be validated once it comes in. If valid, it will result a new immutable event in the logs. In this case, it is the requests made by the user in the concerence system.
Immutable events: an event that records the result as a past-tense fact. In this case, it is the details of the validated user requests.
Materialized read models: a pre-computed, read-only cache or database table designed specifically to satisfy application queries. In this case, it is the derived read-only tables, dashboards, etc., from the event logs for the reference of the system manager and users.
Rebuilding a view has to be deterministic and following the event log order:
a. Ensure the data consistency: we can always produce the same view in case any data loss
b. Avoid possible errors: for example, a cancellation can only happen after a booking

#### Review

- **Assessment:** Solid
- **What works:** You distinguish a rejectable command, an accepted fact recorded as an immutable event, and query-specific state derived from events. You also correctly connect determinism to reproducibility and ordering to valid state transitions.
- **Clarification:** An event records the resulting past-tense fact, not merely the request details: for example, `SeatsBooked` rather than `BookSeatsRequested`. With the same events, order, and projection code, rebuilding must produce the same state.

### 9. **[Event Sourcing and CQRS]** Identify one advantage and one danger of replaying an event log. How would external exchange rates, personal-data deletion, or sending emails make replay more difficult?

#### My answer

Replay lets us rebuild or add views and rerun corrected projection code. Its danger is that replay may produce a different state or repeat real-world effects. Therefore external inputs must be captured deterministically, personal data must be stored or encrypted so deletion remains possible, and side effects such as email must not run as ordinary replayable projection logic.

#### Review

- **Assessment:** Mostly right
- **What works:** You identify recovery and bug correction as replay benefits, and you correctly spot nondeterministic external data, deletion conflicts, and duplicated side effects as dangers.
- **Clarification:** The exchange-rate problem is determinism, not API idempotency. Replay needs the original rate stored in the event or a historical lookup guaranteed to return the same rate for the event timestamp. Email sending should be kept outside projection replay or protected with durable deduplication.
- **Stronger answer:** Replay lets us rebuild or add views and rerun corrected projection code. Its danger is that replay may produce a different state or repeat real-world effects. Therefore external inputs must be captured deterministically, personal data must be stored or encrypted so deletion remains possible, and side effects such as email must not run as ordinary replayable projection logic.

### 10. **[Dataframes, Matrices, and Arrays]** A data scientist turns a table of user–movie ratings into a sparse user-by-movie matrix for a recommendation model. Why is this transformation useful, and how does dataframe-style data manipulation differ from writing a declarative SQL query?

#### My answer

The sparse matrix stores only observed ratings and exposes the numerical user–movie structure needed by linear-algebra-based recommendation methods. SQL declaratively states the result to compute; dataframe APIs typically apply a sequence of interactive transformations, merges, pivots, and custom functions as the scientist incrementally reshapes a working copy of the data.

#### Review

- **Assessment:** Mostly right
- **What works:** You correctly explain why sparse storage fits mostly missing user–movie combinations and why a numerical matrix is efficient input for recommendation algorithms. You also recognize dataframe APIs' broad transformation toolkit.
- **Clarification:** SQL is also heavily used for analytics and is not limited to retrieving a predefined format. The chapter's key contrast is interaction style: SQL declares the desired result, whereas dataframe work typically uses an incremental series of commands to wrangle a private dataset, often changing its structure on the way to a matrix.
- **Stronger answer:** The sparse matrix stores only observed ratings and exposes the numerical user–movie structure needed by linear-algebra-based recommendation methods. SQL declaratively states the result to compute; dataframe APIs typically apply a sequence of interactive transformations, merges, pivots, and custom functions as the scientist incrementally reshapes a working copy of the data.

### 11. **[Summary — synthesis]** Design a data system for a professional-network product with profiles, employer relationships, multi-hop “people you may know” exploration, operational screens, and offline recommendation training. Which models or derived representations would you use for each need, and what would make your choices stop applying?

#### My answer

- **Profiles:** Store bounded, mostly self-contained profile content as documents when profiles are normally loaded as a whole. This stops fitting if profile components need frequent independent updates or acquire many relationships to shared entities.
- **Employer relationships:** Keep employers, people, and employment records in a normalized relational model because many people can work for many employers and shared employer details should be updated once. If relationship traversal becomes the dominant workload, also derive a graph representation.
- **“People you may know”:** Build a derived property graph from connection, employment, education, and other relationship data so multi-hop paths are natural to query. A graph is unnecessary if recommendations use only a few fixed joins or simple numerical features.
- **Operational screens:** Build relational, document, or denormalized materialized views shaped around each screen's queries. Event sourcing and CQRS may produce these views when audit history and complex state transitions justify the additional replay and consistency complexity; otherwise direct transactional updates are simpler.
- **Offline recommendation training:** Export authoritative data into dataframes for cleaning and feature preparation, then into sparse matrices or arrays when the algorithm requires numerical input. This choice stops applying if the model needs another specialized representation, such as graph embeddings or text/vector features.

#### Review

- **Assessment:** Solid
- **What works:** You choose models from relationship shape and access pattern rather than by product label, distinguish authoritative normalized facts from several derived forms, and state concrete conditions that would invalidate each choice.
- **Clarification:** Make the source-of-truth boundary explicit for operational screens: document/relational transactional state can be authoritative, while denormalized screen-shaped views should be treated as derived and rebuildable. Event sourcing is a separate choice about making the event log authoritative, not a requirement for CQRS-style read views.

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

### Querying and the core data models

**Declarative query language** — A language in which you describe the result or pattern you want, not the sequence of operations for obtaining it. SQL, Cypher, SPARQL, and Datalog are declarative; their engines can change indexes, join algorithms, execution order, or parallelism without requiring the query to change.

**Relational database** — A database based on relations: unordered collections represented in SQL as tables of rows and columns. Relationships are usually represented with keys and reconstructed with joins, which makes the model particularly effective for structured data and many-to-one or many-to-many relationships.

**Document model** — A model that stores a record and its nested one-to-many data together, commonly as JSON. It fits data that forms a self-contained tree and is usually read as a unit, but references and many-to-many relationships can become awkward when the database has weak join support.

**Bounded document** — A document whose contents have a natural, practical limit and belong to one root entity. An order with its line items is usually bounded; a user with every post or follower embedded inside is potentially unbounded. Bounded does not mean fixed-size—it means growth is predictable enough that loading, rewriting, and storing the document as one unit remains safe.

**Impedance mismatch** — The structural mismatch between application objects and relational tables. A nested application object may need to be split across several tables when stored and then reconstructed when read.

**Object-relational mapping (ORM)** — A translation layer that maps application objects to relational rows and generates common SQL operations. It reduces repetitive mapping code, but it cannot erase the differences between the models and can hide inefficient access patterns such as N+1 queries.

**Boilerplate code** — Repetitive code required by a framework or integration even though it contains little domain logic. Mapping rows into objects by hand is boilerplate that an ORM can reduce.

### Structure, duplication, and locality

**Normalization** — Storing a human-meaningful fact once and referring to it elsewhere by a stable ID. This avoids inconsistent duplicate copies and makes updates cheaper, but reads must resolve the references through joins or lookups.

**Denormalization** — Duplicating or prejoining data so reads need less work. It can improve read performance, but uses more storage and makes writes and consistency harder because every copy must be updated.

**Atomicity** — The guarantee that a transaction's writes take effect together or not at all. It helps prevent a crash from leaving only some denormalized copies updated; atomicity does not by itself guarantee that those copies contain logically correct values.

**Hydrating** — Resolving stored IDs into the current human-readable data they identify, effectively performing joins in application code. A timeline may store only post and author IDs, then hydrate them at read time to obtain current text, like counts, names, and profile pictures.

**Data locality (storage locality)** — Keeping related data physically together so it can be retrieved with fewer seeks or index lookups. A document has good locality when most of it is read together, but locality becomes wasteful when a large document is loaded for one small field or rewritten for a tiny update.

**Schema-on-read** — Storing data without the database enforcing one uniform structure and interpreting its implicit schema when it is read. This helps with heterogeneous or evolving data, but every reader must correctly handle old, missing, or differently shaped fields.

**Fractional indexing** — Representing user-controlled order with sortable values that leave room between neighboring items. Moving an item usually changes only its own value rather than renumbering the entire list, although repeated insertions into the same gap may eventually require rebalancing.

### Dimensional modeling for analytics

**Dimensional modeling** — Organizing analytical data around measurable events and the descriptive contexts used to analyze them. Star schemas, snowflake schemas, and one big table are related implementations of this idea.

**Fact table** — The central table of events or measurements, such as one row per product sold or page viewed. It is usually very large and contains measures plus foreign keys to dimensions.

**Dimension table** — A table describing the who, what, where, when, how, or why of facts—for example, products, customers, stores, or dates. Analysts join dimensions to facts to group and filter measurements.

**Star schema** — A fact table surrounded directly by denormalized dimension tables. Its simple join structure is convenient for analysts, at the cost of some duplicated descriptive data.

**Snowflake schema** — A more normalized star schema in which dimensions reference additional **subdimensions**, such as a product dimension referring to separate brand and category tables. It reduces duplication but adds joins and makes analysis more complex.

**One big table (OBT)** — A highly denormalized analytical table that folds dimension attributes into the fact rows, effectively precomputing the joins. It requires more storage but can simplify and sometimes accelerate queries.

### Representing graphs

**Adjacency list** — A graph representation in which each vertex stores or can retrieve the IDs of its immediate neighbors. It is compact for sparse graphs and efficient for traversing from one vertex along its edges.

**Adjacency matrix** — A two-dimensional array whose rows and columns are vertices and whose cells indicate edges. It can waste space for sparse graphs, but its matrix form is useful for numerical and machine-learning operations.

**Property graph model** — A graph model with labeled vertices and directed, labeled edges; both can carry key-value properties. It supports efficient forward and backward traversal and can represent many entity and relationship types in one graph.

**Hypergraph** — A generalization of a graph in which one edge can connect more than two vertices. In a property graph, the same higher-degree relationship is usually modeled as an extra vertex connected to all participants.

**Triple-store model** — A graph-like model that represents every fact as a subject–predicate–object triple. If the object is a value, the triple acts like a property; if it identifies another entity, the predicate acts like an edge.

**RDF (Resource Description Framework)** — A standardized subject–predicate–object data model designed for exchanging and combining graph data. RDF commonly uses URIs as globally distinguishable identifiers so independently created vocabularies do not collide.

**Turtle (triple format)** — A concise, human-readable syntax for encoding RDF triples. Turtle is a serialization format for RDF data, not a separate graph model or query language.

### Languages associated with graphs

**Datalog** — A declarative logic-query language built from facts and rules. Its recursive rules can derive relationships such as arbitrary-length reachability, making it powerful for graph queries even though its underlying model is relational.

**GraphQL** — An API query language through which a client requests a JSON response with exactly the fields and nesting it needs. Despite its name, it does not require a graph database and deliberately omits arbitrary recursive graph searches; clients may traverse only relationships exposed by the server's schema.

### Event-based models

**Event sourcing** — Recording every valid state change as an immutable, ordered event and treating that event log as the source of truth. Current state and query-specific materialized views are derived by replaying the same events in order, so replay must be deterministic and external side effects must be controlled.

**Command query responsibility segregation (CQRS)** — Separating the model used to accept and validate writes from the models optimized for reads. A command is a requested action that may be rejected; after validation it produces an event, from which one or more read models are updated.

**Crypto-shredding** — Encrypting sensitive data with a dedicated key and later deleting the key to make the remaining ciphertext unreadable. It can help reconcile immutable event logs with deletion requirements, but only if every usable copy of the key is removed and the remaining metadata does not itself reveal sensitive information.

## Review

### Pattern across your answers

- Your core decision framework is strong: you usually choose a representation from relationship shape, access pattern, and the work moved onto reads or writes.
- Your best answers trace consequences across a whole lifecycle, especially normalization versus denormalization and the synthesis design.
- The recurring gap is attributing a property too broadly to a model: ACID is not uniquely relational, SQL is not merely for predefined retrieval, and replay determinism is distinct from API idempotency.

### Review next

1. [When to Use Which Model](/workspaces/ddia/content/en/ch3.md:511): focus on the precise signals for document locality versus independent identity, small updates, references, and joins.
2. [Event Sourcing and CQRS](/workspaces/ddia/content/en/ch3.md:1423): separate commands from resulting facts, and deterministic projections from controlled external side effects.
3. [Dataframes, Matrices, and Arrays](/workspaces/ddia/content/en/ch3.md:1552): contrast declarative result specification with incremental dataframe wrangling without treating SQL and dataframes as having mutually exclusive purposes.

### Follow-up

1. A document database offers multi-document ACID transactions. Which profile workload changes would still make normalized tables more attractive, and why?
2. During replay, a historical-rate API always accepts repeated identical requests but returns corrected historical values. Is it idempotent, deterministic for projection rebuilding, both, or neither? Explain.
3. Express the same data-cleaning goal once as a declarative result and once as a sequence of dataframe transformations. What optimization freedom does each interface give its engine?

## Application challenge

Design the data representations for a professional-network service that supports editable profiles, shared employer records, multi-hop connection exploration, an auditable history of profile changes, business analytics, and offline recommendation training.

Choose which representation is authoritative for each kind of fact and which representations are derived. Justify where you would use relational tables, documents, a graph, an event log, a star schema, dataframes, or sparse matrices; identify the consistency and rebuild requirements; and state one workload change that would make you revise the design.

## Spaced review

### 2026-07-13

- [ ] When does a nested document fit better than normalized tables, and what relationship change weakens that fit?
- [ ] Explain normalization as a trade-off among reads, writes, storage, and consistency.
- [ ] Why can a declarative query benefit from database-engine improvements without application changes?

### 2026-07-18

- [ ] Contrast a property graph with RDF triples, and name the query language introduced for each.
- [ ] Distinguish a command, an event, and a materialized read model in an event-sourced system.
- [ ] Why must event replay preserve log order and avoid uncontrolled external side effects?

### 2026-07-27

- [ ] A large profile document receives frequent updates to one independently addressed education entry. Name two costs and one alternative representation.
- [ ] Why are ACID guarantees and the relational data model separate design dimensions?
- [ ] What must remain stable when an optimizer changes a declarative query's execution plan?

### 2026-08-01

- [ ] Design a graph pattern that finds accounts connected through one to four shared devices. What vertex, edge, direction, property, and path constraints must the language express?
- [ ] Contrast an idempotent external call with deterministic materialized-view rebuilding.
- [ ] Where should email sending sit relative to replayable projection logic, and why?

### 2026-08-25

- [ ] Given people, employers, jobs, and skills, decide which relationships belong in documents, normalized tables, or a graph, and justify the boundaries.
- [ ] Compare SQL and dataframe manipulation by interaction style and optimizer freedom, not merely by user or purpose.
- [ ] For a replayed price-conversion view, specify where the exchange rate should come from and how deletion and email side effects should be handled.
