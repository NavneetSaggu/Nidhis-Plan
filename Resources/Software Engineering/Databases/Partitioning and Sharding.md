# Partitioning and Sharding

A **partition** is a division of a logical database or its constituent elements into distinct independent parts. Database **partitioning** is normally done for manageability, performance or availability reasons, as for load balancing.

[https://en.wikipedia.org/wiki/Partition_(database)](https://en.wikipedia.org/wiki/Partition_(database))

**Sharding** is a type of partitioning, such as **Horizontal Partitioning** (HP)

There is also **Vertical Partitioning** (VP) whereby you split a table into smaller distinct parts. Normalization also involves this splitting of columns across tables, but vertical partitioning goes beyond that and partitions columns even when already normalized.

[https://en.wikipedia.org/wiki/Shard_(database_architecture)](https://en.wikipedia.org/wiki/Shard_(database_architecture))

I really like Tony Baco's answer on Quora where he makes you think in terms of schema (rather than columns and rows). He states that...

"**Horizontal partitioning**", or sharding, is replicating [copying] the schema, and then dividing the data based on a shard key.

"**Vertical partitioning**" involves dividing up the schema (and the data goes along for the ride).

[https://www.quora.com/Whats-the-difference-between-sharding-DB-tables-and-partitioning-them](https://www.quora.com/Whats-the-difference-between-sharding-DB-tables-and-partitioning-them)

Oracle's Database Partitioning Guide has some nice figures. I have copied a few excerpts from the article.

[https://docs.oracle.com/cd/B28359_01/server.111/b32024/partition.htm](https://docs.oracle.com/cd/B28359_01/server.111/b32024/partition.htm)

**When to Partition a Table**

Here are some suggestions for when to partition a table:

- Tables greater than 2 GB should always be considered as candidates for partitioning.
- Tables containing historical data, in which new data is added into the newest partition. A typical example is a historical table where only the current month's data is updatable and the other 11 months are read only.
- When the contents of a table need to be distributed across different types of storage devices.

**Partition Pruning**

Partition pruning is the simplest and also the most substantial means to improve performance using partitioning. Partition pruning can often improve query performance by several orders of magnitude. For example, suppose an application contains an Orders table containing a historical record of orders, and that this table has been partitioned by week. A query requesting orders for a single week would only access a single partition of the Orders table. If the Orders table had 2 years of historical data, then this query would access one partition instead of 104 partitions. This query could potentially execute 100 times faster simply because of partition pruning.

**Partitioning Strategies**

- Range
- Hash
- List

You can read their text and visualize their images which explain everything pretty well.

And lastly, it is important to understand that databases are extremely resource intensive:

- CPU
- Disk
- I/O
- Memory

Many DBA's will partition on the same machine, where the partitions will share all the resources but provide an improvement in disk and I/O by splitting up the data and/or index.

While other strategies will employ a "shared nothing" architecture where the shards will reside on separate and distinct computing units (nodes), having 100% of the CPU, disk, I/O and memory to itself. Providing it's own set of advantages and complexities.