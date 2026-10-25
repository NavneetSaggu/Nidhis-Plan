# Escaping References

References to private variables can sometimes escape to outside the class, for eg. any Collections object in a getCollection() function.

This means these objects can be modified; goes against OOP principles.

There’s some workarounds to this:

1. Return a new object, initialized with the value of your attribute, in each get() call. 
    
    ```java
    return new ArrayList<String>(this.phoneNumbers);
    ```
    
2. Return an immutable collection, for eg. unmodifyableMap or unmodifyableList in Java etc.
    
    ```java
    return new Collections.unmodifyableList(this.phoneNumbers);
    ```
    

Only really seems to be an issue in Java. [This](https://www.youtube.com/watch?v=nW48K2a6t7k&ab_channel=VirtualPairProgrammers) video is pretty much summed up in these notes.