# Decorator

Lets you attach new behaviors to objects by placing these objects inside special wrapper objects that contain the behaviors

# Pieces

- **Component Interface:** This is an abstract class or interface that defines the common interface for both the concrete components and decorators. It specifies the operations that can be performed on the objects.
- **Concrete Component:** These are the basic objects or classes that implement the Component interface. They are the objects to which we want to add new behavior or responsibilities.
- **Abstract Decorator**: This is an abstract class that also implements the Component interface and has a reference to a Component object. Decorators are responsible for adding new behaviors to the wrapped Component object.
- **Concrete Decorator**: These are the concrete classes that extend the Decorator class. They add specific behaviors or responsibilities to the Component. Each Concrete Decorator can add one or more behaviors to the Component.

# Example

### **decorators/DataSource.java: A common data interface, which defines read and write operations**

```kotlin
**package** refactoring_guru.decorator.example.decorators;

**public** **interface** **DataSource** {
    **void** writeData(**String** data);

    **String** readData();
}
```

### **decorators/FileDataSource.java: Simple data reader-writer**

```kotlin
**package** refactoring_guru.decorator.example.decorators;

**import** java.io.*;

**public** **class** **FileDataSource** **implements** DataSource {
    **private** **String** name;

    **public** FileDataSource(**String** name) {
        **this**.name = name;
    }

    **@Override
    public** **void** writeData(**String** data) {
        File file = **new** File(name);
        **try** (OutputStream fos = **new** FileOutputStream(file)) {
            fos.write(data.getBytes(), 0, data.length());
        } **catch** (IOException ex) {
            System.out.println(ex.getMessage());
        }
    }

    **@Override
    public** **String** readData() {
        **char**[] buffer = **null**;
        File file = **new** File(name);
        **try** (FileReader reader = **new** FileReader(file)) {
            buffer = **new** **char**[(**int**) file.length()];
            reader.read(buffer);
        } **catch** (IOException ex) {
            System.out.println(ex.getMessage());
        }
        **return** **new** **String**(buffer);
    }
}
```

### **decorators/DataSourceDecorator.java: Abstract base decorator**

```kotlin
**package** refactoring_guru.decorator.example.decorators;

**public** **abstract** **class** **DataSourceDecorator** **implements** DataSource {
    **private** DataSource wrappee;

    DataSourceDecorator(DataSource source) {
        **this**.wrappee = source;
    }

    **@Override
    public** **void** writeData(**String** data) {
        wrappee.writeData(data);
    }

    **@Override
    public** **String** readData() {
        **return** wrappee.readData();
    }
}
```

### **decorators/EncryptionDecorator.java: Encryption decorator**

```kotlin
**package** refactoring_guru.decorator.example.decorators;

**import** java.util.Base64;

**public** **class** **EncryptionDecorator** **extends** DataSourceDecorator {

    **public** EncryptionDecorator(DataSource source) {
        **super**(source);
    }

    **@Override
    public** **void** writeData(**String** data) {
        wrappee.writeData(encode(data));
    }

    **@Override
    public** **String** readData() {
        **return** decode(wrappee.readData());
    }

    **private** **String** encode(**String** data) {
        **byte**[] result = data.getBytes();
        **for** (**int** i = 0; i < result.length; i++) {
            result[i] += (**byte**) 1;
        }
        **return** Base64.getEncoder().encodeToString(result);
    }

    **private** **String** decode(**String** data) {
        **byte**[] result = Base64.getDecoder().decode(data);
        **for** (**int** i = 0; i < result.length; i++) {
            result[i] -= (**byte**) 1;
        }
        **return** **new** **String**(result);
    }
}
```

### **decorators/CompressionDecorator.java: Compression decorator**

