## Why Replicate?

- Keep data geographically close to users (reduce latency)
- Allow system to continue if some parts fail (increase availability)
- Scale out read queries across multiple machines (increase read throughput)

All difficulty in replication lies in **handling changes** to replicated data. Three main algorithms: **single-leader**, **multi-leader**, and **leaderless** replication.

---

## Leaders and Followers

**Leader-based replication** (aka master–slave, active/passive):

1. One replica is the **leader** — all writes must go through the leader
2. Other replicas are **followers** — leader sends change stream to all followers; each applies changes in the same order
3. Reads can go to the leader or any follower; writes go to leader only

![[Screenshot 2026-05-27 at 4.16.50 PM.png]]

Used in: PostgreSQL (v9.0+), MySQL, Oracle Data Guard, SQL Server AlwaysOn, MongoDB, RethinkDB, Espresso. Also used by Kafka, RabbitMQ.

---

### Synchronous vs. Asynchronous Replication

![[Pasted image 20260527161750.png]]

**Synchronous**: leader waits for follower to confirm before reporting success to client. Impractical for all followers to be sync - single node failure stops the whole system.
- ✅ Follower guaranteed to have up-to-date data; if leader fails, follower can take over safely
- ❌ If synchronous follower is slow/down, leader must block all writes

**Semi-synchronous**: one follower is synchronous, rest are asynchronous — guarantees at least 2 nodes are up-to-date.

**Chain Replication**: is a variant of sync replication; leader does not broadcast to all nodes at the same time, they are instead arranged in a chain (used by Azure)
- Write is not considered committed until it has propagated through every node in the chain and the TAIL has confirmed it.
- Each node only needs to forward to the _next_ node, and multiple writes can be _pipelined_ through the chain simultaneously. This is the main advantage of this.
- Write 2 can be at Node 2 while Write 1 is still travelling to the TAIL. 
- Gives much better throughput than a naive synchronous broadcast.
**Recovery**: 
- if the HEAD fails, the next node becomes the new HEAD
- if the TAIL fails, the previous node becomes the new TAIL
- if a middle node fails, it's simply removed and its neighbours are linked
Pros:
- *High throughput*, writes can be pipelined since config manager makes sure writes propagate through and the chain is always connected.
**Cons**:
- *High write latency*: it's proportional to the chain length (latency compounds through each hop)

**Fully asynchronous**: leader doesn't wait for any followers. Widely used in cases where there are a large number of followers, or where followers are geographically distributed (replication lag will be high, sync is not an option)
- ✅ Leader can continue processing even if all followers lag
- ❌ Writes can be lost if leader fails before replicating

---

### Setting Up New Followers

Can't just copy data files — DB is constantly changing, file copy would be inconsistent. Process (without downtime):
1. Take a **consistent snapshot** of leader (most DBs support this; MySQL uses `innobackupex`)
2. Copy snapshot to new follower
3. Follower connects to leader, requests all changes **since the snapshot** (snapshot must be associated with a position in the replication log — PostgreSQL calls it *log sequence number*, MySQL calls it *binlog coordinates*)
4. When follower has processed the backlog, it has **caught up**

---

### Handling Node Outages

**Follower failure — catch-up recovery**: follower keeps a log of received changes. On restart, it requests all changes from leader since the last transaction it processed.

**Leader failure — failover**: one follower must be promoted to leader, clients reconfigured, other followers start consuming from the new leader.

Automatic failover steps:
1. **Detect failure** — most systems use a timeout (e.g. 30 seconds of no heartbeats)
2. **Choose new leader** — typically the replica most up-to-date with old leader; requires consensus (Chapter 9)
3. **Reconfigure** — clients route writes to new leader; old leader must be told it's no longer leader if it rejoins; it should become a follower 

**Note**: typically, *heartbeats* are sent unprompted, *healthchecks* are requested by an external party.

**Failover failure modes:**
- **Lost writes**: new leader may not have received all writes from old leader → unreplicated writes are discarded (violates durability expectations). 
	- **Risk**: risky if an external system is involved. Github had an issue where a new MySQL leader repeated a set of auto-incremented primary keys which had not yet been replicated from the original (failed) leader. The system used to update these keys in Redis; reuse of primary keys caused MySQL and Redis to become inconsistent.
- **Split brain**: two nodes both believe they are the leader → if both accept writes with no conflict resolution, data is lost or corrupted. Fix: setup a *fencing* mechanism (STONITH — Shoot The Other Node In The Head)
	- **Risk**: Fencing can be risky if a fence race occurs, both nodes can shut each other off.
