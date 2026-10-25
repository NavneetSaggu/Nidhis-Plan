# Database Concurrency

# Concurrency Benefits

Concurrency in databases matters a lot, and here’s why it’s a big deal:

- **Simultaneous Access:** Databases often serve many users or processes at once. Concurrency ensures that everyone can access the data they need when they need it, without interference.
- **Transaction Isolation:** Concurrency control mechanisms are put in place to ensure that transactions are isolated from each other, meaning the operations of one transaction are not visible to others until they are complete. This is crucial to maintaining the integrity of the data.
- **Improved Performance:** When multiple operations can occur concurrently, the overall system performance is better. This means faster responses and less waiting time for users.
- **Resource Utilization**: Concurrency leads to more efficient use of database resources, as it can process several operations at once, rather than processing transactions one by one.

# Violations

## Dirty Read

*(Read UNCOMMITED data from another transaction)* 

> Example
> 
> - T1 modifies a value in row 1.
> - T2 reads the updated value in row 1 before T1 commits.
> - T1 aborts, making T2’s read data invalid.

Dirty reads are *similar* to non-repeatable and phantom reads, but relate to reading UNCOMMITTED data, and occur when an UPDATE, INSERT, or DELETE from another transaction is read, and the other transaction has NOT yet committed the data. It is reading "in progress" data, which may not be complete, and may never actually be committed.

## Non-repeatable Read

*(Read COMMITTED data from an UPDATE query from another transaction)*

> Example
> 
> - T1 reads a record.
> - T2 updates/deletes that record and commits.
> - T1 re-reads the record and finds a different value or discovers that the record is gone.

A non-repeatable read occurs, when during the course of a transaction, a row is retrieved twice and the values within the row differ between reads.

## Phantom Read

*(Read COMMITTED data from an INSERT or DELETE query from another transaction)*

> Example
> 
> - T1 runs a query to count rows meeting a condition.
> - T2 inserts/deletes rows affecting this condition and commits.
> - T1 re-runs the query and gets a different count.

A phantom read occurs when, in the course of a transaction, two identical queries are executed, and the collection of rows returned by the second query is different from the first.

## Lost Update

*(Data gets OVERWRITTEN between two concurrent transactions)*

> Example
> 
> - T1 and T2 both read row 1.
> - T1 modifies row1 and commits.
> - T2 updates row1, not knowing the changes made by T1.
> - T2 commits, it will lead to overwriting T1’s changes.

In short, the last update wins.

## Why is Update not a Delete-Insert?

DELETE statements from another transaction, have a very low probability of causing Non-repeatable reads (only when the DELETE statement removes the very same row which your current transaction was querying). But this is a rare case, very unlikely to occur in a database which have millions of rows in each table.

UPDATES are a more frequent job in most use cases rather than actual INSERT or DELETES. This is why UPDATES are treated differently from INSERT-DELETE and the resulting anomaly is also named differently.

There is also an additional processing cost associated with handling for INSERT-DELETEs, rather than just handling the UPDATES.

# Isolation Levels

(In order of increasing processing times)

- READ_UNCOMMITTED
    - prevents nothing. It's the zero isolation level
    - implements no locking
- READ_COMMITTED
    - prevents just one, i.e. Dirty reads
    - implemented using row-level locks, which are acquired and released as data is read and modified
    - read locks are shared and modification locks are exclusive
    - this makes concurrent modification safe but still doesn’t guarantee repeatable reads
    - deadlocks less likely to occur
- REPEATABLE_READ
    - prevents two anomalies: Dirty reads and Non-repeatable reads
    - implemented using row-level locks that are held until the end of the transaction.
    - read locks are shared and modification locks are exclusive
    - this makes concurrent modification safe and also guarantees repeatable reads
    - deadlocks more likely to occur
- SERIALIZABLE
    - prevents all three anomalies: Dirty reads, Non-repeatable reads and Phantom reads
    - implemented using strict locking, where transactions acquire locks on all data they access, and only release the locks at the end of the transaction.
    - deadlocks very likely to occur

|  | **Dirty read** | **Non-repeatable read** | **Phantom read** |
| --- | --- | --- | --- |
| **Serializable** | no | no | no |
| **Repeatable read** | no | no | yes |
| **Read committed** | no | yes | yes |
| **Read uncommitted** | yes | yes | yes |

# Concurrency Control

There’s two ways to do this

# Pessimistic Locking

It assumes that two concurrent accesses are likely to be conflicting, and thus implement an algo that is tweaked to handle cases where concurrent accesses are more often conflicting than not.

## 2 Phase Locking Algorithm

Two-phase locking (2PL) is a concurrency control method used in database systems to ensure **transaction serializability**. Transaction serializability ensures that even when database transactions overlap in time, the final result looks if they had occurred one after another in sequence. The protocol works in two phases:

1. **Locking (Growing) Phase**: In this phase, a transaction may obtain locks on data (rows, tables etc) but cannot release any. There are mainly two types of locks:
    - **Shared Lock**: Allows a transaction to read a resource. Multiple transactions can hold shared locks on the same resource, which means multiple readers can read the same data.
    - **Exclusive Lock**: Allows a transaction to both read and write a resource. If a transaction holds an exclusive lock on a resource, no other transaction can acquire any lock (shared or exclusive) on that resource.
