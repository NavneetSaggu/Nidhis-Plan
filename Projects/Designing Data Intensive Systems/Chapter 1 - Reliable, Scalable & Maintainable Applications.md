
## What is a Data-Intensive Application?

- Most apps today are **data-intensive**, not compute-intensive — the bottleneck is the amount, complexity, and speed of change of data.
- Modern apps are built from standard building blocks:
  - **Databases** — store data for later retrieval
  - **Caches** — remember results of expensive operations to speed up reads
  - **Search indexes** — allow keyword/filter queries
  - **Stream processors** — send messages to another process asynchronously
  - **Batch processors** — periodically crunch large accumulated datasets

### Blurring of categories
- **Redis** — datastore also used as a message queue
- **Apache Kafka** — message queue with database-like durability guarantees

When you combine several tools, the application code acts as the glue — you are now a **data system designer**, not just an application developer. You essentially build a new, special-purpose data system from smaller, general-purpose components. 

The example system below uses
- **Memcached** as application-managed caching layer
- **Elasticsearch** as full-text search server

![[Pasted image 20260526071215.png]]

---

## The Three Goals: Reliability, Scalability, Maintainability

| Property | Meaning |
|---|---|
| **Reliability** | System works correctly even when things go wrong |
| **Scalability** | Ability to handle increased load gracefully |
| **Maintainability** | Different people can work on the system productively over time |

> Systems that **anticipate faults and can cope with them** are preferred over systems that try to prevent all faults.

---

## Reliability
• The application performs the function that the user expected.
• It can tolerate the user making mistakes or using the software in unexpected ways.
• Its performance is good enough under the expected load and data volume.
• The system prevents any unauthorized access and abuse.

**Fault** = one component deviating from spec. **Failure** = system as a whole stops working. Build fault-tolerant systems, not fault-free ones.

> Prefer **tolerating faults** over **preventing faults** (exception: in security — prevention is non-negotioable).

### Hardware Faults
- Hard disks, RAM, power supplies, network switches all fail eventually.
- Traditional response: **redundancy** (RAID, dual power, hot-swappable CPUs, diesel generators).
- As platforms like **AWS** prioritise flexibility and elasticity over single-machine reliability, VM instances can **become unavailable without warning**.
- Scale of modern apps means more machines → higher absolute rate of hardware faults.
- Trend: move toward software fault-tolerance in addition to (or instead of) hardware redundancy.
- **Chaos Monkey** — Netflix tool that deliberately kills instances to test fault tolerance.

### Software Errors
- Hardware faults are random and uncorrelated; software bugs are **systematic** — they tend to trigger under specific conditions and affect many nodes at once.
- Examples:
  - Bug triggered by unhandled input case causing every instance to crash (like the leap second Linux kernel bug)
  - Runaway process consuming shared resources
  - Service slows down, becomes unresponsive, returns corrupted data (memory leaks)
  - **Cascading failures** — one component failure triggers failures in others
- No quick fix; requires: careful assumptions, thorough testing, process isolation, monitoring, allowing processes to crash and restart.

### Human Errors
- Leading cause of outages is **developer's or operation's errors**, not hardware failure.
- Mitigations: well-designed abstractions and APIs, sandbox environments, thorough testing (unit → integration → manual), easy rollback, detailed monitoring and alerting, good operational training.

---

## Scalability

Scalability = ability to cope with **increased load**. Not a binary property — ask: *"If the system grows in X way, what are our options for coping?"*

### Describing Load — Load Parameters
- Load is described by **load parameters** — the right choice depends on the system architecture.
- Example: requests/second, ratio of reads to writes, cache hit rate, number of active users.

#### Twitter Case Study
Two main operations: **post tweet** (4.6k avg / 12k peak writes/sec) and **home timeline** (300k reads/sec).

Two approaches:

**Approach 1 — Pull on read:** Query tweets from all followed users at read time.
- Simple write, expensive read (fan-in at read time).

**Approach 2 — Fan-out on write:** On post, pre-compute and push tweet into each follower's timeline cache.
- Fast read (just look up cache), expensive write (user with 30M followers → 30M writes per tweet).

![[Pasted image 20260526071157.png]]

**Twitter's solution:** Hybrid — most users get fan-out on write; celebrities (high follower count) are fetched and merged at read time.

### Describing Performance — Percentiles

- **Mean** response time is misleading — use **percentiles**.
- **p50** (median): half of requests are faster than this.
- **p95, p99, p999** (tail latencies): the slowest 5%, 1%, 0.1% of requests.

> **Amazon uses p999** (99.9th percentile) as their SLA target — the slowest requests tend to come from customers with the **most data** (i.e., most valuable customers).

#### Queueing Delays & Head-of-Line Blocking
- High percentile response times are often dominated by **queueing delays**.
- A server processing a small number of requests in parallel only needs a **few slow requests to block subsequent ones** — "head-of-line blocking".
- Important: measure response times **on the client side**, not server side.

#### Tail Latency Amplification
- If a user request fans out to multiple backend calls, the **slowest call determines the overall response time**.
- Even if only 1% of individual calls are slow, the chance that *at least one* of N parallel calls is slow grows quickly.

![[Pasted image 20260526071133.png]]

> When load-testing: the client must send requests **independently of response time** — if it waits for the previous response, queues are artificially short and measurements are skewed.

### Approaches for Coping with Load
- **Vertical scaling** (scale up) — more powerful machine. Simple, but has limits and cost.
- **Horizontal scaling** (scale out) — distribute load across many smaller machines. More complex.
- **Elastic systems** — automatically add resources when load increases; useful for unpredictable load.

> The architecture that scales well for one application often doesn't work for another. It is built around **assumptions of which operations are common and which are rare** — those assumptions are the load parameters.

Key factors shaping a scalable architecture:
- Volume of reads / writes
- Volume of data to store
- Complexity of the data
- Response time requirements
- Access patterns

---

## Maintainability

Three design principles to reduce pain over a system's lifetime:

### Operability — Making Life Easy for Operations
Operations teams keep the system running. Good systems should:
- Provide **visibility into runtime behavior** (monitoring, metrics, logging)
- Support **automation and integration** with standard tools
- Have **good documentation** and an easy-to-understand operational model
- Exhibit **good default behavior** while allowing overrides

### Simplicity — Managing Complexity
- As systems grow, **accidental complexity** accumulates — complexity not inherent in the problem itself, but arising from the implementation.
- Accidental complexity makes systems harder to understand, change, and maintain.
- Best tool for removing accidental complexity: **abstraction** — hiding implementation detail behind a clean interface.

### Evolvability — Making Change Easy
- Requirements change constantly. Systems must be easy to modify.
- Closely linked to simplicity — simple systems are easier to change.

---

## Summary

| Goal                | What it means                                               | Threats                                           |
| ------------------- | ----------------------------------------------------------- | ------------------------------------------------- |
| **Reliability**     | System works correctly even when faults occur               | Hardware faults, software bugs, human error       |
| **Scalability**     | System handles increased load without degrading performance | Growing users, data volume, request rate          |
| **Maintainability** | System remains productive for future engineers              | Accidental complexity, poor operability, rigidity |

- Fault-tolerance: **hide certain types of faults** from the end user.
- These three properties are not binary — they must be designed for continuously.