- **Timeout tuning**: too short → unnecessary failovers under load; too long → longer recovery time
	- **Risk**: Leaders usually fail when the system is under unexpectedly heavy load; unnecessary failovers make this situation even worse

There's not easy solutions to the failure modes listed here. This is why *most teams make decision to failover manually*.

---

### Implementation of Replication Logs

**Statement-based replication**: leader logs every SQL statement (INSERT, UPDATE, DELETE) and sends to followers.
- ❌ Nondeterministic functions (`NOW()`, `RAND()`) produce different values per replica
- ❌ Auto-incrementing columns or conditional updates require exact same order of execution; limiting for high concurrency scenarios.
- ❌ Side effects (triggers, stored procedures) may differ unless they are absolutely deterministic
- Used by: MySQL <5.1, VoltDB (with deterministic transactions enforced)

**WAL shipping**: leader sends its write-ahead log to followers; followers build exact same data structures.
- ❌ WAL is tightly coupled to storage engine internals — can't run different DB versions on leader vs. follower (no zero-downtime version upgrades)
- Used by: PostgreSQL, Oracle

**Logical (row-based) log replication**: separate log format decoupled from storage engine, at row granularity.
- INSERT: new values of all columns
- DELETE: enough info to identify row (primary key, or all columns)
- UPDATE: row identity + new values
- ✅ Backward compatible — leader and follower can run different software versions since logical logs are decoupled from storage engine.
- ✅ Easier for external systems like custom indexes, caches, or data warehouses to parse (this is called *change data capture*)
- Used by: MySQL binlog (row-based mode)

**Trigger-based replication**: Used when more flexibility is needed than the database's built-in replication provides — e.g. replicating only a subset of data, replicating between different types of databases, or applying custom conflict resolution logic.

A trigger registers custom application code that executes automatically when a data change (write transaction) occurs. 
Optionally, the trigger logs the change into a separate table; an external process reads from that table and applies the necessary logic to replicate the data change to another system.

- ❌ Greater overhead than other replication methods
- ❌ More prone to bugs and limitations than built-in replication
- ✅ Very flexible — can filter, transform, and route changes arbitrarily
- Used by: Databus (Oracle), Bucardo (Postgres)

---

## Problems with Replication Lag

With async replication + read-scaling, reads may return stale data — **eventual consistency**. Replication lag is usually <1 second in normal operation, but can grow to minutes under heavy load or network issues. Three anomalies to be aware of:

> It would be better if application developers didn't have to worry about these subtle replication issues and could just trust their databases to "do the right thing." This is why **transactions** exist. Single-node transactions have long existed; many distributed DBs abandoned them claiming they're too expensive — but that's an oversimplification (covered in Chapters 7 & 9).

### Read-After-Write Consistency (Read-Your-Writes)

Problem: user submits data, reads from stale replica → appears as if data was lost.

![[Pasted image 20260527171314.png]]

Solutions:
- Always read user's **own** data from the leader (e.g. always read your own profile from leader, other users' profiles from followers)
- If **most things are potentially editable by the user**, the above won't scale — instead: track time of last update and for **one minute after**, make all reads from the leader; or monitor replication lag and prevent queries on any follower more than one minute behind
- Client remembers the **timestamp** of its most recent write — system ensures the replica serving that read reflects updates at least until that timestamp (can be a logical timestamp like log sequence number, or actual system clock)
- If replicas span multiple datacenters, requests that need the leader must be **routed to the datacenter containing the leader**

**Cross-device read-after-write** (same user, different devices e.g. desktop + mobile):
- Timestamp approach becomes harder — code on one client doesn't know what updates happened when on another device; **this metadata must be centralised**
- Connections from different devices may be routed to different datacenters — requests from all a user's devices may need to be routed to the **same datacenter**

### Monotonic Reads

Problem: user reads from fresh replica, then stale replica → data appears to go **backward in time** (e.g. a comment appears then disappears on refresh).

![[Pasted image 20260527171747.png]]

Monotonic reads guarantees this doesn't happen. It's a **lesser guarantee than strong consistency, but a stronger guarantee than eventual consistency** — you may see an old value, but once you've read a newer value, you will never see an older one.

Solution: each user always reads from the **same replica** (chosen e.g. by hash of user ID). If that replica fails, reroute.

### Consistent Prefix Reads

Problem: in partitioned databases, different partitions operate independently with no global write ordering — an observer may see replies before questions (causality violation).

![[Pasted image 20260527171856.png]]

Consistent prefix reads guarantees: **if a sequence of writes happens in a certain order, then anyone reading those writes will see them appear in the same order.**

