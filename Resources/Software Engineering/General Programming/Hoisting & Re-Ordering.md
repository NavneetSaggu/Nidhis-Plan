# Hoisting & Re-Ordering

## **Hoisting**

Means that you have pulled some operation out of a loop because the loop itself does not affect the result of the operation. 
Eg. hoisting a conditional test out of the while loop if the variable doesn’t get modified in the loop.

### In Java

```java
while (!stop)
        i++;
```

Might be converted into this code:

```java
if (!stop)
    while (true)
        i++;
```

JVM does (allows) this "optimization" when there is no synchronization block on the given method.

## **Re-Ordering**

Means changing the sequence of instructions in a way that does not affect the result. Typically this would be adjacent instructions with no data dependencies 
Eg. it does not matter which order you perform the following two statements:

```java
int a = x;
int b = y;
```