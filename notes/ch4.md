# Chapter 4 — Storage and Retrieval

## Before reading

### Why this chapter matters

The storage engine underneath a database strongly influences which workloads it handles well, even when databases expose similar APIs. This chapter gives you a practical model for reasoning about read speed, write throughput, disk use, recovery, and analytical query performance—enough to choose and tune systems more deliberately.

### Mental map

1. **Storage and Indexing for OLTP** begins with an append-only log and introduces indexes as a read–write trade-off.
2. **Log-Structured Storage** develops sorted immutable files, in-memory buffering, and background merging into the LSM-tree approach.
3. **B-Trees** presents the contrasting update-in-place, page-oriented approach, followed by a direct comparison with LSM-trees.
4. **Data Storage for Analytics** shifts from small record lookups to large scans, motivating column-oriented storage and CPU-efficient query execution.
5. **Multidimensional and Full-Text Indexes** broadens indexing beyond one-dimensional keys to spatial, textual, and semantic similarity queries.

### Terms to notice

- index
- SSTable, memtable, and LSM-tree
- compaction
- Bloom filter
- B-tree page and branching factor
- read, write, and space amplification
- column-oriented storage
- inverted index and vector embedding

### Watch for

- **Primary data versus index:** an index is derived data that improves selected reads while adding storage and write work.
- **Immutable segments versus update-in-place pages:** LSM-trees and B-trees organize writes around fundamentally different constraints.
- **OLTP versus OLAP access patterns:** fast point operations and range lookups demand a different layout from wide scans and aggregations.
- **Exact versus approximate search:** some specialized indexes trade result accuracy for substantially faster retrieval.

### Useful prior knowledge

A working understanding of keys, records, primary and secondary indexes, and basic relational queries is enough. It helps to remember Chapter 1's OLTP–OLAP distinction and Chapter 3's star-schema terminology; familiarity with logarithmic lookup, sequential versus random access, and CPU caches is useful but not required.

### Reading strategy

For each storage structure, track four questions: what is written, what stays in memory, what happens during a read, and what background maintenance is required. Treat the LSM-tree/B-tree comparison as the chapter's central contrast, then notice how the same principle—arranging data around expected queries—reappears in analytics and specialized indexes.

## Focus questions

Answer in your own words; uncertainty is useful—mark anything you’re unsure about.

### 1. ★ **[Storage and Indexing for OLTP]** Why is an append-only file fast for writes but slow for reads, and what new costs arise when you add an index?

#### My answer


### 2. **[Log-Structured Storage]** Which limitations of an in-memory hash index motivate storing records sorted by key in SSTables?

#### My answer


### 3. ★ **[Constructing and merging SSTables]** Reconstruct the lifecycle of a write in an LSM storage engine, from arrival through memtable persistence and later compaction.

#### My answer


### 4. **[Bloom filters and Compaction strategies]** Why can a Bloom filter safely avoid some SSTable reads despite being probabilistic, and how could the workload influence the choice of compaction strategy?

#### My answer


### 5. ★ **[B-Trees]** How does a B-tree locate and update a key, and why do page splits and crash recovery complicate that mechanism?

#### My answer


### 6. ★ **[Comparing B-Trees and LSM-Trees]** For a write-heavy service that also requires predictable point-read latency, how would you reason about choosing between an LSM-tree and a B-tree?

#### My answer


### 7. **[Multi-Column and Secondary Indexes]** How does column order constrain a concatenated index, and how does storing values in or near an index change read and write behavior?

#### My answer


### 8. ★ **[Data Storage for Analytics and Column-Oriented Storage]** Why does an analytical query that scans many rows benefit from column-oriented storage, and when would that layout become less attractive?

#### My answer


### 9. **[Query Execution: Compilation and Vectorization]** How do query compilation and vectorized processing take different routes to reducing CPU cost during large analytical scans?

#### My answer


### 10. **[Materialized Views and Data Cubes]** A dashboard repeatedly computes the same aggregates, but users occasionally invent new filters. What would materializing results improve, and where would it stop helping?

#### My answer


### 11. ★ **[Multidimensional and Full-Text Indexes; Vector Embeddings]** How does the shape of a query lead to different index designs for geographic ranges, keyword conjunctions, and semantic similarity, and where does approximation enter?

#### My answer


## Closed-book recall

Close the chapter before completing these prompts; do not consult the text.

### Three most important ideas


### A surprising trade-off


### What I can explain confidently


### What remains unclear


## Concepts

### Indexes and log-structured storage

**Index** — An additional structure derived from primary data that organizes lookup keys so queries can avoid scanning every record. Each index trades extra disk space and write work for faster reads, so its value depends on the application's query patterns.

**Sorted String Table (SSTable)** — An immutable file of key-value pairs sorted by key, with each key appearing at most once in that file. Sorting enables efficient range scans, sparse indexes, compression, and low-memory merging of several files.

