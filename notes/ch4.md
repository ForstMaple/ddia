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

Append-only file is fast for writes because it is just writing a new record at the end of the file instead of overwriting a value. It is slow for reads because we need to scan the whole file to find the latest occurrence of a key.
An index will incur additional space to store them. There will be computation resources used when we update the index when new data come in.

#### Review

- **Assessment:** Solid.
- **What works:** You correctly connect append-only writes to minimal write work, linear scans to slow lookups, and indexes to extra space and write maintenance.
- **Clarification:** Indexes can slow writes substantially because every relevant index must stay consistent with the primary data; their value therefore depends on the actual query patterns.

### 2. **[Log-Structured Storage]** Which limitations of an in-memory hash index motivate storing records sorted by key in SSTables?

#### My answer

- In-memory hash index is not persistent. It will be lost when we restart the database.
- Memory is expensive to scale as the data grow.
- We need complicated logic to avoid hash collisions.
- It does not support range queries very well.
- Sorting also enables sparse indexes, compression, and low memory merging that reclaims obsolete log entries.

#### Review

- **Assessment:** Mostly right.
- **What works:** You identified the full-memory key directory, restart/rebuild issue, awkward range queries, and collision/growth concerns that make disk-backed hash tables unattractive.
- **Clarification:** The in-memory hash table itself does not require “a lot of random I/O”; that problem arises when trying to put the hash table on disk. Sorting also enables sparse indexes, efficient range scans, compression, and low-memory merging that reclaims obsolete log entries.


### 3. ★ **[Constructing and merging SSTables]** Reconstruct the lifecycle of a write in an LSM storage engine, from arrival through memtable persistence and later compaction.

#### My answer (revised)

- When a new write comes in, it will be stored in an in-memory ordered map (red-black tree, skip list, trie, etc.). Such data structure supports fast insertions and lookups, and it can be read back in a sorted order. This is called a "memtable".
  - When there is a new write, the engine also keep a separate log of the incoming data. In this case, even if the database or the system crashes, we can recover the memtable and avoid losing any data. Once the memtable is flushed, we can delete these logs.
- When the memtable grows above a certain size (typically a few megabytes), it will be written to the disk as an SSTable. It will end up as the latest segment of the database along with other old segments. It also has its own index of the contents. Meanwhile, the memory for the outdated memtable will be freed and accept new writes.
- When looking up a value, we look it up in the sequence of memtable, latest segment, and old segment.
- The database will periodically merge and compact the segment files. Outdated and overwritten values will be deleted.

#### Review

- **Assessment:** Solid. The earlier durability gap is resolved.
- **What improved:** You now state that every incoming write is also recorded in a separate durable log, allowing the unflushed memtable to be reconstructed after a crash, and that the corresponding log can be discarded after the memtable is safely flushed. Your lifecycle from write through compaction is complete.

### 4. **[Bloom filters and Compaction strategies]** Why can a Bloom filter safely avoid some SSTable reads despite being probabilistic, and how could the workload influence the choice of compaction strategy?

#### My answer

Bloom filters apply hash functions on the keys and produce a set of numbers that are interpreted as indexes into the array of bits.
While bloom filters can have false positives - the bits are set to 1 by other keys by accident, but it can efficiently and accurately tell when the queried key does not exist in the SSTable, and we can safely skip that table.

- Size-tiered Compaction: When serveral SSTables reach roughtly the same size, merge them into a bigger table. Files of different sizes can cover overlapping range of keys, so querying a key may need to check different SSTables. Meanwhile, more temporary space will be necessary for large merges, both the inputs and outputs. Size-tiered compaction has higher write throughputs, suitable for scenariso where writes are dominant and reads are relatively rare.
- Leveled Compaction: SSTables are divided into different levels with increasing size limits. At each level, the tables are partitioned to cover non-overlapping key ranges. When merging, a table is only merged with the next-level tables whose key ranges overlap. This makes the compaction smaller and more incremental. Also, a point read only needs to go through a few candidate tables each level. However, one incoming range may repeatedly rewrite overlapping data already in lower levels, causing a write ampification. Therefore, leveled comapction performs better when reads are dominant and writes only update a small set of hot keys.

#### Review

