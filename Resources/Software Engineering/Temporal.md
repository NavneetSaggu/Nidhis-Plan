# Temporal

# Temporal

The Temporal Application will consist of the following pieces:

1. A [Workflow](https://docs.temporal.io/workflows) written in your programming language of choice such as Java. 
A Workflow defines the overall flow of the application.
2. An [Activity](https://docs.temporal.io/activities) is a method that encapsulates business logic prone to failure (e.g., calling a service that may go down). 
These Activities can be automatically retried upon some failure.
3. A [Worker](https://docs.temporal.io/workers), provided by the Temporal SDK, which runs your Workflow and Activities reliably and consistently.

Temporal applications are built using an abstraction called Workflows. You'll develop those Workflows by writing code in a general-purpose programming language such as Java.

Conceptually, a Workflow defines a sequence of steps. With Temporal, those steps are defined by writing code, known as a [Workflow Definition](https://docs.temporal.io/workflows#workflow-definition), and are carried out by running that code, which results in a [Workflow Execution](https://docs.temporal.io/workflows#workflow-execution).

These Workflow Executions orchestrate the execution of [Activities](https://docs.temporal.io/activities), which execute a single, well-defined action, such as calling another service, transcoding a media file, or sending an email message. In the money transfer application, you have three Activity methods, `withdraw`, `deposit`, and `refund`. These symbolize the movement of funds between accounts.

## How It Works

You'll have **three main components** deployed separately:

### **Temporal Service (Cluster)**

- This is the **Temporal Server**, which handles workflow execution history, task queues, and persistence.
- It typically runs as a **separate microservice** (self-hosted or on Temporal Cloud).
- Stores workflow state in a **database (PostgreSQL, MySQL, Cassandra, etc.)**.
- Usually runs **3+ nodes** for fault tolerance.

### **Worker Nodes (Executing Workflows & Activities)**

- These are separate **worker services** that execute your **workflow logic and activities**.
- You write **custom business logic** in these worker processes.
- Workers poll Temporal for tasks from their respective task queues and execute them.
- Multiple worker instances **can be horizontally scaled**.

### **API Service (Your Backend, Accepting User Requests)**

- This is your **actual application backend** (e.g., a Spring Boot service).
- It receives API requests from your **frontend (Flutter, Web, etc.)**.
- It uses the **Temporal SDK** to start workflows or send signals to running workflows.
- But it **does not execute workflows or activities itself**—that’s the worker’s job.

Here's the flow of a request

1. **User makes an API request** → Your API service receives it.
2. **API service starts a workflow** by calling the Temporal Service.
3. **Temporal Service persists the workflow state** and schedules tasks in a queue.
4. **Worker nodes pick up tasks from the queue** and execute the activities.
5. **Workers complete tasks & return results** → Temporal tracks progress.
6. **API can poll or get signals** about the workflow status.

## Pros

- No manual handling of retries or error cases; ensure atomicity in complex processes.
    
    Any failures in a single step ensures rollback of the entire transaction.
    
    This is needed because there's a number of instances where our flow is made of a number of succeeding steps, all of which have to succeed.
    
- Ease of being able to set timeouts on our system.
- Scheduling purging jobs and calculation cron jobs.
- Great visibility into each workflow; each user journey.
    
    What step is each user at; how much time is each step taking .. is all visible on the dashboard
    
- No real circuit breaker; but provision for retries with back-offs and final termination.

## **Cons**

- You're now deploying three clusters
    - We CAN make this two, by combining API service & Workers into a single cluster, but there you lose the benefit of being able to separately scale these two
- It doesn't offer physical isolation in it's multi-tenancy solution (namespaces)
    - Data from all the namespaces is stored in the same DB, just encrypted with different keys.
- It might introduce a lot of redundancy, based on how we implement it
    - I build my system around Temporal vs
    - I build my system around separate database and just use Temporal to update this database as necessary using workflows and activities (This is what I would recommend)
- Each workflow is composed of atomic, idempotent activities that need to be retried or rolled back based on what kind of operation it does.
    - You'll have to think more carefully about the timeouts and retry configs you give to your activities
- A new workflow is instantiated for each step in a user's journey (the way we're thinking of using it)
    - Depending on how lightweight this object is, it could impact our NFR performance?

## Temporal Design Doctrine

- Things get slightly tricky here, because temporal will retry all activities by default, so all your activity operations should be idempotent, or have some mechanism built in to check for idempotency. (This is also one of it's primary pros)
- You cannot use dependency injection into workflows (nor can you blindly convert all existing code to workflows because workflows have logic/determinism constraints). But you can use dependency injection in activities. See [this extension](https://github.com/temporalio/sdk-dotnet/tree/main/src/Temporalio.Extensions.Hosting) and [this sample](https://github.com/temporalio/samples-dotnet/tree/main/src/DependencyInjection).
- Workflow should be queried directly for the current state of an order while it is open.
    - The DB is used to list and search for specific orders and maintain historical information.
    - You never want workflows polling DB for some *state* field.
    - Workflows should not rely on external DBs to decide what to do next. They should keep all the relevant information inside and use these DBs as information sinks.
    - Even though you'll see temporal's internal DBs, we do not generally query on these to get any relevant data about the workflows.
- **Temporal allows you to set different timeout values (Heartbeat, StartToClose, ScheduleToStart, etc.) for different activity types** by specifying **custom ActivityOptions per activity.**
- One important pattern is to always update these external DBs from workflow code.
    - If you need to issue an update don’t update DB then start/signal a workflow.
    - Always start/signal workflow which in turn should update the DB and perform other actions. This way multiple types of race conditions and failure scenarios are eliminated.
- You can only query closed workflow executions up to the set retention period.
    - A worker cluster must be running for workflow to serve queries and its results aren’t persisted anywhere.
    - Data that you need about workflows (such as order details) will all have to be worker queries
    - Any data that you want to keep persisting after a workflow has finished executing must be persisted in a separate DB
- You can choose have to setup three separate services and three separate clusters for separately scaling each, as per your needs.
    - The API service
    - The temporal service, which schedules and orchestrates workflows
    - The workers, with all of their business logic

```
+---------------------+        +-----------------------+        +----------------------+
|   Flutter Frontend  |  --->  |   Backend API Service |  --->  | Temporal Service     |
+---------------------+        +-----------------------+        +----------------------+
                  								                                     |             |
                  								                                     |             |
                  								                                     v             v
                  			                   +--------------------------------+    +----------------------+
                  			                   |   Temporal Worker Service      |    |   Temporal DB        |
                  			                   | (Executes workflows/activities)|    | (Stores state)      	|
                  			                   +--------------------------------+    +----------------------+
```

### Doubts

- Which task queue do these query requests go to for the workers to execute them? Will these go to the same task queue as the workflow task?