**Sparse** — A sparse index stores entries for only some keys—often the first key of each SSTable block—then relies on sorting to scan the small block where the target must lie. In a sparse bitmap, the analogous idea is that relatively few bits are set, making compressed representations attractive.

**Red-black tree** — A self-balancing binary search tree that keeps insertion, lookup, and ordered traversal at $O(\log n)$. It is one possible in-memory ordered map for a memtable.

**Skip list** — A sorted linked structure with additional, progressively sparser pointer layers that let a lookup skip over many entries. Its expected insertion and lookup cost is $O(\log n)$, and ordered iteration is straightforward, making it another memtable option.

**Trie** — A prefix tree whose path edges represent characters or bits of a key. Shared prefixes can make prefix lookup and ordered traversal efficient; the same idea later supports finite-state structures used for fuzzy term search.

**Memtable** — The mutable, sorted in-memory component of an LSM engine. Writes enter it after being recorded in a durability log; when it reaches a threshold, it is flushed in key order as an immutable SSTable.

**Tombstone** — A deletion marker written in place of immediately removing every older copy of a key. Reads treat it as a deletion, and compaction eventually uses it to discard older values; it can itself be removed only after it has reached all data it must suppress.

**Log-Structured Merge Tree (LSM-tree)** — A storage design that converts cheap writes to an in-memory ordered structure into immutable sorted files, then merges and compacts those files in the background. It usually favors write throughput, but reads may need to reconcile several SSTables and compaction consumes I/O.

**Bloom filter** — A compact probabilistic set representation that hashes each key to several bits. A zero bit proves a key is absent, while all-one bits mean only “possibly present,” so an LSM engine can safely skip many SSTables at the cost of occasional false-positive reads.

**Size-tiered compaction** — Similarly sized SSTables are merged into successively larger files. It supports high write throughput, but can leave more overlapping files to check during reads and needs substantial temporary disk space when large files merge.

**Leveled compaction** — SSTables are organized into levels and partitioned into smaller key ranges, with limited overlap at older levels. Incremental compaction improves read efficiency and disk-space usage, generally at the cost of more rewriting than size-tiered compaction.

### B-trees and storage-engine trade-offs

**B-tree** — A balanced tree of fixed-size disk pages whose internal keys divide the key space into ranges. Lookups follow a small number of child references to a leaf; updates overwrite pages in place, splitting full pages and possibly propagating splits toward the root.

**Leaf page** — A page at the bottom of a B-tree containing individual keys and either their values or references to those values. Linked sibling leaves in some variants also make ordered range scans efficient.

**Branching factor** — The number of child-page references an internal B-tree page can hold. A high branching factor—often hundreds—keeps the tree shallow, so even a very large index needs only a few page reads per lookup.

**Torn page** — A page only partly written because a crash or hardware failure interrupted a non-atomic page overwrite. It may contain a mixture of old and new data, so recovery mechanisms such as a WAL and checksums are needed to detect or repair it.

**Write-ahead log (WAL)** — An append-only record of changes that must be made durable before the corresponding B-tree pages are modified. After a crash, replaying the WAL restores a consistent tree and preserves acknowledged changes that had not yet reached their pages.

**Copy-on-write** — Instead of overwriting a page, the database writes the modified version elsewhere and creates new parent pages pointing to it, eventually publishing a new root. This avoids exposing a partly updated tree and preserves old versions, but creates additional pages that must later be reclaimed.

**Backpressure** — Flow control applied when foreground writes arrive faster than flushing or compaction can absorb them. An LSM engine may delay or suspend operations when memtables fill, preventing unbounded memory growth but causing latency spikes.

**Random writes** — Many small writes to scattered storage locations, typical when unrelated B-tree pages are updated. They have lower throughput than large contiguous writes, especially on HDDs, and can also increase SSD garbage-collection work.

**Sequential writes** — Fewer, larger writes to contiguous regions, as when an LSM engine flushes or compacts an entire segment. They use storage bandwidth more efficiently, although the logical data may later be rewritten by compaction.

**Write amplification** — The ratio of bytes actually written by the storage system to the minimum bytes represented by application writes. WAL writes, whole-page updates, memtable flushes, and repeated compactions all increase it, reducing write throughput and accelerating SSD wear.

**Fragmentation in a B-tree** — Deletions and page splits can leave unused pages or free space scattered inside the database file. The space may be reusable internally but is difficult to return to the operating system without a rewrite or maintenance process such as vacuuming.

### Index placement and OLAP interaction

**Clustered index** — An index that stores the actual rows directly in index-key order rather than merely pointing elsewhere. Reads near the clustering key gain locality, but there can be only one physical clustering order and updates may move or split stored rows.

**Heap file** — A collection of rows stored without index-key order; secondary indexes point to row locations or identifiers in it. This decouples row placement from any one index, but an indexed lookup often requires an extra heap access.

**Covering index** — An index that includes all columns needed by a particular query, allowing the query to be answered without fetching the base row. It saves reads but duplicates data, consumes space, and adds work to writes.