2. **Unlocking (Shrinking) Phase:** The transaction releases all the locks in this phase but cannot acquire any new ones. locks can be released as soon as the transaction no longer needs the locked resource

In 2PL, multiple readers can access data simultaneously through shared locks. However, this system struggles with concurrent reads and writes: reader and writer transactions interfere with each other since readers’ shared locks are incompatible with writers’ exclusive locks.

The lock compatibility would be like this:

| Lock Compatibility | Shared Lock | Exclusive Lock |
| --- | --- | --- |
| Shared Lock | Yes | No |
| Exclusive lock | No | No |

### Example

To illustrate two-phase locking, let’s consider two transactions, T1 and T2:

**Transaction T1** wants to read a record A and then update it. **Transaction T2** wants to read the same record A and then read record B.

Here’s how two-phase locking would work in this scenario:

1. `*T1 starts and requests a shared lock on record A.*`
2. `*T1 is granted the shared lock since no one else has a lock on A.*`
3. `*T1 reads record A.*`
4. `*Before writing, T1 upgraded its lock from shared to exclusive lock on record A.*`
5. `*T2 starts and requests a shared lock on record A, but must wait because T1 has an exclusive lock.*`
6. `*T1 writes to record A (still in the locking phase).*`
7. `*After writing, T1 enters the unlocking phase, releasing all the locks on record A and then commits(The unlocking/shrinking phase presents several scenarios, discussed in types of 2PL below)*`
8. `*As soon as the lock is released, T2 acquires a shared lock.*`
9. `*T2 reads record A and then requests and is granted a shared lock on record B.*`
10. `*T2 reads record B and then completes its operation.*`

T1 must complete its operations before T2 can read it, ensuring that T2 gets the updated value. This example shows how 2PL implements serialization of transactions by implementing locks and prevents inconsistent reads and updates.

### Deadlocks

A deadlock occurs when two or more transactions are waiting for each other to release locks, and none of them can proceed.

**Example**

Transaction T1 and T2 are running concurrently

1. `*T1 requests and acquires an exclusive/shared lock on Row 1 in the sales table.*`
2. `*T2 requests and acquires an exclusive/shared lock on Row 2 in the same table.*`
3. `*T1 now requests an exclusive lock on Row 2 but has to wait because T2 holds the lock.*`
4. `*T2 requests an exclusive lock on Row 1 but also has to wait because T1 holds the lock.*`

Transactions can get stuck in a deadlock when each waits for the other to release its locks. To address this, the database must detect and resolve the deadlock, typically by aborting/rollback one of the transactions.

### Cascading Rollbacks

Cascading rollback happens when the failure of one transaction forces rollbacks of other transactions. It occurs when other transactions read uncommitted data from the failed transaction, leading to a situation known as **dirty reads**.

**Example**

1. `*T1 starts,acquires exclusive lock on Row 1, writes data against row 1 and then unlocks row 1 during shrinking phase.*`
2. `*T2 starts, acquires shared lock and reads the uncommitted data from Row 1 (the changes made by T1).*`
3. `*T3 starts and reads the uncommitted data from Row 1 as well, or perhaps reads some derivative data from T2 that originated with T1's changes.*`
4. `*T1 encounters a problem and must be rolled back, undoing the changes to Row 1.*`
5. `*Because T2 and T3 have read and possibly acted on this now-rolled-back data, their operations may be based on invalid data states. Therefore, they too must be rolled back to ensure data consistency which depicts cascading rollback effect.*`

### Reduced Concurrency

In 2PL, transactions maintain their locks until completion, potentially limiting the number of transactions that can be processed concurrently.. If another transaction needs the same locked resources, it has to wait, which can cause delays and a backlog of pending transactions.

**Example**

1. `*T1 starts a long-running operation, like generating a report, which requires a shared lock on many rows in the accounts table.*`
2. `*T2 and T3, which want to update some of the locked rows, are blocked until T1 completes and releases its locks, even though they might be quick operations.*`

## Strict 2PL

Strict 2PL ensures that transactions retain all their exclusive locks until they complete, either by committing or aborting. By holding onto write-locks until a transaction is fully finalized, it prevents other transactions from accessing the locked data, thereby **minimizing the risk of cascading rollbacks**. Shared locks can be released before commit/rollback.

**Example**

1. `*T1 starts and writes to Row 1.*`
2. `*T1 acquires a shared lock on Row 2.*`
3. `*T2 starts and try to acquire shared lock on Row 1 but since it is exclusively locked for writes by T1, T2 has to wait*`
4. `*T3 starts and try to acquire write lock on Row 2 but since it is shared locked by T1, T3 has to wait*`
5. `*T1 encounters a problem and rolled back, undoing the changes to Row 1.(Here it can release the shared lock on Row 2 before rollback and must release exclusive lock on Row 1 after rollback)*`
6. `*At this step, the database doesn’t need to rollback transactions T2 and T3 because they didn’t read the uncommitted data from T1.**(No cascading effect)***`
7. `*Now T3 can take write lock on Row 2 and after T2 rollbacks successfully, T2 can take shared lock on Row1.*`

