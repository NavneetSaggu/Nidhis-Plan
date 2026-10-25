# Databases

[Pagination (MySQL)](Databases/Pagination%20(MySQL)%20012a139bfa764280a5bc9d41866f2707.md)

[ACID](Databases/ACID.md)

[Database Concurrency](Databases/Database%20Concurrency.md)

[_Locking Implementation](_Locking%20Implementation.md)

[_RDBMS](_RDBMS.md)

[Partitioning and Sharding](Databases/Partitioning%20and%20Sharding.md)

[_Bloom Filters](_Bloom%20Filters.md)

# Categorisation

![Untitled](Databases/Untitled.png)

| Property | Relational (Confusingly Called SQL) | Non Relational (Confusingly Called NoSQL) |
| --- | --- | --- |
| Data Models | Data organised into rows & columns. 
Use structured schema with tables to represent data and define relationships between tables. | Can use various data models, including document-oriented, key-value, column-family, and graph databases |
| Schema | Have a rigid predefined schema that specifies the structure of the data, and takes effort to modify. | Are schema-less or have a flexible schema, allowing for more dynamic and adaptable data structures. |
| Query Language | Use the SQL query language. | Use query languages specific to their data model
Document-oriented databases may use JSON-like query languages 
Key-value stores use simple key-based retrieval
Can also use SQL like queries |
| Properties | Follow ACID (Atomicity, Consistency, Isolation, Durability) properties, which ensure data integrity and transaction reliability. | Follow BASE (Basically Available, Soft state, Eventually consistent) model, which prioritizes availability and scalability over strict consistency. This means that NoSQL databases may allow for eventual consistency, where data may take some time to propagate across all nodes in a distributed system. |
| Scalability | Scaling horizontally (across multiple servers) without violating ACID is a challenge and may require complex partitioning strategies.
Scaling is hard not because it’s not supported. but because of the promises that RDBMS makes about consistency. | Designed to be horizontally scalable from the outset, and therefore do not promise ACID. |
| Use Cases | Suited for applications with structured data, complex queries, and transactions, where consistency is required; such as financial systems, e-commerce platforms, and traditional relational applications. | Good fit for applications with rapidly changing data, unstructured or semi-structured data, and high scalability requirements, including social media, IoT, and content management systems. |

If your data model has a lot entities with a lot of relationships your better off with SQL imo

If the data can be stored in a **relatively** structured manner with rows and columns then use SQL.

If the data is highly unstructured then do not use SQL.

[https://www.linkedin.com/pulse/why-amazon-google-netflix-facebook-switched-nosql-shannon-block-cfe/](https://www.linkedin.com/pulse/why-amazon-google-netflix-facebook-switched-nosql-shannon-block-cfe/)

# Types Of NoSQLs

- **OLAP (Online Analytics Processing) DB**
Use cases with heavy use of filters and aggregations (slice and dice) over several dimensions is, imho, a OLAP use case.
- **Time series Database**
Use time series if the timestamp is the most important feature and you seldomly aggregate/filter over other dimensions.

> Use case for both: consider a write heavy use case with O(million) writes a day, say you’re tracking truck locations for an IOT app.
Druid is actually olap and timeseries. We're using it for years without issues. Ingest from Kafka, serve data to grafana / superset. Queried mostly using SQL. So far so good for analytics.
Push your raw data into Kafka, then use it as data lake. That way you can easily switch between databases if they don't fit your use case.
> 
- OLTP (Online Transaction Processing) DB

# Choosing The Right DB

![Untitled](Databases/Untitled%201.png)

# Choosing The Right DB