- **Assessment:** Solid.
- **What works:** You explain the Bloom filter’s one-sided guarantee correctly and connect size-tiered versus leveled compaction to write throughput, read amplification, write amplification, and temporary space.
- **Clarification:** Strategy choice should also consider sustained compaction bandwidth and disk headroom, since either strategy can cause latency or capacity trouble when background work cannot keep up.

### 5. ★ **[B-Trees]** How does a B-tree locate and update a key, and why do page splits and crash recovery complicate that mechanism?

#### My answer

When looking up a key in a B-tree, it starts at the root page, where the references to its child pages are stored. Child pages stores a continuous range of keys, and the references tell the boundaries of keys that each page is storing. We may need to go futher into the child pages' child pages until we hit the leaf page, where the individual keys are stored. The leaf page either contains the key value inline or a reference to the page where the value can be found.

When updating a key, we first go through a similar process of the lookup and find the leaf page storing the key. Then, we rewrite the pages with the new value. If adding a new key, we need to find the page that is supposed to store the key and add it to that page. However, we may need to split the page and its parent page (sometimes all the way to the root page) into two half-full pages if the page is full.

Since we need to overwrite a page to perform an update, we have to consider cases where the database crashes in midst of the overwrite. Some pages may become corrupted. B-tree tends to implement a "write-ahead log" to make itself resilient of crashes.

#### Review

- **Assessment:** Solid.
- **What works:** You correctly describe root-to-leaf traversal, in-place leaf updates, cascading page splits, and the danger of partial multi-page updates or torn pages.
- **Clarification:** The term is **write-ahead log (WAL)**: the modification is made durable in the WAL before the corresponding tree pages are written, enabling recovery to a consistent state.

### 6. ★ **[Comparing B-Trees and LSM-Trees]** For a write-heavy service that also requires predictable point-read latency, how would you reason about choosing between an LSM-tree and a B-tree?

#### My answer
While it may depend the specific requirments on write throughput and point-read latency, LSM is likely to be a better option in this case.

- LSM provides better write throughput because it incurs sequential I/O compared with B-tree's random I/O, though LSM can also encounter bottlenecks when the memtable fills up too fast and disk I/O cannot catch up. At the same time, LSM flush/compaction can also create latency spikes. It will be good to evaluate the tail latency after the merging and compactions kicks off and determine whether it still meets the requirement of the "predictable point-read latency".
- B-tree has more predictable point-read latency because each lookup goes through one page at one level, and the depth of the tree is usually small enough. LSM may need to look up a few different SSTables at different stages of compactions, but the Bloom filter can help reduce the number of disk I/O needed.

#### Review

- **Assessment:** Mostly right.
- **What works:** You identify the central tension: LSM’s sequential, batched writes usually favor throughput, while a shallow B-tree gives a more bounded point-read path.
- **Clarification:** “Predictable latency” makes the decision depend heavily on the percentile SLO. LSM flush/compaction backpressure can create latency spikes, so benchmark the sustained workload after compactions begin; a strict tail-latency requirement can justify a B-tree despite lower write throughput.

### 7. **[Multi-Column and Secondary Indexes]** How does column order constrain a concatenated index, and how does storing values in or near an index change read and write behavior?

#### My answer (revise)

The column order of the concatenated determines how the data are sorted. It is only helpful when we want to make a query with the leading columns. For example, if we have a concatenated index (A, B, C), it is only helpful when we make a query to find certain combinations of (A), (A, B), or (A, B, C).

- Clustered indexes: the values are part of the index structure
  - Read: the value is or is part of the key per se
  - Write: clustered indexes determine the physical order that a row is stored in the disk. If the clustered index is not a self-increment id or something similar, there will be some overhead to insert or update a value, which can also cause page splits.

- Stroing the reference to the actual data
  - The value can be the primary key of the row or a referene to a location on the disk (heap file)
  - Read: we need to use primary key or the reference to navigate to the value in the heap file
  - Write: if possible, the heap file can be updated in-place, but if the new value is longer than the preivous one, we may need to find a new place in the heap to store it. After that, we need to update all affected indexes to point to the new location in the heap, or we need to leave a forward pointer at the old heap location to point to the new one.