Solution: ensure causally related writes go to the **same partition** (can't always be done efficiently); or use algorithms that explicitly track causal dependencies (version vectors — covered in Detecting Concurrent Writes).

---

## Multi-Leader Replication

**Multi-leader** (master–master, active/active): more than one node accepts writes; each leader simultaneously acts as a follower to the other leaders.

### Use Cases

**Multi-datacenter operation**: one leader per datacenter. Within each datacenter: regular leader–follower replication. Between datacenters: each datacenter's leader replicates its changes to leaders in other datacenters asynchronously.

![[Pasted image 20260527161810.png]]

|                   | Single-leader                                                                              | Multi-leader                                                                                                     |
| ----------------- | ------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------- |
| Write latency     | Every write must go over the internet to leader datacenter — adds significant latency      | Write processed in local datacenter; inter-DC delay hidden from users (assuming replication isn't strictly sync) |
| Datacenter outage | Failover required to promote follower in another DC                                        | Each DC continues operating independently; replication catches up when DC comes back                             |
| Network problems  | Very sensitive to inter-DC link problems — writes made synchronously over public-internet  | Async replication tolerates network interruptions; temporary outage doesn't prevent writes                       |

**DBs or tools support multi-leader replication OOTB**
- MySQL has Tungsten
- PostgreSQL has BDR
- Oracle has GoldenGate

**Clients with offline operation**: each device acts as a local leader (accepts writes). Sync happens when reconnected — replication lag may be hours or days. (e.g. calendar apps, CouchDB)

**Collaborative editing**: e.g. Google Docs, Etherpad. For no conflicts: acquire a lock (equivalent to single-leader with transactions). For faster collaboration: make unit of change very small (single keystroke), avoid locking — but requires full conflict resolution.

Multi-leader replication is often a retrofitted feature — autoincrementing keys, triggers, integrity constraints can all be problematic. Often considered **dangerous territory** — avoid if possible.

### Handling Write Conflicts

![[Pasted image 20260527161830.png]]

Key difference from single-leader: in multi-leader, **both writes succeed locally**, and the conflict is only detected asynchronously — potentially too late to ask the user to resolve it. (Making conflict detection synchronous removes the main advantage of multi-leader.)

**Conflict avoidance** (simplest, most recommended): route all writes for a particular record through the **same leader** — conflicts cannot occur. Works until you need to reroute (e.g. DC failure, user moves location) — at that point conflict avoidance breaks down.

**Converging to consistent state** — all replicas must arrive at the same final value (order of writes is undefined in multi-leader, so there's no obvious "last" write). Options:
- **Last Write Wins (LWW)**: give each write a timestamp, highest wins. Popular but dangerously prone to data loss.
- **Replica-ID precedence**: writes from higher-numbered replica always win. Also implies data loss.
- **Merge values**: e.g. order alphabetically and concatenate ("B/C")
- **Explicit conflict structure**: record all information, resolve in application code at a later time

**Custom conflict resolution logic** — most multi-leader tools allow application code to resolve:
- **On write**: as soon as a conflict is detected in the replication log, a conflict handler is called (runs in background, cannot prompt user, must execute quickly — e.g. Bucardo uses a Perl snippet)
- **On read**: all conflicting writes stored; on next read, multiple versions returned to app; app resolves (may prompt user) and writes result back — e.g. CouchDB

**Automatic conflict resolution**:
- **CRDTs** (Conflict-free Replicated Data Types): data structures (sets, maps, counters, etc.) that auto-merge concurrent edits sensibly — Riak 2.0
- **Mergeable persistent data structures**: track history explicitly like Git; three-way merge
- **Operational transformation**: algorithm behind Google Docs / Etherpad, designed for concurrent editing of ordered lists (e.g. characters in a document)

### Multi-Leader Replication Topologies

![[Pasted image 20260527161846.png]]

- **Circular**: each node receives writes from one node, forwards to one other (MySQL default). Nodes tag writes with all node IDs passed through to prevent infinite loops. **Problem**: if just one node fails, it interrupts the flow of replication messages between all other nodes — requires manual reconfiguration.
- **Star**: one root node forwards to all others. Same single-point-of-failure problem.
- **All-to-all** (most general): every leader sends to every other. More fault-tolerant — multiple paths. **Problem**: some network links may be faster than others → replication messages can **overtake** each other → writes arrive out of causal order at some replicas (e.g. an UPDATE arriving before the INSERT it depends on).

![[Pasted image 20260527173416.png]]

To correctly order causally dependent writes, **version vectors** can be used. However, conflict detection is poorly implemented in many systems — **PostgreSQL BDR does not provide causal ordering of writes; Tungsten Replicator for MySQL doesn't even try to detect conflicts**. Read docs carefully and test thoroughly.

---

## Leaderless Replication

Any replica can accept writes directly. Inspired by Amazon **Dynamo** (not DynamoDB — that uses single-leader). Open-source Dynamo-style: **Riak**, **Cassandra**, **Voldemort**.

In some leaderless implementations the client sends writes directly to several replicas; in others a **coordinator node** does this on behalf of the client — but unlike a leader, the coordinator does not enforce ordering of writes. This difference has profound consequences for how the DB is used.

### Writing When a Node Is Down

No failover — it simply doesn't exist. Client sends write to **all n replicas in parallel**; write succeeds once **w** replicas acknowledge. The client ignores the fact that one replica missed the write.

On reads, client sends request to **r replicas in parallel** and uses version numbers to determine which value is newer.

![[Pasted image 20260527173436.png]]

**Read repair**: when client reads from multiple replicas and detects a stale response, it writes the newer value back to the stale replica. Works well for **frequently read** values.

**Anti-entropy process**: background process that constantly looks for differences between replicas and copies missing data. Does **not** copy writes in any particular order; may have significant delay. Not all systems implement this — Voldemort does not. Without anti-entropy, **rarely-read values** may be permanently stale (never repaired because never read → reduced durability).

### Quorum Reads and Writes

With **n** replicas, **w** write confirmations required, **r** nodes queried per read:

> **w + r > n** → the sets of nodes written to and nodes read from must overlap by at least one node → at least one r node has the latest write

![[Pasted image 20260527173608.png]]

Common choice: n is an **odd number** (typically 3 or 5); w = r = ⌈(n+1)/2⌉. But quorums don't need to be majorities — any overlap of at least one node suffices.

Writes and reads are **always sent to all n replicas in parallel** — w and r only determine how many responses to wait for before considering the operation successful.

**Tuning**:
- Lower w/r → lower latency, higher availability, but higher chance of stale reads
- w = n, r = 1 → fast reads, but a single failed node causes all writes to fail
- w + r ≤ n → more likely to read stale values; allows lower latency and higher availability at cost of consistency

**Quorum w+r>n still doesn't guarantee strong consistency** — edge cases:
- Sloppy quorum: writes may land on different nodes than reads → no guaranteed overlap
- Concurrent writes with LWW: clock skew can silently drop writes
- Concurrent read + write: undetermined whether old or new value is returned
- Partial write failure (e.g. disk full on some nodes): not rolled back; subsequent reads may or may not return the value
- Node failure + restore from old replica: number of replicas storing latest value can fall below w

Dynamo-style DBs are generally optimised for use cases that can tolerate eventual consistency. In particular, you usually do **not** get read-after-write, monotonic reads, or consistent prefix reads. **Stronger guarantees require transactions or consensus** (Chapters 7 & 9).

### Sloppy Quorums and Hinted Handoff

If a network interruption cuts a client off from the w or r "home" nodes for a value, it faces a choice: return an error, or accept the write on **other reachable nodes** that aren't the designated home nodes. The latter is a **sloppy quorum**.

Sloppy quorums are particularly useful for increasing **write availability** — as long as any w nodes are available, the DB accepts writes. These characteristics make leaderless DBs appealing for **high availability and low latency** use cases that can tolerate occasional stale reads.

**Hinted handoff**: once the network heals, writes temporarily held on "wrong" nodes are forwarded to the correct home nodes.

- Even with w + r > n, you **cannot be sure to read the latest value** with sloppy quorums — the latest write may be on nodes outside the normal n until hinted handoff completes. A sloppy quorum is really just a **durability assurance**, not a true quorum.
- Enabled by default in **Riak**; disabled by default in **Cassandra** and Voldemort

**Multi-datacenter leaderless** (Cassandra, Voldemort): n includes nodes across all datacenters; client typically waits for quorum only within its **local datacenter** — cross-DC writes happen asynchronously. Riak keeps client-to-DB traffic local to one datacenter; cross-DC replication happens asynchronously in the background.

### Monitoring Staleness

Leader-based: easy to measure — subtract follower's log position from leader's = replication lag.

Leaderless: no fixed write order, no log position → much harder. Without anti-entropy, a stale value could be **ancient** — if a value is only infrequently read, the stale replica may never get repaired. Research on quantifying staleness exists but is not yet standard practice.

---

## Detecting Concurrent Writes

Even with strict quorums, Dynamo-style DBs allow several clients to concurrently write to the same key → conflicts arise during normal writes, read repair, and hinted handoff. The problem: events arrive in a different order at different nodes due to variable network delays and partial failures.

![[Pasted image 20260527174011.png]]

### Last Write Wins (LWW)

Force an arbitrary ordering by attaching a **timestamp** to each write; highest timestamp wins. LWW is the **only** supported conflict resolution in Cassandra; optional in Riak.

- ✅ Achieves eventual convergence
- ❌ Durability loss — concurrent writes all reported as successful to client, but only one survives; others are silently discarded
- ❌ Can even drop writes that are **not concurrent** due to clock skew
- It doesn't really make sense to say one concurrent write happened "first": **we say the writes are concurrent, so their order is undefined**; LWW forces an arbitrary order on them
- **Only safe usage**: ensure a key is written **once** and then treated as **immutable** — use a UUID as the key, so there's never a concurrent write to the same key

### The "Happens-Before" Relationship and Concurrency

**B is *causally dependent* on A** if B knows about A, depends on A, or **B builds upon A**. Two operations are **concurrent** if neither knows about the other — regardless of wall-clock time (exact time doesn't matter; a slow network means two ops can be concurrent even if they occur far apart in time).

Three possible relationships between operations A and B:
- A happened before B → B should overwrite A
- B happened before A → A should overwrite B
- A and B are **concurrent** → conflict, needs resolution

An algorithm needs to determine which case applies. If one happened before another, the later operation should overwrite the earlier one; if concurrent, we have a conflict.

### Capturing the Happens-Before Relationship (Single Replica)

Algorithm for a single-server DB, read the two sets of rules given below first, then this diagram, and it'll make sense

![[Pasted image 20260527174045.png]]

- Server maintains a **version number** per key; increments on every write; stores new version number along with the written value
- On **read**: server returns all values not yet overwritten + latest version number. A client **must read a key before writing**
- On **write**: client must include version number from prior read, and must **merge all values received in that prior read**
- Server receives write with version V:
  - **Overwrites** all values with version ≤ V (they've been merged into the new write)
  - **Keeps** all values with version > V (they are concurrent with this write)

**Explicit versioning/sibling rules:**

1. **No version number sent by client** → write is concurrent with everything (treated as version 0 / "I've seen nothing"). Does not overwrite anything — returned as one of the values on subsequent reads.
2. **Client sends version V** → supersedes all siblings stored with version ≤ V (overwrite them); concurrent with all siblings stored with version > V (keep them as siblings)
3. **Every write bumps the global version counter** for the key, regardless of whether it supersedes or is a concurrent write
4. **Each sibling carries the global version number** stamped at the time it was written (implicit in the book — never stated explicitly)
5. **All siblings are returned to the client on every read**; the client is responsible for merging them when writing back

### Merging Concurrently Written Values (Siblings)

Riak calls concurrent values **siblings**. Client must merge them — same problem as multi-leader conflict resolution.

- Simple approach: pick one based on version/timestamp (LWW) → data loss
- For sets: take the **union** of siblings (works for additions)
- For **deletions**: can't simply delete — must use a **tombstone** (marker with version number indicating deletion), otherwise deleted items reappear when unioned with a sibling that still has the item
- **CRDTs** (e.g. Riak's datatype support) can auto-merge siblings including deletions

### Version Vectors (Multiple Replicas)

A single version number is insufficient when multiple replicas accept concurrent writes. Solution: **version number per replica per key**. Each replica increments its own version number when processing a write, and also tracks version numbers seen from other replicas.

The collection of all replicas' version numbers = **version vector** (sometimes called *vector clock*, though technically different — version vectors are the correct structure for comparing replica state). Riak 2.0 uses *dotted version vectors* (encodes it as a string called *causal context*).

Version vectors are sent from replicas to clients on read; sent back by client on write. Allows DB to distinguish overwrites from concurrent writes. Safe to **read from one replica and write back to a different one** — siblings may be created, but no data is lost as long as siblings are correctly merged.

---

## Summary

| Approach | Writes accepted by | Reads | Conflict resolution |
|---|---|---|---|
| **Single-leader** | Leader only | Leader or followers (may be stale) | No conflicts — writes serialised at leader |
| **Multi-leader** | Any leader | Any node | Required — must resolve concurrent writes |
| **Leaderless** | Any replica (w of n) | Multiple replicas (r of n) | Required — LWW or sibling merging |

**Consistency models for replication lag:**
- **Read-after-write**: users always see data they themselves submitted
- **Monotonic reads**: users don't see data from an earlier point in time after having previously read newer data
- **Consistent prefix reads**: see causally related writes in the correct order
