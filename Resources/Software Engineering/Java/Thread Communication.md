# Thread Communication

Imagine you have two threads that need to collaborate on a task. Each thread may need to send data to the other, signal when they have completed their work, or coordinate their actions to avoid conflicts. Thread communication allows threads to work together seamlessly and efficiently.

# **SHARED VARIABLES**

One of the most common ways for threads to communicate in Java is through shared variables. Shared variables are variables that multiple threads can access and modify. However, without proper synchronization, accessing shared variables from multiple threads can lead to data corruption and unexpected behavior.

Let’s consider a simple example where two threads increment a shared counter:

java

```java
public class SharedVariableExample {
    private static int counter = 0;
    public static void main(String[] args) {
        Thread thread1 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                counter++;
            }
        });
        Thread thread2 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                counter++;
            }
        });
        thread1.start();
        thread2.start();
        try {
            thread1.join();
            thread2.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(“Final Counter Value: “ + counter);
    }
}
```

In this example, we have two threads, thread1 and thread2, both incrementing the counter variable 1000 times. We start both threads and then use join() to wait for them to finish before printing the final value of the counter.

However, running this code may not always produce the expected result. The reason is that both threads access and modify the counter variable simultaneously without any synchronization. This can lead to a race condition where the final value of the counter may not be 2000 as expected.

To fix this issue, we need to use synchronization mechanisms to ensure that only one thread can access the counter variable at a time.

# **SYNCHRONIZATION**

Synchronization is the process of controlling access to shared resources to prevent race conditions and data corruption. In Java, you can use the synchronized keyword to synchronize methods or blocks of code.

Let’s modify our previous example to use synchronization:

```java
public class SynchronizationExample {
    private static int counter = 0;
    public static synchronized void incrementCounter() {
        for (int i = 0; i < 1000; i++) {
            counter++;
        }
    }
    public static void main(String[] args) {
        Thread thread1 = new Thread(() - > {
            incrementCounter();
        });
        Thread thread2 = new Thread(() - > {
            incrementCounter();
        });
        thread1.start();
        thread2.start();
        try {
            thread1.join();
            thread2.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(“Final Counter Value: “+counter);
    }
}
```

In this modified example, we have created a synchronized method incrementCounter() that ensures only one thread can execute it at a time. This prevents concurrent access to the counter variable and eliminates the race condition.

By using synchronization, we can ensure that the final value of the counter will always be 2000, as expected.

# **INTER-THREAD COMMUNICATION**

In addition to sharing variables, threads may need to communicate with each other explicitly. Java provides several mechanisms for inter-thread communication, including wait(), notify(), and notifyAll().

> Essentially, the synchronised monitor is enough to guarantee isolated execution between different threads. wait() notify() is only used to order execution of different threads that are executing in parallel.
> 

## **wait()**

The wait() method is used to make a thread wait until a specific condition is met. When a thread calls wait(), it releases the lock on the object it is synchronized on, allowing other threads to acquire the lock and execute their code. The thread will remain in a waiting state until another thread calls notify() or notifyAll() on the same object.

Here’s an example of using wait() and notify() for inter-thread communication:

```java
public class WaitNotifyExample {
    public static void main(String[] args) {
        Object lock = new Object();
        Thread producer = new Thread(() - > {
            synchronized(lock) {
                System.out.println(“Producer: Producing data…”);
                try {
                    Thread.sleep(2000);
                    lock.notify(); // Notify the waiting thread
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
        Thread consumer = new Thread(() - > {
            synchronized(lock) {
                System.out.println(“Consumer: Waiting
                    for data…”);
                try {
                    lock.wait(); // Wait for notification
                    System.out.println(“Consumer: Data received!”);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
        producer.start();
        consumer.start();
    }
}
```

In this example, the producer thread produces data and notifies the consumer thread when the data is ready. The consumer thread waits for the notification using the wait() method. This ensures that the consumer thread only proceeds when the producer has finished its work.

## **notify() and notifyAll()**

The notify() method is used to wake up one of the waiting threads that are blocked on the same object. If multiple threads are waiting, notify() will wake up one of them arbitrarily. If you want to wake up all waiting threads, you can use notifyAll().

Here’s an example using notify() and notifyAll():

```java
public class NotifyExample {
    public static void main(String[] args) {
        Object lock = new Object();
        Runnable task = () - > {
            synchronized(lock) {
                System.out.println(Thread.currentThread().getName() + “: Waiting…”);
                try {
                    lock.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + “: Resumed!”);
            }
        };
        Thread thread1 = new Thread(task);
        Thread thread2 = new Thread(task);
        thread1.start();
        thread2.start();
        try {
            Thread.sleep(1000); // Sleep for 1 second
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        synchronized(lock) {
            // Wake up one waiting thread
            lock.notify();
            // Wake up all waiting threads
            // lock.notifyAll();
        }
    }
}
```

In this example, two threads (thread1 and thread2) are waiting for a notification from the main thread. The main thread uses notify() to wake up one of the waiting threads. If you replace lock.notify() with lock.notifyAll(), both waiting threads will be awakened.

When notifyAll() is called on a monitor, all threads that are waiting on that monitor will be awakened simultaneously. However, although multiple threads are awakened, they will not start executing in parallel. Instead, they will re-acquire the lock on the monitor one by one in an unspecified order.

# MESSAGE QUEUE

You could have a [`BlockingQueue`](http://download.oracle.com/javase/1,5.0/docs/api/java/util/concurrent/BlockingQueue.html) of message objects. Other threads would place messages onto the queue. As part of the `while(true)` loop, thread `A` would [`poll`](http://download.oracle.com/javase/1,5.0/docs/api/java/util/concurrent/LinkedBlockingQueue.html#poll()) the queue and process any messages that have arrived.

```java
class A extends Thread{
 List<Object>  objs = something ;//init it
 BlockingQueue<Message> queue = new LinkedBlockingQueue<Message>();
 void run(){
     while(true){
       Message msg;
       while ((msg = queue.poll()) != null) {
         // process msg
       }
       // do other stuff
     }
   }
}
```