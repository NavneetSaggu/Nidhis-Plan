# Atomics

## **Introduction**

A shared mutable object leads to problems when concurrency is involved.

```java
public class Counter {
    int counter;

    public void increment() {
        counter++;
    }
}
```

The increment operation (*counter++*) looks atomic but has 3 operations: fetch value, increment, write new value.

Multiple threads try to get and update the value at the same time, it may result in lost updates.

Think of atomics as volatiles, but with atomicity guarantees for eve more complex operations.

## Locks

Use the *synchronized* keyword in the *increment* method signature. This ensures that only one thread enters method at a time.

Additionally, we need to add the *volatile* keyword to ensure proper reference visibility among threads.

```java
public class SafeCounterWithLock {
    private volatile int counter;

    public synchronized void increment() {
        counter++;
    }
}
```

Using locks hit performance; all threads contend for the lock, those that don’t get the lock get suspended. 

The process of suspending and then resuming a thread is very expensive; context switching overhead can become greater than computation cost.

## Atomic Operations

There are special non-blocking algorithms for concurrent environments which use low-level atomic machine instructions such as compare-and-swap (CAS), to ensure data integrity.

These do not cause threads to get suspended, instead threads are simply told if operation was successful or not.

- Working of CAS [READ IF CURIOUS]
    
    A typical CAS operation works on three operands:
    
    1. The memory location on which to operate (M)
    2. The existing expected value (A) of the variable
    3. The new value (B) which needs to be set
    
    **The CAS operation updates atomically the value in M to B, but only if the existing value in M matches A, otherwise no action is taken.**
    
    In both cases, the existing value in M is returned. This combines three steps – getting the value, comparing the value, and updating the value – into a single machine level operation.
    
    When multiple threads attempt to update the same value through CAS, one of them wins and updates the value. **However, unlike in the case of locks, no other thread gets suspended**; instead, they're simply informed that they did not manage to update the value. The threads can then proceed to do further work and context switches are completely avoided.
    
    One other consequence is that the core program logic becomes more complex. This is because we have to handle the scenario when the CAS operation didn't succeed. We can retry it again and again till it succeeds, or we can do nothing and move on depending on the use case.
    

## Atomics in Java

The most common atomic variable classes are [AtomicInteger](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/atomic/AtomicInteger.html), [AtomicLong](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/atomic/AtomicLong.html), [AtomicBoolean](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/atomic/AtomicBoolean.html), and [AtomicReference](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/atomic/AtomicReference.html). For atomically updating *int*, *long*, *boolean,* and object reference respectively. Class methods are:

- *get()* – gets value from memory, with changes from other threads, equivalent to reading from *volatile.*
- *set()* – sets value to memory, changes visible to other threads, equivalent to writing to *volatile.*
- *lazySet()* – eventually writes the value to memory, maybe reordered with subsequent relevant memory operations. One use case is nullifying references, for the sake of garbage collection, which is never going to be accessed again. In this case, better performance is achieved by delaying the null *volatile* write
- *compareAndSet()* – same as described [here](Atomics.md), returns true when it succeeds, else false
- *weakCompareAndSet()* – same as described [here](Atomics.md), but weaker in the sense, that it does not create happens-before orderings. This means that it may not necessarily see updates made to other variables. [As of Java 9](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/atomic/AtomicInteger.html#weakCompareAndSet(int,int)), this method has been deprecated in all atomic implementations in favor of [*weakCompareAndSetPlain()*](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/atomic/AtomicInteger.html#weakCompareAndSetPlain(int,int)). The memory effects of *weakCompareAndSet()* were plain but its names implied volatile memory effects. To avoid this confusion, they deprecated this method and added four methods with different memory effects such as *weakCompareAndSetPlain()* or [*weakCompareAndSetVolatile()*](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/atomic/AtomicInteger.html#weakCompareAndSetVolatile(int,int))

A thread-safe counter implemented with *AtomicInteger* is shown in the example below:

```java
public class SafeCounterWithoutLock {
    private final AtomicInteger counter = new AtomicInteger(0);

    public int getValue() {
        return counter.get();
    }
    public void increment() {
        while(true) {
            int existingValue = getValue();
            int newValue = existingValue + 1;
            if(counter.compareAndSet(existingValue, newValue)) {
                return;
            }
        }
    }
}
```

As you can see, we retry the *compareAndSet* operation and again on failure, since we want to guarantee that the call to the *increment* method always increases the value by 1.