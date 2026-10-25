# System Design

[Writing Good APIs](System%20Design/Writing%20Good%20APIs.md)

[Microservices](System%20Design/Microservices.md)

[Twitter](System%20Design/Twitter.md)

[_The Dual Write Anti-Pattern](_The%20Dual%20Write%20Anti-Pattern.md)

![Untitled](System%20Design/Untitled.png)

# High Level Concepts To Learn About

Lock yourself in a room a learn these 5 programming concepts (you'll thank me later):

1. SOLID Principles

The 5 principles of object-orientated programming that creates clean, maintainable and scalable code.

2. Domain-Driven Design

Improve your understanding of complex business domains and learn to model them using DDD principles.

3. Clean Architecture

Structure your code in a way that enforces separation of concerns, promotes testability and ensures your application remains flexible.

4. Test-Driven Development

Write more reliable and maintainable code. Write tests before you write the code to help ensure your code meets the required specifications and reduces the likelihood of bugs.

5. Continuous Integration & Continuous Delivery

Streamline your software development process, enabling faster delivery through automated builds, tests and deployment.

# Books

[https://www.audible.in/pd/Designing-Data-Intensive-Applications-Audiobook/B08VKM4CWP?https%3A%2F%2Fwww.audible.in%2F%3F&source_code=AUDTM13403072200B3&gclid=CjwKCAiA0cyfBhBREiwAAtStHAcEp7OtzU-HDvw4P_hLsJCF-z6RBQ7dwmnFcyh6tdkEFFAWO0DfDhoCyHQQAvD_BwE](https://www.audible.in/pd/Designing-Data-Intensive-Applications-Audiobook/B08VKM4CWP?https%3A%2F%2Fwww.audible.in%2F%3F&source_code=AUDTM13403072200B3&gclid=CjwKCAiA0cyfBhBREiwAAtStHAcEp7OtzU-HDvw4P_hLsJCF-z6RBQ7dwmnFcyh6tdkEFFAWO0DfDhoCyHQQAvD_BwE)
This is apparently the bestest ever book that covers all the essentials and more.

# Free Resources

Look at ByteByteGo’s infographics they are amazing.

[Quastor](http://quastor.org/) - Hat tip to the ExperiencedDevs subreddit for this rec. It's a free email newsletter that sends out summaries of engineering blog posts (usually from big tech companies). They had a good post recently on why/how khan academy rewrote their backend from python to go.

[System Design Interview YouTube Channel](https://www.youtube.com/c/SystemDesignInterview) - Unfortunately it doesn't seem like this channel is active anymore, but it has really awesome videos on System Design where they walk you through how to solve a question. Their step by step guide is a must watch.

[System Design Primer Repo on GitHub](https://github.com/donnemartin/system-design-primer) - I assume everyone already knows this resource but I’ll just link it anyway. This is an amazing repo that goes through all the core concepts of system design interviews and provides tons of links for additional reading. If you’re completely new, this is should be your starting point.

Paid Resources

[Designing Data Intensive Applications by Martin Kleppmann](https://dataintensive.net/) - This is an amazing book to get a high level overview of different data-related technologies (SQL vs. NoSQL dbs, OLAP vs. OLTP, data lakes vs. warehouses, and a bunch more). You can view a very detailed summary of the book’s content here but I’d highly recommend purchasing it and giving it a read.

*Taken from this previous [post](https://www.reddit.com/r/cscareerquestions/comments/s0e54f/warning_systems_expert_by_the_algoexpert_guy_is/)*

I can also add [Hussein Nasser's YouTube channel](https://www.youtube.com/c/HusseinNasser-software-engineering). He produces great content on backend engineering.

[https://www.reddit.com/r/cscareerquestions/comments/ufhjjk/what_are_the_best_resources_to_study_system_design/](https://www.reddit.com/r/cscareerquestions/comments/ufhjjk/what_are_the_best_resources_to_study_system_design/)

Read Quastor

[https://github.com/donnemartin/system-design-primer](https://github.com/donnemartin/system-design-primer)

[https://dataintensive.net/](https://dataintensive.net/)

[https://www.youtube.com/c/SystemDesignInterview](https://www.youtube.com/c/SystemDesignInterview)

[https://www.youtube.com/c/HusseinNasser-software-engineering](https://www.youtube.com/c/HusseinNasser-software-engineering)

[https://www.reddit.com/r/ExperiencedDevs/comments/nsssv9/what_are_the_best_resources_to_crack_system/](https://www.reddit.com/r/ExperiencedDevs/comments/nsssv9/what_are_the_best_resources_to_crack_system/)

[https://www.youtube.com/watch?v=2BWr0fsDSs0&ab_channel=sudoCODE](https://www.youtube.com/watch?v=2BWr0fsDSs0&ab_channel=sudoCODE)

# Case Studies

## Idempotency Tokens

[https://www.youtube.com/watch?v=IP-rGJKSZ3s&ab_channel=TomScott](https://www.youtube.com/watch?v=IP-rGJKSZ3s&ab_channel=TomScott)

## Design A Parking Garage System

[https://www.youtube.com/watch?v=NtMvNh0WFVM&ab_channel=Exponent](https://www.youtube.com/watch?v=NtMvNh0WFVM&ab_channel=Exponent)
This guy made a parking booking system, here are the things I learned from this

- It’s totally fine to use RDBMS if that’s what you’re comfortable with. PostGres MySQL is good.
- Think of what datatypes would be best for what columns.
- Think of what would be public and internal endpoints, what are the security or abuse concerns?
- Think about any race conditions in the database that you’re designing
- Define what would be the primary keys and secondary keys in your database
- What is the trade-off with adding enums in SQL? You get all the benefits of using an int in terms of integer. But you get flexibility issues; in PostGres it’s hard to delete enums from a table and things. Look up how enums work in SQL.
- Think if your DB query params are read or write heavy, setup read replicas if they are read heavy.
- Think of consistency; do we want our data to be strongly or weakly consistent? Think of tradeoff with latency. So if you put read lock on a read heavy database for all writes, will that be reasonable?
- Can you read lock based on a particular column or a shard?
- Think of what columns you can use to shard your database.

## What Technologies To Use When

### Cache

[Cache](System%20Design/Cache.md)

Options: Redis, Memcache
Use: Redis (Redis is a modern version of Memcache). Redis understands data structures like list etc, so performing list append is faster in Redis wrt Memcache which sees all values as a blob.

### File storage

Why? To store images, videos, files etc. These are blob storages. DBMS is used when we have to perform query on the data. But a file is something we just serve.
Options: S3

### Text search engine

Netflix: search for a movie
Amazon: search for a product
Uber: search for a location
Options: Elastic Search, Solr (build over Lucene)
Important: They are search engines, not databases. They give availability and redundancy but no guarantee on data, so data may be lost. Store critical data somewhere else.

### Fuzzy text search

Why? User may type wrong spelling. For eg, instead of Airport, may type Ariport.
Options: Elasticsearch, Solr

### Timeseries database

Why? For metrics
Options: InfluxDB, OpenTSDB 
Note: These are kinds of RDBMS with some customization. In it, a user will write always in append-only mode and won't edit old data. Also, queries will be bulk queries on a time range. So, optimized for that.

### Data Warehouse/ Big Data

Why? Want to store huge data for analytics. Eg for Amazon, which country gives more revenue, which geography has more transactions, which product is sold where most etc.
Options: Hadoop
Note: This is more for offline processing.

### RDBMS

Why? When information is structured. And when we need ACID guarantees. For eg, in banking, an amount is debited but not credited, different account balance for different queries etc
Options: MySQL, PostgreSQL, Oracle

### NoSQL (Document DB)

Why? For Amazon, building catalogue. A catalogue can have different types of items with different attributes. Eg, T-shirt, Washing machine, Refrigerator, Milk, Medicine etc. This can be stored as JSON in RDBMS but NoSQL DBs are designed to handle such kinds of queries in a more optimized manner.
Options: MongoDB, CouchBase

### NoSQL (Columnar DB)

Why? When data is ever increasing. Eg, Uber drivers keep sending their location data every few minutes. And if drivers keep increasing, data will keep growing exponentially. But queries will be low only on this huge data, like what locations for a given driver id.
Options: Cassandra, Hbase