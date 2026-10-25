## Overview

A **transaction** groups multiple reads and writes into a logical unit — either the whole thing succeeds (commit) or fails (abort/rollback). With transactions, the application can ignore partial failure and retry safely.

Transactions are not a law of nature. They simplify the programming model by letting the database handle certain error scenarios and race conditions (**safety guarantees**). But they have costs — sometimes weakening or abandoning them is worthwhile for performance or availability.

---

## ACID Properties

### Atomicity
- **Not** about concurrency, like the Java keyword (that's Isolation)
- About what happens when a fault occurs *mid-write*: if the transaction can't complete, it is **aborted** and all writes so far are **discarded**
- The defining feature: **all-or-nothing** guarantee → application can safely retry
- Better term would have been *abortability*

### Consistency
- ACID "C" = application-defined **invariants** that must always hold (e.g., debits = credits in a bank's DB)
- **Not a database property** — the DB can't enforce arbitrary invariants; it's the application's responsibility to write transactions that preserve them (using DB's atomicity or isolation guarantees)
- Atomicity, Isolation, and Durability are DB properties. Consistency is an application property. The "C" in ACID was reportedly "tossed in to make the acronym work."

### Isolation
- Concurrently executing transactions are **isolated** from each other — they can't step on each other's toes
- Classic textbook definition: **serializability** — result is the same as if transactions ran serially, one at a time
- In practice, serializable isolation is rarely used due to performance cost; weaker levels are common

![[Pasted image 20260529081640.png]]
*Lost Update Problem*
### Durability
- Once a transaction commits, the data will not be lost — even on hardware fault or crash
- Single-node: write to non-volatile storage (disk / SSD) + write-ahead log for recovery
- Replicated DB: data successfully copied to N nodes before acknowledging commit
- **Perfect durability doesn't exist** — various real-world risks:
  - Disk is only accessible if machine is alive (vs replicated systems that stay available)
  - Correlated faults (power outage, bug) can knock out all replicas simultaneously → in-memory data lost (vs disk that will survive)
  - Async replication can lose recent writes if leader fails before replication
  - SSDs: `fsync` not guaranteed to work correctly after sudden power cut; 30–80% of drives develop bad blocks in first 4 years; SSDs slowly lose data when not powered on
  - Silent data corruption can spread to replicas and backups before detection
- Best practice: use multiple techniques together (disk + replication + backups)

---

## Single-Object vs Multi-Object Operations

**Multi-object transactions** are needed when multiple pieces of data must stay in sync:
- Foreign key references in relational DBs (insert must keep references valid)
- Denormalized data in document DBs (e.g., unread counter + email record must update together)
- Secondary indexes (record and its indexes must update atomically)

![[Pasted image 20260529083703.png]]
*Dirty Read: one transaction reads another transaction’s uncommitted writes.*

![[Pasted image 20260529083727.png]]
*Atomicity: if an error occurs, prior writes from that transaction are undone.*

**Single-object writes** — storage engines universally provide atomicity + isolation at the single-object level (via write-ahead log + per-object locks). This handles:
- Partial network writes (10 KB of 20 KB JSON sent before connection drops)
- Power failure mid-overwrite
- Concurrent reads of partially-written objects

Some DBs also offer atomic single-object operations: `increment`, `compare-and-set`. These are **not** transactions — they're "lightweight transactions" (marketing term). A real transaction groups **multiple operations on multiple objects**.

### The need for multi-object transactions
Many distributed datastores abandoned multi-object transactions (hard to implement across partitions, can hurt availability/performance). But they are genuinely needed for the use cases above. Nothing *fundamentally* prevents them in distributed DBs — see Chapter 9.

### Handling Errors and Aborts

ACID DBs: if danger of violating guarantees → abort entirely rather than remain half-finished.

Leaderless replication datastores: "best effort" → won't undo what it already did. Error recovery is the application's responsibility.

**Retrying aborted transactions** is simple but not perfect:
- If network failed *after* commit (server committed, client thinks it failed) → retry causes duplicate execution — need application-level deduplication
- If error is due to overload → retrying makes it worse. Use exponential backoff + limit retries
- Only retry for **transient** errors (deadlock, network, failover); **permanent** errors (constraint violation) should not be retried
- If transaction has side effects outside the DB (e.g., sending email) → side effect may happen even if TX aborts. Use 2PC to coordinate (Ch9)
- If client crashes during retry → writes lost

---

## Weak Isolation Levels

Concurrency bugs are hard to test — they only trigger with unlucky timing, and are nondeterministic. DBs hide concurrency via transaction isolation. Full **serializability** has performance cost → DBs use weaker levels in practice. These are widely misunderstood and inconsistently implemented.

### Read Committed

Two guarantees:
1. **No dirty reads** — only see data that has been committed
2. **No dirty writes** — only overwrite data that has been committed

#### No Dirty Reads

![[Pasted image 20260529143436.png]]
*No dirty reads: user 2 sees the new value for x only after user 1’s transaction has committed.*

Why dirty reads are bad:
- Another transaction may see partial update (some writes committed, others not yet) → confusing/incorrect decisions
- Aborted transaction's writes might be visible before rollback → data that was "never committed" was seen

Implementation: DB keeps **both old and new value** for any object being written. Reads see the old value until the write transaction commits (no read lock needed).

#### No Dirty Writes

![[Pasted image 20260529143456.png]]
*With dirty writes, conflicting writes from different transactions can be mixed up.*

A dirty write = overwriting an uncommitted value. Prevention: delay second write until first transaction commits or aborts (via row-level locks on writes).

Prevents mishaps like: Alice and Bob both try to buy the same car → without this protection, Bob could win the listing update but Alice could win the invoice update.

**What read committed does NOT prevent**: the race condition in fig7-1 (two counter increments). The second write happens *after* the first committed — not a dirty write, but still incorrect. → Needs "Preventing Lost Updates."

Default isolation level in: Oracle 11g, PostgreSQL, SQL Server 2012, MemSQL.

---

### Snapshot Isolation and Repeatable Read

**Read skew** (nonrepeatable read): a transaction reads data at different points in time and sees inconsistent results (e.g., Alice sees $500 in account 1 before transfer, and $400 in account 2 after — total appears $100 short).

![[Pasted image 20260529143513.png]]

Read skew is *acceptable* under read committed (both values were committed at time of reading), but problematic for:
- **Backups**: multi-hour backup may capture inconsistent state → permanent inconsistency if restored
- **Analytic queries / integrity checks**: scan over large parts of DB → nonsensical results if data changes mid-query

**Snapshot isolation** (solution): each transaction reads from a **consistent snapshot** — the state of the DB as of when the transaction started. Even if data changes later, the transaction always sees its snapshot.

Supported by: PostgreSQL, MySQL InnoDB, Oracle, SQL Server.

#### Implementing Snapshot Isolation (MVCC)

Key principle: **readers never block writers, writers never block readers.**

Implementation uses **Multi-Version Concurrency Control (MVCC)**:
- Each transaction gets a unique, always-increasing **transaction ID (txid)**
- Every written object is tagged with `created_by = txid` and `deleted_by = txid` (soft delete)
- An **update** is internally a **delete + create** (old row marked deleted_by, new row created)
- Garbage collection eventually removes rows no longer visible to any transaction

![[Pasted image 20260530171328.png]]

**Visibility rules** — an object is visible to a transaction if:
1. The transaction that *created* it had already committed when the reader's transaction started
2. The transaction that *deleted* it (if any) had NOT yet committed when the reader's transaction started

At the start of each transaction, the DB records all in-progress transactions. Any writes from those transactions (even if they later commit) are ignored.

**Indexes and MVCC**:
- Index points to all versions of an object; query filters out non-visible versions
- PostgreSQL optimization: avoids index updates if different versions of same object fit on same page
- CouchDB/Datomic/LMDB approach: append-only / copy-on-write B-trees — each write creates a new B-tree root (new snapshot). No need to filter by txid; old roots remain intact. Requires background compaction/GC.

#### Naming Confusion: Repeatable Read

Snapshot isolation is called different things:
- Oracle: "serializable"
- PostgreSQL, MySQL: "repeatable read"
- IBM DB2: "repeatable read" means *serializable*
Reason: SQL standard was defined before snapshot isolation was invented. It defines "repeatable read" which looks similar but is not identical. Nobody really knows what "repeatable read" means across implementations.

#### Naming Confusion: Lost Update vs Dirty Write

|                        | Lost Update      | Dirty Write                    |
| ---------------------- | ---------------- | ------------------------------ |
| Overwrites             | committed write  | uncommitted write              |
| Danger                 | silent data loss | inconsistent state on rollback |
| Prevented by row locks | ✗ no             | ✓ yes                          |
So if you have an only write level lock and not a transaction level lock, you're fucked



---

### Preventing Lost Updates

**Lost update problem**: two transactions do a read-modify-write cycle concurrently. One modification is lost because the second write doesn't include the first modification ("later write clobbers earlier write").

Common patterns where this occurs:
- Incrementing a counter / updating account balance
- Making a local change to a complex value (e.g., adding element to JSON list)
- Two users editing a wiki page simultaneously

#### Solutions

**1. Atomic write operations** (best when applicable)
```sql
UPDATE counters SET value = value + 1 WHERE key = 'foo';
```
Implemented via exclusive lock on object during read-modify-write (also called *cursor stability*), or by forcing atomic ops onto a single thread. MongoDB, Redis provide atomic operations for their types. ORM frameworks can accidentally generate unsafe read-modify-write code.

**2. Explicit locking**
```sql
BEGIN TRANSACTION;
SELECT * FROM figures WHERE name = 'robot' AND game_id = 222 FOR UPDATE;
-- validate move logic
UPDATE figures SET position = 'c4' WHERE id = 1234;
COMMIT;
```
`FOR UPDATE` locks all returned rows. Works but easy to forget a lock somewhere → race condition.

**3. Automatically detecting lost updates**
Allow concurrent read-modify-write cycles; if transaction manager detects a lost update → abort and retry. Efficient in conjunction with snapshot isolation.
- **Detects**: PostgreSQL repeatable read, Oracle serializable, SQL Server snapshot isolation
- **Does NOT detect**: MySQL/InnoDB repeatable read
- Less error-prone than manual locks — detection is automatic

**4. Compare-and-set**
```sql
UPDATE wiki_pages SET content = 'new content'
WHERE id = 1234 AND content = 'old content';
```
Allows update only if value hasn't changed since last read. Check whether the update took effect and retry if not. **Caution**: if DB allows WHERE clause to read from an old snapshot, this may not prevent lost updates.

**5. Conflict resolution in replicated DBs**
Locks and compare-and-set assume **one up-to-date copy** — invalid with multi-leader or leaderless replication (concurrent writes on different nodes, async replication).

Instead: allow concurrent writes to create **conflicting versions (siblings)**, resolve after the fact via:
- Application code merging siblings
- Commutative atomic operations (e.g., increment, add-to-set) — apply in any order on any replica and get same result. Used by Riak 2.0 CRDTs.
- **LWW (last write wins)**: prone to lost updates. Unfortunately the default in many replicated DBs.

---

### Write Skew and Phantoms

**Write skew**: two transactions read the same objects, then *each updates different objects* based on what it read. Neither transaction individually makes an invalid write, but together they violate an invariant.

Example: two doctors both see ≥2 doctors on call → both decide to go off call → both commit → 0 doctors on call.

![[Pasted image 20260530175204.png]]

Write skew is a **generalisation of lost updates**:
- Lost update: two transactions update the *same* object
- Write skew: two transactions update *different* objects based on a shared premise

#### Preventing Write Skew — Options and Limitations

| Approach | Works for write skew? |
|---|---|
| Atomic single-object ops | ❌ Multiple objects involved |
| Auto-detect lost updates | ❌ Not detected in most snapshot isolation implementations |
| DB constraints | ⚠️ Only if constraint involves a single object; multi-object constraints rare |
| Explicit `SELECT FOR UPDATE` | ✅ Works if the locked rows are the ones modified |
| Serializable isolation | ✅ Only reliable solution |

For the doctors example: `SELECT FOR UPDATE` works because the rows checked (on-call doctors) are later modified. But for other write skew patterns, `SELECT FOR UPDATE` can't attach locks to anything (nothing yet exists to lock).

#### More Write Skew Examples

- **Meeting room booking**: check no conflicts → insert booking (two concurrent bookings can slip through snapshot isolation)
- **Multiplayer game**: two pieces move to same position (lock prevents same piece, not same position)
- **Claiming a username**: check not taken → create account (unique constraint is the fix here)
- **Preventing double-spending**: check balance ≥ amount → insert spending → balance can go negative with two concurrent spends

#### Phantom Reads

The write skew pattern:
1. **SELECT** checks a condition (≥2 doctors on call, no conflicting booking, etc.)
2. Application decides to proceed based on result
3. Application **writes** (INSERT/UPDATE/DELETE) — this write *changes the precondition* of step 2

A **phantom** = a write in one transaction changes the result of a search query in another transaction. Snapshot isolation avoids phantoms in read-only queries, but *not* in read-write transactions with this pattern.

`SELECT FOR UPDATE` can't fix phantoms where the search returns 0 rows (no rows to attach lock to).

#### Materializing Conflicts

Artificially introduce a **lock object** into the DB to attach locks to.

Example for meeting room booking: pre-create a table of all (room, time_slot) pairs for the next 6 months. Each booking transaction locks the relevant rows (`SELECT FOR UPDATE`). The table is purely for locking — not for storing booking data.

**Downsides**: hard and error-prone to figure out, leaks concurrency control into application data model. Use as a **last resort** — serializable isolation is much preferable.

---

## Serializability

Serializable isolation = strongest guarantee. The DB ensures that even if transactions run in parallel, the result is identical to some serial execution. Prevents **all** race conditions.

Three approaches to implement serializability:

---

### 1. Actual Serial Execution

Execute one transaction at a time, single-threaded, no concurrency. By definition serializable.

Became feasible around 2007 due to:
- RAM got cheap enough to keep entire working dataset in memory → no disk I/O waiting
- OLTP transactions are short (few reads/writes); analytics are read only and can run on consistent snapshots outside the serial loop

Used by: VoltDB/H-Store, Redis, Datomic.

Faster than lock-based systems for some workloads (no locking overhead), but **limited to one CPU core**.

#### Stored Procedures (Required)
Even though a transaction is committed within the same HTTP request, interactive client/server style wastes time waiting for network round trips. 
With serial execution, a slow transaction stalls everyone. Solution: submit the **entire transaction as a stored procedure** — the DB executes it in one shot, no network/disk I/O waiting mid-transaction.

![[Pasted image 20260530181042 1.png]]

*Traditional stored procedures*: bad reputation (PL/SQL, T-SQL are ugly languages; SPs are hard to debug, version, test, monitor, push metrics from). They also have a bigger blast radius than a service since they're shared by many services, poorly written SP can destroy DB performance and affect all services. 
*Modern implementations* use general-purpose languages: VoltDB → Java/Groovy, Datomic → Java/Clojure, Redis → Lua.

VoltDB replication: executes the same stored procedure on each replica (rather than copying writes) → **stored procedures must be deterministic**.

#### Partitioning for Scale

Single-threaded execution limits throughput to one CPU core. Scaling option: partition data so each transaction only touches one partition → each partition has its own transaction thread → throughput scales linearly with CPU cores.

**Cross-partition transactions**: the stored procedure must run in lock-step across all involved partitions → significant coordination overhead. VoltDB reports ~1,000 cross-partition writes/sec, orders of magnitude below single-partition throughput. Not scalable by adding nodes.

#### Constraints for Serial Execution to Work

- Every transaction must be **small and fast** (one slow transaction stalls all)
- Active dataset must **fit in memory**
- Write throughput must be low enough for a single CPU core, or partitioned without cross-partition coordination
- Cross-partition transactions are possible but expensive

---

### 2. Two-Phase Locking (2PL)

> Note: 2PL ≠ 2PC (two-phase commit — covered in Ch9). Completely different things.

The standard approach for serializability for ~30 years. Used in MySQL InnoDB (serializable isolation level) and SQL Server, DB2 (repeatable read level).

**Core rule**: readers block writers, writers block readers (contrast with snapshot isolation where they never block each other).

> **Why 2PL and not just MVCC?** Snapshot isolation gives you *read stability* — you always see a consistent snapshot and won't see concurrent writes. But it doesn't give you *premise stability*: transaction A reads X, makes a decision, and writes based on that decision — but by commit time, X may have changed (write skew). Under 2PL, B cannot write X until A commits, so A's premise is guaranteed to still be true at commit time. MVCC protects what you *see*; 2PL protects what you *act on*.

Specifically:
- **A reads, B wants to write** → B must wait until A commits/aborts
- **A writes, B wants to read** → B must wait until A commits/aborts  
- **A reads, then upgrades to write** → upgrades shared lock to exclusive lock
- Locks held until **end of transaction** (this is the "two-phase": acquire phase → release phase at commit)

#### Lock Modes

Each object in the DB has a lock (shared or exclusive mode):
- **Read** → acquire **shared lock** (multiple transactions can hold shared simultaneously; blocked if exclusive lock exists)
- **Write** → acquire **exclusive lock** (no other lock of any kind allowed; all must wait)
- **Read then write** → upgrade shared → exclusive lock

#### Deadlocks

With many locks, A waits for B's lock and B waits for A's lock. DB automatically **detects deadlocks** and aborts one transaction (application must retry).

#### Performance of 2PL

Big downside: significantly worse throughput and latency than weak isolation.
- Overhead of acquiring/releasing many locks
- Reduced concurrency — any potential race condition causes one transaction to wait
- Long-running transactions can cause queue cascades: one slow transaction with many locks can stall the entire system
- Deadlocks are more frequent than under weaker isolation → wasted work from retries
- Latency is unstable and high at high percentiles under contention

#### Predicate Locks

Needed to prevent **phantoms**. A predicate lock belongs to all objects matching a search condition (including objects that don't exist yet but might be inserted):

```sql
SELECT * FROM bookings
WHERE room_id = 123 AND end_time > '12:00' AND start_time < '13:00';
```

Rules:
- Reading: acquire **shared predicate lock** on the condition. Must wait if any exclusive lock matches
- Writing (INSERT/UPDATE/DELETE): check if old/new value matches any existing predicate lock. Must wait if yes

This prevents all write skew and phantoms, achieving full serializability.

**Problem**: checking predicate locks is expensive if many active transactions have many locks.

#### Index-Range Locks (Next-Key Locking)

Practical approximation of predicate locks used by most 2PL databases. Simplify predicate by locking a **larger set of objects** (safe, because any write matching original predicate also matches the approximation):

- Lock all bookings for room 123 at *any time* (approximation of room 123, noon–1pm)
- Or lock all rooms in the *noon–1pm* range (approximation of room 123, noon–1pm)

Implementation: attach shared lock to an index entry (e.g., `room_id = 123` index entry). Any write to the same index range encounters the lock and must wait.

If no suitable index exists → fall back to locking the entire table (safe, but hurts performance severely).

---

### 3. Serializable Snapshot Isolation (SSI)

A newer algorithm (2008, Michael Cahill's PhD thesis) providing **full serializability with small performance penalty** over snapshot isolation. Used in PostgreSQL (serializable level since 9.1) and FoundationDB.

**Pessimistic vs optimistic concurrency control**:
- 2PL = **pessimistic**: if anything might go wrong (indicated by a lock), wait until safe
- Serial execution = pessimistic to the extreme: each transaction holds an exclusive lock on the entire DB/partition
- SSI = **optimistic**: transactions proceed without blocking; check for violations at commit time. If violated → abort and retry

Optimistic CC performs badly under **high contention** (many aborts + retries amplify load). Performs well when there's spare capacity and low contention.

#### How SSI Works

Based on **snapshot isolation** — reads from a consistent snapshot. Adds detection of **serialization conflicts** to determine which transactions to abort.

**Core problem**: a transaction may act on an **outdated premise** — it read data, made a decision, but by commit time the data it read has changed.

Two cases to detect:

**Case 1: Detecting stale MVCC reads** (uncommitted write occurred *before* the read)

![[Pasted image 20260603133700.png]]

- Transaction 43 reads Alice's on-call status from snapshot; transaction 42 has already modified it but is uncommitted at read time → 43 ignores the write per MVCC rules
- When 43 wants to commit, 42 has since committed → 43's premise is now stale
- DB tracks which transactions' writes were ignored. At commit time: if any ignored writes are now committed → abort the transaction
- Why wait until commit? At read time, the transaction might be read-only (no risk of write skew). Also, 42 may abort or may still be uncommitted when 43 commits → no need to abort unnecessarily. This preserves snapshot isolation's support for long-running reads.

> **Read-only transactions are never aborted by SSI.** Even if many concurrent writes commit to the rows being read, a read-only transaction always completes successfully. SSI only aborts a transaction when it *wrote* something based on a stale premise (an instance of *write skew*) — a pure read transaction has no writes, so there is no serialization violation possible. The tripwire tracking still happens in the background, but at commit time the check ("did you act on stale data?") is vacuously false for a read-only transaction → it always passes. This makes SSI excellent for long-running analytical queries running alongside an OLTP write workload.

**Case 2: Detecting writes that affect prior reads** (write occurs *after* the read)

![[Pasted image 20260603133732.png]]

- Transactions 42 and 43 both read on-call doctors for shift 1234 → both recorded against the `shift_id = 1234` index entry
- When a transaction writes (updates on-call status), it checks the index for other transactions that recently read that data
- Instead of blocking those readers, it **notifies** them: their data may be outdated (like a tripwire)
- Transaction 42 commits first → succeeds (transaction 43's write hasn't taken effect yet). Transaction 43 tries to commit → sees that 42 already committed a conflicting write → must abort

#### Performance of SSI

- **vs 2PL**: no blocking for reads or writes — query latency more predictable and less variable. Read-only queries on consistent snapshots need no locks. Particularly good for read-heavy workloads.
- **vs Serial execution**: not limited to one CPU core. FoundationDB distributes serialization conflict detection across machines → scales to high throughput even with cross-partition transactions.
- Abort rate affects performance: long read-write transactions more likely to conflict and abort. SSI requires fairly short read-write transactions. Less sensitive to slow transactions than 2PL or serial execution.
- Tracking granularity trade-off: fine-grained tracking → fewer unnecessary aborts but more bookkeeping overhead; coarse-grained → faster but more aborts than strictly needed.

---

## Summary: Race Conditions and Isolation Levels

| Race Condition | Description | Prevented By |
|---|---|---|
| **Dirty read** | Read another TX's uncommitted writes | Read committed + |
| **Dirty write** | Overwrite another TX's uncommitted write | Read committed + |
| **Read skew** (nonrepeatable read) | Different parts of DB seen at different times | Snapshot isolation (MVCC) |
| **Lost update** | Concurrent read-modify-write; one update lost | Atomic ops / locks / auto-detection / SSI |
| **Write skew** | Two TXs read same data, update different objects, violate invariant | Serializable isolation only |
| **Phantom read** | Write changes result of a search query in another TX | Snapshot isolation (read-only); index-range locks or SSI (read-write) |

| Serializability Approach | How | Pros | Cons |
|---|---|---|---|
| **Serial execution** | Single thread + stored procedures | Simple, no race conditions | Single CPU core; dataset must fit in memory |
| **Two-phase locking (2PL)** | Shared/exclusive locks; writers block readers and vice versa | Full serializability | High latency, unstable under contention, deadlocks |
| **SSI** | Optimistic; detect conflicts at commit time | Full serializability, good performance, scalable | High abort rate under high contention |
