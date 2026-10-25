# Designing Data-Intensive Applications — Table of Contents
*Martin Kleppmann, O'Reilly 2017 | 585 pages*

---

## Part I: Foundations of Data Systems (p. 14)

### Chapter 1. Reliable, Scalable & Maintainable Applications (p. 15)
- Thinking About Data Systems — p. 16
- Reliability — p. 18
  - Hardware Faults — p. 19
  - Software Errors — p. 20
  - Human Errors — p. 21
  - How Important Is Reliability? — p. 22
- Scalability — p. 22
  - Describing Load — p. 23
  - Describing Performance — p. 25
  - Approaches for Coping with Load — p. 29
- Maintainability — p. 30
  - Operability: Making Life Easy for Operations — p. 31
  - Simplicity: Managing Complexity — p. 32
  - Evolvability: Making Change Easy — p. 33
- Summary — p. 34

### Chapter 2. Data Models & Query Languages (p. 38)
- Relational Model Versus Document Model — p. 39
  - The Birth of NoSQL — p. 40
  - The Object-Relational Mismatch — p. 40
  - Many-to-One and Many-to-Many Relationships — p. 44
  - Are Document Databases Repeating History? — p. 47
  - Relational Versus Document Databases Today — p. 49
- Query Languages for Data — p. 53
  - Declarative Queries on the Web — p. 55
  - MapReduce Querying — p. 57
- Graph-Like Data Models — p. 60
  - Property Graphs — p. 61
  - The Cypher Query Language — p. 63
  - Graph Queries in SQL — p. 64
  - Triple-Stores and SPARQL — p. 66
  - The Foundation: Datalog — p. 71
- Summary — p. 74

### Chapter 3. Storage & Retrieval (p. 79)
- Data Structures That Power Your Database — p. 80
  - Hash Indexes — p. 82
  - SSTables and LSM-Trees — p. 86
  - B-Trees — p. 89
  - Comparing B-Trees and LSM-Trees — p. 93
  - Other Indexing Structures — p. 95
- Transaction Processing or Analytics? — p. 100
  - Data Warehousing — p. 101
  - Stars and Snowflakes: Schemas for Analytics — p. 103
- Column-Oriented Storage — p. 105
  - Column Compression — p. 107
  - Sort Order in Column Storage — p. 109
  - Writing to Column-Oriented Storage — p. 111
  - Aggregation: Data Cubes and Materialized Views — p. 111
- Summary — p. 113

### Chapter 4. Encoding & Evolution (p. 119)
- Formats for Encoding Data — p. 120
  - Language-Specific Formats — p. 121
  - JSON, XML, and Binary Variants — p. 122
  - Thrift and Protocol Buffers — p. 125
  - Avro — p. 130
  - The Merits of Schemas — p. 135
- Modes of Dataflow — p. 136
  - Dataflow Through Databases — p. 137
  - Dataflow Through Services: REST and RPC — p. 139
  - Message-Passing Dataflow — p. 144
- Summary — p. 147

---

## Part II: Distributed Data (p. 152)

### Chapter 5. Replication (p. 156)
- Leaders and Followers — p. 157
  - Synchronous Versus Asynchronous Replication — p. 158
  - Setting Up New Followers — p. 160
  - Handling Node Outages — p. 161
  - Implementation of Replication Logs — p. 163
- Problems with Replication Lag — p. 166
  - Reading Your Own Writes — p. 167
  - Monotonic Reads — p. 169
  - Consistent Prefix Reads — p. 170
  - Solutions for Replication Lag — p. 172
- Multi-Leader Replication — p. 173
  - Use Cases for Multi-Leader Replication — p. 173
  - Handling Write Conflicts — p. 176
  - Multi-Leader Replication Topologies — p. 180
- Leaderless Replication — p. 182
  - Writing to the Database When a Node Is Down — p. 182
  - Limitations of Quorum Consistency — p. 186
  - Sloppy Quorums and Hinted Handoff — p. 188
  - Detecting Concurrent Writes — p. 189
- Summary — p. 197

### Chapter 6. Partitioning (p. 203)
- Partitioning and Replication — p. 204
- Partitioning of Key-Value Data — p. 205
  - Partitioning by Key Range — p. 206
  - Partitioning by Hash of Key — p. 207
  - Skewed Workloads and Relieving Hot Spots — p. 209
- Partitioning and Secondary Indexes — p. 210
  - Partitioning Secondary Indexes by Document — p. 210
  - Partitioning Secondary Indexes by Term — p. 212
- Rebalancing Partitions — p. 213
  - Strategies for Rebalancing — p. 214
  - Operations: Automatic or Manual Rebalancing — p. 217
