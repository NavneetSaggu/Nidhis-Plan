# ACID

- **Atomicity**:
    - Transactions are treated as single, indivisible units of work.
    - All operations within a transaction either complete successfully or are rolled back entirely if any part fails.
- **Consistency**:
    - Transactions transition the database from one consistent state to another.
    - The database maintains consistency before and after transaction execution.
    - Constraints like unique keys and foreign keys ensure data consistency.
- **Isolation**:
    - Multiple transactions can execute concurrently without interference.
    - Each transaction is isolated from others until completion.
    - Isolation prevents issues like dirty reads, non-repeatable reads, and phantom reads.
- **Durability**:
    - Committed transaction changes are permanent and survive system failures.
    - Changes are saved permanently to the database, ensuring recovery even after a system crash.