- Covering indexes / indexes with included columns
  - Read: the value is not part of the key itself, but it is stored within the index, so we don't have to got to the heap file to get the value
  - Write: first, we need to update the heap file; after that we need to update the values stored within ithe index

#### Review

- **Assessment:** Needs revision.
- **What works:** Your leftmost-prefix explanation is correct, and you recognize the locality-versus-duplication trade-off among clustered, heap-backed, and covering indexes.
- **Clarification:** The earlier gap remains in the clustered-index description. The index search key determines ordering and lookup; the stored row is the payload and is not thereby “part of the key.” Maintaining a clustered index also does not re-sort the whole table on every write. The engine locates the target page and updates, splits, or relocates pages as needed. Your covering-index description correctly says that included columns are stored in the index without becoming search-key columns.
- **Stronger answer:** A concatenated index on `(A, B, C)` is ordered lexicographically, so it efficiently supports predicates beginning with `A`, such as `A`, `(A, B)`, or `(A, B, C)`, but generally not `B` alone. A heap-backed secondary index stores a row reference, making the index smaller and writes cheaper but requiring another lookup to fetch the row. A clustered index stores the row with the ordered index entry, improving read locality but making inserts, updates, and page splits more expensive. A covering index duplicates selected non-key columns beside the search key so covered queries avoid the heap lookup, at the cost of more index space and write maintenance.

### 8. ★ **[Data Storage for Analytics and Column-Oriented Storage]** Why does an analytical query scans many rows benefit from column-oriented storage, and when would that layout become less attractive?

#### My answer (revise)

- If the table has a lot of columns, column-oriented storage enables the engine to load only the necessary columns instead of the whole row. This makes queries more efficient.
- When all rows are sorted in an appropriate order and there are only a small amount of distinct values, the data can be highly compressed and reduce the usage of disk space.
- If some columns are too large or if different subset of columns often fit into different tasks, we can consider storing them in different servers in a distrbuted structure. Nevertheless, it can be more difficult to keep the row order consistent among all the columns.

The column-oriented storage also has some drawbacks that make it less attractive in some cases:

- If we often need to read a lot of columns every time, it will be expensive and inefficient for column-oriented storage to combine all the columns, compared with row-oriented storage.
- For any write operations in a column-oriented storage, it will be stored in the memory at first and merged into the database in bulk. If we have many and frequent writes, it will incur very high overhead for the column-oriented storage. Meanwhile, queries have to combine the results in memory and disk at the same time.
- Column-oriented storage usually requires a careful and complex schema design to optimize the query efficiency and storage.

#### Review

- **Assessment:** Needs revision.
- **What works:** You correctly explain column pruning, compression, costly row reconstruction, and why buffered bulk writes fit columnar storage better than frequent point updates.
- **Clarification:** The earlier distribution gap remains. Columns in the same row group rely on the same row order, so independently distributing or reordering them would make reconstructing rows difficult and can force network joins. Distributed analytical systems normally partition by sets of rows (often as row groups or shards), while each partition stores its local columns together and aligned.
- **Stronger answer:** Columnar storage helps scans that touch many rows but few columns because it reads only those columns, compresses similar adjacent values well, and lets the CPU process compact arrays efficiently. It is less attractive for point lookups or queries needing most of each row, and for frequent small updates, because reconstructing rows and updating several column files is costly. Systems therefore buffer updates and merge them in batches. For distributed execution, they normally partition rows across machines and retain aligned column chunks within each partition, rather than placing unrelated columns independently on different servers.


### 9. **[Query Execution: Compilation and Vectorization]** How do query compilation and vectorized processing take different routes to reducing CPU cost during large analytical scans?

#### My answer

Query compilation:
The query engine will generate the code to execute the query. The code will iterate through the rows and load the columns of interest into an output buffer. Then, the query engine will compile the code into machine code, which can be executed by CPU efficiently on the loaded column-encoded data.

Vectorized processing:
The query is executed not row by row by in batches. The database will have a fixed set of predefined operators, and we can pass arguments to them and get back a batch of values. This makes use of parallelism and efficient bitwise operations of the CPU.

#### Review