```kotlin
**package** refactoring_guru.decorator.example.decorators;

**import** java.io.ByteArrayInputStream;
**import** java.io.ByteArrayOutputStream;
**import** java.io.IOException;
**import** java.io.InputStream;
**import** java.util.Base64;
**import** java.util.zip.Deflater;
**import** java.util.zip.DeflaterOutputStream;
**import** java.util.zip.InflaterInputStream;

**public** **class** **CompressionDecorator** **extends** DataSourceDecorator {
    **private** **int** compLevel = 6;

    **public** CompressionDecorator(DataSource source) {
        **super**(source);
    }

    **public** **int** getCompressionLevel() {
        **return** compLevel;
    }

    **public** **void** setCompressionLevel(**int** value) {
        compLevel = value;
    }

    **@Override
    public** **void** writeData(**String** data) {
        wrappee.writeData(compress(data));
    }

    **@Override
    public** **String** readData() {
        **return** decompress(wrappee.readData());
    }

    **private** **String** compress(**String** stringData) {
        **byte**[] data = stringData.getBytes();
        **try** {
            ByteArrayOutputStream bout = **new** ByteArrayOutputStream(512);
            DeflaterOutputStream dos = **new** DeflaterOutputStream(bout, **new** Deflater(compLevel));
            dos.write(data);
            dos.close();
            bout.close();
            **return** Base64.getEncoder().encodeToString(bout.toByteArray());
        } **catch** (IOException ex) {
            **return** **null**;
        }
    }

    **private** **String** decompress(**String** stringData) {
        **byte**[] data = Base64.getDecoder().decode(stringData);
        **try** {
            InputStream in = **new** ByteArrayInputStream(data);
            InflaterInputStream iin = **new** InflaterInputStream(in);
            ByteArrayOutputStream bout = **new** ByteArrayOutputStream(512);
            **int** b;
            **while** ((b = iin.read()) != -1) {
                bout.write(b);
            }
            in.close();
            iin.close();
            bout.close();
            **return** **new** **String**(bout.toByteArray());
        } **catch** (IOException ex) {
            **return** **null**;
        }
    }
}
```

### **Demo.java: Client code**

```kotlin
**package** refactoring_guru.decorator.example;

**import** refactoring_guru.decorator.example.decorators.*;

**public** **class** **Demo** {
    **public** **static** **void** main(**String**[] args) {
        **String** salaryRecords = "Name,Salary\nJohn Smith,100000\nSteven Jobs,912000";
        DataSourceDecorator encoded = **new** CompressionDecorator(
                                         **new** EncryptionDecorator(
                                             **new** FileDataSource("out/OutputDemo.txt")));
        encoded.writeData(salaryRecords);
        DataSource plain = **new** FileDataSource("out/OutputDemo.txt");

        System.out.println("- Input ----------------");
        System.out.println(salaryRecords);
        System.out.println("- Encoded --------------");
        System.out.println(plain.readData());
        System.out.println("- Decoded --------------");
        System.out.println(encoded.readData());
    }
}
```

### **OutputDemo.txt: Execution result**

```kotlin
Input ----------------
Name,Salary
John Smith,100000
Steven Jobs,912000
- Encoded --------------
Zkt7e1Q5eU8yUm1Qe0ZsdHJ2VXp6dDBKVnhrUHtUe0sxRUYxQkJIdjVLTVZ0dVI5Q2IwOXFISmVUMU5rcENCQmdxRlByaD4+
- Decoded --------------
Name,Salary
John Smith,100000
Steven Jobs,912000
```

- **Extending Functionality**: When you have a base component with basic functionality, but you need to add additional features or behaviors to it dynamically without altering its structure. Decorators allow you to add new responsibilities to objects at runtime.
- **Multiple Combinations of Features**: When you want to provide multiple combinations of features or options to an object. Decorators can be stacked and combined in different ways to create customized variations of objects, providing flexibility to users.
- **Legacy Code Integration**: When working with legacy code or third-party libraries where modifying the existing codebase is not feasible or desirable, decorators can be used to extend the functionality of existing objects without altering their implementation.
- **GUI Components**: In graphical user interface (GUI) development, decorators can be used to add additional visual effects, such as borders, shadows, or animations, to GUI components like buttons, panels, or windows.
- **Input/Output Streams**: Decorators are commonly used in input/output stream classes in languages like Java. They allow you to wrap streams with additional functionality such as buffering, compression, encryption, or logging without modifying the original stream classes.

# Where is it used?

- Consider a video streaming platform where users can watch movies and TV shows. Each video content may have additional features or options available, such as subtitles, language preferences, video quality options, and audio enhancements.
- Make Pizza, Dress, anything with multiple options that may or make not be applied
- Suppose we are building a coffee shop application where customers can order different types of coffee. Each coffee can have various optional add-ons such as milk, sugar, whipped cream, etc. We want to implement a system where we can dynamically add these add-ons to a coffee order without modifying the coffee classes themselves.