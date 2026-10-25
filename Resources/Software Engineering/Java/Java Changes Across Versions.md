# Java Changes Across Versions

- Java SE 1.0 - The initial release of Java in 1996, included basic features such as the Java virtual machine (JVM), applet support, and the Java Development Kit (JDK).
- Java SE 1.1 - Released in 1997, introduced new features such as JDBC (Java Database Connectivity), inner classes, and RMI (Remote Method Invocation).
- **Java SE 1.2 - Released in 1998, introduced the Collections framework, JIT (Just-in-Time) compiler, and support for reflection.**
- Java SE 1.3 - Released in 2000, included new features such as the Jar Indexing API, Java Sound API, and the HotSpot JVM.
- Java SE 1.4 - Released in 2002, introduced the assert keyword, regular expressions, and NIO (New Input/Output) API.
- **Java SE 5.0 - Released in 2004, introduced major language enhancements such as generics, annotations, autoboxing, and enhanced for-loop.**
- Java SE 6 - Released in 2006, introduced new features such as scripting language support, JDBC 4.0, and improved JMX (Java Management Extensions) API.
- Java SE 7 (LTS release) - Released in 2011, introduced new features such as the try-with-resources statement, multi-catch statement, and support for dynamic languages.
- **Java SE 8 (LTS release) - Released in 2014, introduced major language enhancements such as lambda expressions, the Stream API, and the Date and Time API, and optionals.**

Up until Java 8, the Oracle website offered JREs and JDKs as separate downloads - even though the JDK also always included a JRE in a separate folder. With Java 9 that distinction was basically gone, and you are *always* downloading a JDK. The directory structure of JDKs also changed, with not having an explicit JRE folder anymore.

We also shifted from using a 1.x versioning to x versioning.

- Java SE 9 - Released in 2017, introduced the Java Platform Module System (JPMS), which provides better modularization and enhanced security.
- Java SE 10 - Released in 2018, introduced new features such as local variable type inference and the ability to run Java applications directly from a source code file.
- **Java SE 11 (LTS release) - Released in 2018, introduced support for HTTP/2 and TLS 1.3, as well as new features such as the `var` keyword, Unicode 10.0, improved garbage collection.**
- Java SE 12 - Released in 2019, introduced new features such as switch expressions and improvements to the garbage collector.
- Java SE 13 - Released in 2019, introduced new features such as text blocks, which make it easier to work with multi-line strings.
- Java SE 14 - Released in 2020, introduced new features such as records, which provide a compact way to define classes that are primarily used for holding data. Standardized switch expressions, heplful NPEs.
- Java SE 15 - Released in 2020, introduced new features such as sealed classes, which restrict the subclasses that can be defined for a class.
- Java SE 16 - Released in 2021, introduced new features such as pattern matching for instanceof and improved garbage collection.
- Java SE 17 (LTS release) - Released in 2021, introduced new features such as pattern matching for instanceof and improved garbage collection. Sealed Classes allow you to control who is allowed to subclass your class.
- Java SE 19 - Released in 2022, added a couple of exciting features, like `*Virtual Threads*` and the new `*Foreign Function & Memory API*`, as well as `*Structured Concurrency*` and the `*Vector API*` - but they are all in preview mode, thus subject to change across the next releases.
- Java SE 20 - Released in 2023, continuing iterative work on previous features, but all WIP.