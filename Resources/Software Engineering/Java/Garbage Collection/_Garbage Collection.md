# Garbage Collection

[https://www.youtube.com/watch?v=XrNgF2sbhGQ&ab_channel=VirtualPairProgrammers](https://www.youtube.com/watch?v=XrNgF2sbhGQ&ab_channel=VirtualPairProgrammers)

TODO: ADD STUFF FROM HERE [https://github.com/DeepakVadgama/java-interview/blob/master/topics/core/garbage-collection.md](https://github.com/DeepakVadgama/java-interview/blob/master/topics/core/garbage-collection.md)

- The scope of a stack is very tight, since it is based on code blocks.
- At times we want objects live for longer and be accessible outside.
- In C, C++, Pascal and other languages, we can choose whether want objects to be stored on the stack or on the heap.
- Java automatically creates all objects on the heap.
    
    <aside>
    📌 There are some optimizations made by the JVM when it can tell for sure, that an object created will not be accessed outside a limited scope, and in such cases object may be evacuated to the stack.
    
    </aside>
    
    <aside>
    📌 There are also special optimizations made for strings. See [here](Core%20Java.md).
    
    </aside>
    

## Deallocating Memory

In C, C++ etc. you must state when an object is no longer needed by calling free().

Objects not freed continue to consume memory (even after app is shutdown), until memory fills up and we’re forced to restart the system.

This is called a **Memory Leak**.

Java does this by itself, thus it’s a **Managed Language**.

## Managed Language

Java avoids memory leaks by

- **Runnning on a Virtual Vachine.** 
**new** keyword doesn’t take memory directly from the OS, but via the VM. This ensures that unless the VM has a leak, OS level leaks are impossible
    
    <aside>
    📌 VM is another function written in C to control allocation and freeing of memory for objects on OS.
    
    </aside>
    
- **Garbage Collection strategy**
An automatic process figures out which objects are no longer needed and deallocates memory for them.

## Eligibility for Garbage Collection

Any object on the heap not reachable through a reference from the stack is eligible for garbage collection.

Important note, we say unreachable and not unreferenced. A referenced object can still be unreachable.

## Soft Leaks (not an official term)

Since a JVM will eventually release all of it’s allocated memory back to OS, a traditional leak is not possible.

But suppose, an object is referenced on the stack even though it will never be used again.

We should not be keeping objects live, for longer than we want to.

One good trick to checking for leaks is running on smaller heap size (-Xmx10m for eg).

Our app should not be consuming most of our available heap, and it should have regular dips; indicating gc is successfuly freeing up memory for us.

## Generational Collection

Instead of searching for all objects to be removed, GC looks for all objects to be retained and it rescues them.

This is called **Mark and Sweep.** 

### Mark and Sweep

1. All threads in the app are paused. This is called a **Stop The World Event**.
2. All live references are checked.
3. All objects found at the end of a reference are marked as being alive.
4. A full scan of the heap is done, all objects not marked can then be cleaned up.
5. Objects being retained are moved to a single contiguous block of memory, called **Compacting**.

### Problems

If there are a lot of objects that are not garbage, this process will take a long time and app will get noticeably frozen.

### Solution

Generational GC. Why?

1. Most objects don’t live very long
2. If object survives one GC, it’s more likely to live forever

### Dividing The Heap

- **Young Generation**
    - Smaller
    - All new objects created here
    - Fills up quickly
    - When full, GC takes place, but only for Young
    - GC of this is called a **minor collection**
    - Since most objects don’t live long, most stuff here is garbage, and this space is also small; therefore GC is quick, compacting is quick.
    - Surviving objects (after n number of GCs) are moved to Old
- **Old Generation**
    - Bigger
    - GC of this is called a **major collection**
    - Most objects here are live, and space is also bigger, therefore GC takes longer, compacting takes longer.
    

Young Generation is divided into

- **Eden**
    - New objects are created here
- S0
    - Space where survivors of first GC get moved to from Eden
- S1
    - Space where survivors of the next GC get moved to from both S0 and Eden; so that S0 gets empty.

Here’s a VisualVM screenshot to make it easier

![Screenshot 2022-05-02 at 3.20.32 AM.png](Garbage%20Collection/Screenshot_2022-05-02_at_3.20.32_AM.png)

## PermGen

Introduced on Java 6

Objects here are never GC’ed

If this fills up, app crashes.

In Java 6 Internalized Strings are placed here.

Meta data for every class we create is placed here.

If we redeploy on a server enough number of times, our PermGen can run out of space, from only the class metadata that is loaded onto it for each deployment, and never GC’ed.

## MetaSpace

No PermGen since Java 6, it has MetaSpace. Has no cap, all memory is available for this.