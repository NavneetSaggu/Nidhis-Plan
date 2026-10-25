# Spark

REFERENCE DOCS:

- [https://spark.apache.org/docs/2.1.1/programming-guide.html](https://spark.apache.org/docs/2.1.1/programming-guide.html) excellent source, really covers the basics of the framework very well
- [https://spark.apache.org/docs/2.1.1/sql-programming-guide.html](https://spark.apache.org/docs/2.1.1/sql-programming-guide.html) excellent source for spark sql, dataframes and datasets

Apache Spark is an open-source cluster computing framework. Its primary purpose is to handle the real-time generated data, but it also runs batch jobs.

# Map Reduce vs Spark

| Map Reduce | Spark |
| --- | --- |
| **Batch Processing**: MapReduce is primarily designed for batch processing of data.
**Disk-Based Processing**: It relies heavily on disk storage for intermediate data between map and reduce stages.
**Java-Centric**: Originally, MapReduce was closely associated with Java, making it Java-centric.
**Complex DAGs**: Representing complex workflows may require chaining multiple MapReduce jobs.
**Slower Processing**: Due to the disk-based nature, MapReduce can be slower for iterative algorithms and interactive data analysis. | **Batch and Stream Processing**: Spark supports both batch processing and stream processing, offering more versatility.
**In-Memory Processing**: It emphasizes in-memory processing, reducing the need for extensive disk I/O.
**Polyglot Support**: Spark provides APIs in multiple languages like Scala, Java, Python, and R, offering more flexibility.
**Directed Acyclic Graph (DAG)**: Spark uses a more flexible and efficient DAG execution engine, enabling complex workflows without chaining jobs.
**Faster Performance**: Thanks to in-memory processing and other optimizations, Spark generally performs faster, especially for iterative algorithms. |

# Local Setup Instructions

Setting up spark cluster on local

1. Download the spark repo from here [https://spark.apache.org/downloads.html](https://spark.apache.org/downloads.html)
2. Untar it, cd into the folder
3. Add a file at ./conf/spark-env.sh with the following configs
    
    ```bash
    #!/usr/bin/env bash
    
    # CONFIGS SEPCIFIED
    
    # Set the number of cores for each Spark worker
    export SPARK_WORKER_CORES=1
    
    # Set the amount of memory for each Spark worker (e.g., 4g for 4 gigabytes)
    export SPARK_WORKER_MEMORY=1g
    
    # Set number of worker instances
    export SPARK_WORKER_INSTANCES=2
    ```
    
4. Run ./sbin/start-master.sh
5. Open [http://localhost:8080/](http://localhost:8080/)
6. Run ./sbin/start-worker.sh <get URL from the dashboard>
7. If you want to see history dashboard, you have to enable event logging using the following steps
    1. make a ./conf/spark-defaults.conf file
    2. Add the following details to it
        
        ```bash
        spark.eventLog.enabled		true
        spark.history.fs.logDirectory	<path to directory you want to store event logs in>
        ```
        
    3. When setting up spark session, make sure to use the same event log path
        
        ```java
        SparkSession spark = SparkSession.builder()
                        .appName("BatchIngestionTest")
                        .master("spark://BLRETV-H0CC4Q7.local:7077")
                        .config("spark.eventLog.enabled", "true")
                        .config("spark.eventLog.dir", "file:<path to directory you want to store event logs in>")
        ```
        
    4. Run ./sbin/start-history-server.sh
    5. Open [localhost:18080](http://localhost:18080) to see history dash
8. When you run your java app on this cluster, make sure it has the same spark version, or you’ll get a very obscure error.
In case you want to see dash of a job that’s still running, use localhost:4040

# Crux Of Spark

## Spark Core

1. Spark Core is the foundational component of the Apache Spark framework.
2. It provides the basic functionality for distributed data processing in Spark.
3. Spark Core **emphasizes in-memory processing**, which significantly speeds up iterative algorithms and interactive data analysis by reducing the need for extensive disk I/O.
4. Spark Core handles the **distribution of tasks across a cluster** of machines, enabling parallel processing of data.
5. It incorporates **fault tolerance** through lineage information, allowing the reconstruction of lost data in case of node failures.
6. Spark Core can integrate with **various cluster managers**, such as Apache Mesos, Hadoop YARN, or Spark's standalone cluster manager.
7. Spark core by itself has **no data storage**, it relies on either HDFS, or external DBs.

## Cluster Managers

1. Cluster managers are responsible for managing and allocating resources in a distributed computing environment. 
    1. Handle the distribution of tasks and data across a cluster of machines (each machine can have multiple nodes). They consider factors such as data locality to minimize data transfer overhead.
    2. Also coordinate activities of individual nodes in cluster, ensuring efficient resource utilization and optimal task scheduling.
    3. Ensure fault tolerance by monitoring node health and redistributing tasks in case of  failures.
    4. They discover nodes as they are added to cluster, allowing Spark to dynamically adapt to changes in the cluster size.
2. Spark can work with various cluster managers 
    1. Apache Mesos
    2. Hadoop YARN
    3. Spark's standalone cluster manager (used for local setup, with multiple nodes on same machine)
3. **Job Isolation:** Cluster managers provide job isolation, ensuring that different Spark applications running on the same cluster do not interfere with each other's resources.
4. **Monitoring and Logging:** They often offer monitoring and logging capabilities to track the performance and health of the cluster, aiding in troubleshooting and optimization.

## RDDs (**Resilient Distributed Datasets**)

1. RDD is a fundamental data structure in Spark, representing an **immutable, fault tolerant, distributed collection of objects** that are **spread across a cluster** of machines and can be **operated on in parallel**.
    1. **Immutability:** RDDs are immutable, meaning once created, their content cannot be changed. However, transformations can create new RDDs.
    2. **Fault Tolerance:** RDDs achieve fault tolerance through lineage information, allowing the system to reconstruct lost data in case of node failures by reapplying transformations.
    3. **Parallel Processing:** RDDs support parallel processing of data. Operations on RDDs are automatically parallelized across the distributed computing infrastructure.
2. RDDs support two types of operations: 
    1. **Transformations** (e.g., map, filter) 
        1. Transformations create new RDDs. A sequence of tranformations will create a sequence of RDDs, which is what is referred to as the Spark DAG.
        2. Transformations on RDDs are lazily evaluated, meaning they are not executed immediately. Actions trigger the actual computation, optimizing the execution plan.
    2. **Actions** (e.g., count, collect, save and reduce)
        1. Actions trigger computations and return results to the driver program.
        2. Every time you trigger an action, the entire Spark DAG will run from start to end, and compute each RDD from scratch.
3. RDDs can be **persisted in memory or stored on disk** to reuse them efficiently across multiple stages of a Spark application.
4. RDDs allow users to define custom partitioning schemes to control the distribution of data across the cluster. (Have not explored yet)

## Caching In Spark

By default, each transformed RDD may be recomputed each time you run an action on it.

However, you may also *persist* an RDD in memory using the `persist` (or `cache`) method.

There is also support for persisting RDDs on disk, or replicated across multiple nodes.

Example where recomputation of transformation occurs in Spark; here map function will execute over the entire dataset twice.

```java
// Create an initial RDD
List<Integer> data = Arrays.asList(1, 2, 3, 4, 5);
JavaRDD<Integer> initialRDD = sc.parallelize(data);

// Define a transformation - map to double the values
JavaRDD<Integer> transformedRDD = initialRDD.map(x -> {
    System.out.println("Mapping: " + x);
    return x * 2;
});

// Trigger an action - count the elements
long count1 = transformedRDD.count();
System.out.println("Count 1: " + count1);

// Perform another action - collect the elements
List<Integer> collectedData = transformedRDD.collect();
System.out.println("Collected Data: " + collectedData);
```

# Spark Architecture

![Untitled](Spark/Untitled.png)

1. **Driver Program:**
    - **Role:** The driver program is the main entry point for Spark applications. It contains the application logic, creates SparkContext, and coordinates the execution of tasks on the cluster. The spark code that we write works as a driver program.
    - **Functionality:** It defines the high-level control flow of the application and interacts with the cluster manager to distribute tasks.
2. **SparkContext:**
    - **Role:** SparkContext is the entry point for any Spark functionality. It establishes a connection to the Spark cluster and manages the distributed processing of data.
    - **Functionality:** SparkContext enables the driver program to communicate with the cluster manager and coordinate the execution of tasks across the worker nodes.
    - **In Code**: **`SparkSession`** is higher-level and provides a more convenient interface for working with structured data, while **`JavaSparkContext`** is lower-level and suitable for RDD-based programming. In modern Spark applications, you'll often find the use of **`SparkSession`** for higher-level abstractions and **`JavaSparkContext`** for more fine-grained control when needed.
    - A typical Spark application makes one Spark session, and that session is used throughout the application's lifecycle.e
3. **Cluster Manager:**
    - **Role:** The cluster manager is responsible for managing and allocating resources in a distributed environment. It is the bridge between the driver program and the worker nodes.
    - **Functionality:** It schedules tasks, monitors node health, handles fault tolerance, and ensures efficient resource utilization.
4. **Worker Nodes:**
    - **Role:** Worker nodes are the individual machines in the Spark cluster that execute tasks assigned by the driver program.
    - **Functionality:** They perform computations, store data, and communicate with the driver program and cluster manager.
5. **Executor:**
    - **Role:** Executors are processes that run on worker nodes and execute tasks assigned by the driver program.
    - **Functionality:** Executors manage the execution of individual tasks, store intermediate data in memory, and communicate with the driver program and cluster manager.
6. **Resilient Distributed Datasets (RDDs):**
    - **Role:** RDDs are the fundamental data structure in Spark, representing distributed collections of objects.
    - **Functionality:** RDDs support parallel processing, fault tolerance, and provide a set of transformations and actions for data processing.
7. **Directed Acyclic Graph (DAG) Scheduler:**
    - **Role:** The DAG scheduler is responsible for optimizing the execution plan of Spark applications by building a directed acyclic graph of stages.
    - **Functionality:** It breaks down the application into stages, optimizing task execution and minimizing data shuffling.
8. **Task Scheduler:**
    - **Role:** Task Scheduler sits within the SparkContext and is responsible for assigning tasks to executor processes on worker nodes.
    - **Functionality:** It considers data locality and resource availability, optimizing task assignment for efficient processing.

# Sequence To Run A Spark Job

![Screenshot 2023-12-20 at 2.19.35 AM.png](Spark/Screenshot_2023-12-20_at_2.19.35_AM.png)

1. **Initialization:**
    - You create a Spark application, typically written in a programming language such as Scala, Java, Python, or R.
    - The application includes the necessary Spark dependencies and configurations.
2. **SparkContext Creation:**
    - The application initializes a SparkContext, which serves as the entry point for Spark functionality.
    - The SparkContext establishes a connection to the Spark cluster and prepares to execute tasks.
3. **Cluster Manager Interaction:**
    - The SparkContext communicates with the chosen cluster manager (e.g., Apache Mesos, Hadoop YARN, or Spark's standalone cluster manager) to request resources for the application.
4. **Task Distribution:**
    - The application logic (driver program) is divided into tasks, and these tasks are distributed to the worker nodes by the cluster manager.
5. **Executor Initialization:**
    - Executors are started on the worker nodes to execute the tasks.
    - Each executor runs in its own Java process and communicates with the driver program.
6. **RDD Creation and Transformation:**
    - The application creates Resilient Distributed Datasets (RDDs) or DataFrames, representing distributed collections of data.
    - RDDs undergo transformations (e.g., map, filter) to define the series of operations to be performed on the data.
7. **DAG Creation:**
    - The application logic, represented by RDD transformations, is converted into a Directed Acyclic Graph (DAG) of stages by the DAG scheduler.
8. **Stage Execution:**
    - The DAG is broken down into stages, which are executed in order. Each stage contains tasks that can be executed in parallel.
    - Tasks within a stage are sent to the task scheduler for assignment to available executors.
9. **Task Scheduling:**
    - The task scheduler assigns tasks to the executors based on data locality, resource availability, and other factors.
    - Executors start executing the tasks in parallel.
10. **Data Processing:**
    - Executors process the data in parallel, applying the transformations defined by the RDD or DataFrame operations.
    - Intermediate data is stored in memory or on disk for efficient reuse.
11. **Result Collection (Action):**
    - Actions (e.g., collect, count) trigger the execution of the DAG and cause the results to be collected to the driver program.
12. **Job Completion:**
    - The entire job is considered complete when all tasks have been executed, and the results are collected to the driver program.
13. **Resource Deallocation:**
    - The cluster manager deallocates resources, and executors are shut down on the worker nodes.
14. **SparkContext Shutdown:**
    - The SparkContext is shut down, releasing the resources and ending the Spark application.

# Spark SQL / Spark 2.0

Spark SQL was built as an improvement over Hive, to overcome the following limitations

- Hive allows SQL-like queries over large datasets but historically operated using MapReduce, making it slower for certain workloads.
- Hive does not allow you to resume a job if it fails in the middle.

Here are it’s high level features:

1. **DataFrames and Datasets:**
    - Spark SQL introduces the concept of DataFrames and Datasets, which are higher-level abstractions for representing structured data. They provide a more user-friendly API for working with data, and Spark SQL seamlessly integrates with these abstractions.
        
        DataFrames are essentially just untyped datasets. Which explains why Dataset<Rows> are DataFrames for Java API. Throughout this document, we will often refer to Scala/Java Datasets of Rows as DataFrames.
        
        |  | RDDs | DataSets | DataFrames (Scala, no applicable to Java) |
        | --- | --- | --- | --- |
        | Definition | RDDs are the foundational abstraction in Spark and represent a distributed collection of objects. They are low-level and provide a fault-tolerant, immutable, and parallel processing model. | Abstraction Level: Datasets are an extension of DataFrames and provide a type-safe, higher-level abstraction. Operate on domain objects with compiled lambda functions. | Abstraction Level: DataFrames are a higher-level abstraction built on top of RDDs. They represent distributed collections of data organized into named columns and tables, similar to a relational database.
         |
        | Functional vs Procedural | More Procedural
        Encourage **how-to** rather than what-to | More Declarative
        Encourage **what-to** rather than how-to | More Declarative
        Encourage **what-to** rather than how-to |
        | Type System | RDDs are dynamically typed, meaning that the types of objects within an RDD are checked at runtime.
        They are only compile time type safe.
         | Datasets are statically typed, allowing the compiler to catch type-related errors during development, providing enhanced robustness. Users can seamlessly convert between DataFrames and Datasets. | DataFrames are dynamically typed, similar to RDDs, but they provide a more expressive API for working with structured data.
         |
        | Optimizations | RDDs do not benefit from Spark's Catalyst optimizer or Tungsten execution engine, and users have to manage optimizations manually.
         | Datasets benefit from both the Catalyst optimizer and Tungsten execution engine, and the use of static typing allows more aggressive optimizations. | DataFrames leverage the Catalyst query optimizer and Tungsten execution engine, providing automatic optimizations for query plans. |
        | API | RDDs have a more functional programming API, and users have explicit control over partitioning and data distribution.
         | Datasets offer an API that is similar to DataFrames but includes additional type-specific operations. They provide a balance between the expressiveness of DataFrames and the type safety of RDDs. | DataFrames provide a more SQL-like, declarative API for expressing data manipulations, making them more user-friendly for structured data processing. |
        | Ease Of Use | RDDs require more manual handling of low-level details. Users need to explicitly define transformations and actions on the RDD, and the syntax can be more complex. |  | DataFrames offer a higher-level API inspired by SQL and pandas. They provide a more intuitive and declarative syntax for expressing data manipulations, making it easier for users familiar with SQL or data analysis tools. |
        | When To Use | Low level control needed
        Dealing with unstructured data
        Manipulate data with lamba functions rather than DSL
        When you dont care about schema of data
        When you dont care about optimizations |  |  |
        | Syntax | RDD in Java | Dataset<Row> in Java | Dataset<Pojo> in Java |
2. **Simple Optimisation Problem With RDD**
    
    ![Untitled](Spark/Untitled%201.png)
    
3. **More Difficult Code in RDD**
    
    ![Untitled](Spark/Untitled%202.png)
    
4. **Structured APIs in Spark**
Analysis errors are reported before starting job
    
    
    |  | SQL | DataFrames | DataSets |
    | --- | --- | --- | --- |
    | Syntax Errors | Runtime | Compile Time | Compile Time |
    | Analysis Errors | Runtime | Runtime | Compile Time |
5. **Catalyst Optimizer**
After a bunch of optimizations, everything still gets decomposed to RDDs.
This is must like the Java JIT compiler, generates RDD code at runtime
    
    ![Untitled](Spark/Untitled%203.png)
    
    ![Untitled](Spark/Untitled%204.png)
    
6. **External Data Sources:**
    - Spark SQL supports a wide range of data sources, allowing users to read, write, and query data in various formats.
    It also supports reading and writing data in CSV format. Users can specify various options for parsing CSV files, such as delimiter, header, and schema inference.
7. **Programmatic SQL Queries:**
    - Users can express SQL queries programmatically using Spark SQL's API, allowing for dynamic and flexible query construction within Spark applications.
    
    ```scala
    // Example of programmatic SQL query using Spark SQL API
    val result = spark.sql("SELECT * FROM myTable WHERE age > 21")
    ```
    
8. **Optimizations and Catalyst Optimizer:**
    - Spark SQL employs the Catalyst query optimizer, which optimizes the execution plan of SQL queries. It performs optimizations like predicate pushdown, constant folding, and rule-based transformations to improve query performance.

# Optimising Spark Jobs

Optimisation approaches to be taken depend on where the bottlenecks are in your job. Spark’s Web UI is usually a good place to start investigating these kinds of questions.

## Shuffling

File shuffling is a process in distributed computing frameworks, like Apache Spark, where data is reorganized and redistributed across the nodes in a cluster. It often occurs as part of certain operations that require data to be grouped or aggregated based on a specific key. Shuffling can be an expensive operation in terms of both time and resources.

Here are key points about file shuffling in the context of Apache Spark:

1. **Why Shuffling Occurs:**
    - Shuffling is typically triggered by operations that require the redistribution of data across partitions. Examples include **`groupByKey`**, **`reduceByKey`**, and **`join`** operations, where data needs to be grouped or aggregated based on a specific key.
2. **Data Movement:**
    - During shuffling, data is moved between partitions and nodes in the cluster to ensure that records with the same key end up in the same partition, facilitating the subsequent processing steps.
3. **Performance Impact:**
    - Shuffling can have a significant impact on the performance of a Spark job. It involves the exchange of data between nodes, leading to network and disk I/O overhead.
4. **Partitioning and Skewness:**
    - The effectiveness of shuffling depends on the choice of partitioning strategy. Skewed data, where certain keys have significantly more records than others, can lead to uneven distribution and longer processing times.
5. **Optimizations:**
    - Spark incorporates various optimizations to minimize the impact of shuffling. This includes using in-memory caching, compressing data during the shuffle, and optimizing the network transfer of data.
6. **Monitoring Shuffling:**
    - Spark provides tools to monitor and analyze shuffling activities, such as the Spark web UI, which displays metrics related to shuffling, including the amount of data shuffled and the time taken for shuffling operations.
7. **Tuning and Configuration:**
    - Users can tune certain configurations related to shuffling, such as adjusting the size of in-memory buffers, configuring the degree of parallelism, and setting memory fractions for shuffling operations.

## Broadcast

Broadcast variables allow the programmer to keep a read-only variable cached on each machine rather than shipping a copy of it with tasks. They can be used, for example, to give every node a copy of a large input dataset in an efficient manner.

# **Closure**

The **closure** is those variables and methods which must be visible for the executor to perform its computations on the RDD. This closure is serialized and sent to each executor along with tasks.

To execute jobs, Spark breaks up the processing of RDD operations into tasks, each of which is executed by an executor. Spark computes the task’s **closure** prior to execution.

## Potential Closure Issues

You must understand the scope and life cycle of variables and methods when executing code across a cluster. RDD operations that modify variables outside of their scope can be a frequent source of confusion. 

```java
int counter = 0;
JavaRDD<Integer> rdd = sc.parallelize(data);

// Wrong: Don't do this!!
rdd.foreach(x -> counter += x);

println("Counter value: " + counter);
```

The variables within the closure sent to each executor are now copies and thus, when **counter** is referenced within the `foreach` function, it’s no longer the **counter** on the driver node. There is still a **counter** in the memory of the driver node but this is no longer visible to the executors!

# Transformations

## **map() vs mapPartitions()**

Spark `map()` and `mapPartitions()` transformations apply the function on each element/record/row of the DataFrame/Dataset and returns the new DataFrame/Dataset.

- [map()](https://sparkbyexamples.com/spark/spark-map-vs-mappartitions-transformation/#map) – Spark `map()` transformation applies a function to each row in a DataFrame/Dataset and returns the new transformed `Dataset`.
- [mapPartitions()](https://sparkbyexamples.com/spark/spark-map-vs-mappartitions-transformation/#mappartitions) – This is exactly the same as map(); the difference being, Spark `mapPartitions()` provides a facility to do heavy initializations (for example Database connection) once for each partition instead of doing it on every DataFrame row. This helps the performance of the job when you dealing with heavy-weighted initialization on larger datasets.

## Map vs UDF

Simply, `map` is more flexible than `udf`. With `map`, there is no restriction on the number of columns you can manipulate within a row. Say you want to derive the value for 5 columns of the data and delete 3 columns. You would need to do `withColumn`/`udf` 5 times, then a `select`. With 1 `map` function, you could do all of this.

Here is how the options stacked up against each other in 2018.

```java
+----------------+------------------+-------------+---------+
|     Method     | Engineering Ease | Performance | Average |
+----------------+------------------+-------------+---------+
| Native         |                1 |           1 |       1 |
| UDF            |                2 |           3 |     2.5 |
| Map            |                2 |           3 |     2.5 |
| Custom Native  |                3 |           1 |       2 |
+----------------+------------------+-------------+---------+
1 is best
3 is worst
```

- Developers should always try to use Spark-native functions over all else.
- If that’s not possible, developers are advised to try extending Spark’s native functions library.
- If there is a need to put together something quickly, UDFs are appropriate although these are not recommended for jobs running in production pipelines.
- Map based functions should be generally avoided.

## Checkpointing

There’s two types of checkpointing in Spark.

This is mostly only relevant for stream processing use cases; we can skip this for now.