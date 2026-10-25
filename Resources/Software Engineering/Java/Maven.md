# Maven

# I**nheritance** and A**ggregation**

From [https://devflection.com/posts/2020-04-12-maven-part-3/](https://devflection.com/posts/2020-04-12-maven-part-3/)

## **Inheritance**

To inherit configuration from a parent project, the "parent" element is added to the child project POM, which includes the unique identifiers of the parent project (groupId, artifactId, and version). The optional "relativePath" element specifies a relative directory for Maven to search for the parent project before searching local and remote repositories.

An example of a child project POM:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
		 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		 xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">

		<modelVersion>4.0.0</modelVersion>

		<!-- Here we define the parent -->
		<parent>
        <groupId>com.devflection</groupId>
        <artifactId>maven-parent-project</artifactId>
        <version>1.0-SNAPSHOT</version>
				<!-- This is optional -->
				<relativePath>../maven-parent-project</relativePath>
	  </parent>

		<artifactId>maven-inheritance-example</artifactId>
		<packaging>jar</packaging>

</project>
```

- Adding the parent element to a project in Maven allows it to inherit most of the configuration from the parent POM.
- Inheritance includes the groupId, version, and most other configurations done in the parent POM.
- The only elements that are not inherited are the artifactId, name, and prerequisites.
- The parent project only needs to set the packaging element to "pom".

An example of a parent project POM:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
		 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		 xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">

		<modelVersion>4.0.0</modelVersion>

    <groupId>com.devflection</groupId>
    <artifactId>maven-parent-project</artifactId>
    <version>1.0-SNAPSHOT</version>
		<!-- This needs to be set for parent and aggregator projects -->
		<packaging>pom</packaging>

</project>
```

## **Dependency management and plugin management**

- **Dependency management** allows for configuring default values for dependencies and their versions in the parent project POM.
- **Plugin management** works similarly, with plugin configurations defined in the parent project POM and referenced in child project POMs.
- This simplifies version management as changing versions in the parent project applies to all child projects.
- Overwriting values in child projects allows for customised configurations without duplicating code.
- An example parent project POM with dependencyManagement and pluginManagement configurations can be provided.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
		 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		 xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">

		<modelVersion>4.0.0</modelVersion>

    <groupId>com.devflection</groupId>
    <artifactId>maven-parent-project</artifactId>
    <version>1.0-SNAPSHOT</version>
		<packaging>pom</packaging>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>junit</groupId>
                <artifactId>junit</artifactId>
                <version>4.11</version>
                <scope>test</scope>
            </dependency>
            <dependency>
                <groupId>org.apache.commons</groupId>
                <artifactId>commons-lang3</artifactId>
                <version>3.9</version>
            </dependency>
        </dependencies>
    </<dependencyManagement>>

    <build>
        <pluginManagement>
            <plugins>
                <plugin>
                    <artifactId>maven-antrun-plugin</artifactId>
                    <version>1.8</version>
                    <executions>
                        <execution>
                            <phase>clean</phase>
                            <configuration>
                                <target>
                                    <echo>Hello world from the antrun plugin!</echo>
                                </target>
                            </configuration>
                            <goals>
                                <goal>run</goal>
                            </goals>
                        </execution>
                    </executions>
                </plugin>
            </plugins>
        <pluginManagement>
		</build>

</project>

```

In this POM, we configured the dependencies and the plugin, but they are not used on every child project by default. The child projects still need to reference the dependency or plugin if they want to use/execute it.The way to reference them is to just add them to the dependency or plugin element, but we need only the name of the dependency or plugin, since everything else is configured in the parent POM.

An example of the child POM:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
		 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		 xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">

	<modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.devflection</groupId>
        <artifactId>maven-parent-project</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <artifactId>maven-child-project</artifactId>
	<packaging>jar</packaging>

    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <artifactId>maven-antrun-plugin</artifactId>
            </plugin>
        </plugins>
	</build>

</project>

```

We can see that we can omit the version and scope in the dependencies since it is taken from the parent project and in the plugin, we just configure the name of the plugin we want to run.

> If we are using dependency management, we might encounter some issues with transitive dependencies. An example of this might be that our project has a dependency to apache-commons library version 3.7, and we also have another dependency to a library, that has a dependency to apache-commons library version 3.9. So we are transitively depending on apache-commons version 3.9. But since our dependency management overrides and forces the second dependency to use an older version of apache-commons, we might have some failure.
> 

## **Aggregation**

The second concept is **aggregation**, which allows us to execute a Maven command against the aggregator project and the same command then gets propagated to the modules below that are specified in the aggregator project in the **module** element.Same as it was for the parent project, an aggregator project also has to have the packaging of the project set to *pom*.

An example aggregator project POM:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
		 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		 xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">

	<modelVersion>4.0.0</modelVersion>

    <groupId>com.devflection</groupId>
    <artifactId>maven-aggregator-project</artifactId>
    <version>1.0-SNAPSHOT</version>
	<packaging>pom</packaging>

      <modules>
        <module>my-first-module</module>
        <module>my-second-module</module>
    </modules>

</project>

```

If we run **mvn package** against the aggregator project, it will propagate the same command to *my-first-module* and *my-second-module*.An important point is that the order of how the modules appear in the POM does not matter because Maven will figure out the dependencies between them and build them in the correct order based on that.

So if in our example, the project *my-first-module* depends on the project *my-second-module*, Maven will first run the command against project *my-second-module* and then against the project *my-first-module*, which is the opposite order of how we wrote them in the modules element.

## **Summary**

We can see that **inheritance** is useful when we have some base configuration that is shared across multiple projects. We can extract the configuration to a parent project and have all of the child projects inherit from the parent. This makes configuration easier to manage since we only need to make changes on one file (the parent POM), and it also simplifies the child project POM files.

On the other hand **aggregation** is useful when we have multiple projects that are somehow linked (maybe they depend on each other or they form a logical unit together) and it makes sense for them to all be built together.

A lot of real-world projects often use both concepts in the same project.This means that the same root project is both a parent and an aggregator. So what we have is a parent aggregator project that contains the shared configuration of all the child projects and also lists all the child projects as modules. Then we can execute a Maven command against this root aggregator project and the Maven command is forwarded to all the child projects that are specified as modules.

The benefit of this approach is that we can separate our big project into multiple smaller modules, which are still built together by executing a single Maven command against the root aggregator project.

## **Example**

For our example, we will pretend we are working for a bank that specializes in calculating mortgages.We will create a monorepo that will contain all of our separate projects for different parts. The project structure will look like:

```
>devflection-bank
    >devflection-bank-core
    >devflection-bank-desktop
    >devflection-bank-web

```

The root project *devflection-bank* will be a parent and aggregator project. The core module will contain our business logic (i.e. calculating mortgages), the desktop module will contain the desktop app GUI code and the web-app module will contain the web application code. Both of the app modules will depend on the core module.

Let us have a look at the POMs and we will note down the important parts for this article.

## **Parent POM**

The parent POM:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.devflection</groupId>
    <artifactId>devflection-bank</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>

    <modules>
        <module>devflection-bank-core</module>
        <module>devflection-bank-desktop</module>
        <module>devflection-bank-web</module>
    </modules>

    <properties>
        <apache.commons.lang3.version>3.9</apache.commons.lang3.version>
        <junit.version>4.11</junit.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>junit</groupId>
                <artifactId>junit</artifactId>
                <version>${junit.version}</version>
                <scope>test</scope>
            </dependency>
            <dependency>
                <groupId>org.apache.commons</groupId>
                <artifactId>commons-lang3</artifactId>
                <version>${apache.commons.lang3.version}</version>
            </dependency>
            <dependency>
                <groupId>com.devflection</groupId>
                <artifactId>devflection-bank-core</artifactId>
                <version>1.0-SNAPSHOT</version>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <pluginManagement>
            <plugins>
                <plugin>
                    <artifactId>maven-antrun-plugin</artifactId>
                    <version>1.8</version>
                    <executions>
                        <execution>
                            <phase>clean</phase>
                            <configuration>
                                <target>
                                    <echo>Hello from the antrun plugin!</echo>
                                </target>
                            </configuration>
                            <goals>
                                <goal>run</goal>
                            </goals>
                        </execution>
                    </executions>
                </plugin>
            </plugins>
        </pluginManagement>
    </build>

</project>

```

The important parts for this example are:

- Packaging type is set to POM
- All three child projects are listed as modules
- Dependency management specifies the versions for the shared dependencies (Apache commons, JUnit and our core module)
- Plugin management configures an example maven-antrun-plugin with some defaults

## **Core module POM**

Next, lets look at the core module POM:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.devflection</groupId>
        <artifactId>devflection-bank</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <artifactId>devflection-bank-core</artifactId>
    <packaging>jar</packaging>

    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
<!-- This is the plugin we configured in the parent and is reused by all modules --><plugin>
                <artifactId>maven-antrun-plugin</artifactId>
                <executions>
                    <execution>
                        <configuration>
                            <target>
                                <echo>Hello from devflection-bank-core!</echo>
                            </target>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

</project>

```

Here the important points are:

- We defined the parent project
- We defined the dependencies (JUnit and Apache commons), but without the version, since this is managed by the parent
- We defined the example plugin, but we only have to define the name and the things we want to override (in this case we want to change the actual message)

## **Dekstop app POM**

Next up is the desktop module:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.devflection</groupId>
        <artifactId>devflection-bank</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <artifactId>devflection-bank-desktop</artifactId>
    <packaging>jar</packaging>

    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
        </dependency>
        <dependency>
            <groupId>com.devflection</groupId>
            <artifactId>devflection-bank-core</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
<!-- This plugin is used to create an executable jar file for the desktop app--><plugin>
                <artifactId>maven-assembly-plugin</artifactId>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <archive>
                        <manifest>
                            <mainClass>com.devflection.bank.desktop.DevflectionBankGUI</mainClass>
                        </manifest>
                    </archive>
                    <descriptorRefs>
                        <descriptorRef>jar-with-dependencies</descriptorRef>
                    </descriptorRefs>
                </configuration>
            </plugin>
<!-- This is the plugin we configured in the parent and is reused by all modules --><plugin>
                <artifactId>maven-antrun-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>

```

For the desktop app, the important points are:

- We defined the parent
- We defined the dependencies (here we also depend on the core module) without versions since they are managed by the parent
- We reference the example plugin that is in the parent, except here we are not overriding anything

## **Webapp POM**

And finally we have our webapp:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.devflection</groupId>
        <artifactId>devflection-bank</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <artifactId>devflection-bank-web</artifactId>
    <packaging>jar</packaging>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.2.6.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>com.devflection</groupId>
            <artifactId>devflection-bank-core</artifactId>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
<!-- This plugin is used to package the Spring Boot application into an executable jar file --><plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <mainClass>com.devflection.DevflectionBankWebapp</mainClass>
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
<!-- This is the plugin we are overriding from the parent POM --><plugin>
                <artifactId>maven-antrun-plugin</artifactId>
                <executions>
                    <execution>
                        <configuration>
                            <target>
                                <echo>Hello from devflection-bank-webapp!</echo>
                            </target>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

</project>

```

The important points here are:

- We defined the parent
- Dependency management is defined so we can inherit the version for Spring. More about this [here](http://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html#Importing_Dependencies)
- We defined the dependencies without versions
- We defined the example plugin as well and overrode the message here as well

# Packaging

## Introduction

[Maven](https://www.baeldung.com/maven) offers many default packaging types that include a *jar*, *war*, *ear*, *pom*, *rar*, *ejb*, and *maven-plugin*. **Each packaging type follows a build lifecycle that consists of phases.** Usually, every phase is a sequence of goals and performs a specific task.

**Different packaging types may have different goals in a particular phase.** For example, in the package phase of *jar* packaging type, *maven-jar-plugin*‘s jar goal is executed. Conversely, for a *war* project, *maven-war-plugin*‘s war goal is executed in the same phase.

## jar

Java archive – or *jar* – is one of the most popular packaging types. Projects with this packaging type produce a compressed zip file with the *.jar* extension. It may include pure Java classes, interfaces, resources, and metadata files.

To begin with, let's look at some of the default goal-to-build-phase bindings for the *jar*:

- resources: *resources*
- compiler: *compile*
- resources: *testResources*
- compiler: *testCompile*
- surefire: *test*
- **jar: *jar***
- install: *install*
- deploy: *deploy*

Without delay, let's define the packaging type of a *jar* project:

```xml
<packaging>jar</packaging>
```

**If nothing has been specified, Maven assumes the packaging type is a *jar.***

## war

Simply put, a web application archive – or *war* – contains all files related to a web application. It may include Java servlets, JSPs, HTML pages, a deployment descriptor, and related resources. Overall, *war* has the same goal bindings as a *jar*, but with one exception —the package phase of the *war* has a different goal, which is *war*.

Without a doubt, *jar* and *war* are the most popular packaging types in the Java community. A detailed [difference between these two](https://www.baeldung.com/java-jar-war-packaging) might be an interesting read.

Let's define the packaging type of a web application:

```xml
<packaging>war</packaging>
```

**The other packaging types *ejb*, *par*, and *rar* also have similar lifecycles, but each has a different package goal. (**ejb:ejb or par:par or rar:rar**)**

## pom

Among all packaging types, *pom* is the simplest one. It helps to create aggregators and parent projects.

**An aggregator or [multi-module project](https://www.baeldung.com/maven-multi-module) assembles submodules coming from different sources.** These submodules are regular Maven projects and follow their own build lifecycles. The aggregator POM has all the references of submodules under the *modules* element.

**A parent project allows you to define the inheritance relationship between POMs.**  The parent POM shares certain configurations, plugins, and dependencies, along with their versions. Most elements from the parent are inherited by its children — exceptions include *artifactId*, *name*, and *prerequisites*.

Because there are no resources to process and no code to compile or test. Hence, the artifacts of pom projects generate themselves instead of any executable.

Let's define the packaging type of a multi-module project:

```xml
<packaging>pom</packaging>
```

**Such projects have the simplest lifecycle that consists of only two steps: *install* and *deploy*.**

# Dependency Resolution