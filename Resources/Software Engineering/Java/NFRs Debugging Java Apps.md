# NFRs / Debugging Java Apps

## Types Of Issues in Java  Apps

### Latency Issues

---

There us an increase in latency for the service

Things to check

1. Is an API throughput more than expected?
    1. Rate limit 
    2. Identify and rogue clients
2. Are all APIs impacted?
    1. Single API: It could be a local issue, some buggy code, some functional issue
    2. All APIs: This could be an issue with a common  dependency, or an issue with the system.
3. Dependency Latencies
Look at the dependencie being called on the upstream, are they latent too?
4. Datastore Metrics
    1. Is there any latency increase?
    2. Are there any contention issues?
    3. Are there any IO issues?

### Increased Failure Rate

---

Increase in faliures in the service requests

Things to check

1. Is an API throughput more than expected?
    1. Rate limit 
    2. Identify and rogue clients
2. Recent Deployment?
    1. Functional Issues
    2. Code Issues
3. Client Side Failures
    1. Timeouts: Tuning / latency issues
    2. Other errors
4. Server Side Failures

### Application Crash

---

Application crashes at times 

1. OOM?
    1. Memory leaks
    2. Need more memory?
2. Liveness probe failures
    1. Are threads blocked?
    2. Is our throughput too high / should we ratelimit?
    3. High start up time could be causing probe failures

> **DASHBOARDS**
99% of the time, you don’t need to take heap dump or run a JFR if you have good metrics, you are acquainted with the dashboards, and you know how to read these dashboards.
> 

### System Related Issues

---

1. CPU
2. Memory
3. Contention / Locks
Multiple threads fighting for a single resource. They usually use mutex or locks or semaphores or things.
4. IO

## Stack Traces

---

Stack trace gives us the execution stack. This stack trace is given at the thread level.

## CPU Profiling

---

What unit is CPU usage measured in? CPU usage is a time measure. Using 50% of the CPU means you’re using the CPU 50% of the time.

Top will tell you these times for the last one second. i.e. in the last one second, how much time did your process use the CPU for?

### How is CPU Profiling Instrumented?

This is calculate by sampling. Look at what process is running in the CPU at regular intervals, that’ll give you an indicator of what % of time your CPU is spending executing what process.

So within your code, if you wish to identify the methods that are taking time, within your Java app (not the time taken by the method, but methods that are taking time), that is done by sampling. Multiple profilers have different sampling techniques, and some are more accurate than the others.

## Understanding Memory Allocation

---

- Each object needs memory location.
- If there are a lot of threads parallely asking for new Object() creation, we will have to put synchronization on it, they will have to queue up for allocation, and that will lead to contention.
- To solve this, they came up with TLABs

TLAB

- Thread Local Allocation Buffer
- Leased space for each thread, all new allocation operations for that thread will be on it’s own TLAB, and there will be no contention
- If full, new TLAB is allocated, this is slow process
- Objects bigger than TLAB are allocated in Eden, this is a slow process

TLAB sizes are automatically calculated. JVM itself optimizes the size.

## Memory Profiling

---

What objects do you create? What GC pressure is your app under?

- More objects
    - More memory
    - Higher GC pressure
    - Higher pause times

### How is Memory Profiling Instrumented?

- Instumenting every object allocation is expensive
- Thus, our profiler only monitors for when new TLABs are created, and when objects are allocated outside the TLAB.
- With this, we can get object counts and sizes.

## Contention and Lock Profiling

---

Look at [_Thread Lifecycle](_Thread%20Lifecycle.md). SImply take a thread dump, look at what threads are blocked, and what are they blocked on. Take multiple thread dumps to validate, since thread dumps are only a snapshot and contention could be a one-off.

Synchronized blocks and IO are the usual culprits here.

## GC Tuning

---

There is no point GC tuning anymore, if you have any GC problems. just move to JDK 17 and use ZGC

### To See Threads and Metrics Being Emitted

---

```sql
# to get dashboard
http://localhost:<admin port of java app>

# to get threads
http://localhost:<admin port of java app>/threads

# to get metrics
http://localhost:<admin port of java app>/metrics?pretty=true
```

### To Run a JFR

---

Running a Flight Recording can increase memory usage by up to 100% (At one point, 40% of memory was being used by Flight Recorder)

```sql
# Make sure you add these to java args
-Dcom.sun.management.jmxremote 
-Dcom.sun.management.jmxremote.port=<app port> 
-Dcom.sun.management.jmxremote.authenticate=false 
-Dcom.sun.management.jmxremote.ssl=false 
-XX:+UnlockCommercialFeatures 
-XX:+FlightRecorder

sudo jcmd <pid of java process you want to run JFR on> JFR.start duration=150s filename=~/cm2.jfr

scp navneet.saggu@10.32.201.90:/home/navneet.saggu/cm2.jfr /Users/navneet.saggu/cm4.jfr
```

### To Connect to JConsole

---

```sql
## Connect socket using secure connection of any IP you can ssh to before connection JMC / JConsole using
ssh -N -D 5600 <IP>

## On JConsole, use
# Terminal :
jconsole -J-DsocksProxyHost=localhost -J-DsocksProxyPort=5600 -J-DsocksNonProxyHosts=
# JConsole : 
10.32.201.90:26810
```

### To Check CPU Utilization

---

Refer to [this](../Linux.md)

### <FK Specific> To See Metrics Being Captured by Cosmos

---

```sql
http://10.47.4.67/api/suggest?type=metrics&max=5000&q=<app id with which you're pushing to cosmos>
```