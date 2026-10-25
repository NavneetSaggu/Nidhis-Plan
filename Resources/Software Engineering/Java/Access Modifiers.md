# Access Modifiers

Specify accessibility or scope of a field, method, constructor, or class.

Four types:

| Access Modifier | within class | within package | outside package by subclass only | outside package |
| --- | --- | --- | --- | --- |
| **Private** | Y | N | N | N |
| **Default** | Y | Y | N | N |
| **Protected** | Y | Y | Y | N |
| **Public** | Y | Y | Y | Y |

In words:

1. **Private**: Access within class → cannot be accessed from outside the class.
2. **Default** (when unspecified): Access within package → cannot be accessed from outside package. 
3. **Protected**: Access within package, outside package through child class → cannot be accessed from outside package without making child class.
4. **Public**: Access everywhere → within the class, outside the class, within the package and outside the package.

There are many non-access modifiers, such as static, abstract, synchronized, native, volatile, transient, etc.