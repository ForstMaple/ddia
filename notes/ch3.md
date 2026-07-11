# Chapter 3 — Data Models and Query Languages

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