- Request Routing — p. 218
  - Parallel Query Execution — p. 220
- Summary — p. 220

### Chapter 7. Transactions (p. 224)
- The Slippery Concept of a Transaction — p. 225
  - The Meaning of ACID — p. 226
  - Single-Object and Multi-Object Operations — p. 231
- Weak Isolation Levels — p. 236
  - Read Committed — p. 237
  - Snapshot Isolation and Repeatable Read — p. 240
  - Preventing Lost Updates — p. 245
  - Write Skew and Phantoms — p. 249
- Serializability — p. 254
  - Actual Serial Execution — p. 255
  - Two-Phase Locking (2PL) — p. 260
  - Serializable Snapshot Isolation (SSI) — p. 264
- Summary — p. 269

### Chapter 8. The Trouble with Distributed Systems (p. 275)
- Faults and Partial Failures — p. 276
  - Cloud Computing and Supercomputing — p. 277
- Unreliable Networks — p. 279
  - Network Faults in Practice — p. 281
  - Detecting Faults — p. 282
  - Timeouts and Unbounded Delays — p. 283
  - Synchronous Versus Asynchronous Networks — p. 286
- Unreliable Clocks — p. 289
  - Monotonic Versus Time-of-Day Clocks — p. 290
  - Clock Synchronization and Accuracy — p. 291
  - Relying on Synchronized Clocks — p. 293
  - Process Pauses — p. 297
- Knowledge, Truth, and Lies — p. 302
  - The Truth Is Defined by the Majority — p. 302
  - Byzantine Faults — p. 306
  - System Model and Reality — p. 308
- Summary — p. 312

### Chapter 9. Consistency & Consensus (p. 321)
- Consistency Guarantees — p. 322
- Linearizability — p. 324
  - What Makes a System Linearizable? — p. 325
  - Relying on Linearizability — p. 330
  - Implementing Linearizable Systems — p. 332
  - The Cost of Linearizability — p. 335
- Ordering Guarantees — p. 339
  - Ordering and Causality — p. 339
  - Sequence Number Ordering — p. 343
  - Total Order Broadcast — p. 348
- Distributed Transactions and Consensus — p. 352
  - Atomic Commit and Two-Phase Commit (2PC) — p. 354
  - Distributed Transactions in Practice — p. 360
  - Fault-Tolerant Consensus — p. 364
  - Membership and Coordination Services — p. 370
- Summary — p. 373

---

## Part III: Derived Data (p. 384)

### Chapter 10. Batch Processing (p. 387)
- Batch Processing with Unix Tools — p. 389
  - Simple Log Analysis — p. 389
  - The Unix Philosophy — p. 392
- MapReduce and Distributed Filesystems — p. 395
  - MapReduce Job Execution — p. 397
  - Reduce-Side Joins and Grouping — p. 401
  - Map-Side Joins — p. 406
  - The Output of Batch Workflows — p. 409
  - Comparing Hadoop to Distributed Databases — p. 412
- Beyond MapReduce — p. 417
  - Materialization of Intermediate State — p. 417
  - Graphs and Iterative Processing — p. 422
  - High-Level APIs and Languages — p. 424
- Summary — p. 427

### Chapter 11. Stream Processing (p. 435)
- Transmitting Event Streams — p. 436
  - Messaging Systems — p. 437
  - Partitioned Logs — p. 442
- Databases and Streams — p. 447
  - Keeping Systems in Sync — p. 448
  - Change Data Capture — p. 450
  - Event Sourcing — p. 453
  - State, Streams, and Immutability — p. 455
- Processing Streams — p. 460
  - Uses of Stream Processing — p. 461
  - Reasoning About Time — p. 464
  - Stream Joins — p. 468
  - Fault Tolerance — p. 472
- Summary — p. 475

### Chapter 12. The Future of Data Systems (p. 484)
- Data Integration — p. 485
  - Combining Specialized Tools by Deriving Data — p. 485
  - Batch and Stream Processing — p. 489
- Unbundling Databases — p. 494
  - Composing Data Storage Technologies — p. 494
  - Designing Applications Around Dataflow — p. 499
  - Observing Derived State — p. 504
- Aiming for Correctness — p. 510
  - The End-to-End Argument for Databases — p. 511
  - Enforcing Constraints — p. 516
  - Timeliness and Integrity — p. 519
  - Trust, but Verify — p. 523
- Doing the Right Thing — p. 528
  - Predictive Analytics — p. 528
  - Privacy and Tracking — p. 531
- Summary — p. 538

---

## Back Matter
- Glossary — p. 548
- Index — p. 554