**Drill-down** — Moving from a summarized analytical view to finer-grained detail, such as yearly sales → monthly sales → individual transactions. It changes the aggregation level rather than merely filtering the same level.

**Slicing and dicing** — Selecting and rearranging subsets of an OLAP dataset across dimensions—for example, viewing one region and comparing products by month. It lets analysts inspect the same facts from different dimensional perspectives.

### Column-oriented storage and compression

**Column-oriented storage** — Values from each column are stored together within row blocks rather than storing complete rows together. Analytical queries can read only the few columns they need and compress them well, while point updates and reconstruction of whole rows are less convenient.

**Shredding/Stripping** — Encoding nested records by separating their fields into column streams while retaining enough repetition/definition information to reconstruct the original nesting. Formats such as Parquet use this to bring columnar benefits to document-shaped data.

**Bitmap encoding** — For a column with $n$ distinct values, create $n$ bitmaps with one bit per row; a bit is set when that row has that value. Filters combine these bitmaps quickly with bitwise AND/OR, and low-cardinality or sparse data compresses well.

**Run-length encoding (RLE)** — Replace consecutive repetitions with a value and run length, or store lengths of alternating zero/one runs in a bitmap. Sorting similar values together creates longer runs and therefore better compression.

**Roaring bitmaps** — A bitmap representation that partitions the integer space and chooses a compact container representation for each partition, such as a bitmap or a list/run representation. It stays compact across both dense and sparse regions while supporting fast set operations.

**Wide-column model** — A row-oriented model in which each row may have a large, flexible set of columns, often grouped into column families. Despite the name, it is distinct from column-oriented analytical storage because values belonging to one row are stored together.

### Query execution and precomputation

**Operators** — The stages in a physical query plan—such as scan, filter, join, and aggregate—that transform batches or streams of records. Planning chooses their implementations, order, placement, and parallelism; execution cost depends heavily on how efficiently each operator processes data.

**Query compilation** — Generate specialized machine code for a particular query rather than repeatedly interpreting its plan row by row. The resulting tight loops reduce dispatch and branching overhead, though compilation itself has a startup cost.

**Vectorized processing** — Interpret a plan using predefined operators that process batches of column values rather than one row at a time. Batching improves cache locality, reduces function-call overhead, and enables SIMD; here “vector” means a data batch, not an embedding.

**Materialized aggregates** — Precomputed `COUNT`, `SUM`, or similar results stored as materialized views. Repeated reads become much faster, but every underlying change must update or refresh the stored result and unanticipated groupings still require raw data.

**Data cube (OLAP cube)** — A grid of materialized aggregates grouped along selected dimensions, with rollups that summarize across one or more dimensions. It makes supported multidimensional queries very fast but cannot answer filters or dimensions that were not represented when the cube was designed.

### Multi-attribute and text indexes

**Concatenated index** — A multi-column index that forms one ordered key by appending fields in a declared order, such as `(last_name, first_name)`. It efficiently supports predicates on the leftmost prefix, but not generally a predicate only on a later field.

**Multidimensional index** — An index that narrows several dimensions simultaneously, such as latitude and longitude, instead of imposing only a lexicographic field order. Spatial trees and space-filling-curve schemes group nearby points, making multi-axis range queries efficient.

**Inverted index** — A mapping from each term to the documents containing it—the reverse of mapping each document to its terms. Intersecting or unioning term entries answers keyword conjunctions and disjunctions efficiently.

**Postings list** — The document identifiers associated with one term in an inverted index, often augmented with frequency or position data. Sorted IDs, sparse bitmaps, and compression make intersections fast and compact.

**Levenshtein automation** — A finite-state machine that accepts strings within a chosen edit distance of a query term. Intersecting it with a trie-like automaton of indexed terms finds typo-tolerant matches without comparing the query against every term.

### Semantic search and vector indexes

**Retrieval-augmented generation (RAG)** — Retrieve documents relevant to a prompt, place their contents in the model's context, and then generate an answer grounded in that retrieved material. Vector search is one possible retrieval mechanism; RAG is the larger pipeline, not an index type.

**Multimodal** — Able to process more than one modality, such as text and images, in a shared model or embedding space. A multimodal embedding model can make semantically related items from different media searchable by proximity.

**Flat index** — Store every embedding directly and compare the query with every vector. This exact search provides full recall but has linear query cost, so it becomes slow as the collection grows.

**Inverted file (IVF) index** — Cluster vectors into partitions and search only selected partitions near the query. It greatly reduces distance computations but is approximate because a nearby vector may lie across a partition boundary.

- **Centroids** — Representative center vectors used to assign stored vectors—and later a query—to IVF partitions.
- **Probes** — The number of IVF partitions searched for a query; more probes improve recall but increase latency and computation.

**Hierarchical Navigable Small World (HNSW) index** — An approximate nearest-neighbor graph with sparse upper layers for long jumps and increasingly dense lower layers for local refinement. Search descends layer by layer toward nearer vectors, trading some recall and substantial memory/build cost for fast queries.

## Review

## Application challenge

## Spaced review
