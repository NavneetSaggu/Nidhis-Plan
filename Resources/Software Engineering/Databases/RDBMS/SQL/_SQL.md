# SQL

[SQL Indexes (Keys)](SQL/SQL%20Indexes%20(Keys)%2051aa88812ba94026b9a36c9cbcb087ed.md)

[Composite Indexes](SQL/Composite%20Indexes.md)

[Views](SQL/Views.md)

[Niche Queries](SQL/Niche%20Queries.md)

[_SQL Locks](_SQL%20Locks.md)

[Multi Master Replication](SQL/Multi%20Master%20Replication.md)

[Logging](SQL/Logging.md)

[Conflict Resolution](SQL/Conflict%20Resolution.md)

# Basics

## **Joins**

![SQLJoins.png](SQL/SQLJoins.png)

- **Inner join**: Rows common in both T1 and T2
- **Left outer join**: All rows in T1
- **Right outer join**: All rows in T2
- **Full join**: All rows in T1 and T2
- **Cross join**: All rows in T1 * all rows in T2

## **Keys**

- **Primary key**: Uniquely identify the row. Cannot be null.
- **Candidate key**: Can be chosen as primary key
- **Composite key**: Combination of multiple columns
- **Foreign key**: Column referencing other table's primary key

## **Indexes**

- Improves speed of data retrieval.
- Primary and foreign keys are indexed by default.
- **Non-clustered index**: Rows are unordered (stored in heap). While index is stored separately as sorted column.
- **Clustered index**: The rows themselves are ordered (thus there can be only 1 clustered index per table).
- **Composite index**: Index on multiple columns (c1,c2,c3). Order of columns should match the where clauses for maximum efficiency.
- **Cardinality**: Uniqueness of rows (eg: PassportID vs Gender column values).
- **Bitmap-index**: Used when cardinality is very low (eg: Gender column).
- **Data structures used**: Bit arrays, hashmaps or B+Trees (most common).

## **Queries**

- **Union**: Merges content of 2 structurally compatible tables
- **Group By**: Used to aggregate (avg, sum, count) values. Aggregate column should be same as group-by column.
- **Having**: Adding where clauses on top of group-by.
- **Explain:** Clause can be added to see how query was executed (query plan)

## **Frequently asked queries**

- Find all departments with sales more than 1000
    
    ```sql
    SELECT department, SUM(sales) AS "Total sales" FROM order_details	
    GROUP BY department
    HAVING SUM(sales) > 1000;
    ```
    
- Print all employee ids with their manager ids
    
    ```sql
    SELECT e1.emp_id, e1.emp_mgr_id 
    FROM employee e1 LEFT JOIN employee e2 
       ON e1.emp_mgr_id = e2.emp_id
    ```
    
- Print all manager names with count of direct reportees
    
    ```sql
    SELECT e2.ename, count(e1.ename) 
    FROM employee_s e1 LEFT JOIN employee_s e2 
      ON e1.manager_id = e2.eid 
    group by e2.ename;
    ```
    
- Print 10th highest salary
    
    ```sql
    SELECT Salary FROM
    (  SELECT DISTINCT Salary FROM Employee ORDER BY Salary DESC LIMIT 10 ) 
    AS Emp ORDER BY Salary LIMIT 1;
    ```
    

# Data Types

## Timestamp

You can use the `DATE()` function to extract the date portion of the timestamp:

```sql
SELECT * FROM table
WHERE DATE(timestamp) = '2012-05-25'

```

If you have an index on the timestamp column, this would be faster because it could utilize the index.

```sql
SELECT * FROM table
WHERE timestamp BETWEEN '2012-05-25 00:00:00' AND '2012-05-25 23:59:59'
```

Remember to put quotes around date strings

# **Resources**

- [Cheat sheet](http://files.zeroturnaround.com/pdf/zt_sql_cheat_sheet.pdf)
- [Clustered vs Non-clustered indexes](https://docs.microsoft.com/en-us/sql/relational-databases/indexes/clustered-and-nonclustered-indexes-described)
- [DB Index Wiki](https://en.wikipedia.org/wiki/Database_index)