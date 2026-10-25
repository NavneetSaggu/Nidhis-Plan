# SQL Locks

From:  [https://www.scaler.com/topics/sql/lock-table/](https://www.scaler.com/topics/sql/lock-table/)

Documentation: [https://docs.oracle.com/database/121/CNCPT/consist.htm#CNCPT020](https://docs.oracle.com/database/121/CNCPT/consist.htm#CNCPT020)

## Overview

Locks are placed on SQL Server resources **(like rows, columns, tables, etc.)** to prevent multiple transactions from using the same resource at the same time. Locks are also used to **adhere to the ACID properties** during a transaction. A Lock helps in avoiding the concurrency and the deadlock situation in a multi-user environment. In the SQL server database, locks are divided into six types, **Extent (X), Shared (S), Update (U), Intent (I), Schema (Sch), and Bulk Update (BU) lock**.

### Pre-requisites

- [What is Transaction in DBMS?](https://www.scaler.com/topics/dbms/transaction-in-dbms/)
- [ACID properties in DBMS](https://www.scaler.com/topics/dbms/acid-properties-in-dbms/)

## Scope

**This article explains**:

- How and where to use Locks in a database management system.
- Types of Locks and Locking Mechanism in **SQL**.

## Introduction

Relational Databases are used in various software sectors and a relational database should meet the ***Atomicity*, *Consistency*, *Isolation*, and *Durability* (ACID)** requirements during a transaction on the database resources.

The ACID properties include the following conditions for a successful transaction:

- **Atomicity:** It means that if any transaction on the data is performed, it should be perfectly completed or not executed at all.
- **Consistency:** Consistency refers to the condition that each transaction must modify the data in approved ways only. It means that security requirements must be followed to ensure that the database remains precise and accurate during a transaction.
- **Isolation:** This attribute ensures that all transactions are independent of one another, and keeps transactions separate until they're completely executed.
- **Durability:** This attribute ensures that the database keeps a track of the pending changes when a transaction is being executed for the server to recover from an unexpected shutdown.

Locks are used in a database to ensure that the ACID properties are followed when several **transactions are performed simultaneously** on the database resources. Consider a real-life example, when you are trying to book a movie ticket, and you are trying to take the top center seat but some other person is also trying to book the same seat at the same time as you are, then one seat will be allocated to two persons if there are no ACID properties in place and it will create problems for both users, but if an ongoing transaction follows ACID properties and has a lock placed on the resources **(like rows)** inside the table, no other transaction (user) can access or update that resource until the lock is released. It will solve the **concurrency problem** in various fields.

## Types of Resources in a SQL Server

Let us look at the types of resources where locks can be placed in a SQL server database. We place locks on **rows, keys, tables, pages, extent**, and **databases** to avoid the use of these resources by two users simultaneously.

![](SQL%20Locks/types-of-resources-in-a-sql-server.webp)

| Resource Name | Description |
| --- | --- |
| **RID** | RID refers to the Row ID, a RID helps in locating a row that contains certain information such as **slots, pages, files, etc**., and helps in the **faster identification** of certain rows. RID is used for locking a single row in the table. |
| **Key** | Key refers to a single column or a set of columns that are used to identify rows distinctively, keys make sure there aren't any rows containing duplicate data. It can be a **primary key, secondary key, or foreign key** that is used to lock a row within an index. |
| **Page** | A Page is the smallest data storage unit in the SQL server database, it refers to an **8-KB database page**. When a lock is applied to secure a page, another user cannot modify the data on the page. |
| **Extent** | Extents are the fundamental unit of space management. It is a group of 8 contiguous pages, **i.e., 64KB size**, and a lock on extent will secure **8 contiguous** pages. |
| **Table** | Tables are similar to spreadsheets in which data is logically structured in a **row-column manner**. Each column represents a field, and each row represents a distinct record of the respective field. A lock on the table will restrict the usage of all the **contents from the table**. |
| **Database** | A database is made up of a series of tables each table holds a unique set of datasets. A lock on a database prevents users with **read-only permission** to modify the data. |

## Types/Modes of Locks in a SQL Server

**There are various kinds of locks in SQL which are applied on a transaction, let's discuss them one by one below**:

![](SQL%20Locks/types-of-locks-in-a-sql-server.webp)

### 1. Exclusive Lock (X):

The exclusive locks come in handy during a **Data Manipulation Language (DML)** operation such as **INSERT**, **UPDATE**, and [**DELETE**](https://www.scaler.com/topics/delete-query-in-sql/) query. When **exclusive lock (X)** is applied to a transaction, it prohibits other people from accessing the locked resources. That is, an exclusive lock can only hold one transaction on a resource at a time. When a transaction wants to **edit the page or row data, this lock is imposed**.

**Example:**

```
BEGIN TRAN

USE LOCKS

UPDATE employees SET title = 'Software Engineer II' WHERE emp_id = 101

/* The following code block selects information about locks held by current transactions using sys.dm_tran_locks. The WHERE clause filters out locks on the database resource type. This means that only locks on other resource types such as tables, pages, rows, etc. will be returned. */

SELECT resource_type, request_mode, resource_description
FROM   sys.dm_tran_locks
WHERE  resource_type <> 'DATABASE'
ROLLBACK
```

We can see in the below image (server state) that an exclusive lock (X) is placed on the key resource type when we use the UPDATE statement in a transaction to change the title = 'Software Engineer II', where emp_id = 101.

![](SQL%20Locks/exclusive-lock.webp)

### 2. Shared Lock (S):

A shared lock (S) is applied on a resource when a transaction reads data without changing it. In a shared lock, a different transaction can read the same data, but a transaction attempting to modify the data will be **blocked, until** the shared lock from the resource is removed. Shared locks (S) are not visible as the shared locks are placed for the time till the SELECT statement is executed and are released when we get the selected data from sys.dm_tran_locks. To show the locks, an addition of WITH HOLDLOCK is required.

**Example:**

```
BEGIN TRAN

USE LOCKS

SELECT * FROM employees WITH (HOLDLOCK)
WHERE emp_id = 102
SELECT resource_type, request_mode, resource_description
FROM   sys.dm_tran_locks
WHERE  resource_type <> 'DATABASE'
ROLLBACK
```

We can see in the below image (server state) that a shared lock (S) is placed on the key resource type when we use the SELECT statement with HOLDLOCK command in a transaction.

![](SQL%20Locks/shared-lock.webp)

### 3. Update Lock (U):

To avoid [deadlock situations](https://www.scaler.com/topics/dbms/deadlock-in-dbms/), an update lock is applied to the database resources. The update lock is a mix of an **Exclusive lock** and a **Shared lock**. When a **DML query is performed**, first the SQL Server must locate the data it wishes to edit, so an update lock is utilized to prevent lock **conversion deadlocks**. An update lock, similar to an exclusive lock, can only be kept on a single transaction, but it is **also feasible** to set a shared lock on a resource that already has an update lock. We use the UPDLOCK command to impose an update lock.

**Example:**

```
BEGIN TRAN

USE LOCKS

SELECT * FROM employees WITH (UPDLOCK)
WHERE emp_id < 102
SELECT resource_type, request_mode, resource_description
FROM   sys.dm_tran_locks
WHERE  resource_type <> 'DATABASE'
ROLLBACK
```

We can see in the below image (server state) that an update lock (U) is placed on the key resource type when we use the SELECT statement with the UPDLOCK command in a transaction.

![](SQL%20Locks/update-lock.webp)

### 4. Intent Lock (I):

This lock informs the SQL Server that a user has taken out a lock at a **lower level in the lock hierarchy**, this eliminates the need for SQL to verify all lower levels for incompatible lock types, which **improves efficiency**. It is divided into three types:

- **Intent Exclusive (IX):** If this lock is in place on a page or a row, the transaction targets to modify various lower hierarchy resources by **getting exclusive (X)** locks on those resources separately.
- **Intent Shared (IS):** If this lock is in place on a page or row, then the transaction targets to read various lower hierarchy resources by placing a shared lock (S) on those resources separately.
- **Intent Update (IU):** Intent update locks are only applied to page resources, and the lock gets converted into an exclusive lock once the update query is executed.

**Example:**

```
BEGIN TRAN

USE LOCKS

UPDATE TOP(5) employees
SET dept_no = 4WHERE title = 'Software Engineer'
SELECT resource_type, request_mode, resource_description
FROM   sys.dm_tran_locks
WHERE  resource_type <> 'DATABASE'
ROLLBACK
```

We can see in the below image (server state) that an intent **exclusive lock (IX)** is placed on the page and object resource type when we use the UPDATE statement with the TOP(5) employees command in a transaction to set the dept_no = 4 where the title is 'Software Engineer'.

![](SQL%20Locks/intent-lock.webp)

### 5. Schema Lock (Sch):

We use schema locks when a transaction to create or alter a table **(Data Definition Language statement)** is used in the SQL Server database.

- **Schema Modification (Sch-M):** It is used when a Data Definition Language statement (like **CREATE**, **ALTER**, etc.) runs, it blocks access to the locked object data while the structure of the object is changed.
- **Schema Stability (Sch-S):** This lock is applied when a schema-dependent query is built and run, and also when an execution plan is generated to secure the resources.

**Example:**

```
BEGIN TRAN

USE LOCKS

CREATE TABLE departments

SELECT resource_type, request_mode, resource_description
FROM   sys.dm_tran_locks
WHERE  resource_type <> 'DATABASE'
ROLLBACK
```

We can see in the below image **(server state)** that a Schema modification (Sch-M) is placed on the HOBT and object resource type and Schema stability (Sch-S) is placed on the METADATA resource type when we use the CREATE statement in a transaction to create a departments table.

![](SQL%20Locks/schema-lock.webp)

### 6. Bulk Update Lock (BU):

Bulk Update lock is intended to be used for bulk import processes when supplied with a **TABLOCK** argument. Other processes will be unable to access a database during the bulk load execution if a **bulk update lock** is in place. For example, when we try to insert tens of thousands of rows in a table in one go it places a **Bulk Update Lock (BU)** till the execution of the query is completed.

## Locking Hierarchy

SQL Server introduced the locking hierarchy, which is required while accessing or updating data. The lock structure starts at the top with the **database**, then, **tables** and **pages** to the **row** at the last. Locks are placed from the top to the bottom hierarchy, as this prevents the deadlock and race conditions from happening. The figure below illustrates it better:

![](SQL%20Locks/locking-hierarchy.webp)

A shared lock is always present on the database level during a transaction linked to the database. The shared lock on a database level is used to prevent the database from being dropped or a database backup from being restored over the current database. When a SELECT query is used to read data, for example, a shared lock (S) is placed on the database level, an intent shared lock (IS) is placed on the table and page level, and a shared lock (S) is placed on the row level.

**Let's look at the locking hierarchy when a user wants to read the data from the database, i.e., by using a SELECT statement**:

![](SQL%20Locks/locking-hierarchy-while-reading-data.webp)

In the case of Data Modification **(like insertion, update, deletion of data)**, a shared lock (S) is placed on the database level, an intention shared lock (IS) on the table and page, and an exclusive or update lock (X or U) on the row level is placed to prevent deadlock situation and to prevent the database from being dropped.

Let's look at the **locking hierarchy** when a user wants to **modify** the data using the **Data Manipulation Language**, i.e. **INSERT**, **UPDATE** and **DELETE** statements.

![](SQL%20Locks/locking-hierarchy-while-modifying-data.webp)

## Lock Compatibility Matrix and Examples

**At the row level, we can use the three lock options as listed below**:

- Shared (S)
- Exclusive (X)
- Update (U)

**Let's see the compatibility** of these locks with each other in the below table, i.e., which locks can we apply to a resource that has a **specific lock already in place**.

| Mode of Lock | Exclusive (X) | Shared (S) | Update (U) |
| --- | --- | --- | --- |
| Exclusive (X) | ❌ | ❌ | ❌ |
| Shared (S) | ❌ | ✅ | ✅ |
| Update (U) | ❌ | ✅ | ❌ |
- ✅ : Compatible with each other
- ❌ : Not compatible with each other

**We can interpret from the above** table that, it is feasible to set a shared lock on a resource that already has an update lock or we can't place an exclusive lock with any **other lock, etc**.

**At the table level, we can use the five lock options as listed below:**

- Shared (S)
- Intent shared (IS)
- Exclusive (X)
- Intent exclusive (IX)
- Shared with intent exclusive (SIX)

We can similarly interpret the below table that, it is not feasible to set an exclusive lock (X) with any other lock or intent shared (IS) is compatible with all the locks except the exclusive lock, etc. **Let's look at the table below:**

| Mode of Lock | Exclusive (X) | Shared (S) | Intent exclusive (IX) | Intent shared (IS) | Shared with intent exclusive (SIX) |
| --- | --- | --- | --- | --- | --- |
| Exclusive (X) | :x: | :x: | :x: | :x: | :x: |
| Shared (S) | :x: | :heavy_check_mark: | :x: | :heavy_check_mark: | :x: |
| Intent exclusive (IX) | :x: | :x: | :heavy_check_mark: | :heavy_check_mark: | :x: |
| Intent shared (IS) | :x: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| Shared with intent exclusive (SIX) | :x: | :x: | :x: | :heavy_check_mark: | :x: |
- :heavy_check_mark: : Compatible with each other
- :x: : Not compatible with each other

## Locking Mechanism

We are using MySQL database to execute the Lock mechanism. If you do not have MySQL, download it from [here](https://dev.mysql.com/downloads/mysql/) for your respective operating system.

### 1. Creating a Database

**Syntax for creating a database:**

```
CREATE DATABASE <database_name>;

USE <database_name>;

```

Creating a database named **LOCKS** in the MySQL database.

```
-- creating a LOCKS database in MySQLCREATE DATABASE LOCKS;

-- activating the LOCKS database for useUSE LOCKS;

```

**MySQL Workbench illustration of the above SQL commands**:

![](SQL%20Locks/creating-a-database-locks.webp)

### 2. Table Creation

**Syntax for creating a table:**

```
CREATE TABLE <table_name> (
    <column_name1> <data_type>,
    <column_name2> <data_type>,
    <column_name3> <data_type>,
   ...
);

```

Creating a table named **employees** in the **LOCKS** database.

```
CREATE TABLE employees (
    emp_id INT NOT NULL PRIMARY KEY,
    emp_name VARCHAR(255) NOT NULL,
    age INT,
    title VARCHAR(255),
    dept_no INT);

```

![](SQL%20Locks/creating-table-employees.webp)

### 3. Inserting Data into the Table

**Syntax for inserting a row into the table:**

```
INSERT INTO <table_name> VALUES (column1_value, column2_value, column2_value, ...);

```

Inserting values in the **employees** table.

```
INSERT INTO employees VALUES (101, "Abhishek", 22, "Software Engineer", 4);

INSERT INTO employees VALUES (102, "Yuvraj", 22, "Developer Advocate", 2);

```

![](SQL%20Locks/inserting-values-in-employees-table.webp)

### 4. View Table

**Syntax to view table in the output:**

```
SELECT * FROM <table_name>
```

Viewing the **employees** table.

```
SELECT * FROM employees;

```

![](SQL%20Locks/view-table.webp)

### 5. Applying and Removing Locks

Two types of Lock can be placed on MySQL database resources:

- **READ Lock**
- **WRITE Lock**

These locks are discussed below:

### READ LOCK in MySQL

- In a **READ** lock **(also known as a shared lock)**, the user can only read data from a table, and cannot modify the data.
- Multiple sessions can obtain a **READ** lock for a table **simultaneously**. Other sessions can also read data from the table without having to acquire the lock.
- The MySQL session with a **READ** lock in place can only read data from the table and cannot write anything until the **READ** lock is freed. Also, **other sessions will not be unable to write data** to the table. Write operations from another session will be kept in a running... state until the **READ** lock is freed.

**Syntax to apply a READ lock:**

```
LOCK TABLE <table_name> READ;

```

**Syntax to remove lock:**

```
UNLOCK TABLES;

```

**This statement will remove the locks from the table of the active database**.

Let's see how we can apply and remove a **READ** lock from the **employees** table.

First, create a MySQL connection in the MySQL workbench, connect to the MySQL database using your password, then use the below syntax to get your current MySQL session id.

```
SELECT connection_id();

```

![](SQL%20Locks/connection-id.webp)

Let's place a **READ** lock on the **employees** table now.

```
LOCK TABLE employees READ;

```

Now, let's try to **UPDATE** the title of emp_id = 101 to "Advanced Software Engineer".

```
UPDATE employees SET title="Advanced Software Engineer" where emp_id = 101;

```

MySQL throws an error as follows:

```
Error Code: 1099. Table 'employees' was locked with a READ lock and can't be updated

```

![](SQL%20Locks/update-error-with-read-lock.webp)

We can see that, we can't write/modify our table contents once we have applied a **READ** lock on the **employees** table. Let's try to modify the content of the table from a different session.

**Let's create another MySQL** connection in the MySQL workbench, again connect to the MySQL database using your password, then use the below syntax to get your current MySQL session id.

```
SELECT connection_id();

```

![](SQL%20Locks/connection-id-2.webp)

Again, let's try to run the same query.

```
UPDATE employees SET title="Advanced Software Engineer" where emp_id = 101;

```

As the first session already has a **READ** lock in place on the **employees** table, MySQL keeps the above update query in the running... state on this second session. We can't modify the **employees** table until and unless the lock is **removed from the table from session 1.**

![](SQL%20Locks/update-query-waiting.webp)

We can use **SHOW PROCESSLIST;** to get detailed information about the query. We can see this in the image below.

![](SQL%20Locks/show-process-list.webp)

Let's remove the **READ** lock from the table so that the session 2 query will be executed:

```
UNLOCK TABLES;

```

![](SQL%20Locks/unlock-table.webp)

Let's check session 2, we can see that the update query which was in running state is executed now.

![](SQL%20Locks/session-2-query-executed.webp)

### WRITE LOCK in MySQL

- In **WRITE** lock **(also known as an exclusive lock)**, the user can only write/modify and read the data of a table of the current session only.
- Other sessions can't write or read the data until the **WRITE** lock is freed.

**Syntax to apply a WRITE lock:**

```
LOCK TABLE <table_name> WRITE;

```

**Syntax to remove lock:**

```
UNLOCK TABLES;

```

This statement will **remove the locks** from the table of the active database.

Now, **let's see how we can apply** a **WRITE** lock on the **employees** table.

Create two MySQL connections in the **MySQL workbench**, and connect to the MySQL database using your password.

Let's place a **WRITE** lock on the **employees** table now in session 1.

```
LOCK TABLE employees WRITE;

```

Now, let's try to **INSERT** a value into the table.

```
-- writing data into the table session 1INSERT INTO employees VALUES (103, "Rohan", 24, "Application Engineer", 3);

-- reading data from the table session 1SELECT * FROM employees;

```

![](SQL%20Locks/write-lock-session-1.webp)

We can see that, we can write/modify and read data from the **employees** table from session 1.

Let's try to **INSERT** another value from session 2 of MySQL.

```
-- writing data into the table session 1INSERT INTO employees VALUES (104, "Anupam", 26, "Software Engineer II", 5);

-- reading data from the table session 1SELECT * FROM employees;

```

![](SQL%20Locks/running-in-write-lock.webp)

We can see that, we can't read or write data into the table from session 2 of the MySQL database once we have applied a **WRITE** lock on the **employees** table.

Let's remove the **WRITE** lock from the table so that the session 2 query will be executed:

```
UNLOCK TABLES;

```

**Let's check session 2**, we can see that the update query which was in running state is executed now and a row is inserted into the table.

![](SQL%20Locks/session-query-executed.webp)

## Conclusion

- **Locks are placed on SQL** resources to **avoid** several transactions to access the same resource simultaneously, such as rows **accessed or manipulated** by a transaction by two users at the same time.
- Locks in SQL are used to adhere to the [ACID properties in DBMS](https://www.scaler.com/topics/dbms/acid-properties-in-dbms/) during a transaction.
- Locks are applied to the database resources like **RID**, **Key**, **Page**, **Extent**, **Table**, and **Database**.
- Locks are of 6 types, **Extent (X)**, **Shared (S)**, **Update (U)**, **Intent (I)**, **Schema (Sch)**, and **Bulk Update (BU)** lock.