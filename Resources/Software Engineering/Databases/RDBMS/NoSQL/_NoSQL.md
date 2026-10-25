# **NoSQL Secret Sauce: The LSM Tree**

## TLDR

The Log Structured Merge tree (LSM tree) is the secret sauce behind NoSQL databases like Cassandra, as it is optimized for fast writes and efficient merging and compaction, making it ideal for ingesting large amounts of data from various sources.

## **Key insights**

- 🌳 The secret sauce behind NoSQL databases like Cassandra is the Log Structured Merge tree, which is optimized for fast writes, making it ideal for ingesting huge amounts of data from various sources like mobile applications and IoT devices.
- 📝 The LSM tree structure in NoSQL databases allows for efficient sequential IO, making writes fast on any storage media.
- 💡Updates to existing object keys in SSTables do not overwrite the old entries, but instead add a new entry to the most recent SSTable, which supersedes any previous entries.
- 📚 The sorted nature of SSTables allows for efficient merging and compaction, making the LSM tree a simple and efficient data structure.
- 🌳 The LSM tree approach used in NoSQL databases involves merging and compacting SSTables in the background to cap the number of tables that need to be searched through for each read.
- 💡Compaction strategies in NoSQL databases, such as Size Tiered Compaction and Leveled Compaction, have different optimizations for write throughput and read optimization.
- 💡There are many optimization strategies for LSM trees that aim to provide read performance closer to that of the B-tree, highlighting the ongoing efforts to improve the efficiency of NoSQL databases.
- 🌳 The use of bloom filters and summary table in LSM trees allows for efficient lookup of keys, reducing the number of random I/O required.

## How It Works

- All writes are batched and written to a memtable, which is implemented as a balanced binary tree.
    ![Untitled](NoSQL/Untitled.png)
    
- This sorted batch is now written in an append only fashion to the disk, to what’s called an SSTable (**Sorted Strings Table**). This is sequential disk IO, and is thus very fast.    
    ![Untitled](NoSQL/Untitled%201.png)
    
- Any updates are just written to another SSTable, and the value from the latest SSTable is meant to overwrite all older existing values.
    ![Untitled](NoSQL/Untitled%202.png)
    
- Any deletes are kept track of by a tombstone marker, which tell us that the key no longer exists.    
    ![Untitled](NoSQL/Untitled%203.png)
    
- A user first reads memtable for the value, and then looks up the key sequentially in the series of sorted SSTables. (This reads are further sped up by use of bloom filters, and min-max ranges that are kept at the DB level)
    
    ![Untitled](NoSQL/Untitled%204.png)
    
- This makes writes very fast, but also means that there is a lot of redundant data in the database, in the form of out of date values, and keys that don’t exist anymore.
To counter this increasing number of LSM tables, we use compaction to merge SSTables into levels. This compaction is done by a background cron process.
- This results in the formation of a hierarchical data structure called the LSM tree
    
    ![Untitled](NoSQL/Untitled%205.png)
    
- So to reiterate, memtable is flushed to disk persisted SSTables
    
    ![Untitled](NoSQL/Untitled%206.png)
    
    and when there’s too many SSTables, they are compacted to levels
    
    ![Untitled](NoSQL/Untitled%207.png)
    
- There’s two strategies to compaction;
    - Size Tiered
        - Cassandra
        - Optimized for write throughput
    - Levelled Compaction
        - RocksDB
        - Optimized for read throughput
- Compaction strategy is critical here.
- There are many optimization strategies that try to provide read performance closer to that of B Trees that are used in SQL databases.
    
    ![Untitled](NoSQL/Untitled%208.png)
    
    - **Summary Table**: keeps min max ranges of keys present on a disk block. This allows system to skip searches on those disk blocks where key doesn’t fall in range. 
    Naive search will us a lot of IO, even though all SSTables are sorted; going through all the SSTables is expensive. This saves us from that.
    - **Bloom Filter**: returns a firm no, if a key is not present, and a probable yes if it could be.
    Most systems use a bloom filter at each level. 
    This saves us from having to go through all the levels to check if a key exists, in case if a key is missing from a DB altogether.

1. Single deletion → will whole file be deleted?
2. Is this for write heavy systems?