## Overview

**Partitioning** (a.k.a. sharding) = breaking a large dataset into smaller subsets distributed across nodes.
- Terminology: *shard* (MongoDB, Elasticsearch, SolrCloud), *region* (HBase), *tablet* (Bigtable), *vnode* (Cassandra, Riak), *vBucket* (Couchbase)
- Goal: spread data and query load **evenly** across nodes to scale horizontally
- Each record belongs to exactly **one** partition; each partition acts as a small independent database
- **Skew** = uneven distribution; a disproportionately loaded partition = **hot spot**

---

## Partitioning + Replication

- Partitioning is almost always combined with replication for fault tolerance
- Each partition's leader is on one node; followers on other nodes
- A node can be leader for some partitions and follower for others
- Choice of partitioning scheme is **independent** of replication scheme

![[Pasted image 20260527200543.png]]

---

## Partitioning Key-Value Data

### Key Range Partitioning

- Assign a continuous range of keys to each partition (like encyclopedia volumes)
- Boundaries can be set manually or automatically by the DB
- Keys within a partition kept **sorted** → efficient range scans
- Range scans can treat key as a concatenated index (e.g., fetch all readings from a particular month) (explained in Cassandra Compound Key Hybrid)
- **Downside**: hot spots if access patterns cluster in a range (e.g., timestamp keys → all writes go to "today" partition)
  - Fix: prefix key with another field (e.g., sensor name + timestamp) to distribute writes
- Used by: Bigtable, HBase, RethinkDB, MongoDB (pre-2.4)

![[Pasted image 20260527200524.png]]

### Hash Partitioning

- Apply a hash function to the key; assign each partition a **range of hash values**
- Good hash functions distribute keys evenly even if inputs are similar
- Common hash functions: MD5 (Cassandra, MongoDB), Fowler–Noll–Vo (Voldemort)
- **Do not use** language built-in `hashCode()` — results can differ across processes
- **Downside**: loses sort order → range queries must go to all partitions in some DBs (Mongo)

![[Pasted image 20260527200603.png]]

> **Consistent Hashing** (Karger et al.): uses randomly chosen partition boundaries to avoid central control. Works poorly for databases in practice — prefer the term *hash partitioning* instead.

#### Cassandra Compound Key Hybrid

- Declare a **compound primary key**: `(partition_key, sort_columns...)`
- Only the **first column** is hashed to determine the partition
- Remaining columns are used as a concatenated index for **sorted storage within the partition**
- Enables efficient range scans for a primary key (e.g., all tweets by a user are on one single partition, sorted by time)
- Range queries across the first (hashed) column are not possible

### Skewed Workloads and Hot Spots

- Even with hash partitioning, a single extremely hot key (e.g., celebrity user ID) can still overload one partition
- Most systems cannot automatically compensate — **application responsibility**
- Common technique: append a small random number (e.g., 2-digit) to the hot key
  - Spreads writes across 100 keys / 100 partitions
  - **Trade-off**: reads now have to query all 100 keys and merge results
  - Adds bookkeeping overhead — only worth it for known hot keys

---

## Partitioning and Secondary Indexes

Secondary indexes don't map neatly to partitions — two main approaches:

### Document-Partitioned Index (Local Index)

- Each partition maintains its own secondary index covering **only documents in that partition**
- Writes only touch one partition — no cross-partition coordination needed
- Also called a **local index**
- **Reads require scatter/gather**: query must be sent to all partitions and results merged
  - Prone to tail latency amplification, poor perf
- Used by: MongoDB, Riak, Cassandra, Elasticsearch, SolrCloud, VoltDB

![[Pasted image 20260528160507.png]]

### Term-Partitioned Index (Global Index)

- A single global index covers data from **all** partitions, but the index itself is also partitioned, so we have two sets of partitions
	- Partition on the primary key - the *data partition* - holding the documents
	- Partition on the secondary index term - the *index partition* - holding references to data partitions
- Partitioned by term (or hash of term) — not by document
  - Partitioning by term → supports efficient range scans on indexed values
  - Partitioning by hash of term → more even load distribution
- **Reads are efficient**: client queries just the partition(s) containing the relevant term
- **Writes are slow and complex**: a single document write may touch multiple index partitions
  - Ideally needs a distributed transaction across all affected partitions (not always supported)
  - In practice, global secondary indexes are often **updated asynchronously** (e.g., DynamoDB: normally within a fraction of a second but may lag during faults)
- Used by: Riak search, Oracle data warehouse (optional)

![[Pasted image 20260528160624.png]]

---

## Rebalancing Partitions

**Rebalancing** = moving load/data between nodes as cluster changes (needed when adding more CPUs, or more disk, or on node failure).

Minimum requirements:
- Load (read / write / storage) distributed fairly after rebalancing
- DB continues accepting reads/writes during rebalancing
- Move only as much data as necessary (minimum possible network and disk IO, faster rebalancing)

Strategies for partition assignment (and rebalancing)

