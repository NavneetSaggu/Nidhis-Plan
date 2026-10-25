# MongoDB

# Download & Setup

Download formula for mongdb 

`brew tap mongodb/brew`

Update brew formulae

`brew update`

Install MongoDB

`brew install mongodb-community@7.0`

## What comes with?

The installation includes the following binaries:

- The [`mongod`](https://www.mongodb.com/docs/manual/reference/program/mongod/#mongodb-binary-bin.mongod) server
- The [`mongos`](https://www.mongodb.com/docs/manual/reference/program/mongos/#mongodb-binary-bin.mongos) sharded cluster query router
- The MongoDB Shell, [`mongosh`](https://www.mongodb.com/docs/mongodb-shell/#mongodb-binary-bin.mongosh)

## Where are the files?

To find exact file paths `brew --prefix`

|  | **Apple Silicon Processor** |
| --- | --- |
| [`configuration file`](https://www.mongodb.com/docs/manual/reference/configuration-options/) | `/opt/homebrew/etc/mongod.conf` |
| [`log directory`](https://www.mongodb.com/docs/manual/reference/configuration-options/#mongodb-setting-systemLog.path) | `/opt/homebrew/var/log/mongodb` |
| [`data directory`](https://www.mongodb.com/docs/manual/reference/configuration-options/#mongodb-setting-storage.dbPath) | `/opt/homebrew/var/mongodb` |

## How to start?

`brew services start mongodb-community@7.0`

# **Improving Performance with MongoDB**

### Challenges with Relational Databases:

1. **Scaling Limitations:**
    - Scaling with relational databases involves buying larger servers, but this becomes impractical when there's no larger server available.
    - Spreading databases across two servers, especially for active/active clusters, is challenging in traditional databases like MySQL, PostgresSQL, and even Oracle.
2. **Active/Active Cluster Challenges:**
    - Active/active clusters face difficulties in maintaining consistency and synchronizing data updates between servers.
    - Traditional relational databases struggle with queries when data is distributed across multiple servers.

### MongoDB's Innovative Approach:

1. **BSON Document Storage:**
    - MongoDB stores data in BSON documents, ensuring self-contained data without complex relationships.
    - In contrast, CouchDB uses JSON.
    - Each document contains all the necessary information, allowing easy distribution across multiple servers.
2. **Linear Scalability:**
    - MongoDB's query approach, looking for specific keys and values in documents, enables almost linear scalability.
    - The data can be spread across numerous servers, and each server independently processes queries for its content.
3. **Sharding for Scalability:**
    - MongoDB doesn't provide master/master replication but offers sharding to split data across multiple machines.
    - Sharding allows each machine to handle updates for different dataset portions, facilitating scalability.
4. **Cost-Efficient Scaling:**
    - MongoDB's design allows for scaling to potentially hundreds of machines without the need for expensive solutions like Oracle's Real Application Clusters.

In summary, MongoDB's BSON document storage and sharding capabilities provide a scalable and efficient solution, overcoming the challenges associated with active/active clusters in traditional relational databases.

# Concepts in SQL and MongoDB

Before we compare SQL terminology to MongoDB terminology, let us first understand some basic concepts of SQL and MongoDB.

| Field | This is a **name and value** pair. |
| --- | --- |
| Document | A group of fields is called a document. |
| Collection | A group of documents clubbed together, is called a collection. |
| Database | A physical container to keep multiple collections is called a database. |

Now, let us see how these MongoDB concepts match up with SQL terminology.

| **SQL Terminology** | **MongoDB Terms** |
| --- | --- |
| Database | Database |
| Table | Collection |
| Row | Document |
| Column | Field |
| Index | Index |
| Primary Key | Primary Key |
| Aggregation | Aggregation Pipeline |
| SELECT INTO NEW_TABLE | $out |
| MERGE INTO TABLE | $merge |
| table joins | $lookup, embedded document |

# General Features

1. Generating Keys
    - In MongoDB, a document represents the unit of storage, equivalent to a row in an RDBMS.
    - Documents can store complex information like lists, dictionaries, and nested structures.
    - Unlike a fixed row in a traditional database, a MongoDB document can have any number of keys and values.
    - A key in MongoDB is similar to a column name in an RDBMS and is used to reference data within the document.
    - MongoDB requires each document to have a unique identifier called **`_id`**; if not provided, MongoDB generates a unique value.
    - There's a debate on whether to use a unique key provided by the database or generate one manually; the trend favors using the database-provided key for uniqueness and independence.
    - MongoDB can adapt well to changes in human-created unique numbers, unlike manually generated keys.
    - Developers' preferences vary regarding using default keys or creating their own; the choice often depends on personal preference.
    - Ultimately, users must decide whether to use a confident and likely unchanged key or rely on the default key provided by MongoDB.
2. **Flexibility of MongoDB:**
    - MongoDB is inherently flexible; collections can contain diverse items.
    - While items within a collection don't have to be similar, practical benefits arise when dealing with indexing and advanced queries if there is some schema to a collection.
3. **Dynamic Collection Creation:**
    - Collections are effectively created on demand in MongoDB.
    - A collection is created when attempting to save a document that references it.
4. **Dynamic Application Behavior:**
    - Dynamic collection creation enables the development of highly dynamic applications.
    - MongoDB allows on-the-fly creation of indexes and execution of other database-level commands.
5. **BSON Usage**
    - MongoDB uses BSON for document-oriented storage.
    - BSON is a binary form of JSON but may not always be more space-efficient.
    - BSON is designed to make data processing faster and supports quick traversal and indexing.
    - MongoDB prioritizes speed over space efficiency.
    - BSON is easy to convert to a programming language's native data format, simplifying code and aligning with MongoDB's goals.
    - BSON provides extensions to JSON, allowing storage of binary data and specific datatypes.
    - BSON is not likely to significantly impact how you use MongoDB in your everyday work.
6. **Dynamic Queries**
    - MongoDB supports dynamic queries, allowing running queries without preplanning.
    - CouchDB, a competitor to MongoDB, does not support dynamic queries; it uses a static query approach.
    - CouchDB is schemaless with dynamic data but static queries defined in advance (the opposite of an RDBMS)
    - CouchDB recommends predefined queries for production to avoid poor performance.
    - CouchDB allows writing queries as map and reduce functions, but it has a steep learning curve.
    - MongoDB is easier to use, allowing users to provide document parts for matching, and it handles the rest.
    - MongoDB supports advanced features like map and reduce functions, but users can ease into using them gradually.
7. **Indexing:**
    - MongoDB offers extensive support for document indexing, crucial for managing large document collections.
    - Indexing allows MongoDB to quickly locate relevant documents instead of scanning each one.
    - All documents in MongoDB are automatically indexed on the **`_id`** key, ensuring uniqueness.
    - Users can create custom indexes and decide whether to enforce uniqueness, with caution to avoid accidental data deletion.
    - MongoDB supports indexing on keys like lastname and can handle duplicates, useful for practical scenarios.
    - Indexing is not limited to top-level keys; it can be applied to embedded documents and supports composite indexes on multiple keys for efficient queries.
8. **Query Profiler:**
    - Includes a built-in profiling tool to analyze how it determines which documents to return.
    - The tool is useful for identifying query performance issues and suggests improvements, often through the addition of indexes.
    - The MongoDB Profiler provides valuable information, especially for complex queries experiencing slow performance.
9. **Writing In Place:**
    - Many databases, including MongoDB, use multi-version concurrency control (MVCC) for updating rows or documents.
    - MVCC allows multiple users to see different versions of data during transactions to prevent changes partway through.
    - MongoDB updates information in-place, allowing updates wherever the data is, without allocating extra space or touching indexes.
    - MongoDB performs lazy writes, reducing disk I/O by writing to disk once every second, compared to the more frequent writes in CouchDB.
    - CouchDB guarantees safe and quick data writes to disk but impacts performance significantly.
    - MongoDB's approach makes it faster, especially for scenarios with frequent updates, but it doesn't guarantee data safety on disk.
    - For critical data like billing or accounts receivable, a traditional RDBMS may be a better solution due to guaranteed data safety.
10. **Storing Binary Data:**
    - GridFS is MongoDB's solution for storing binary data in the database.
    - BSON can save up to 4MB of binary data in a document.
    - GridFS is suitable for storing large binary files like movie clips, high-quality audio, or files several hundred megabytes in size.
    - GridFS stores file metadata in the "files" collection and breaks down data into chunks stored in the "chunks" collection.
    - Storing data using GridFS is easy, scalable, and supports range operations for retrieving specific parts of a file.
    - GridFS is typically used through MongoDB drivers in programming languages for ease of use.
    - Designed for speed and scalability, MongoDB with GridFS is suitable for working with large data files.
11. **Replicating Data:**
    - MongoDB lacks certain data storage guarantees present in RDBMS databases for specific reasons.
    - Implementing these features would slow down the database, increase program complexity, and might not prevent data loss due to hardware failures.
    - MongoDB emphasizes data safety, and initially used master-slave replication for a safety net.
    - Master-slave replication has been deprecated and replaced with replica sets in MongoDB.
    - Replica sets have one primary server (similar to a master) handling write requests, ensuring proper write handling.
    - Write operations are logged in the primary's 'oplog,' which is replicated by secondary servers to stay up-to-date.
    - In case of the primary server failure, one of the secondaries takes over as the primary to handle client write requests.
12. **Implementing Sharding:**
    - Auto-sharding is a significant feature in MongoDB for large-scale deployments.
    - MongoDB manages data splitting and recombination automatically in auto-sharding scenarios.
    - Auto-sharding ensures data goes to the right server and optimizes query execution efficiently.
    - Developers experience no difference when interacting with a MongoDB database with multiple shards.
    - Auto-sharding is not yet production-ready, but when implemented, it will enhance MongoDB's scalability significantly.
    - For smaller projects, a single MongoDB instance might be sufficient, but for scalable projects like Facebook or Amazon, auto-sharding is crucial.
13. **Map-Reduce Support:**
    - MapReduce can be intimidating for some, requiring a different way of thinking about data.
    - CouchDB relies solely on MapReduce, contributing to its steep learning curve.
    - MongoDB doesn't mandate the use of map and reduce functions; it provides a simple querying syntax similar to MySQL.
    - MongoDB offers map and reduce functions in JavaScript for users who want them, running on the server.
    - The map function finds documents meeting specific criteria, and the results are processed by the reduce function.
    - The reduce function typically returns a new document with derived information.
    - If SQL GROUP BY is needed, map and reduce functions are suitable tools in MongoDB.
    - MongoDB's map and reduce functions are not poor imitations of CouchDB; they can be used for various tasks instead of MongoDB's innate query support.

# Installation

## What apps come with?

| --bsondump | Reads contents of BSON-formatted rollback files. |
| --- | --- |
| --mongo | The database shell. |
| --mongod | The core database server. |
| --mongodump | Database backup utility. |
| --mongoexport | Export utility (JSON; CSV; TSV); not reliable for backup. |
| --mongofiles | Manipulates files in GridFS objects. |
| --mongoimport | Import utility (JSON; CSV; TSV); not reliable for recoveries. |
| --mongooplog | Pulls oplog entries from another mongod instance. |
| --mongoperf | Check disk I/O performance. |
| --mongorestore | Database backup restore utility. |
| --mongos | Mongodb sharding routerprocess. |
| --mongosniff | Sniff/traces MongoDB database activity in real time; Unix-like systems only. |
| --mongostat | Returns counters of database operation. |
| --mongotop | Tracks/reports MongoDB read/write activities. |
| --mongorestore | Restore/import utility. |

## mongod

- **`mongod`** starts the MongoDB service or daemon.
- Various flags can be set when launching **`mongod`** for different configurations.
- Flags include specifying the database path (--dbpath), showing version information (--version), and printing diagnostic system information (--sysinfo).
- The full list of options is available with the --help flag.
- Defaults can be used by typing **`mongod`** in the shell or command prompt to start the service.

# Data Model

- MongoDB has various types of collections, with the default collection being expandable.
- Capped collections in MongoDB have a size limit, replacing the oldest document with newer ones.
- Every collection has a unique name starting with a letter, possibly an underscore when created.
- Collection names can include numbers and letters but cannot use $ symbol, empty string, start with "system," or contain null characters.
- Recommended to keep collection names simple and short, with a maximum of 118 characters minus database and period characters.
- The combination of database name and collection name is called a namespace. (databasename.collectionname)
- A database has a default limit of 24,000 namespaces, with each collection using at least two.
- Each collection accounts for one namespace for itself and one for the default _id index, with additional namespaces for more indexes. If your collection's primary namespace is "myDB.myCollection," the namespace for the **`_id`** index would be "myDB.myCollection.id."
- This allows up to 12,000 collections per database, assuming one index per collection.
- The limit on namespaces can be increased by providing the nssize parameter when executing the MongoDB service application (mongod).

## Embedding vs Referencing Documents

- **Embedding vs. Referencing:**
    - **Embedding:** Placing data (e.g., arrays) directly into the document itself.
    - **Referencing:** Creating a reference to another document containing specific data.
- **Relational vs. MongoDB Approach:**
    - In relational databases, multiple tables are often needed for related information.
    - MongoDB prefers embedding data for efficiency, tidiness, and faster access.
- **Example: Storing CD Data:**
    - **Relational Structure:**
        
        ```java
        |_media (DB)
        	 |_cds (TABLE)
        			 |_id, artist, title, genre, releasedate (SCHEMA)
        	 |_ cd_tracklists (TABLE)
        			 |_cd_id, songtitle, length (SCHEMA)
        ```
        
    - **Nonrelational Structure:**
        
        ```java
        |_media
        	 |_items
        			 |_<document>
        
        {
          "Type": "CD",
          "Artist": "Nirvana",
          "Title": "Nevermind",
          "Genre": "Grunge",
          "Releasedate": "1991.09.24",
          "Tracklist": [
            {
              "Track": "1",
              "Title": "Smells Like Teen Spirit",
              "Length": "5:02"
            },
            {
              "Track": "2",
              "Title": "In Bloom",
              "Length": "4:15"
            }
          ]
        }
        ```
        
- **Efficiency of Embedding:**
    - MongoDB loads all CD information from one document, avoiding multiple queries.
    - Embed data whenever possible for efficiency.
- **Creating References in MongoDB:**
    - Can be done manually or automatically using the DBRef specification.
    - More flexibility with DBRef when collections change between documents.

## The _id field

**Object Identifier (_id Key):**

- **Automatically Added:**
    - _id key added automatically to every MongoDB document.
    - First attribute in each new document, even if not specified in code.
- **Mandatory Element:**
    - _id key is mandatory for each document in the collection.
- **Default Value:**
    - If not specified manually, MongoDB assigns a special BSON datatype.
    - BSON datatype consists of a 12-byte binary value.
- **Structure of Default Value:**
    - 4-byte timestamp (seconds since epoch, January 1st, 1970).
    - 3-byte machine ID, 2-byte process ID, and 3-byte counter.
    - Stored in Big Endian format.
    - This, all keys are in increasing order.
- **Big Endian and Little Endian:**
    - Big Endian: Most significant value saved first.
    - Little Endian: Least significant value saved first.
- **Supported by Drivers:**
    - All MongoDB drivers support this special BSON datatype.
- **Creation Using ObjectId():**
    - Drivers and MongoDB shell support invoking ObjectId() for _id key creation.
    - Optionally, you can specify your own value for _id.