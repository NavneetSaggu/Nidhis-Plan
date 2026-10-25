## Faults and Partial Failures

- Single-computer behavior is **deterministic** — same operation always produces same result; hardware fault → total crash, not partial failure
- Distributed systems: **partial failures** — some parts broken while others work fine; *nondeterministic* — same operation may sometimes work, sometimes not
- You may not even know whether an operation succeeded, because message delivery time is also nondeterministic

---

## Cloud Computing vs. Supercomputing

| Supercomputers (HPC) | Internet Services (Cloud) |
|---|---|
| Stop entire cluster on node failure; restart from checkpoint | Must stay online; no acceptable downtime |
| Specialized hardware, shared memory / RDMA | Commodity hardware, IP/Ethernet |
| Nodes are relatively reliable | Higher failure rates; assume something always broken |
| Crash = total failure escalation | Must tolerate partial failure in software |

- Internet services: must tolerate failed nodes and continue as a whole → enables rolling upgrades, live node replacement
- Cloud environments: kill slow VM and request a new one
- Geographically distributed deployments: inter-DC communication over slow/unreliable internet

> **Key principle**: Build reliable systems from unreliable components (TCP over IP, ECC over noisy channels — but always with limits)

---

## Unreliable Networks

Shared-nothing systems communicate *only* via network. The internet and most datacenter networks are **asynchronous packet networks** — no delivery guarantees.

### 6 Possible Failure Modes (when you send a request)

![[fig8-1.png]]

1. Request lost (cable unplugged)
2. Request queued / delayed (network or recipient overloaded)
3. Remote node crashed
4. Remote node temporarily unresponsive (e.g., GC pause)
5. Remote node processed request, but **response lost**
6. Remote node processed request, but **response delayed**

→ All 6 are **indistinguishable** to the sender. Sender cannot tell why no response arrived.

### Detecting Faults

