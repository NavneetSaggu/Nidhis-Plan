# Volatile

[https://www.javatpoint.com/volatile-keyword-in-java](https://www.javatpoint.com/volatile-keyword-in-java)

- Ensures that variable is not cached and always read from the main memory.
- Used when we want to modify variable in different threads; threads will now always read updated value of variable
- Can be used either with primitive type or objects.
- Cannot be used with classes or methods, only variables.
- Prevents the compiler from doing any reordering or any optimization.

# **What Problems Does It Solve?**

Mostly used to ensure thread safety for simple variables.

- Counters
- Flags
- Status Enums
- Singleton Instances [Singletons](../Singletons.md)

Without the **`volatile`** keyword, there is a risk of the worker thread caching the value of the **`flag`** variable and not noticing changes made by other threads, leading to incorrect behaviour. The use of **`volatile`** helps in this scenario by ensuring the visibility of changes to the shared variable across threads.

# **When To Use It?**

• You can use the volatile keyword with variables. Using volatile keyword with classes and methods is illegal. 

• It guarantees that value of the volatile variable will always be read from the main memory, not from the local thread cache. 

• If you declared variable as volatile, Read and Writes are atomic 

• It reduces the risk of memory consistency error. 

• Any write to volatile variable in Java establishes a happen before relationship with successive reads of that same variable. 

• The volatile variables are always visible to other threads. 

• The volatile variable that is an object reference may be null. 

• When a variable is not shared between multiple threads, you do not need to use the volatile keyword with that variable.
****

# **Difference Between Synchronization and Volatile Keyword**

| Volatile Keyword | Synchronization Keyword |
| --- | --- |
| Volatile keyword is a field modifier. | Synchronized keyword modifies code blocks and methods. |
| The thread cannot be blocked for waiting in case of volatile. | Threads can be blocked for waiting in case of synchronized. |
| It improves thread performance. | Synchronized methods degrade the thread performance. |
| It synchronizes the value of one variable at a time between thread memory and main memory. | It synchronizes the value of all variables between thread memory and main memory. |
| Volatile fields are not subject to compiler optimization. | Synchronize is subject to compiler optimization. |

# **Example of Volatile Keyword**

In the following example, we have defined a class which increases the counter value. The run () method in the VolatileThread.java gets the updated value and old value when the thread begins execution. In the main class, we define the array of thread. 

```java
public class VolatileData
{
    private volatile int counter = 0;
    public int getCounter()
    {
        return counter;
    }
    public void increaseCounter()
    {
        ++counter;      //increases the value of counter by 1   
    }
}  

-----------------------------------------------------------------------------

public class VolatileThread extends Thread
{
    private final VolatileData data;
    public VolatileThread(VolatileData data)
    {
        this.data = data;
    }
    @Override
    public void run()
    {
        int oldValue = data.getCounter();
        System.out.println("[Thread " + Thread.currentThread().getId() +
                "]: Old value = " + oldValue);
        data.increaseCounter();
        int newValue = data.getCounter();
        System.out.println("[Thread " + Thread.currentThread().getId() +
                "]: New value = " + newValue);
    }
}  

-----------------------------------------------------------------------------

public class VolatileMain
{

    private final static int noOfThreads = 2;
    public static void main(String[] args) throws InterruptedException
    {
        VolatileData volatileData = new VolatileData(); //object of VolatileData class  
        Thread[] threads = new Thread[noOfThreads];     //creating Thread array   
        for(int i = 0; i < noOfThreads; ++i)
            threads[i] = new VolatileThread(volatileData);
        for(int i = 0; i < noOfThreads; ++i)
            threads[i].start();                 //starts all reader threads  
        for(int i = 0; i < noOfThreads; ++i)
            threads[i].join();                  //wait for all threads  
    }
}
```