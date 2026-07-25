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


### 2. **[Introduction — Declarative Query Languages]** Contrast a declarative query with a hand-written imperative algorithm. Why can the same declarative query benefit from a later database-engine improvement without application changes?

#### My answer


### 3. ★ **[The Object-Relational Mismatch / When to Use Which Model]** A user profile contains a small, bounded set of addresses, education entries, and preferences that are almost always fetched together. Why might a document representation fit well? What change to the access pattern or relationships would make a relational design more attractive?

#### My answer


### 4. ★ **[Normalization, Denormalization, and Joins]** A company name and logo appear on millions of user profiles but occasionally change. Compare storing copies on each profile with storing one organization record referenced by ID. Trace the read, write, consistency, and storage consequences.

#### My answer


### 5. **[Many-to-One and Many-to-Many Relationships]** A platform must answer both “which organizations has this person worked for?” and “which people worked for this organization?” How would a normalized relational design represent this relationship, and why is copying references on both sides risky?

#### My answer


### 6. **[Stars and Snowflakes: Schemas for Analytics]** For a retailer’s historical sales analysis, distinguish a fact table from dimension tables. Why might analysts prefer a star schema to a more normalized snowflake schema, and when can denormalization be relatively safe here?

#### My answer


### 7. ★ **[Graph-Like Data Models]** You are building a fraud-investigation tool that must find several-hop links among people, devices, accounts, and transactions. Why is a graph model a natural fit, and what must a query language express to support this task well?

#### My answer


### 8. ★ **[Event Sourcing and CQRS]** A conference system records registrations, cancellations, and room-capacity changes. Explain the respective roles of commands, immutable events, and materialized read models. Why must rebuilding a view be deterministic and process events in log order?

#### My answer


### 9. **[Event Sourcing and CQRS]** Identify one advantage and one danger of replaying an event log. How would external exchange rates, personal-data deletion, or sending emails make replay more difficult?

#### My answer


### 10. **[Dataframes, Matrices, and Arrays]** A data scientist turns a table of user–movie ratings into a sparse user-by-movie matrix for a recommendation model. Why is this transformation useful, and how does dataframe-style data manipulation differ from writing a declarative SQL query?

#### My answer


### 11. **[Summary — synthesis]** Design a data system for a professional-network product with profiles, employer relationships, multi-hop “people you may know” exploration, operational screens, and offline recommendation training. Which models or derived representations would you use for each need, and what would make your choices stop applying?

#### My answer


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

Declarative Query Language
Relational Database
Document Model
Impedance Mismatch
Object-Relational MappingBoilerplate code
Data locality
Normalization
Denormalization
Atomicity
Hydrating

Star Schema

- Fact table
- Dimensional Table
Snowflake Schema
- Subdimensions

Dimensional Modeling
One Big Table
Fractional Indexing

Schema-on-read
Storage locality

Adjacency list
Adjacency matrix
Property graph model
Triple store model
Hypergraph
Turtle (Triple format)
RDF (Resource Description Framework)
Datalog
GraphQL

Event Sourcing
Command Query Responsibility Segregation
Crypto Shredding

## Review

Complete the focus questions and closed-book recall first. Then review whether you can justify a model from its relationships and access patterns—not merely name a database product—and whether you can explain what work each choice moves onto reads, writes, consistency handling, or derived-view maintenance.

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

### 2026-08-11

- [ ] Given people, employers, jobs, and skills, decide which relationships belong in documents, normalized tables, or a graph, and justify the boundaries.
- [ ] Compare a star schema, a dataframe, and a sparse matrix by the analytical task each makes natural.
- [ ] Explain why “schemaless” storage does not mean that an application has no schema assumptions.
