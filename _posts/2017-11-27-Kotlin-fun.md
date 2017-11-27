---
layout: default
title: "Kotlin fun"
date: 2017-11-27
---

# Overview

Kotlin has been getting a lot of press recently, most-likely because it’s now officially supported as an Android development language.

Spring is also providing more-and-more support in Framework 5 and Boot 2.

I’ve taken some time to look into Kotlin. I’ll not get into the pros and cons (apart from saying that it is fully compatible with Java and its massive ecosystem), but thought it worth sharing a trivial Boot app that shows it in action.

# Data Classes

Data classes cut out a lot of boilerplate code for typical simple beans. For a Customer class with id and name properties:

```kotlin
data class Customer(val name: String, var id: Long? = null)
```

Much like Lombok this gives a lot out-of-the-box, such as getters and setters, `toString()`, `equals()` and `hashCode()` with zero effort. It can also be used directly in Java code:

```java
  Customer c = new Customer("Bob", null);
  String name = c.getName();
```

# Services

## Interface

A simple CustomerService interface looks quite Java-like, except that types are declared after the variable / function:

```kotlin
  interface CustomerService {
    fun all(): Collection<Customer>
    fun byId(id: Long): Customer?
    fun insert(customer: Customer)
  }
```  

## Implementation

Kotlin can use all the usual Java libraries, such as Spring’s JDBC support for implementing the CustomerService:

```kotlin
  @Service
  @Transactional
  class JdbcTemplateCustomerService(private val jdbcTemplate: JdbcTemplate) : CustomerService {

    override fun all(): Collection<Customer> = jdbcTemplate.query("SELECT * FROM CUSTOMERS") { rs, index ->
        Customer(rs.getString("NAME"), rs.getLong("ID"))
    }
 
    // other methods / functions omitted
  }
```
 
As you can see, lambdas are a key part of the language. The part in curly braces is not the method body, but a lambda implementing the RowCallbackHandler functional interface. The JdbcTemplate is auto-wired into the class constructor.

# REST Controller

A RestController is similarly terse:

```kotlin
  @RestController
  class CustomerRestController(private val customerService: CustomerService) {

    @GetMapping("/customers")
    fun customers() = customerService.all()

  }
``` 

# Wiring it Together

Finally the wiring is straightforward:

```kotlin
  fun main(args: Array<String>) {
    SpringApplicationBuilder()
            .sources(JdbcApplication::class.java)
            .run(*args)
  }
```

For a bit more fun, the application can be seeded with a few Customers by altering the main method:

```kotlin
  fun main(args: Array<String>) {
    SpringApplicationBuilder()
            .initializers(beans {
                bean { SpringTransactionManager(ref()) }
                bean {
                    ApplicationRunner {
                        val customerService = ref<CustomerService>()
                        arrayOf("Alice", "Bob", "Charlie")
                                .map { Customer(name = it) }
                                .forEach { customerService.insert(it) }
                        customerService.all().forEach { println(it) }
                    }
                }
            })
            .sources(JdbcApplication::class.java)
            .run(*args)
  }
```

The beans `{ bean { } bean { } … }` stuff is declaring Spring beans using a Kotlin-specific DSL. On start-up it creates 3 Customers, and prints them to the console. Wiring up the SpringTransactionManager simply ensures that they are committed to the database.

It should be noted that all this can reside in a single file, say `JdbcApplication.kt` with a `@SpringBootApplication` annotation to perform all the auto-configuration, including set up of an H2 in-memory database:

```kotlin
  @SpringBootApplication
  class JdbcApplication
``` 

Whether it should be in a single file is debatable, however there is a growing movement towards grouping functionality by feature rather than logical layer. Micro-services are an example of feature-rather-than layer functional organisation.

# Run via curl

Finally:

```
u0886@VD-JAVA01-02 ~
$ curl http://localhost:8080/customers
[{"name":"Alice","id":1},{"name":"Bob","id":2},{"name":"Charlie","id":3}]
```

A `pom.xml` file for reference:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>jdbc</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>kotlin-jdbc</name>
    <description>Demo project for Spring Boot</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.0.M6</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <kotlin.compiler.incremental>true</kotlin.compiler.incremental>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.jetbrains.kotlin</groupId>
            <artifactId>kotlin-stdlib-jre8</artifactId>
        </dependency>
        <dependency>
            <groupId>org.jetbrains.kotlin</groupId>
            <artifactId>kotlin-reflect</artifactId>
        </dependency>
        <dependency>
            <groupId>org.jetbrains.exposed</groupId>
            <artifactId>exposed</artifactId>
            <version>0.8.9</version>
        </dependency>
        <dependency>
            <groupId>org.jetbrains.exposed</groupId>
            <artifactId>spring-transaction</artifactId>
            <version>0.8.9</version>
        </dependency>
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
        </dependency>
    </dependencies>

    <build>
        <sourceDirectory>${project.basedir}/src/main/kotlin</sourceDirectory>
        <testSourceDirectory>${project.basedir}/src/test/kotlin</testSourceDirectory>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            <plugin>
                <artifactId>kotlin-maven-plugin</artifactId>
                <groupId>org.jetbrains.kotlin</groupId>
                <configuration>
                    <compilerPlugins>
                        <plugin>spring</plugin>
                    </compilerPlugins>
                </configuration>
                <dependencies>
                    <dependency>
                        <groupId>org.jetbrains.kotlin</groupId>
                        <artifactId>kotlin-maven-allopen</artifactId>
                        <version>${kotlin.version}</version>
                    </dependency>
                </dependencies>
            </plugin>
        </plugins>
    </build>

    <repositories>
        <repository>
            <id>spring-milestones</id>
            <name>Spring Milestones</name>
            <url>https://repo.spring.io/milestone</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>

    <pluginRepositories>
        <pluginRepository>
            <id>spring-milestones</id>
            <name>Spring Milestones</name>
            <url>https://repo.spring.io/milestone</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </pluginRepository>
    </pluginRepositories>

</project>
```