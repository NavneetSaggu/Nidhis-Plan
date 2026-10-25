# Synchronized

[https://www.baeldung.com/java-synchronized](https://www.baeldung.com/java-synchronized)

## **Overview**

A piece of logic marked with *synchronized* becomes a synchronized block, **allowing only one thread to execute at any given time,** preventing race conditions.

## Synchronization

```java
public class BaeldungSynchronizedMethods {

    private int sum = 0;

    public void calculate() {
        setSum(getSum() + 1);
    }
}
```

- Illustrating race condition [READ IF CURIOUS]
    
    ```java
    @Data
    public class SynchronizedMethods {
        private int sum = 0;
        public void calculate() {
            setSum(getSum() + 1);
        }
    }
    
    @Test
    public void givenMultiThread_whenNonSyncMethod() {
        ExecutorService service = Executors.newFixedThreadPool(3);
    	  SynchronizedMethods summation = new SynchronizedMethods();
    
        IntStream.range(0, 1000)
          .forEach(count -> service.submit(summation::calculate));
        service.awaitTermination(1000, TimeUnit.MILLISECONDS);
    
        assertEquals(1000, summation.getSum());
    }
    ```
    
    We're using an *ExecutorService* with a 3-threads pool to execute the *calculate()* 1000 times.
    
    If we executed this serially, the expected output would be 1000, but **our multi-threaded execution fails almost every time** with an inconsistent actual output:
    
    ```java
    java.lang.AssertionError: expected:<1000> but was:<965>
    at org.junit.Assert.fail(Assert.java:88)
    at org.junit.Assert.failNotEquals(Assert.java:834)
    ...
    ```
    
    Of course, we don't find this result unexpected.
    
    We can use synchronization to resolve this race condition.
    

We can use the *synchronized* keyword on different levels:

- Instance methods
- Static methods
- Code blocks

For a *synchronized* block, Java internally uses a [monitor](https://www.baeldung.com/cs/monitor), (monitor lock or intrinsic lock) for synchronization. 

Monitors are bound to an object; all synchronized blocks of the same object can have only one thread executing them at the same time.

## Synchronizing Instance Method

We can add the *synchronized* keyword in the method declaration to make the method synchronized:

```
public synchronized void synchronisedCalculate() {
    setSum(getSum() + 1);
}
```

Instance methods are *synchronized* over the instance of the class owning the method, which means only one thread per instance of the class can execute this method.

### Synchronizing Static Method

Static methods are *synchronized* just like instance methods:

```java
 public static synchronized void syncStaticCalculate() {
     staticSum = staticSum + 1;
 }
```

These methods are *synchronized* on the *Class* object associated with the class. Since only one *Class* object exists per JVM per class, only one thread can execute inside a *static synchronized* method per class, irrespective of the number of instances it has.

### Synchronizing Code Block in Instance Method

Sometimes we don't want to synchronize the entire method, only some instructions within it. We can achieve this by *applying* synchronized to a block:

```java
public void performSynchronisedTask() {
    synchronized (this) {
        setCount(getCount()+1);
    }
}
```

Notice that we passed a parameter *this* to the *synchronized* block. This is the monitor object. The code inside the block gets synchronized on the monitor object. Simply put, only one thread per monitor object can execute inside that block of code.

### Synchronizing Code Block in Static Method

If the method was *static*, we would pass the class name in place of the object reference, and the class would be a monitor for synchronization of the block:

```java
public static void performStaticSyncTask(){
    synchronized (SynchronisedBlocks.class) {
        setStaticCount(getStaticCount() + 1);
    }
}
```

### Lock Used for Synchronized is Re-entrant

This means the current thread can acquire the same *synchronized* lock over and over again while holding it:

```java
Object lock = new Object();
synchronized (lock) {
    System.out.println("First time acquiring it");

    synchronized (lock) {
        System.out.println("Entering again");

         synchronized (lock) {
             System.out.println("And again");
         }
    }
}
```

As shown above, while we're in a *synchronized* block, we can acquire the same monitor lock repeatedly.