# Functional Interface

### **Functional Interface (Java) — Short Definition**

A **functional interface** is an interface that has **exactly one abstract method**. It can have **default methods**, **static methods**, and **`Object` methods** (like `toString()`), but **only one abstract method**.

It’s the foundation for **lambda expressions** and **method references** in Java.

Eg. Runnable is a functional interface

```java

@FunctionalInterface
public interface MyFunctionalInterface {
    // ✅ One abstract method (required)
    void doSomething(String input);

    // ✅ Default method (optional)
    default void log(String message) {
        System.out.println("Log: " + message);
    }

    // ✅ Static method (optional)
    static void printInfo() {
        System.out.println("This is a functional interface");
    }

    // ✅ Overriding Object class methods is allowed
    boolean equals(Object obj);
}
```

### ⚡ Usage with Lambda:

```java
MyFunctionalInterface action = input -> System.out.println("Doing: " + input);
action.doSomething("work");
action.log("done");
MyFunctionalInterface.printInfo();
```

### 🔍 **Implicit Capturing — What Does It Mean?**

When you write a lambda expression in Java, it can **"capture" variables from the surrounding scope** — without you passing them explicitly. These are called **captured variables** or **captured context**.

Java handles this under the hood by **creating an object** (like an anonymous class) that **stores** references to the captured variables.

---

### ⚠️ Rules of Capturing:

1. ✅ You can capture:
    - Final variables
    - *Effectively final* variables (not marked final but never reassigned)
2. ❌ You **can't** capture a variable that changes after the lambda is defined.

---

### 🔧 Example:

```java
public class LambdaCaptureExample {
    public void process() {
        String prefix = "Hello "; // effectively final

        MyFunctionalInterface f = (input) -> {
            System.out.println(prefix + input); // captures 'prefix'
        };

        f.doSomething("World");
    }
}
```

Here, the lambda **implicitly captures** `prefix`, even though we didn’t pass it to the lambda directly. Java compiler turns this into a structure like:

```java
new MyFunctionalInterface() {
    public void doSomething(String input) {
        System.out.println(prefix + input);
    }
}
```

And `prefix` gets stored inside the synthetic class.

### 🧪 **Example: Lambda with Runnable**

```java
public class LambdaRunnableExample {
    public static void main(String[] args) {
        String taskName = "Download"; // effectively final

        Runnable task = () -> {
            System.out.println("Starting task: " + taskName);
        };

        Thread thread = new Thread(task);
        thread.start();
    }
}
```

---

### 🔍 What’s happening:

- `taskName` is **captured** by the lambda.
- Since it’s **effectively final** (not reassigned), Java allows it.
- The lambda becomes the implementation of `Runnable.run()`.