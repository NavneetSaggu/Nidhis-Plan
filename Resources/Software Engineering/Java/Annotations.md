# Annotations

These can be put on classes, objects, parameters, functions; just about everything.

Annotations can be created to be simply used as labels

```java
@VeryImportantClass
public class Test {
	public @interface VeryImportantClass {}
	
	public static void main(String[] args) {
		System.out.println("In an important class");
	}
}
```

## Making Custom Annotations

Annotations can be customised by using the following annotations

- **@Target** 
Specifies what kind of java element this annotation can be used on. This can be left out if we want the annotation to be usable for any Java element.
    - ElementType.TYPE is given if the annotation is to be used with classes
    - ElementType.METHOD is given if the annotation is to be used with methods
    
    Multiple targets can be listed as an array
    
    ```java
    @Target({ElementType.TYPE, ElementType.METHOD})
    public @interface MyAnnotation {}
    ```
    
- **@Retention**
99% of the time we’ll use RetentionPolicy.RUNTIME. This tells Java to keep the annotation around through the running of the program so that the other code can look at this annotation and use it while the program is running.
    
    
    | RetentionPolicy.SOURCE | refers to the source code, discarded during compilation. It will not be available in the compiled class. |
    | --- | --- |
    | RetentionPolicy.CLASS | refers to the .class file, available to java compiler but not to JVM . It is included in the class file. |
    | RetentionPolicy.RUNTIME | refers to the runtime, available to java compiler and JVM . |
- hehe