- **Assessment:** Solid.
- **What works:** You distinguish specialized generated machine code from interpreted, predefined operators that process column batches.
- **Clarification:** “Parallelism” needs one qualification: batching can use SIMD, but it does not inherently require multiple CPU cores or threads.

  Suppose the predicate is `product_sk = 42 AND store_sk = 7`:

  - **Compilation specializes the work.** Instead of interpreting a query-plan data structure for every row—asking which operator to call, which column to read, and which comparison to perform—the engine generates one loop containing those exact comparisons. Constants such as `42` and `7` can be embedded in the machine code, operators may be fused, and intermediate values can stay in CPU registers. This removes repeated dispatch, function calls, and unpredictable branches. The resulting small, predictable instruction sequence keeps the CPU pipeline busy and may also be automatically compiled into SIMD instructions.
  - **Vectorization amortizes the work.** The engine calls a predefined equality operator once for a batch such as 1,024 `product_sk` values, rather than dispatching it 1,024 times. The operator walks contiguous column memory in a tight loop; caches and hardware prefetchers work well because access is sequential. SIMD can compare several values with `42` in one instruction. The engine can produce two bitmaps for the two predicates and combine many row results at once with a bitwise `AND`.

  Thus compilation exploits the CPU by making one query’s instruction stream highly specialized, whereas vectorization exploits it by feeding a general operator a large, regular block of data. Both favor sequential memory access, cache locality, predictable tight loops, pipelining, SIMD, and direct processing of compressed data.

### 10. **[Materialized Views and Data Cubes]** A dashboard repeatedly computes the same aggregates, but users occasionally invent new filters. What would materializing results improve, and where would it stop helping?

#### My answer

If the dashboard repeated computes the same aggregates, we can consider caching the values according to the different dimensions of these aggregates (data cubes). In this case, the dashboard can load the results efficiently.
However, data cubes do not have very good flexibility. If the aggregate is too complex (involving too many dimensions) or we often needs very customized query, then materializing will not be able to help much.

#### Review

- **Assessment:** Solid.
- **What works:** You correctly frame materialization as precomputing recurring dimension-based aggregates and identify its inability to serve unanticipated filters or groupings.
- **Clarification:** The other side of the trade-off is write/refresh work: underlying changes must be propagated to the materialized result, while raw data is still needed for unsupported questions.

### 11. ★ **[Multidimensional and Full-Text Indexes; Vector Embeddings]** How does the shape of a query lead to different index designs for geographic ranges, keyword conjunctions, and semantic similarity, and where does approximation enter?

#### My answer (revise)

For geographic ranges, we usually want to filter by both longitude and latitude. An index with a single dimension is not very helpful.
For keyword conjunctions, we need to break down the original text into different words or tokens, and we need to find similar words in addition to identical words. This turns a query effectively into a multidimensional query. In this case, we usually use an inverted index.
For semantic similarity, we need to find values that are close in meanings, even if they are completely different words. In this case, we use embedding models to translate a text document into a vectory of floating-point values. Then, we use cosine similarity, Euclidean distance, or other functions to determine how close there are semantically.

#### Review

- **Assessment:** Needs revision.
- **What works:** You distinguish two-dimensional geographic predicates, term-based inverted indexes, and embedding-based semantic distance.
- **Clarification:** The earlier gap remains: approximation is not applied in the same way to all three queries.
- **Stronger answer:**

  - **Geographic range:** A map-window query constrains latitude and longitude simultaneously, so an R-tree, Bkd-tree, grid, or space-filling-curve index groups nearby points and prunes regions outside the window. The intended predicate is exact: return every point inside the rectangle. Some spatial indexes may first return a conservative set of candidates and then apply the exact coordinates as a final filter, but they should not silently omit an in-range point.
  - **Keyword conjunction:** An inverted index maps each exact term to its postings list. For `red AND apples`, intersecting the two lists or bitmaps exactly returns documents containing both indexed terms. Stemming, synonyms, typo tolerance, or tokenization may broaden what counts as a matching term, but that is query interpretation—not approximate nearest-neighbor search.
  - **Semantic similarity:** An embedding model maps the query and documents to points in a high-dimensional space, and a distance function defines which indexed vectors are nearest. A **flat index** computes the distance to every stored vector, so it finds the exact nearest vectors under that distance function, although the embedding itself is only a learned representation of meaning. **IVF** searches only selected vector partitions; a true neighbor can lie just across an unsearched partition boundary. More probes search more partitions, improving recall at the cost of time. **HNSW** navigates a proximity graph from coarse to dense layers; its greedy, limited exploration can take a promising path yet miss the true nearest node. Searching more candidates improves recall but costs more time.

  “Approximate” therefore means that IVF or HNSW may fail to return one of the mathematically closest stored vectors that an exhaustive flat scan would return. It does not mean cosine or Euclidean distance is estimated incorrectly for the candidates actually compared. The trade-off is lower latency and fewer distance calculations in exchange for imperfect **recall**.


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

