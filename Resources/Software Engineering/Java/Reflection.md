# Reflection

[https://www.baeldung.com/java-reflection](https://www.baeldung.com/java-reflection)

## Summary

Reflection is used to describe code which is able to inspect other code in the same system (or itself).

### Use 1 : Call Function x() On Object If Exists, Used By Frameworks

Say you have an object of an unknown type in Java, and you would like to call a 'doSomething' method on it if one exists. Java's static typing system isn't designed to support this unless the object conforms to a known interface, but using reflection, your code can look at the object and find out if it has a method called 'doSomething' and then call it if you want to.

So, to give you a code example of this in Java (imagine the object in question is foo) :

```java
Method method = foo.getClass().getMethod("doSomething", null);
method.invoke(foo, null);
```

### Use 2 : Look For Classes/Fields With Annotations, Used By Frameworks

One very common use case in Java is the usage with annotations. JUnit 4, for example, will use reflection to look through your classes for methods tagged with the @Test annotation, and will then call them when running the unit test.

And finally, yes, the concepts are pretty much similar in other statically typed languages which support reflection (like C#). In dynamically typed languages, the use case described above is less necessary (since the compiler will allow any method to be called on any object, failing at runtime if it does not exist), but the second case of looking for methods which are marked or work in a certain way is still common.

## Definition

Allows us to **inspect** or/and **modify** runtime attributes of classes, interfaces, fields, and methods. Used when we don't know their names at compile time.

We can instantiate new objects, invoke methods, and get or set field values using reflection.

## **Project Setup**

Classes used for this are bundled in [*java.lang.reflect*](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/reflect/package-summary.html) package. Just import:

```
import java.lang.reflect.*;
```

## The *getClass()* method

To get access to the class, method, and field of an instance we call the *getClass* method  

The returned *class* object (runtime representation of class) provides methods for accessing information about a class.

## Quick Start

```java
public class Person {
    private String name;
    private int age;
}
```

1. A *Person* object, that’s type *Object*
2. Try and find fields of the class of this object at runtime

```java
@Test
public void givenObject_whenGetsFieldNamesAtRuntime_thenCorrect() {
    Object person = new Person();
    Field[] fields = person.getClass().getDeclaredFields();

    List<String> actualFieldNames = getFieldNames(fields);

    assertTrue(Arrays.asList("name", "age")
      .containsAll(actualFieldNames));
}
```

```java
private static List<String> getFieldNames(Field[] fields) {
    List<String> fieldNames = new ArrayList<>();
    for (Field field : fields)
      fieldNames.add(field.getName());
    return fieldNames;
}
```

# More Complex Example

```java
public interface Eating {
    String eats();
}
```

*Animal* implementation of the *Eating* interface:

```java
public abstract class Animal implements Eating {

    public static String CATEGORY = "domestic";
    private String name;

    protected abstract String getSound();

    // constructor, standard getters and setters omitted
}
```

*Locomotion* which describes how an animal moves:

```java
public interface Locomotion {
    String getLocomotion();
}
```

*Goat* which extends *Animal* and implements *Locomotion*. 

```java
public class Goat extends Animal implements Locomotion {

    @Override
    protected String getSound() {
        return "bleat";
    }

    @Override
    public String getLocomotion() {
        return "walks";
    }

    @Override
    public String eats() {
        return "grass";
    }

    // constructor omitted
}
```

## Using Reflections

### Inspect Name Of Class (or Field / Method)

```java
@Test
public void givenObject_whenGetsClassName_thenCorrect() {
    Object goat = new Goat("goat");
    Class<?> clazz = goat.getClass();

    assertEquals("Goat", clazz.getSimpleName());
    assertEquals("com.baeldung.reflection.Goat", clazz.getName());
    assertEquals("com.baeldung.reflection.Goat", clazz.getCanonicalName());
}
```

### Create Class Object From Class Name

```java
@Test
public void givenClassName_whenCreatesObject_thenCorrect(){
    Class<?> clazz = Class.forName("com.baeldung.reflection.Goat");

    assertEquals("Goat", clazz.getSimpleName());
    assertEquals("com.baeldung.reflection.Goat", clazz.getName());
    assertEquals("com.baeldung.reflection.Goat", clazz.getCanonicalName());
}
```

Name passed to static *forName* method should include package; or we get *ClassNotFoundException*.

### Inspect Modifiers Used For Class (or Field / Method)

We can determine the modifiers used in a class by calling the *getModifiers* method which returns an *Integer.* Each modifier is a flag bit which is either set or cleared.

The [*java.lang.reflect.Modifier*](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/reflect/Modifier.html) class offers static methods that analyze the returned *Integer* for the presence or absence of a specific modifier.

```java
@Test
public void givenClass_whenRecognisesModifiers_thenCorrect() {
    Class<?> goatClass = Class.forName("com.baeldung.reflection.Goat");
    Class<?> animalClass = Class.forName("com.baeldung.reflection.Animal");

    int goatMods = goatClass.getModifiers();
    int animalMods = animalClass.getModifiers();

    assertTrue(Modifier.isPublic(goatMods));
    assertTrue(Modifier.isAbstract(animalMods));
    assertTrue(Modifier.isPublic(animalMods));
}
```

### Inspect Package Of Class

```java
@Test
public void givenClass_whenGetsPackageInfo_thenCorrect() {
    Goat goat = new Goat("goat");
    Class<?> goatClass = goat.getClass();
    Package pkg = goatClass.getPackage();

    assertEquals("com.baeldung.reflection", pkg.getName());
}
```

### Get SuperClass of Class

```java
@Test
public void givenClass_whenGetsSuperClass_thenCorrect() {
    Goat goat = new Goat("goat");
    String str = "any string";

    Class<?> goatClass = goat.getClass();
    Class<?> goatSuperClass = goatClass.getSuperclass();

    assertEquals("Animal", goatSuperClass.getSimpleName());
    assertEquals("Object", str.getClass().getSuperclass().getSimpleName());
}
```

### Get List Of Interfaces Implemented

```java
@Test
public void givenClass_whenGetsImplementedInterfaces_thenCorrect(){
    Class<?> goatClass = Class.forName("com.baeldung.reflection.Goat");
    Class<?> animalClass = Class.forName("com.baeldung.reflection.Animal");

    Class<?>[] goatInterfaces = goatClass.getInterfaces();
    Class<?>[] animalInterfaces = animalClass.getInterfaces();

    assertEquals(1, goatInterfaces.length);
    assertEquals(1, animalInterfaces.length);
    assertEquals("Locomotion", goatInterfaces[0].getSimpleName());
    assertEquals("Eating", animalInterfaces[0].getSimpleName());
}
```

If subclass does not directly implement interface with the *implements* keyword, that interface will not appear in the returned list. Eating does not appear for Goat object.

TODO: Read through and summarise constructor, field, method inspection

# **Inspecting Constructors**

With Java reflection, we can **inspect constructors** of any class and even **create class objects at runtime**. This is made possible by the [*java.lang.reflect.Constructor*](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/reflect/Constructor.html) class.

Earlier on, we only looked at how to get the array of *Constructor* objects, from which we were able to get the names of the constructors.

In this section, we will focus on how to retrieve specific constructors. In Java, as we know, no two constructors of a class share exactly the same method signature. So we will use this uniqueness to get one constructor from many.

To appreciate the features of this class, we will create a *Bird* subclass of *Animal* with three constructors. We will not implement *Locomotion* so that we can specify that behavior using a constructor argument, to add still more variety:

```
public class Bird extends Animal {
    private boolean walks;

    public Bird() {
        super("bird");
    }

    public Bird(String name, boolean walks) {
        super(name);
        setWalks(walks);
    }

    public Bird(String name) {
        super(name);
    }

    public boolean walks() {
        return walks;
    }

    // standard setters and overridden methods
}
```

Let's confirm by using reflection that this class has three constructors:

```
@Test
public void givenClass_whenGetsAllConstructors_thenCorrect() {
    Class<?> birdClass = Class.forName("com.baeldung.reflection.Bird");
    Constructor<?>[] constructors = birdClass.getConstructors();

    assertEquals(3, constructors.length);
}
```

Next, we will retrieve each constructor for the *Bird* class by passing the constructor's parameter class types in declared order:

```
@Test
public void givenClass_whenGetsEachConstructorByParamTypes_thenCorrect(){
    Class<?> birdClass = Class.forName("com.baeldung.reflection.Bird");

    Constructor<?> cons1 = birdClass.getConstructor();
    Constructor<?> cons2 = birdClass.getConstructor(String.class);
    Constructor<?> cons3 = birdClass.getConstructor(String.class, boolean.class);
}
```

There is no need for assertion since when a constructor with given parameter types in the given order does not exist, we will get a *NoSuchMethodException* and the test will automatically fail.

In the last test, we will see how to instantiate objects at runtime while supplying their parameters:

```
@Test
public void givenClass_whenInstantiatesObjectsAtRuntime_thenCorrect() {
    Class<?> birdClass = Class.forName("com.baeldung.reflection.Bird");
    Constructor<?> cons1 = birdClass.getConstructor();
    Constructor<?> cons2 = birdClass.getConstructor(String.class);
    Constructor<?> cons3 = birdClass.getConstructor(String.class,
      boolean.class);

    Bird bird1 = (Bird) cons1.newInstance();
    Bird bird2 = (Bird) cons2.newInstance("Weaver bird");
    Bird bird3 = (Bird) cons3.newInstance("dove", true);

    assertEquals("bird", bird1.getName());
    assertEquals("Weaver bird", bird2.getName());
    assertEquals("dove", bird3.getName());

    assertFalse(bird1.walks());
    assertTrue(bird3.walks());
}
```

We instantiate class objects by calling the *newInstance* method of *Constructor* class and passing the required parameters in declared order. We then cast the result to the required type.

It's also possible to call the default constructor using the *Class.newInstance()* method. However, this method has been deprecated since Java 9, and we shouldn't use it in modern Java projects.

For *bird1*, we use the default constructor which from our *Bird* code, automatically sets the name to bird and we confirm that with a test.

We then instantiate *bird2* with only a name and test as well, remember that when we don't set locomotion behavior as it defaults to false, as seen in the last two assertions.

# **Inspecting Fields**

Previously, we only inspected the names of fields, in this section, **we will show how to** **get and set their values at runtime**.

There are two main methods used to inspect fields of a class at runtime: *getFields()* and *getField(fieldName)*.

The *getFields()* method returns all accessible public fields of the class in question. It will return all the public fields in both the class and all superclasses.

For instance, when we call this method on the *Bird* class, we will only get the *CATEGORY* field of its superclass, *Animal*, since *Bird* itself does not declare any public fields:

```
@Test
public void givenClass_whenGetsPublicFields_thenCorrect() {
    Class<?> birdClass = Class.forName("com.baeldung.reflection.Bird");
    Field[] fields = birdClass.getFields();

    assertEquals(1, fields.length);
    assertEquals("CATEGORY", fields[0].getName());
}
```

This method also has a variant called *getField* which returns only one *Field* object by taking the name of the field:

```
@Test
public void givenClass_whenGetsPublicFieldByName_thenCorrect() {
    Class<?> birdClass = Class.forName("com.baeldung.reflection.Bird");
    Field field = birdClass.getField("CATEGORY");

    assertEquals("CATEGORY", field.getName());
}
```

We are not able to access private fields declared in superclasses and not declared in the child class. This is why we are not able to access the *name* field.

However, we can inspect private fields declared in the class we are dealing with by calling the *getDeclaredFields* method:

```
@Test
public void givenClass_whenGetsDeclaredFields_thenCorrect(){
    Class<?> birdClass = Class.forName("com.baeldung.reflection.Bird");
    Field[] fields = birdClass.getDeclaredFields();

    assertEquals(1, fields.length);
    assertEquals("walks", fields[0].getName());
}
```

We can also use its other variant in case we know the name of the field:

```
@Test
public void givenClass_whenGetsFieldsByName_thenCorrect() {
    Class<?> birdClass = Class.forName("com.baeldung.reflection.Bird");
    Field field = birdClass.getDeclaredField("walks");

    assertEquals("walks", field.getName());
}
```

If we get the name of the field wrong or type in an in-existent field, we will get a *NoSuchFieldException*.

We get the field type as follows:

```
@Test
public void givenClassField_whenGetsType_thenCorrect() {
    Field field = Class.forName("com.baeldung.reflection.Bird")
      .getDeclaredField("walks");
    Class<?> fieldClass = field.getType();

    assertEquals("boolean", fieldClass.getSimpleName());
}
```

Next, we will look at how to access field values and modify them. To be able to get the value of a field, let alone setting it, we must first set it's accessible by calling *setAccessible* method on the *Field* object and pass boolean *true* to it:

```
@Test
public void givenClassField_whenSetsAndGetsValue_thenCorrect() {
    Class<?> birdClass = Class.forName("com.baeldung.reflection.Bird");
    Bird bird = (Bird) birdClass.getConstructor().newInstance();
    Field field = birdClass.getDeclaredField("walks");
    field.setAccessible(true);

    assertFalse(field.getBoolean(bird));
    assertFalse(bird.walks());

    field.set(bird, true);

    assertTrue(field.getBoolean(bird));
    assertTrue(bird.walks());
}
```

In the above test, we ascertain that indeed the value of the *walks* field is false before setting it to true.

Notice how we use the *Field* object to set and get values by passing it the instance of the class we are dealing with and possibly the new value we want the field to have in that object.

One important thing to note about *Field* objects is that when it is declared as *public static*, then we don't need an instance of the class containing them, we can just pass *null* in its place and still obtain the default value of the field, like so:

```
@Test
public void givenClassField_whenGetsAndSetsWithNull_thenCorrect(){
    Class<?> birdClass = Class.forName("com.baeldung.reflection.Bird");
    Field field = birdClass.getField("CATEGORY");
    field.setAccessible(true);

    assertEquals("domestic", field.get(null));
}
```

# **Inspecting Methods**

In a previous example, we used reflection only to inspect method names. However, Java reflection is more powerful than that.

With Java reflection, we can **invoke methods at runtime** and pass them their required parameters, just like we did for constructors. Similarly, we can also invoke overloaded methods by specifying parameter types of each.

Just like fields, there are two main methods that we use for retrieving class methods. The *getMethods* method returns an array of all public methods of the class and superclasses.

This means that with this method, we can get public methods of the *java.lang.Object* class like *toString*, *hashCode*, and *notifyAll*:

```
@Test
public void givenClass_whenGetsAllPublicMethods_thenCorrect(){
    Class<?> birdClass = Class.forName("com.baeldung.reflection.Bird");
    Method[] methods = birdClass.getMethods();
    List<String> methodNames = getMethodNames(methods);

    assertTrue(methodNames.containsAll(Arrays
      .asList("equals", "notifyAll", "hashCode",
        "walks", "eats", "toString")));
}
```

To get only public methods of the class we are interested in, we have to use *getDeclaredMethods* method:

```
@Test
public void givenClass_whenGetsOnlyDeclaredMethods_thenCorrect(){
    Class<?> birdClass = Class.forName("com.baeldung.reflection.Bird");
    List<String> actualMethodNames
      = getMethodNames(birdClass.getDeclaredMethods());

    List<String> expectedMethodNames = Arrays
      .asList("setWalks", "walks", "getSound", "eats");

    assertEquals(expectedMethodNames.size(), actualMethodNames.size());
    assertTrue(expectedMethodNames.containsAll(actualMethodNames));
    assertTrue(actualMethodNames.containsAll(expectedMethodNames));
}
```

Each of these methods has the singular variation which returns a single *Method* object whose name we know:

```
@Test
public void givenMethodName_whenGetsMethod_thenCorrect() throws Exception {
    Bird bird = new Bird();
    Method walksMethod = bird.getClass().getDeclaredMethod("walks");
    Method setWalksMethod = bird.getClass().getDeclaredMethod("setWalks", boolean.class);

    assertTrue(walksMethod.canAccess(bird));
    assertTrue(setWalksMethod.canAccess(bird));
}
```

Notice how we retrieve individual methods and specify what parameter types they take. Those that don't take parameter types are retrieved with an empty variable argument, leaving us with only a single argument, the method name.

Next, we will show how to invoke a method at runtime. We know by default that the *walks* attribute of the *Bird* class is *false*, we want to call its *setWalks* method and set it to *true*:

```
@Test
public void givenMethod_whenInvokes_thenCorrect() {
    Class<?> birdClass = Class.forName("com.baeldung.reflection.Bird");
    Bird bird = (Bird) birdClass.getConstructor().newInstance();
    Method setWalksMethod = birdClass.getDeclaredMethod("setWalks", boolean.class);
    Method walksMethod = birdClass.getDeclaredMethod("walks");
    boolean walks = (boolean) walksMethod.invoke(bird);

    assertFalse(walks);
    assertFalse(bird.walks());

    setWalksMethod.invoke(bird, true);

    boolean walks2 = (boolean) walksMethod.invoke(bird);
    assertTrue(walks2);
    assertTrue(bird.walks());
}
```

Notice how we first invoke the *walks* method and cast the return type to the appropriate data type and then check its value. We then later invoke the *setWalks* method to change that value and test again.

# Why To Not Use Reflection

Because everything reflection does is at runtime, it is not eligible for any compile time optimizations. This makes reflection very slow and processing-heavy.

Because you refer to methods and field by their names, if there are any changes to either of them in the future, your entire reflection code will break down.