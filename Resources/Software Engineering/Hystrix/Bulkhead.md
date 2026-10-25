# Bulkhead

Hystrix has two different approaches to the bulkhead, thread isolation and semaphore isolation.

### Thread Isolation

The standard approach is to hand over all requests to component **C** to a separate thread pool with a fixed number of threads and no (or a small) request queue.

### Semaphore Isolation

The other approach is to have all callers acquire a permit (with 0 timeout) before requests to **C**. If a permit can't be acquired from the semaphore, calls to **C** are not passed through.

### Differences

The advantage of the thread pool approach is that requests that are passed to **C** can be timed out, something that is not possible when using semaphores.