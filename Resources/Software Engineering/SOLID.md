# SOLID

# Introduction

TODO: EXPOUND ON THESE

[https://www.digitalocean.com/community/conceptual_articles/s-o-l-i-d-the-first-five-principles-of-object-oriented-design#single-responsibility-principle](https://www.digitalocean.com/community/conceptual_articles/s-o-l-i-d-the-first-five-principles-of-object-oriented-design#single-responsibility-principle)

- S = Single Responsibility - Each class/instance should have only 1 responsibility
- O = Open Closed - Objects should be open for extension closed for modification
- L = Liskov substitution - Instance objects can be replaced with subtypes without altering correctness
- I = Interface Segregation - Multiple specific interfaces are better than big general one
- D = Dependency Injection - High level and Low level modules should both depend on abstraction

## Software Design Patterns

TODO: Get stuff from here [https://refactoring.guru/design-patterns](https://refactoring.guru/design-patterns)

## **Other design principles**

- Code to interface not implementation
- Favor composition over inheritance (inheritance should NOT only be for reuse)
- Encapsulate what changes
- Strive for loose coupling between objects
- Principle of least knowledge = Talk only to immediate friends

# Single Responsibility

A class should have one and only one reason to change, meaning that a class should have only one job.

1. The class executing the business logic should not be the one making DB calls, or external HTTP calls.
2. If in future we shift to another DB, or start making GRPC clients, this should not affect our existing business logic; so keep these seperate, always.

# Open-Close Principle

This means that a class should be extendable without modifying the class itself.

Consider the following classes

```java
class Calculator {
	List<Shape> shapes;
	public Integer getAreas() {
		// different formulae to calculate area for different shapes
	};
}

interface Shape {
}

Triangle implements Shape
Square implements Shape
etc.
```

1. So if an area calculator class has different implementations for calculating area for different shapes, the formula for calculating area should ideally reside in the shape class itself.
2. Such that, in future introduction of shapes, the calculator class itself doesn’t need changes

# Liskov substitution

If class B is subtype of A, then we should be able to replace A with B without breaking the program.

This can be made sure by ensuring that subclasses extend the capabilities of the parent and not narrow it down.

Here is an example where this is violated

```jsx
interface Bike {
	void turnEngineOn();
	void accelerate();
}

class Motorbike implements Bike {
	boolean isEngineOn; 
	int speed;
	
	public void turnEngineOn() {
		this.isEngineOne = true;
	}
	
	public void accelerate() {
		this.speed += 10;
	}
}

class Bicycle implements Bike {
	public void turnEngineOn() {
		throw new AssertionError("there is no engine");
	}
	
	public void accelerate() {
		this.speed += 10;
	}
}
```

1. Here `Bicycle` narrowed down the functionality of the `Bike` interface.
2. Subtypes should always be able to replace parents

# Interface Segregation

A client should never be forced to implement an interface that it doesn’t use, or clients shouldn’t be forced to depend on methods they do not use.

Consider the following classes

```java
interface RestaurantEmployee {
	void washDishes();
	void serveCustomers();
	void cookFood();
}

class Waiter implements RestaurantEmployee {
	public void washDishes() {
		throw new AssertionError("don't know how to");
	}
	public void serveCustomers()...
	public void cookFood()...
}
```

1. This is wrong, since not all implementations of the `RestaurantEmployee` will be able to perform the functions assigned to it.
2. We should instead segregate common concerns to seperate interfaces.

```jsx
interface IWaitStaff {
	void serveCustomers();
	void takeOrders();
}

interface IKitchenStaff {
	void washDishes();
	void cookFood();
}
```

# Dependency Inversion

### What is a dependency?

If a class A has a need to use the object, or functions of another class B, then A has a dependency on B.

```java
public class TaskHandler {
	
	public DbDao dbDao;
	
	public TaskHandler(DbDao dbDao) {
		this.dbDao = dbDao;
	}

	public TaskResponse handle() {
		DbObject = dbDao.getObject();
		// do some more processing
		return taskResponse;
	}
}
```

Here TaskHandler has a dependency on DbDao.

### Dependency Injection

Giving an object it’s instance variables is called dependency injection.

The simplest way to do it is to use constructor injection

```java
Circle circle = new Circle(5, "red", SOLID);
```

Simply instanstiate an object with it’s dependents whenever they’re needed.

### Inversion of Control (in context of dependencies)

Here are the many definitions of IoC.

- Objects do not create other objects on which they rely on to do their work. Instead they get the objects that they need from an outside source.
- The Dependency Inversion principle states that our classes should depend upon interfaces or abstract classes instead of concrete classes and functions.

What this means in practice is that our User class does not have any direct dependency on a particular implementation; any implementation that meets the interface requirements can be used.

This page explains clearly why we need this [https://github.com/google/guice/wiki/Motivation](https://github.com/google/guice/wiki/Motivation)