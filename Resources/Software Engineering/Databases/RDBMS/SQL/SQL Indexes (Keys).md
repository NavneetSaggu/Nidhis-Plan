# SQL Indexes (Keys)

Check these videos for indexes (35 onwards [https://www.youtube.com/watch?v=NGslt99VOCw&list=PL08903FB7ACA1C2FB&index=37&ab_channel=kudvenkat](https://www.youtube.com/watch?v=NGslt99VOCw&list=PL08903FB7ACA1C2FB&index=37&ab_channel=kudvenkat)) and look at these resources used in this slide [https://www.pragimtech.com/blog/sql-optimization/how-do-sql-indexes-work/](https://www.pragimtech.com/blog/sql-optimization/how-do-sql-indexes-work/)

If you don’t have indexes on your tables, your SQL will scan through all the rows in the table each time you execute a query.  This is called a tables can, and this is done when SQL doesn’t have the right index to help that query.

When we create an index, the db generates a method to rapidly find data  based on one or multiple columns.

# Check Indexes On Table

```sql
# Use this to check indexes on table
SHOW INDEXES FROM table_name;
# INDEX and KEYS are synonyms, so can also run 
SHOW KEYS FROM tablename IN databasename;
```

[https://www.mysqltutorial.org/mysql-index/mysql-show-indexes/](https://www.mysqltutorial.org/mysql-index/mysql-show-indexes/)

---

# Querying Without Indices

Suppose we have table

```sql
create table person {
	first_name varchar(10),
	last_name varchar(10 ),
	date_of_birth date
};
```

If this table has 100,000,000 records, running any given select query can take 4-5 seconds, since each time all the rows are scanned. Thus, all queries, even those with comparisons, take multiple seconds to run

---

# Creating Single Column Index

Suppose you create an index on firstName to speed up queries.

```sql
CREATE INDEX person_first_name_index on person (first_name);
```

This query will take a while as the db will scan all rows and index each name.

### What Queries Will Speed Up?

Now, let’s run some queries

```sql
SELECT COUNT(*) FROM person WHERE first_name = "Emma";
```

This will take only 0.5 seconds.

If we run another query, say

```sql
SELECT COUNT(*) FROM person WHERE last_name = "Smith";
```

It’ll go back to taking 4-5 seconds

Same for DoB queries. these will not speed up.

But there are still other queries where the first_name index will improve performance. 

So any query that has first_name involved in it, and narrows down the to-be-scanned rows of our database, will be sped up

---

# Creating Multi Column Index

```sql
CREATE INDEX person_first_name_last_name_idx on person (last_name, first_name);
```

Important: The order of the columns that you give your multi column index matters. Think of this as sorting the data first by last_name, and then by first_name.

It’ll now take 7 minutes to execute this.

### What Queries Will Speed Up?

```sql
SELECT COUNT(*) FROM person WHERE
	last_name = "Williams"
AND
	first_name = "John";
```

Will only take 36 milliseconds to run this query.

Indexes require extra storage; DB will need to create new records and modify all existing records to create an index.

More on Composite Indexes [Composite Indexes](Composite%20Indexes.md) 

---

# Clustered Indices

- Determines the physical order of data in a table.
- This is why table can only have one clustered index.
- Primary key constraint creates clustered indexes automatically if no clustered index already exists in the table.
- We can see data being indexed by the cluster indexed column if we run a simple select query.
- While there can only be one clustered index, we can have multiple columns in this index. Think of a telephone directory, and how the records in it are arranged by last name and then by first name.
- This will be a composite clustered index.
- Unique clustered indices are also used to enforce Primary  Key constaints in table columns.

```sql
CREATE CLUSTERED INDEX employee_last_name_first_name_idx 
ON employee (last_name, first_name);
```

Again, the order of columns given for the index matter.

# Non-Clustered Indices

- This is analogous to an index in a textbook that doesn’t necessarily have it’s chapters in order. The index is stored in a seperate place and data is stored in a seperate place. This is different from index in a telephone directory or a dictionary. In those, the data itself is arranged in an order, for books you have a seperate page telling you what is present where.
- A table can have more than one non-clustered indices.
- In the index data table itself, the data is stored in order of the index key, but this order is unrelated to how the main tables data is stored.

# Comparison Clustered vs Non-Clustered

- Clustered indices are slightly faster than a non-clustered index because non-clustered index has to refer back to the table if the selected column is not present in the index. i.e. non-clustered index will have to look at two different table, however clustered index has all the data in one single table
- Clustered doesn’t take extra space as all the data is in the table itself, non-clustered needs making an extra table.
- We can make only one clustered index, we can make multiple non-clustered indices.

# Unique Clustered Indices

Unique index is used to enforce uniqueness of key values in the index. 

By default, primary key creates unique clustered index.

Both clustered and non-clustered indices can be unique.

Unique constraint vs Unique Index: No major difference; behind the scenes of a unique constraint, we use a unique index. 

# Advantages to Indices

Speed up lookup queries

# Disadvantages to Indices

Can slow down Insert and Delete and Update statements.

Important to choose where you want to put indices and not go using them blindly.