### ❌ Hash mod N (Don't Do This)

- `hash(key) mod N` assigns keys to nodes directly
- **Problem**: if N changes, almost every key must be remapped → excessive data movement

### ✅ Fixed Number of Partitions

- Create **many more partitions than nodes** at setup (e.g., 1000 partitions across 10 nodes = 100 partitions/node)
- When a new node joins: it steals a few partitions from each existing node
- Only **partition-to-node assignment** changes; key-to-partition mapping stays fixed
- Supports hardware heterogeneity: assign more partitions to more powerful nodes
- **Challenge**: choosing the right number upfront is hard if dataset size is highly variable
  - Too many partitions → management overhead
  - Too few → limits future growth (partition count = max node count)
- Used by: Riak, Elasticsearch, Couchbase, Voldemort

![[Pasted image 20260528161751.png]]

### ✅ Dynamic Partitioning

- Used by key range–partitioned databases (HBase, RethinkDB)
- Partition **splits** when it exceeds a configured size (HBase default: 10 GB); **merges** when it shrinks below a threshold
- Similar to B-tree top-level splits
- Number of partitions adapts automatically to data volume
- **Caveat**: empty DB starts with a single partition → all writes initially go to one node
  - Mitigation: **pre-splitting** — configure initial partition boundaries when DB is empty
- Also works with hash-partitioned data (MongoDB since 2.4 supports both)

### ✅ Partitioning Proportional to Nodes (Cassandra/Ketama)

- Fixed number of **partitions per node** (Cassandra default: 256 per node)
- Total partitions grows with number of nodes; partition size stays roughly stable
- When new node joins: randomly picks existing partitions to split, takes ownership of one half of each
  - Randomization may cause unfair splits, but averages out over many partitions
  - Cassandra 3.0 introduced a deterministic algorithm to avoid unfair splits
- Requires hash-based partitioning (since partitions are carved from hashed values and not raw keys)

### Automatic vs. Manual Rebalancing

- **Fully automatic**: system decides when to move partitions — less operational work but unpredictable
- **Fully manual**: admin explicitly configures partition assignments
- Middle ground (e.g., Couchbase, Riak, Voldemort): system generates suggested assignment, admin must approve it
- **Risk of full automation**: can trigger cascading failures — a slow node may be falsely detected as dead, rebalancing adds more load to it, making it worse
- Human-in-the-loop is slower but prevents operational surprises

---

## Request Routing

**Problem**: how does a client know which node to connect to for a given key? (Partition-to-node mapping changes as cluster rebalances — a form of *service discovery*)

### Three Approaches

1. **Client → any node**: node handles request if it owns the partition, otherwise forwards to the correct node
2. **Client → routing tier**: a partition-aware load balancer forwards requests to the right node; does no actual data processing
3. **Client-side routing**: client knows the partition assignment and connects directly to the right node

![[Pasted image 20260528214504.png]]

### ZooKeeper-Based Coordination

- Nodes register themselves in ZooKeeper
- ZooKeeper maintains the **authoritative mapping** of partitions → nodes
- Routing tier or partition-aware clients **subscribe** to ZooKeeper for updates
- On partition ownership change / node add/remove → ZooKeeper notifies subscribers
- Used by: HBase, SolrCloud, Kafka, LinkedIn Espresso (via Helix)
- MongoDB: similar architecture using its own *config server* + *mongos* routing tier

![[Pasted image 20260528214521.png]]

### Gossip Protocol (No External Coordinator)

- Cassandra and Riak disseminate cluster state changes via **gossip protocol** among nodes
- Client can connect to any node; that node forwards if needed (approach 1)
- More complexity in DB nodes, but **no dependency on ZooKeeper**
- Couchbase: does not rebalance automatically; uses *moxi* routing tier that learns routing from cluster nodes

> IP addresses of nodes don't change as frequently as partition assignments — DNS is sufficient for finding node addresses.

---

## Parallel Query Execution (MPP)

- NoSQL systems typically support only simple single-key or scatter/gather queries
- **MPP** (Massively Parallel Processing) relational databases (used for analytics) support complex queries: joins, filtering, grouping, aggregation
- MPP query optimizer breaks queries into **execution stages**, many of which run in parallel across nodes
- Full-table-scan style queries benefit most from parallel execution
- Covered in depth in Chapter 10

---

## Summary

| Approach | Range Scans | Hot Spot Risk | Rebalancing |
|---|---|---|---|
| Key range partitioning | ✅ Efficient | ⚠️ High (sequential keys) | Dynamic splits |
| Hash partitioning | ❌ All partitions | ✅ Low | Fixed partitions or dynamic |
| Cassandra compound key | ✅ Within partition | ✅ Low (hashed first key) | Proportional to nodes |

| Secondary Index | Write Cost | Read Cost |
|---|---|---|
| Document-partitioned (local) | Low (single partition) | High (scatter/gather all partitions) |
| Term-partitioned (global) | High (multi-partition, async) | Low (single partition per term) |