**Size-tiered compaction** — When several SSTables reach roughly the same size, they are merged into one larger SSTable; several outputs of that size can later be merged into a still larger one. Because files of different sizes may cover the same key ranges, a read may need to check several SSTables, and a large merge temporarily needs room for both its inputs and output. The trade-off is high write throughput: data is usually rewritten less aggressively, making this strategy attractive when writes dominate and reads are relatively rare.

**Leveled compaction** — SSTables are divided into levels with increasing size limits and, beyond the newest level, partitioned so files in the same level cover non-overlapping key ranges. When a file is pushed down, it is merged only with the next-level files whose key ranges overlap, making compaction smaller and more incremental. A point read therefore has few candidate files per level, and obsolete versions consume less space; the cost is greater write amplification because one incoming range may repeatedly rewrite overlapping data already in lower levels. This usually suits read-heavy workloads and workloads that repeatedly update a small set of hot keys.

The contrast is therefore not whether compaction happens—both strategies merge sorted files and remove obsolete values—but how eagerly they impose order:

- **Size-tiered:** merge by similar file size → fewer rewrites, more overlapping files, higher read and temporary-space costs.
- **Leveled:** merge by level and key range → more rewrites, fewer read candidates, tighter space usage.

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

**Pattern across your answers:** You reason well from physical layout to I/O behavior and consistently identify the main read/write trade-offs. You have now closed the LSM durability gap. The remaining pattern is precision about boundaries between adjacent ideas—search keys versus stored payload, logical columns versus distribution units, SIMD versus multithreading, and exact predicates versus approximate vector retrieval.

**Review next:**

1. Vector indexes: exact flat search versus the recall/latency trade-offs of IVF and HNSW.
2. Storing values within the index: search keys, clustered payloads, heap references, and covering columns.
3. Columnar execution: row-group distribution and how compilation and batching exploit CPU architecture.

**Follow-up:**

1. After an acknowledged LSM write but before a memtable flush, which persisted structure makes recovery possible, and when can its entries be discarded?
2. Why can an included column let a query avoid a heap lookup even though it is not part of the index’s search key?
3. Which tuning decision in an approximate vector index trades more work and latency for better recall?

## Application challenge

You are choosing storage for an event-ingestion service that receives 80,000 writes per second. Most reads are point lookups, and the API has a 20 ms p99 latency SLO. Traffic bursts can last 15 minutes, available SSD space is limited to 1.4 times the live dataset, and a compliance deletion must become physically unrecoverable within 24 hours.

State:

- the storage-engine and compaction decision you would make;
- which Chapter 4 concepts support it;
- your assumptions;
- how the choice could fail;
- what requirement change would make you choose an alternative.

## Spaced review

### 2026-08-24

- [ ] Review complete

1. Reconstruct an acknowledged LSM write from WAL append through memtable flush and eventual compaction.
2. Compare the point-read path and likely latency variance of a B-tree with those of an LSM-tree.

#### My review answers


### 2026-08-29

- [ ] Review complete

1. Explain how clustered, heap-backed, and covering indexes place row data differently and how each placement changes reads and writes.
2. An analytics table has 120 columns, but a query scans four. Why does columnar layout help, and why is distributing one column per server not the core scaling model?

#### My review answers


### 2026-09-21

- [ ] Review complete

1. Match these queries to suitable index families: map-window filtering, two-keyword conjunction, and semantic nearest-neighbor search.
2. Contrast exact flat vector search with IVF or HNSW, including what is traded for lower latency.

#### My review answers
