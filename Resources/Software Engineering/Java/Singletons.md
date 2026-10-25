# Singletons

# Vanilla Singletons Are Not Thread Safe

```
package com.journaldev.designpatterns;

public class ASingleton {

	private static ASingleton instance = null;

	private ASingleton() {
	}

	public static ASingleton getInstance() {
		if (instance == null) {
			instance = new ASingleton();
		}
		return instance;
	}

}

```

- getInstance() method is not thread-safe
- Multiple threads can enter the if loop and create multiple instances

# How to achieve thread-safety in Singleton Class?

There are three ways through which we can achieve thread safety.

## **Create the instance variable at the time of class loading.**

**Pros**

- Thread safety without synchronization
- Easy to implement

**Cons**

- Early creation of resource that might not be used in the application.
- The client application can’t pass any argument, so we can’t reuse it. For example, having a generic singleton class for database connection where client application supplies database server properties.

## **Synchronize the getInstance() method**.

**Pros**

- Thread safety is guaranteed.
- Client application can pass parameters
- Lazy initialization achieved

**Cons**

- Slow performance because of locking overhead.
- Unnecessary synchronization that is not required once the instance variable is initialized.

## **Use synchronized block inside the if loop and volatile variable**

**Pros**

- Thread safety is guaranteed
- Client application can pass arguments
- Lazy initialization achieved
- Synchronization overhead is minimal and applicable only for first few threads when the variable is null.

**Cons**

- Extra if condition

Looking at all the three ways to achieve thread-safety, I think the third one is the best option. In that case, the modified class will look like this:

```
package com.journaldev.designpatterns;

public class ASingleton {

	private static volatile ASingleton instance;
	private static Object mutex = new Object();

	private ASingleton() {
	}

	public static ASingleton getInstance() {
		ASingleton result = instance;
		if (result == null) {
			synchronized (mutex) {
				result = instance;
				if (result == null)
					instance = result = new ASingleton();
			}
		}
		return result;
	}

}
```

## Making Sense Of The Code

### Important

Just getting the singleton does not need any sync, since it's just a read. So, just synchronize the setting of the Sync would do. Unless two treads try to create the singleton at start up at the same time, then you need to make sure check if the instance is set twice (one outside and one inside the sync) to avoid resetting the instance in a worst case scenario.

### Less Important

The local variable `result` seems unnecessary. But, it’s there to improve the performance of our code. In cases where the instance is already initialized (most of the time), the volatile field is only accessed once (due to “return result;” instead of “return instance;”). This can improve the method’s overall performance by as much as 25 percent. If you think there are better ways to achieve this or if the thread-safety is compromised in the above implementation, please comment and share it with all of us.