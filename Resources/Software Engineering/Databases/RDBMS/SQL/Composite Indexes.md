# Composite Indexes

Resource: [https://www.mysqltutorial.org/mysql-index/mysql-composite-index/#:~:text=MySQL allows you to create,two columns%2C and so on](https://www.mysqltutorial.org/mysql-index/mysql-composite-index/#:~:text=MySQL%20allows%20you%20to%20create,two%20columns%2C%20and%20so%20on).

Composite Index can consist of up to 16 columns.

Composite indexes are used for 

- queries that test all columns in the index
- queries that test the first columns, the first two columns, and so on.

If you specify the columns in the right order in the index definition, a single composite index can speed up many kinds of queries.

# Creating Index

```sql
CREATE TABLE table_name (
    c1 data_type PRIMARY KEY,
    c2 data_type,
    c3 data_type,
    c4 data_type,
    INDEX index_name (c2,c3,c4)
);

# OR

CREATE INDEX index_name 
ON table_name(c2,c3,c4);
```

# Using Index

All these queries would use the index

```sql
SELECT * FROM table_name WHERE c1 = v1; 
SELECT * FROM table_name WHERE c1 = v1 AND c2 = v2; 
SELECT * FROM table_name WHERE c1 = v1 AND c2 = v2 AND c3 = v3;
```

**WARNING Index won’t work if the columns do not form a leftmost prefix of the index; for example, in this case:**

```sql
SELECT * FROM table_name WHERE c1 = v1 AND c3 = v3;
```