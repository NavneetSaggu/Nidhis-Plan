## Storage Engines — Two Families

- **Log-structured** storage engines (LSM-Trees, SSTables)
- **Page-oriented** storage engines (B-Trees)

You probably won't implement your own, but you need to understand what they do under the hood to tune them for your workload.

---

## Data Structures That Power Your Database

### The Simplest Possible Database (Hash Index)

A log-structured store at its most basic: every `db_set` call **appends** a key-value pair to a file. `db_get` scans backwards for the most recent entry.

```
#!/bin/bash
db_set () {
	echo "$1,$2" >> database
}
db_get () {
	grep "^$1," database | sed -e "s/^$1,//" | tail -n 1
}
```

- **Write performance**: excellent — appending to a file is generally the fastest possible write
- **Read performance**: O(n) — scans entire file on lookup → needs an index
- **Index cost**: every index slows down writes, since it must be updated on every write


---

### Hash Index

**Keep an in-memory hash map** where every key maps to the **byte offset** of its most recent value in the data file. This is essentially what **Bitcask** (Riak's default storage engine) does.
- Offers high-performance reads and writes — as long as **all keys fit in RAM**
- Values can be larger than RAM (stored on disk, looked up by offset)
- Best suited to: **high write volume, not too many distinct keys** (e.g. URL → hit counter)

![[Pasted image 20260525210043.png]]

**Compaction**: to avoid running out of disk space, break the log into **segments** of a certain size, then **compact** segments by throwing away duplicate keys and keeping only the most recent value per key.

![[Pasted image 20260525210005.png]]

- Merging and compaction of frozen segments runs in a **background thread** — reads and writes continue on the old segments in the meantime
- After merging is complete, reads switch to the new merged segment, old ones are deleted
- Each segment has its own **in-memory hash table**; lookup checks the most recent segment first, then older ones. Keeping the number of segments small means not many hash maps to check.

**Implementation details that matter:**
- **File format**: binary (encoded string length + raw unescaped bytes) is faster and simpler than CSV
- **Deleting records**: append a special **tombstone** marker; merging process discards the deleted key's values
- **Crash recovery**: in-memory hash maps are lost on crash — speed up recovery by storing **snapshots** of each segment's hash map on disk
- **Partially written records**: include **checksums** so corrupted log entries can be detected and ignored
- **Concurrency control**: one writer thread (log is strictly sequential); data file segments are immutable so can be **read concurrently by multiple threads**

**Why append-only is good:**
- **Sequential writes** are much faster than random writes, especially on spinning disks — and also preferable on SSDs
- Crash recovery is simpler — no partially-overwritten records
- Merging avoids data fragmentation over time

**Limitations of hash indexes:**
- **Hash table must fit in memory** — can't have more keys than RAM
- **Range queries are inefficient** — can't scan keys between `kitty00000` and `kitty99999`; must look up each key individually

---

### SSTables and LSM-Trees

**SSTable (Sorted String Table)**: same segment/compaction approach as hash indexes, but with one change — **key-value pairs within each segment are sorted by key**.

This requires that keys only appear once per merged segment (compaction ensures this).

**Merging segments** is now simple and efficient — like mergesort. Even if segments are bigger than available memory, merging works because it reads sequentially.

![[Pasted image 20260525232036.png]]

**Sparse in-memory index**: you no longer need to keep an index of *every* key in memory. Because keys are sorted, you can keep a **sparse index** (one key per few kilobytes of segment file) and binary-search between known offsets.

![[Pasted image 20260525232129.png]]

**Compression saves space and IO bandwidth**: Since the entire chunk of any given offset is scanned for every read query, the chunk can be compressed before writing to disk

**How writes work (maintaining sort order in memory):**
1. **Write** incoming keys to an in-memory balanced tree (e.g. **red-black tree** or **AVL tree**) — called a **memtable**. These allow inserting in any order and reading back in sorted order.
2. When the memtable exceeds a threshold, **write it out to disk as an SSTable segment** — it's already sorted, so this is efficient. New writes go to a fresh memtable.
3. **Reads**: check the memtable first, then the most recent on-disk segment, then older segments.
4. Run **merging and compaction** in the background periodically.
5. **Crash protection**: keep a separate append-only log on disk for every memtable write (unsorted is fine — only used for recovery). Discard this log when the memtable is flushed to disk.

**Used in**: LevelDB, RocksDB, Cassandra, HBase — all inspired by Google's **Bigtable** paper. The indexing structure was originally described as a **Log-Structured Merge-Tree (LSM-Tree)** by O'Neil et al.

**Lucene** (used by Elasticsearch and Solr) uses a similar approach for its term dictionary — a key is a word (term) and the value is a postings list of document IDs containing that word.

**Bloom filters**: used to optimise lookups for keys that don't exist — rather than checking every segment file, a Bloom filter can tell you if a key definitely doesn't exist in a segment.

**Compaction strategies:**

|                         | **Size-Tiered (STCS)**                                                                                                                      | **Leveled (LCS)**                                                                                                                                       |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Core idea**           | Group SSTables of similar size together. When enough accumulate, merge into one larger file. Results in tiers of progressively larger files | Organise SSTables into levels (L1, L2, L3...). Each level has a limit on size of SSTables. Files merge down into the next level when a level fills up.  |
| **Motivation**          | Compact as infrequently as possible, in large batches. Optimise for write throughput above all else.                                        | Keep reads fast and space usage predictable. Each SSTable in a level has a fixed range of keys; ranges for all SSTables in a level are non-overlapping. |
| **Write amplification** | *Low* — data is not rewritten frequently                                                                                                    | *High* — data gets rewritten multiple times as it moves down levels                                                                                     |
| **Read amplification**  | *High* — must check multiple tiers of SSTables to find a key                                                                                | *Low* — bounded number of files to check per read (only one SSTable checked per level thanks to no-overlapping ranges), fast and predictable            |
| **Space amplification** | *High* — old and new versions of data coexist across tiers until compaction catches up                                                      | *Low* — levels ensure minimal redundant data at any point                                                                                               |
| **Compaction pattern**  | Infrequent but large and disruptive when it happens. Can cause latency spikes                                                               | Frequent but small and steady. More predictable, less disruptive to latency                                                                             |
| **Best for**            | Write-heavy workloads where read latency is less critical                                                                                   | Read-heavy workloads where consistent performance and space efficiency matter                                                                           |
| **Used by**             | Cassandra (default), HBase, RocksDB (optional)                                                                                              | RocksDB (default), LevelDB, TiKV, Cassandra (optional)                                                                                                  |

---

### B+ Trees

The standard index implementation in virtually every relational database (MySQL InnoDB, PostgreSQL, Oracle, SQL Server) and many non-relational ones. The book calls these "B-trees" but real DB indexes use **B+ Trees** — an important distinction.

**Structure:**
- Break the database into **fixed-size pages** (traditionally 4KB), reading/writing one page at a time
- **Internal (non-leaf) nodes**: store only keys + pointers to child pages — **no values**
- **Leaf nodes**: store all keys + values (or pointers to heap rows in case of non-clustered indices) — this is where all actual data lives
- Leaf nodes are **linked together in a doubly-linked list** — enables efficient range scans
- The number of child pointers per internal node is the **branching factor** (typically several hundred)
- One page is designated the **root** — all lookups start here

![[Pasted image 20260526000540.png]]

**Why B+ over plain B-tree:**
- Internal nodes hold no values → they're smaller → **higher branching factor** → shallower tree → fewer page reads per lookup
- All values at the leaf level → **range scans are fast**: find the start key, then walk the linked leaf list sequentially without re-traversing the tree

**Depth and capacity:**
- Depth is always O(log n). Most databases fit in **3 or 4 levels** — a 4-level tree with 4KB pages and branching factor 500 can store up to **256 TB**.

**Updating a value**: find the leaf page, overwrite in place, write the page back. The **overwrite does not change the page's location** — all references stay intact.

**Adding a key**: find the correct leaf page, insert. If the page is **overfull**, split into two half-full pages and push the middle key up to the **parent** to cover the new child. Of the parent itself is full, the same recursive operation continues upwards.

![[Pasted image 20260526000638.png]]

**Making B+ Trees reliable:**
- Writes **overwrite pages in place** — unlike LSM-trees which only append. A crash mid-write (e.g. during a page split) can corrupt the index.
- **Write-ahead log (WAL)**: every modification is appended to a WAL before being applied to tree pages. Used to recover a consistent state after a crash.
- **Concurrency control**: multiple threads protected via **latches** (lightweight locks) on pages being modified

**Extra implementation notes:**
- Some engines store **abbreviated keys** in internal nodes (just enough to act as boundaries) — fitting more keys per page, increasing branching factor
- Leaf pages are ideally laid out **sequentially on disk** for efficient range scans, though maintaining this as the tree grows is hard — LSM-trees naturally produce sequential files
- Some B+ Tree implementations add **sibling pointers** on leaf nodes as well as internal nodes too, speeding up certain traversals

---

### Comparing B-Trees and LSM-Trees

> Rule of thumb: **LSM-trees are typically faster for writes; B-trees are thought to be faster for reads**. Benchmarks are inconclusive — test with your specific workload.

|                     | B-Trees                                                                                                      | LSM-Trees                                                                                                                                                 |
| ------------------- | ------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Writes              | **Entire page** is written for each change. Every key written **twice** — once to WAL, once to tree page.    | **Write amplification** from compaction/merging, but typically lower than B-tree                                                                          |
| Write throughput    | Lower                                                                                                        | Higher — sequential writes are fast, lower write amplification since no full page rewrites                                                                |
| Read speed          | Generally faster — single path through tree                                                                  | Can be slower — must check multiple SSTables                                                                                                              |
| Space usage         | Leaves some fragmentation (page splits)                                                                      | Can be compressed better, produces smaller files                                                                                                          |
| Compaction overhead | None                                                                                                         | Background compaction shares disk bandwidth — if compaction can't keep up with write rate, **unmerged segments keep growing** and reads slow down         |
| SSD concern         | Multiple overwrites per page                                                                                 | Write amplification still relevant — SSDs can only overwrite blocks a limited number of times                                                             |
| Advantages          | **Strong Transactional Semantics** - keys only present in one location, locks on key ranges ensure isolation | **Better compression** — smaller files on disk than B-trees (which leave fragmentation from page splits)                                                  |
| Disdvantages        | **High write amplification**, entire pages rewritten each time.                                              | **Do not throttle incoming writes** — if compaction can't keep up, unmerged segments grow unboundedly, reads slow down, and you can run out of disk space |


---

### Other Indexing Structures

- **Secondary indexes**: index on a non-primary-key column (e.g. `user_id`). Multiple rows can match the same key — solved by either making the value a list of row IDs, or appending a row ID to each key to make it unique.
- **Clustered index**: stores all row data within the index (e.g. InnoDB's primary key). Avoids an extra lookup to a heap file.
- **Nonclustered index**: stores only a reference to the row (pointer to heap file location).
- **Covering index**: a compromise — stores *some* columns within the index alongside the primary key reference. Allows some queries to be answered from the index alone.
- **Concatenated index**: most common multi-column index — appends multiple fields into one key (e.g. `(lastname, firstname)`). Useful for range queries on the first field, less so for arbitrary field combinations. Order is important here.
- **Multi-dimensional indexes**: for querying several columns at once (e.g. lat/lng range). A standard B-tree or LSM-tree can't answer a 2D range query efficiently — options include **space-filling curves** + B-tree, or specialised **R-trees**.
- **Full-text/fuzzy indexes**: allow searching for similar words, synonyms, grammatical variants, typos. **Lucene** uses a trie-like structure and can find words within a certain **edit distance**.

---

### In-Memory Databases

Keeping everything in RAM is an alternative approach — avoid disk entirely.

- **Cache-only** (e.g. Memcached): data is lost on restart, acceptable for pure caching
- **Durable in-memory** (e.g. VoltDB, MemSQL, Redis, Oracle TimesTen): achieve durability via battery-backed RAM, writing a log to disk, periodic snapshots, or replicating state to other machines
- The performance advantage is **not just avoiding disk reads** — it's avoiding the overhead of encoding in-memory data structures into a disk-writable format
- Counterintuitively, in-memory DBs can support **richer data models** that are hard to implement on disk — e.g. Redis offers priority queues, sets, and sorted sets natively
- **Anti-caching**: if the dataset is larger than RAM, evict least-recently-used data to disk and reload on access — similar to what the OS does with virtual memory, but the DB manages it explicitly and more efficiently

---

## Transaction Processing vs. Analytics (OLTP vs. OLAP)

| Property        | OLTP                                                                                                               | OLAP                                                                                  |
| --------------- | ------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------- |
| Read pattern    | Small number of records, fetched by key                                                                            | Aggregate over millions of records                                                    |
| Write pattern   | Random-access, low-latency writes from user input                                                                  | Bulk import (ETL) or event stream                                                     |
| Used by         | End user / customer via web app                                                                                    | Internal analyst for decision support                                                 |
| Data represents | Latest state (current point in time)                                                                               | History of events over time                                                           |
| Dataset size    | GBs to TBs                                                                                                         | TBs to PBs                                                                            |
| Example Query   | **Transaction processing**: add new inventory to the `stock` table on purchase; decrement quantity with every sale | **Analytical Queries**: What was the total rebvenue of each of the stores in January> |

**OLTP databases** are business-critical — DBAs are reluctant to let analysts run ad-hoc queries on them, since those queries are often expensive and can harm performance.

### Data Warehousing

A **data warehouse** is a separate, read-only database that analysts can query freely, without affecting OLTP systems.

- Data is **extracted** from OLTP databases (periodic dump or continuous stream), **transformed** into an analysis-friendly schema, **cleaned**, and **loaded** into the warehouse.
- This process is known as **ETL (Extract–Transform–Load)**.

![[Pasted image 20260526061742.png]]

> On the surface, a data warehouse and an OLTP database may both use a relational data model — but **their internals are very different**, optimised for very different query patterns. Indexing algorithms that work well for OLTP are not very good at analytic queries.

### Stars and Snowflakes: Schemas for Analytics

The most common schema in data warehouses is the **star schema** (also called dimensional modelling):
- A central **fact table** — each row represents an event (e.g. a sale). Fact tables can become extremely large (100+ columns, petabytes of rows).
- Fact table columns are either **attributes** (e.g. price, quantity) or **foreign keys** into **dimension tables**.
- **Dimension tables** represent the *who, what, where, when, why* of the event (e.g. `dim_customer`, `dim_product`, `dim_date`, `dim_store`).

![[Pasted image 20260526062217.png]]

**Snowflake schema**: a variation where dimensions are **further broken down into subdimensions**. More normalised than star schema, but harder to work with for analysts.

> A typical analytic query accesses only **4 or 5 columns** out of a fact table with 100+ columns — this is what makes column-oriented storage so effective.

---

## Column-Oriented Storage

Row-oriented storage stores all values from one row together. Column-oriented storage stores all values from **each column together** — so to read a query's 4 columns out of 100, you only load 4 column files instead of every row.

![[Pasted image 20260526062420.png]]

### Column Compression

Column files often compress very well because the values within a column tend to repeat (e.g. a `product_sk` column referencing 69,000 products across billions of rows will have lots of repetition). A common technique is **bitmap encoding**:

![[Pasted image 20260526062731.png]]

- For each distinct value in a column, store a **bitmap** — one bit per row, 1 if the row has that value, 0 otherwise
- If the number of distinct values is small relative to number of rows, bitmaps are sparse → can be further compressed with **run-length encoding**
- Bitmaps allow efficient execution of operators like `WHERE product_sk IN (31, 68, 69)` — just OR the bitmaps for those values; for `WHERE product_sk = 31 AND store_sk = 3` we can AND the two bitmaps. This works because all columns contain the rows in the same order.
- Column-oriented layouts also lend themselves to **vectorised processing** (SIMD instructions) — tight loops that operate on compressed column data fit in CPU L1 cache, avoiding branch mispredictions


### Sort Order in Column Storage

- Not necessary, sorting can be imposed to use as an index.
- Rows in column files must be stored in the **same row order** across all columns — so row 23 of `product_sk` corresponds to row 23 of `date_key`, etc.
- You can choose to **sort by a particular column** to support most common queries (e.g. `date_key`) — queries that filter by date can then skip irrelevant data. 
- There can be second or third sort keys too, on `product_sk` for example, to support date-ranged and grouped-by-product queries. 
- The first sort column compresses particularly well (long runs of the same value are ideal for run length encoding).
- Can define **multiple sort orders** (stored as separate copies of the data) — like having multiple secondary indexes

### Writing to Column-Oriented Storage

- Compressed columns make writes harder — inserting a row in the middle requires rewriting all column files
- Solution: use an **LSM-tree style in-memory store** for writes (sorted, then flushed to column files in bulk). Reads merge the in-memory store with on-disk column files. This is what **Vertica** does.

### Aggregation: Materialized Views and Data Cubes

- **Materialized view**: a cached result of an aggregate query (e.g. `COUNT`, `SUM`) written to disk. When underlying data changes, the view must be updated (makes writes more expensive).
- **Data cube (OLAP cube)**: a grid of aggregates grouped by different dimensions. For example, a 2D cube might show total sales aggregated by `date` and `product`. This can support higher dimensions as well.

![[Pasted image 20260526070521.png]]

- Advantage: certain queries become very fast — pre-computed aggregates rather than scanning millions of rows
- Disadvantage: not as flexible as raw data — can't query against arbitrary filters that aren't pre-aggregated

---

## Summary

| Engine | Best for | Key mechanism |
|---|---|---|
| **Hash index** | High write rate, all keys fit in RAM | In-memory hash map → byte offset |
| **LSM-Tree / SSTable** | Write-heavy workloads | Memtable → sorted segments → compaction |
| **B-Tree** | Read-heavy, general purpose | Sorted fixed-size pages, overwrites in place |
| **Column store** | OLAP — aggregate queries over few columns of wide tables | Store columns together, compress heavily |

- **OLTP** engines optimise for low-latency reads/writes on small numbers of records by key
- **OLAP** engines optimise for scanning and aggregating huge numbers of rows across a few columns
- Column-oriented storage, compression, and materialized views are the key techniques that make data warehouses fast
