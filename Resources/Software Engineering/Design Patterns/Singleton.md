# Singleton

1. Best way to implement singleton in Java
2. You can keep adding layers and layers of locking, but what ensures single instantiation in the end, is keeping the instance volatile. 
3. This ensures that all threads are aware of the instantiation of our singleton as soon as it is made
4. This also avoids keeping the entire getInstance method synchronized, which would mean that any time an instance is needed, a thread is required to get a monitor lock on the method.
In our handling, we return the instance straightaway if it is available.

```java
// Double Checked Locking based Java implementation of
// singleton design pattern
class Singleton {
    private static volatile Singleton obj = null;
    private Singleton() {}

    public static Singleton getInstance()
    {
        if (obj == null) {
            // To make thread safe
            synchronized (Singleton.class)
            {
                // check again as multiple threads
                // can reach above step
                if (obj == null)
                    obj = new Singleton();
            }
        }
        return obj;
    }
}
```