Feedback you *might* get (but can't count on):
- **RST/FIN packet**: OS closes/refuses TCP → process crashed, port not listening (but node may have crashed *mid-request*)
- **Process death script**: HBase notifies other nodes so takeover can happen without waiting for timeout
- **Switch management interface**: detect link failures at hardware level (ruled out over internet or shared DCs)
- **ICMP Unreachable**: router informs you IP is unreachable (but router itself can't distinguish all failure modes)

Even TCP acknowledgment doesn't prove the *application* processed the request. You need an explicit application-level response.

### Timeouts and Unbounded Delays

- Timeout is the *only sure* fault detection mechanism — but no perfect timeout value exists
- **Long timeout**: long wait; users see errors or delays
- **Short timeout**: detects faults faster, but risk of declaring a node dead when it's just slow → cascading failures (transferred load makes slow node worse; in extreme cases all nodes declare each other dead)
- In an idealized network with max delay `d` and max processing time `r`, timeout of `2d + r` would work — but **real networks have unbounded delays**

### Network Congestion and Queueing

![[fig8-2.png]]

Four sources of queueing:
1. **Switch queue**: multiple nodes sending to same destination → packets queue at switch to be fed into destination network link one by one; if full, packet dropped and must be retransmitted
2. **OS queue**: all CPU cores busy → incoming network requests queue in OS until application ready
3. **VM queue**: VM paused while another VM uses CPU → incoming data buffered by hypervisor
4. **TCP flow control (backpressure)**: sender limits its own rate to avoid overloading receiver → additional sender-side queue before data enters the network

TCP also retransmits on lost packets (based on observed RTT timeout) → application sees the delay, not the loss.

**UDP vs TCP**: UDP skips flow control and retransmission → lower latency variance, but at cost of reliability. Used for VoIP/video where stale data is worthless.

> **TCP backpressure vs. application-level backpressure (Hystrix / reactive streams)**
>
> TCP has backpressure built into the protocol via the **receive window** — the receiver advertises available buffer space in every ACK; the sender cannot have more bytes in-flight than that window allows. As the receiver's socket buffer fills (because the application is slow to read), the advertised window shrinks, the sender slows down, and pressure propagates upstream automatically. No application code needed.
>
> This works well but has limits: TCP operates at the *byte* level with no concept of requests, and it only kicks in once buffers are already full — meaning latency has already grown before any throttling begins. It also can't cross protocol boundaries (e.g., HTTP/1.1 has no native backpressure).
>
> **Application-level backpressure** (Hystrix circuit breaker, Reactor/RxJava reactive streams, gRPC flow control) operates at the *request* level and can do things TCP cannot:
> - **Fail fast**: reject a request immediately if downstream is struggling, rather than queuing it to time out in 30 seconds
> - **Circuit breaking**: once error rate exceeds a threshold, stop sending requests entirely for a cooldown period
> - **Load shedding**: return a degraded/fallback response instead of queuing indefinitely
> - **Cross-protocol propagation**: enforce backpressure across HTTP, message queues, or any other transport where TCP alone won't help
>
> In short: TCP backpressure is automatic and physical — it protects the network link. Application-level backpressure is intentional and semantic — it protects your service logic from being overwhelmed by more work than it can process.

### Adaptive Timeouts

- In multi-tenant clouds, network delays are highly variable (noisy neighbor problem)
- Better than fixed timeouts: continually measure RTT and jitter, auto-adjust timeouts
- **Phi Accrual failure detector** (used in Akka, Cassandra): outputs probability that node has failed based on observed heartbeat timing

### Synchronous vs. Asynchronous Networks

- **Telephone circuit-switching**: reserves fixed bandwidth end-to-end for the call duration → **bounded delay**, no queueing
- **Packet-switching (Ethernet, IP, TCP)**: packets share bandwidth dynamically → **unbounded delays** due to queueing
- Packet switching wins for bursty traffic (web pages, files) because it maximizes utilization; circuit switching would require upfront bandwidth guessing
- **Latency guarantee = static resource allocation = higher cost, lower utilization**
- Dynamic resource sharing (internet) = cheaper, better utilization, but variable delays
- QoS + admission control *could* emulate circuit switching on packet networks, but not enabled in multi-tenant datacenters today

---

## Unreliable Clocks

Each machine has its own quartz clock — not perfectly accurate. NTP synchronizes clocks, but imperfectly.

### Two Clock Types

| | Time-of-Day Clock | Monotonic Clock |
|---|---|---|
| What it measures | Wall-clock time (seconds since epoch) | Time interval (elapsed duration) |
| Linux API | `clock_gettime(CLOCK_REALTIME)` | `clock_gettime(CLOCK_MONOTONIC)` |
| Java API | `System.currentTimeMillis()` | `System.nanoTime()` |
| NTP-synced? | Yes | Not directly; NTP may *slew* its rate |
| Can jump backward? | Yes (NTP reset) | No — always moves forward |
| Cross-machine compare? | Intended to be comparable | **Meaningless** |
| Use for | Timestamps, event times | Timeouts, elapsed durations |

- Monotonic clock may be different per CPU socket; OS compensates but not always perfectly
- NTP can slew monotonic clock rate ±0.05%, but cannot cause it to jump

### Clock Accuracy Problems

- **Quartz drift**: Google assumes 200 ppm → 6 ms drift per 30-second resync, 17 seconds per day
- **Forced reset**: if local clock drifts too far from NTP, it may be forcibly reset → time jumps backward or forward
- **Firewalled NTP**: misconfiguration may go unnoticed; clock error grows silently
- **NTP accuracy limited by network delay**: min error ~35ms over internet; spikes to ~1s on congested networks
- **Wrong NTP servers**: reporting hours-off time; NTP clients use multiple servers and reject outliers, but trust is still required
- **Leap seconds**: 59 or 61-second minute; has crashed many systems; best practice is NTP *smearing* (spread adjustment over a day)
- **VM clock virtualization**: when VM is paused, clock appears to jump forward suddenly
- **User-controlled devices**: hardware clock may be set arbitrarily

> Incorrect clocks cause **silent data loss**, not dramatic crashes — they go unnoticed until damage is done. Monitor clock offsets between all machines; declare nodes dead if drift is too large.

### Timestamps for Ordering Events (LWW Danger)

![[fig8-3.png]]

- Multi-leader replication with **Last Write Wins (LWW)** by timestamp: a node with a lagging clock can have its newer write discarded because its timestamp is older
- In fig8-3: write `x=2` (causally later) has timestamp 42.003s; write `x=1` has 42.004s → node 2 incorrectly discards `x=2`

**Problems with LWW timestamps:**
- Writes can silently disappear — lagging-clock node's writes discarded until clock skew resolves
- Cannot distinguish sequential vs. concurrent writes → needs version vectors for causality
- Millisecond-resolution clocks → same timestamp from two nodes; random tiebreaker can still violate causality
- NTP accuracy is bounded by network RTT → you cannot make NTP accurate enough to safely order concurrent writes

**Logical clocks** (Lamport clocks, vector clocks) — based on incrementing counters — are the safe alternative for ordering events. They capture only relative ordering, not wall time.

### Clock Confidence Intervals

- Clock readings aren't point-in-time values — they're ranges: e.g., "95% confident time is 10.3–10.5 seconds past the minute"
- Most systems don't expose this uncertainty (e.g., `clock_gettime()` returns no error estimate)
- **Google Spanner TrueTime API**: returns `[earliest, latest]` — explicit confidence interval on current time
  - Width depends on time since last sync with accurate source (GPS/atomic clock)
  - Each Google datacenter has GPS receiver or atomic clock → keeps uncertainty to ~7ms

### Spanner: Synchronized Clocks for Global Snapshots

- Snapshot isolation needs monotonically increasing transaction IDs; on a single node, a counter suffices
- Distributed across datacenters: generating a global monotonically increasing txid requires coordination → bottleneck
- Spanner's solution: use TrueTime timestamps as txids
  - If two confidence intervals `[A_e, A_l]` and `[B_e, B_l]` **don't overlap**, then ordering is unambiguous
  - **Commit-wait**: before committing a read-write transaction, Spanner waits for the length of the confidence interval — ensuring any future transaction's interval will not overlap with this one → causality guaranteed

---

## Process Pauses

### The Lease Expiry Bug

```java
while (true) {
    request = getIncomingRequest();
    if (lease.expiryTimeMillis - System.currentTimeMillis() < 10000) {
        lease = lease.renew();
    }
    if (lease.isValid()) {
        process(request);  // ← thread could pause HERE for 15 seconds
    }
}
```

Two problems:
1. Uses time-of-day clock from *different machine* for expiry comparison → clock skew can cause incorrect behavior
2. Even with local monotonic clock: assumes negligible time between `isValid()` check and `process()` — but thread can be paused arbitrarily long in between

### Causes of Unexpected Process Pauses

- **Stop-the-world GC**: JVM GC can pause all threads for *minutes*; even "concurrent" collectors (CMS) need brief stop-the-world phases
- **VM suspend/resume**: VM suspended (memory saved to disk), resumed later; used for live migration; pause duration depends on write rate
- **Laptop lid close**: OS suspends and resumes execution
- **Context switch / hypervisor VM switch**: running thread paused at arbitrary point; **steal time** = CPU time given to other VMs; under heavy load, paused thread may wait long to resume
- **Synchronous disk I/O**: thread blocks waiting for slow I/O; Java classloader lazily loads classes → I/O can happen unexpectedly; amplified if disk is NFS/EBS (network delays)
- **Memory paging / thrashing**: memory access → page fault → load from disk; under memory pressure can cascade to thrashing; servers often disable swapping to prevent this
- **SIGSTOP signal**: immediately freezes process; only resumes on SIGCONT; can be sent accidentally

A paused node cannot detect it was paused — it has no notion of time passing. When it resumes, it may act on stale state (e.g., a lease it believes is valid).

### Real-Time Systems

- **Hard real-time** = specified deadline, failure to meet it = system failure (aircraft, airbags, rockets)
- Requires: RTOS with guaranteed CPU time allocation, libraries with documented worst-case execution, restricted/no dynamic memory allocation, extensive testing
- Very expensive, restricts language/library choices, lower throughput (timely response over performance)
- **Not appropriate for most server-side data systems**

### Limiting GC Impact

- Treat GC pauses as *brief planned node outages*: application stops routing requests to node before GC, drains in-flight requests, then GC runs with no active traffic
- Alternatively: only GC short-lived objects frequently; restart processes periodically before accumulating enough long-lived objects to require full GC (like rolling upgrade)

---

## Knowledge, Truth, and Lies

### A Node Cannot Trust Its Own Judgment

Three scenarios where a node's self-belief is wrong:
1. **Asymmetric network fault**: node receives messages, sends are dropped/delayed → other nodes declare it dead; node can't correct this
2. **GC pause declared dead**: node pauses for a minute; other nodes time out and elect a new leader; node resumes not knowing it was declared dead

→ **Quorum**: decisions require votes from multiple nodes to reduce dependence on any single node. Even if a node believes it's the leader, if a quorum declared it dead, it must step down.

Most common quorum: **absolute majority** (>half the nodes). With `n` nodes, can tolerate `⌊(n-1)/2⌋` failures. Only one majority can exist at a time — no split-brain.

### The Leader and the Lock Problem

Systems often need exactly one of something:
- One leader per partition (avoid split brain)
- One lock holder per resource (avoid concurrent corruption)
- One registrant per username

Even if node *believes* it's the leader, a quorum may have already elected a new one (e.g., after a GC pause or network interruption).

![[fig8-4.png]]

**Fig 8-4 example (HBase bug)**: Client 1 holds file-write lease; pauses for too long → lease expires; Client 2 acquires lease and writes; Client 1 resumes, still believes it has a valid lease, writes → file corruption.

### Fencing Tokens

![[fig8-5.png]]

Solution: every lock grant includes a **fencing token** — a monotonically increasing number from the lock service.

- Storage server **rejects writes with a token older than the highest token it has already processed**
- In fig8-5: Client 1 gets token 33, pauses; Client 2 gets token 34, writes; Client 1 resumes, sends write with token 33 → storage rejects it

If using ZooKeeper as lock service: `zxid` (transaction ID) or `cversion` (node version) serve as fencing tokens — both guaranteed monotonically increasing.

**Critical**: resource itself must check the token — cannot rely on clients checking their own lock status. Services should protect themselves from accidentally abusive clients.

### Byzantine Faults

- **Our assumption throughout this book**: nodes are *unreliable but honest* — they may be slow, unresponsive, or have stale state, but if they respond, they tell the truth
- **Byzantine fault**: a node deliberately or arbitrarily sends wrong/fake responses
- **Byzantine Generals Problem**: `n` generals must agree; some are traitors who send deceptive messages
- **Byzantine fault-tolerant**: system continues correctly even if some nodes malfunction or attackers interfere

When Byzantine fault tolerance is relevant:
- **Aerospace**: radiation can corrupt memory → unpredictable responses; flight control systems must tolerate this
- **Multi-org peer-to-peer systems**: participants may cheat (Bitcoin/blockchain: untrusting parties agreeing on transaction history without central authority)

For datacenter systems: **Byzantine fault tolerance is generally not worth it** — all nodes controlled by same org, radiation not a concern, protocols are complex, require >2/3 correct nodes. If attacker can compromise one node, they can likely compromise all (same software).

Web apps handle untrusted *clients* via input validation/sanitization (SQL injection, XSS) — server remains the authority.

### Weak Forms of Lying (Pragmatic Protections)

Not full Byzantine fault tolerance, but simple and useful:
- **Checksums**: catch corrupted network packets (TCP/UDP checksums catch most; application-level checksums catch the rest)
- **Input validation**: range checks, string length limits → prevent DoS via large allocations; even internal services benefit from basic sanity checks
- **Multiple NTP servers**: client queries all, checks majority agree → detects misconfigured outlier servers

---

## System Models and Reality

### Timing Models

| Model | Assumption | Realistic? |
|---|---|---|
| **Synchronous** | Bounded network delay, bounded process pauses, bounded clock error | No — unbounded delays do occur |
| **Partially synchronous** | Behaves synchronously *most* of the time; occasionally exceeds bounds arbitrarily | Yes — best model for real systems |
| **Asynchronous** | No timing assumptions; no clock; no timeouts | Very restrictive; some algorithms designed for it |

### Node Failure Models

| Model | Assumption |
|---|---|
| **Crash-stop** | Node fails by crashing; once stopped, gone forever (never recovers) |
| **Crash-recovery** | Nodes may crash and later recover; stable storage survives crash; in-memory state lost |
| **Byzantine** | Nodes may do anything — send fake messages, lie, behave arbitrarily |

> **Most realistic model**: partially synchronous + crash-recovery faults

### Correctness Properties

Algorithm correctness is defined by properties it must always satisfy. For fencing tokens:
- **Uniqueness**: no two requests return the same token value
- **Monotonic sequence**: if request `x` completes before request `y` begins, then `token_x < token_y`
- **Availability**: a non-crashed node requesting a token eventually gets a response

### Safety vs. Liveness

| | Safety | Liveness |
|---|---|---|
| Informal | "Nothing bad happens" | "Something good eventually happens" |
| Violation | Can point to exact moment it was broken; damage is done, cannot be undone | May not hold *yet*, but there's hope it will be satisfied in the future |
| Example | Uniqueness, monotonic sequence | Availability (eventual consistency is a liveness property) |

- **Safety must always hold** in all possible situations — even if all nodes crash, safety violations must not occur
- **Liveness can have caveats** — e.g., "a response is received *if* a majority of nodes haven't crashed *and* network eventually recovers"

### Theory vs. Reality

- System models are simplified abstractions — real implementations must handle edge cases the model says can't happen
- Crash-recovery model assumes stable storage survives — but hardware errors or firmware bugs can wipe disks
- Quorum algorithms assume nodes remember stored data — if a node forgets (disk corruption), quorum breaks
- Theoretical analysis uncovers algorithmic bugs; empirical testing catches implementation issues
- Both are necessary — they're complementary, not substitutes
