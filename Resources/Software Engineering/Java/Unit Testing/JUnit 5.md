# JUnit 5

We write tests so that we have an automated framework that helps us verify on an ongoing basis that the code continues to work in the future, and help identify if any later modification  unexpectedly breaks older functionality.

## Why use a testing framework?

Why not simply write another method that runs our original functionality and verifies output is correct?

For this, you’ll have to

1. Preparation
2. Provide Test Inpits
3. Run The Tests
4. Provide Expected Output
5. Verify Result
6. Do Something To Alert Developer if Test Fails

All of these things need to be done for every test we write, JUnit helps with reducing repeated work.

## JUnit 5 particulars

- JUnit 5 is totally different from JUnit 4.
- Engine is main platform that runs the tests.
- Jupiter is the newer API used for running JUnit 5 tests.
- Vintage is the API used for running older JUnit 4 tests.
- So for running tests on  a Maven project, you add two dependencies
    - Jupiter Engine: this is the main platform that runs the tests
    - Jupiter API: this is the API that tells the platform how /  what tests to run.
    
    If we want to run older tests, you will have to add the vintage dependency.
    
- We also add a Maven Surefire plugin that helps maven run JUnit tests, since it can’t do that using the IDE.
- When you add dependency, specify their scope as being test so that they are not included in the final build.

## Writing A JUnit Test

Suppose you have a MathUtils class, which has an add(int x, int y) method.

You put your tests in src/test/java.

You put your test inside the same package as the class you want to test. This helps

- Organise your tests better
- Leverage the package visibility features of Java, so you get access to protected methods.

The  \@Test annotation tells JUnit which methods need to be run for tests, and which classes are Test classes.

These will run without a main method; this means that, as a hack, JUnit can be used as a runner to run code that we want to run everytime we build our project.

No failures in JUnit, mean a success.

## Assertions

- assertQuals is used to determine if the two expected and actual values are equal. This comes in many types.
    - assertEquals: for plain object
    - assertArrayEquals: for comparing each element at corresponding indices in arrays
    - assertIterableEquals: same as assertArrayEquals but for iterables
- fail() fails the test, every time, no matter what happens.
- assertEquals can also have messages that are displayed when our test fails.
- assertThrows helps us check what exceptions are thrown by method. 
Here is the syntax. Your executable can be a lambda as well.

```java
assertThrows(Expected exception, Executable thatWillThrowTheException);
```

## Test Life Cycle

JUnit manages the lifecycle of your test class, it instantiates the class and runs the methods.

By default, JUnit 5 creates a new test class instance for every test method inside the class.

## LifeCycle Hooks

Since JUnit creates new test class instance, we might have to doo the same initialization seperately in each Test method that we write. @BeforeEach can help you do this.

This is where these hooks come in

- BeforeAll: Runs once before all tests run
- BeforeEach: Runs before each tests run
- AfterEach: Runs after each tests run
- AfterAll: Runs once after all tests run

<aside>
💡 **WARNING**
BeforeAll is ran even before the instance is even created.
AfterAll is ran after the instance has been terminate.
Thus, the methods with these annotations, and consequently the attributes that they interact with must be static

</aside>

## AssertAll

Run a bunch of assertions in just one statement.

Each assertion is supplied as a lambda, or any other executable.

```cpp
assertAll(
	() -> assertEquals(4, mathUtils.multiply(2*2)),
	() -> assertEquals(6, mathUtils.multiply(2*3)),
	() -> assertEquals(8, mathUtils.multiply(2*4))
)
```