## Rigorous 2PL

Rigorous 2PL takes a more conservative approach by having a transaction maintain all its locks, both exclusive and shared, until it either commits or aborts. This approach decreases the chances of deadlocks since resources are more strictly controlled, but it also lowers the level of concurrent operations as both shared and exclusively locked resources are held for the duration of the transaction.

**Example**

1. `*T1 starts and writes to Row 1.*`
2. `*T1 acquires a shared lock on Row 2.*`
3. `*T2 starts and try to acquire shared lock on Row 1 but since it is exclusively locked for writes by T1, T2 has to wait*`
4. `*T3 starts and try to acquire write lock on Row 2 but since it is shared locked by T1, T2 has to wait*`
5. `*T1 encounters a problem and rolled back, undoing the changes to Row 1.(Here it releases the shared lock on Row 2 and exclusive lock on Row1 after rollback)*`
6. `*At this step, the database doesn’t need to rollback transactions T2 and T3 because they didn’t read the uncommitted data from T1.**(No cascading effect)***`
7. `*Now T2 can read Row1 and T3 can take exclusive lock on row 2 once T1 rollbacks successfully.*`

These both types are still prone to deadlocks during the locking phase which can be seen from the following example

1. `*T1 starts and locks Row 1 exclusively.*`
2. `*T2 starts and locks Row 2 exclusively.*`
3. `*T1 tries to lock Row 2 but has to wait because T2 holds the lock.*`
4. `*T2 tries to lock Row 1 but has to wait because T1 holds the lock.*`

## Conservative/Static 2PL

Static Two-Phase Locking, or Conservative 2PL, requires transactions to lock all necessary data items exclusively at the start. If a transaction can’t immediately lock all its needed items, it will wait until everything is available. Once it secures all locks, the transaction can proceed, and it can release locks without the stricter constraints found in Strict or Rigorous 2PL.

Note that there is no restriction on releasing or unlocking data items in Static 2PL as we had in Strict or Rigorous 2-PL. While this method ensures a deadlock-free sequence of transactions, it doesn’t eliminate the possibility of cascading rollbacks, unlike Strict or Rigorous 2PL. Therefore, despite its deadlock prevention, Static 2PL doesn’t guarantee strict scheduling, which is a disadvantage in comparison to Strict and Rigorous 2-PL.

**Example**

1. `*T1 acquires lock on Row1 and Row2*`
2. `*T1 reads row1 and writes Row2.*`
3. `*T1 unlocks Row2.*`
4. `*T2 starts and try to acquire lock on Row 2 and reads it(Which is a dirty read)*`
5. `*T1 encounters a problem and rolled back, undoing the changes to Row 1 and releasing all locks*`
6. `*At this step, T2 also has to rollback transaction as dirty read can destroy data integrity*`

# Optimistic Locking

Here, conflicts are assumed to be rare, and transactions are allowed to proceed without acquiring locks preemptively. Instead of locking resources to prevent conflicts, optimistic locking relies on detecting conflicts after transactions have completed and resolving them if they occur. 

## MVCC

MVCC is a strategy that allows multiple users to access a database concurrently without interference from other transactions. PostgreSQL implements MVCC by creating a snapshot of the database state/version at the start of each transaction. This snapshot provides a consistent view of the data as it existed at the moment the transaction began, regardless of subsequent changes made by others.

The main benefit of the MVCC model over traditional locking for concurrency control is that MVCC allows queries to access data without conflicting with write operations. This means

***“Readers never block writers and writers never block readers”***

### Differences between 2PL and MVCC

| **Aspect** | **Multi-Version Concurrency Control (MVCC)** | **Two-Phase Locking (2PL)** |
| --- | --- | --- |
| Locking Mechanism | Uses multiple versions of data to manage concurrent access without locking. | Uses locks on data items to manage access, blocking other transactions until the lock is released. |
| Performance in High-Concurrency | Typically performs better as it avoids locking, reducing contention. | May suffer from lock contention, reducing performance under high concurrency. |
| Handling Read Operations | Read operations do not usually block writes, as each transaction sees a snapshot of data. | Read operations require shared locks, which can block other transactions. |
| Write Operations | Generates new versions of data, allowing concurrent transactions to proceed without waiting. | Holds locks until the work is completed**(check 2PL types)**, potentially delaying concurrent transactions. |
| Phases of Operation | Continuously creates new data versions as transactions proceed, without distinct phases. | Operates in two phases – acquiring locks and then releasing them. |
| Isolation Level Management | Managed by controlling which data version is visible to a transaction using snapshot,commit logs, hintbits and other visibility rules. | Managed through the granularity and duration of locks. |
| Risk of Deadlocks | Lower risk as it avoids locking data items. | Higher risk due to the locking of data items. |
| Overhead | Overhead from maintaining multiple versions of data and periodic cleanup processes like vacuuming. | Significant resource usage for managing locks, particularly in highly concurrent databases. However, since it doesn’t create new rows, there’s no requirement for additional cleanup. |