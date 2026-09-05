# Chapter 5 — Encoding and Evolution

## Before reading

### Why this chapter matters

Changing application code is easy compared with changing every stored record, deployed server, mobile client, and queued message at once. This chapter connects byte-level encoding choices to a system’s ability to perform rolling upgrades and let independently deployed components evolve without losing or misreading data. The formats are examples; the deeper subject is compatibility across time and process boundaries.

### Mental map

1. Begin with the coexistence problem: old and new code, schemas, and data formats can all be active simultaneously.
2. Compare language-specific, textual, and schema-driven binary encodings by portability, safety, efficiency, and evolvability.
3. Follow how Protocol Buffers and Avro encode records and support schema evolution through different mechanisms.
4. Track who encodes and who decodes as data moves through databases, service calls, and workflow engines.
5. Finish with asynchronous dataflow through message brokers and distributed actors, then reconnect every mode to compatibility.

### Terms to notice

- Encoding and decoding
- Backward compatibility and forward compatibility
- Rolling upgrade
- Field tag
- Writer’s schema and reader’s schema
- Remote procedure call (RPC)
- Durable execution
- Message broker

### Watch for

- **Backward versus forward compatibility:** always identify which version wrote the data and which version is trying to read it.
- **Schema present versus schema embedded in every record:** a format can require a schema without repeating field names or the entire schema in each encoded value.
- **Local call versus remote call:** similar-looking APIs do not give local and network operations the same failure behavior.
- **Synchronous versus asynchronous dataflow:** whether the sender waits changes coupling and failure handling, but not the need for compatible encodings.

### Useful prior knowledge

Recall schema-on-write versus schema-on-read from Chapter 3 and why rolling deployment helps evolvability from Chapter 2. Familiarity with JSON, database migrations, HTTP APIs, and queues will make the examples easier, but no knowledge of Protocol Buffers or Avro is assumed.

### Reading strategy

For every schema change or communication mode, write down four roles: old writer, new writer, old reader, and new reader. Ask which pairings must work and what information lets a reader interpret or safely ignore unfamiliar data. Treat the detailed wire-format diagrams as evidence for the compatibility rules rather than facts to memorize byte by byte.

## Focus questions

Answer the core questions first in your own words; uncertainty is useful—mark anything you’re unsure about. Extension questions are optional.

**Core questions — answer these first.**

### 1. ★ **[Encoding and Evolution]** Why do rolling upgrades require both backward and forward compatibility, and how can an older application instance accidentally destroy a field introduced by newer code?

#### My answer


### 2. **[Language-Specific Formats; JSON, XML, and Binary Variants]** Why are language-specific encodings risky for long-lived shared data, and which datatype and portability problems remain when using JSON, XML, or CSV instead?

#### My answer


### 3. ★ **[Protocol Buffers; Field tags and schema evolution]** How do field tags allow Protocol Buffers records to remain compact and evolve, and which schema changes would put compatibility at risk?

#### My answer


### 4. ★ **[Avro; The writer’s schema and the reader’s schema]** How can Avro omit both field names and tags from each record while still allowing a reader with a different schema version to decode it?

#### My answer


### 5. **[Dataflow Through Databases]** Why must a database commonly contain data written under several historical schemas, and what compatibility problems arise when old and new application instances access it concurrently?

#### My answer


### 6. ★ **[The problems with remote procedure calls (RPCs)]** Why is treating a remote request like a local function call a misleading abstraction, especially when a timeout is followed by a retry?

#### My answer


### 7. ★ **[Modes of Dataflow; Data encoding and evolution for RPC; Durable Execution and Workflows; Event-Driven Architectures]** How does the path taken by encoded data change which components must remain compatible and how independently they can evolve?

#### My answer


**Extension questions — optional deeper practice.**

### 8. **[The Merits of Schemas]** What guarantees and tooling can a schema-driven binary format provide that schema-on-read JSON does not, and what does it give up?

#### My answer


### 9. **[Different values written at different times; Archival storage]** How should the observation that “data outlives code” affect a migration plan and the encoding chosen for an archival snapshot?

#### My answer


### 10. **[Load balancers, service discovery, and service meshes]** How would the rate at which service instances change influence a choice among DNS, a discovery registry, and a service mesh?

#### My answer


### 11. **[Durable execution; Message brokers]** A payment workflow and a message consumer may both retry work after failure. What must the surrounding APIs and message-handling code guarantee to prevent retries from duplicating external effects?

#### My answer


## Closed-book recall

Close the chapter before completing these prompts; do not consult the text.

### Three most important ideas


### A surprising trade-off


### What I can explain confidently


### What remains unclear


## Concepts

Rolling Upgrade / Staged Rollout

## Review

## Application challenge

## Spaced review
