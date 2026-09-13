---
layout: post
title:  "Create a Spring Boot Starter"
categories: [java,devex,springboot]
---

# How to Create a Spring Boot Starter

While working on Developer Experience(DevEx), you will have to develop enablers such as Library, Framework or SDK. If you have ever used `spring-boot-starter-web` or `spring-boot-starter-data-jpa`, you already know this magic feeling: you add a dependency, and suddenly beans are configured for you, properties are bound automatically, and everything "just works" as you expect.

In this post we are going to build that same magic ourselves, but with something much simpler to begin: a simple **Calculator** starter.

Here is the roadmap we will follow step by step:

1. `calculator-library`: the plain Java logic, no Spring Boot at all.
2. `calculator-starter-autoconfiguration`: the Spring Boot auto-configuration that turns our library into a bean.
3. `calculator-spring-boot-starter`: the actual "starter" dependency that applications will add.
4. `calculator-starter-sample`: a sample Spring Boot app that consumes the starter and exposes a REST API.

Let's start now!

---

## Before we start, lets first explain context loading and the bean lifecycle

Before looking at the four projects, it helps to understand what Spring Boot does with a starter. A starter does not execute application code by itself. It load dependencies and auto-configuration so that Spring can discover, create, and configure the right beans while the application context is starting.

### How the application context is loaded

When a Spring Boot application starts, `SpringApplication` creates an `ApplicationContext` and prepares its environment from sources such as `application.yml`, `application.properties`, command-line arguments, and environment variables. Spring then loads configuration classes from component scanning and auto-configuration imports. Conditions such as `@ConditionalOnClass`, `@ConditionalOnMissingBean`, and `@ConditionalOnProperty` decide which configuration and bean definitions should be registered.

### The lifecycle of a bean

For a typical singleton bean, the lifecycle looks like this:

1. Spring registers the bean definition while loading configuration.
2. Spring resolves constructor dependencies and creates the bean instance.
3. Dependency injection and property population take place.
4. Bean post-processors run, which may wrap the bean in a proxy for features such as transactions or caching.
5. Initialization callbacks run, including `@PostConstruct` and `InitializingBean` when they are used.
6. The bean is available for injection and use after the context has been refreshed.
7. During application shutdown, destruction callbacks such as `@PreDestroy` and `DisposableBean` are invoked.

The exact timing depends on the scope. Singleton beans are normally created once per context, while prototype beans are created each time they are requested. Web scopes such as request and session create beans according to the web request lifecycle.

### Common conditional loading in auto-configuration

Spring Boot auto-configuration is designed to be selective. Instead of always creating every bean, it evaluates conditions and only activates configuration when the application matches the expected scenario. This is what makes starters feel automatic without being intrusive.

The most common conditional annotations are:

- `@ConditionalOnClass`: activate configuration only when specific classes are present on the classpath. Use this when a starter should only apply if a required library is available. **Example**: you have MySQL dependencie in your `pom.xml` file.
- `@ConditionalOnMissingBean`: create a bean only if the application has not already defined one. Use this to provide sensible defaults while still allowing application-level overrides.
- `@ConditionalOnProperty`: activate configuration or a bean only when a property exists and has the expected value. Use this for feature flags and opt-in behavior. **Example**: You have property `com.mycompany.application.calculator.use=true` you set to `true` to indicate you allows the auto-configuration of your beans.
- `@ConditionalOnBean`: activate configuration only when another bean is already present in the context. Use this when one auto-configured component depends on another integration being enabled first.
- `@ConditionalOnResource`: activate configuration only when a specific resource file exists, such as a file under `classpath:`. Use this when behavior depends on configuration files or metadata being packaged with the application.
- `@ConditionalOnWebApplication`: activate configuration only for web applications. Use this when the beans only make sense in servlet or reactive web environments.
- `@ConditionalOnExpression`: activate configuration when a SpEL expression evaluates to `true`. Use this sparingly, because it is more flexible but usually less clear than property-based conditions.

In practice, production starters usually combine several of these conditions. For example, a configuration class may require a library on the classpath, check that the application is a web app, and back off if the user has already declared a custom bean.

### Different way to use the Calculator library

The same calculator can support several use cases:

- **Plain Java library:** use `Calculator` directly in a command-line tool, another library, or a non-Spring application. No application context is required.
- **Explicit Spring configuration:** declare a `Calculator` with `@Bean` when the application wants complete control over its operation and dependencies.
- **Auto-configured Spring bean:** let the starter create the `Calculator` when the required library is on the classpath and the application has not supplied its own bean.
- **Application override:** define a custom `Calculator` or change starter properties when the application needs different behavior. Conditions such as `@ConditionalOnMissingBean` allow application configuration to take precedence.

Let's really start now!

---

## Step 1 — The plain Java library

Every good starter begins with a good library. This is just regular Java, with **zero** Spring dependency. That's an important design choice: the library should be usable even outside of Spring. It can also be an existing library. **Example**: The Pulsar Spring Boot Starter is based on the Java Pulsar Client.

### `Operation.java`

First, a simple enum to represent the four operations we support:

```java
package com.practice.starter.calculator;

public enum Operation {
    ADDITION,
    SUBTRACTION,
    MULTIPLICATION,
    DIVISION
}
```

### `Calculator.java`

Then we have our `Calculator` class:

```java
package com.practice.starter.calculator;

import java.util.Objects;
import java.util.Random;

public class Calculator {

    private static final int RANDOM_BOUND = 100;

    private final Operation operation;
    private final Random random;

    public Calculator(Operation operation) {
        this(operation, new Random());
    }

    Calculator(Operation operation, Random random) {
        this.operation = Objects.requireNonNull(operation, "operation must not be null");
        this.random = Objects.requireNonNull(random, "random must not be null");
    }

    public int addition(int x, int y) {
        return x + y;
    }

    public int subtraction(int x, int y) {
        return x - y;
    }

    public int multiplication(int x, int y) {
        return x * y;
    }

    public int division(int x, int y) {
        return x / y;
    }

    public int operationCalculation(Operation operation, int x, int y) {
        return switch (operation) {
            case ADDITION -> addition(x, y);
            case SUBTRACTION -> subtraction(x, y);
            case MULTIPLICATION -> multiplication(x, y);
            case DIVISION -> division(x, y);
        };
    }

    public int randomCalculation() {
        int randomNumber1 = random.nextInt(RANDOM_BOUND);
        int randomNumber2 = random.nextInt(RANDOM_BOUND);
        return operationCalculation(operation, randomNumber1, randomNumber2);
    }
}
```

Nothing fancy here: the `Calculator` holds a default `Operation` (used by `randomCalculation()`), and exposes basic math methods. The package-private constructor that accepts a `Random` object is just there to make the class easier to test deterministically.

### `pom.xml` for the library

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.practice.starter.calculator</groupId>
    <artifactId>calculator-library</artifactId>
    <version>1.0-SNAPSHOT</version>
    <name>calculator-library</name>
    <url>http://maven.apache.org</url>

    <properties>
        <maven.compiler.source>21</maven.compiler.source>
        <maven.compiler.target>21</maven.compiler.target>
        <junit-jupiter.version>5.9.3</junit-jupiter.version>
        <maven-surefire-plugin.version>3.0.0-M7</maven-surefire-plugin.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>${junit-jupiter.version}</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
            </plugin>
            <plugin>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>${maven-surefire-plugin.version}</version>
            </plugin>
        </plugins>
        <finalName>calculator-library</finalName>
    </build>
</project>
```

Notice there is no Spring here at all. This module could be published and reused by any Java project, Spring or not.

---

## Step 2 — Turning the library into an auto-configuration

Now that we have our library, we want Spring Boot to be able to automatically create a `Calculator` bean for any application that needs one. This is the job of the **auto-configuration** module.

The idea is:
- Read a property that tells us what the default operation should be.
- Create a `Calculator` bean using that property.
- Only do this if a `Calculator` bean does not already exist (so a user can still override it).
- Only do this if the `Calculator` class is actually on the classpath.

### `CalculatorProperties.java`

First, we describe the configuration property using a simple POJO annotated with `@ConfigurationProperties`:

```java
package com.practice.starter.calculator.autoconfigure;

import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties(prefix = "starter.calculator")
public class CalculatorProperties {

    private String defaultOperation;

    public String getDefaultOperation() {
        return defaultOperation;
    }

    public void setDefaultOperation(String defaultOperation) {
        this.defaultOperation = defaultOperation;
    }
}
```

This means any application using our starter can now set, in its `application.properties`:

```properties
starter.calculator.defaultOperation=ADDITION
```

### `CalculatorAutoConfiguration.java`

Next, the actual auto-configuration class:

```java
package com.practice.starter.calculator.autoconfigure;

import com.practice.starter.calculator.Calculator;
import com.practice.starter.calculator.Operation;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
@ConditionalOnClass(Calculator.class)
@EnableConfigurationProperties(CalculatorProperties.class)
public class CalculatorAutoConfiguration {

    private final CalculatorProperties calculatorProperties;

    @Autowired
    public CalculatorAutoConfiguration(CalculatorProperties calculatorProperties) {
        this.calculatorProperties = calculatorProperties;
    }

    @Bean
    @ConditionalOnMissingBean
    public Calculator calculator() {
        Operation defaultOperation = Operation.valueOf(calculatorProperties.getDefaultOperation());
        return new Calculator(defaultOperation);
    }
}
```

Let's decode the annotations, because this is really the heart of the whole starter mechanism:

- `@ConditionalOnClass(Calculator.class)` — only activate this configuration if the `Calculator` class is present on the classpath. If someone removes the library dependency, this configuration simply won't kick in.
- `@EnableConfigurationProperties(CalculatorProperties.class)` — registers our properties class so Spring can bind `starter.calculator.*` values to it.
- `@ConditionalOnMissingBean` — only create the `calculator` bean if the application hasn't already defined its own `Calculator` bean. This lets developers override the default behavior if they want to.

### Registering the auto-configuration

Spring Boot needs to know that this class exists and should be picked up automatically. Since Spring Boot 2.7+ (and required in Spring Boot 3+/4+), this is done through a simple text file:

`src/main/resources/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`

```
com.practice.starter.calculator.autoconfigure.CalculatorAutoConfiguration
```

That's it. One line, one fully-qualified class name. This file is what makes Spring Boot's auto-configuration scanning find our class without any manual `@Import` in the consuming application.

### `pom.xml` for the auto-configuration module

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>4.1.1</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <groupId>com.practice.starter.calculator</groupId>
    <artifactId>calculator-starter-autoconfiguration</artifactId>
    <version>1.0-SNAPSHOT</version>
    <name>calculator-starter-autoconfiguration</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <maven.compiler.source>21</maven.compiler.source>
        <maven.compiler.target>21</maven.compiler.target>
        <calculator-library.version>1.0-SNAPSHOT</calculator-library.version>
    </properties>

    <dependencies>
        <!-- Source Dependencies -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-autoconfigure</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>com.practice.starter.calculator</groupId>
            <artifactId>calculator-library</artifactId>
            <version>${calculator-library.version}</version>
        </dependency>
        <!-- Test dependencies -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <skip>true</skip>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

Two dependencies matter most here: `spring-boot-autoconfigure` (to use the conditional annotations) and our own `calculator-library`.

---

## Step 3 — Packaging it as a starter

At this point we already have working auto-configuration. So why do we need a separate "starter" module at all?

The convention in the Spring Boot ecosystem is to separate:
- the **auto-configuration** (the logic), from
- the **starter** (the dependency developers actually add to their `pom.xml`).

The starter's `pom.xml` typically just aggregates the auto-configuration module plus any other dependencies the feature needs (like `spring-boot-starter` itself). This keeps things clean and matches how official Spring Boot starters are structured.

### Create the `pom.xml` for `calculator-spring-boot-starter`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>4.1.1</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>

	<groupId>com.practice.starter</groupId>
	<artifactId>calculator-spring-boot-starter</artifactId>
	<version>1.0-SNAPSHOT</version>
	<name>calculator-spring-boot-starter</name>
	<description>Demo project for Spring Boot</description>

	<properties>
		<maven.compiler.source>21</maven.compiler.source>
		<maven.compiler.target>21</maven.compiler.target>
		<calculator-library.version>1.0-SNAPSHOT</calculator-library.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter</artifactId>
		</dependency>
		<dependency>
			<groupId>com.practice.starter.calculator</groupId>
			<artifactId>calculator-starter-autoconfiguration</artifactId>
			<version>${calculator-library.version}</version>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
				<configuration>
					<skip>true</skip>
				</configuration>
			</plugin>
		</plugins>
	</build>
</project>
```

Here there is no Java code in this module at all! It's a "pom-only" style dependency (even though it's packaged as a jar) whose only job is to pull in:

1. `spring-boot-starter`: the base Spring Boot dependency.
2. `calculator-starter-autoconfiguration`: our auto-configuration, transitively bringing in `calculator-library` too.

Any application that wants calculator features now needs to add exactly **one dependency**: `calculator-spring-boot-starter`.

---

## Step 4 — Consuming the starter in a real application

Time for the payoff. Let's build a small Spring Boot REST API that uses our starter.

### Create the `pom.xml` for the sample application

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>4.1.1</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>

	<groupId>com.practice.starter.sample</groupId>
	<artifactId>calculator-starter-sample</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>calculator-starter-sample</name>
	<description>Demo project for Spring Boot</description>

	<properties>
		<maven.compiler.source>21</maven.compiler.source>
		<maven.compiler.target>21</maven.compiler.target>
		<calculator-spring-boot-starter.version>1.0-SNAPSHOT</calculator-spring-boot-starter.version>
	</properties>

	<dependencies>
		<!-- Source dependencies -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-webmvc</artifactId>
		</dependency>
		<dependency>
			<groupId>com.practice.starter</groupId>
			<artifactId>calculator-spring-boot-starter</artifactId>
			<version>${calculator-spring-boot-starter.version}</version>
		</dependency>
		<!-- Test dependencies -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-webmvc-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>
</project>
```

Notice the single line that brings in our whole calculator library feature:

```xml
<dependency>
    <groupId>com.practice.starter</groupId>
    <artifactId>calculator-spring-boot-starter</artifactId>
    <version>${calculator-spring-boot-starter.version}</version>
</dependency>
```

### `application.properties`

We choose the default operation for the random calculation feature:

```properties
starter.calculator.defaultOperation=ADDITION
```

### `Application.java`

A completely standard Spring Boot entry point — no special setup required:

```java
package com.practice.starter.sample;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {

	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}
}
```

### `CalculationController.java`

And finally, a REST controller that simply asks Spring to inject the `Calculator` bean for it:

```java
package com.practice.starter.sample.controller;

import com.practice.starter.calculator.Calculator;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/calculation")
public class CalculationController {

    private final Calculator calculator;

    public CalculationController(Calculator calculator) {
        this.calculator = calculator;
    }

    @GetMapping("/addition")
    public int getAddition(@RequestParam("x") int x, @RequestParam("y") int y) {
        return calculator.addition(x, y);
    }

    @GetMapping("/subtraction")
    public int getSubtraction(@RequestParam("x") int x, @RequestParam("y") int y) {
        return calculator.subtraction(x, y);
    }

    @GetMapping("/multiplication")
    public int getMultiplication(@RequestParam("x") int x, @RequestParam("y") int y) {
        return calculator.multiplication(x, y);
    }

    @GetMapping("/division")
    public int getDivision(@RequestParam("x") int x, @RequestParam("y") int y) {
        return calculator.division(x, y);
    }

    @GetMapping("/random")
    public int getRandomCalculation() {
        return calculator.randomCalculation();
    }
}
```

Look closely: **there is no `@Bean` method for `Calculator` anywhere in this project.** We never call `new Calculator(...)` ourselves. Spring Boot found our `CalculatorAutoConfiguration`, saw that no other `Calculator` bean existed, read the `starter.calculator.defaultOperation` property, and created the bean for us automatically.



### Trying it out

Once the app is running, you can hit the endpoints:

```
GET /api/calculation/addition?x=7&y=5        -> 12
GET /api/calculation/subtraction?x=10&y=3    -> 7
GET /api/calculation/multiplication?x=6&y=4  -> 24
GET /api/calculation/division?x=20&y=5       -> 4
GET /api/calculation/random                  -> a random result using DIVISION
```

---

## Putting it all together

Here is the dependency chain we just built, from the bottom up:

```
calculator-library
        ↑
calculator-starter-autoconfiguration
        ↑
calculator-spring-boot-starter
        ↑
calculator-starter-sample
```

- `calculator-library` knows nothing about Spring. It's pure business logic, reusable anywhere.
- `calculator-starter-autoconfiguration` wraps that logic in a Spring-friendly `@Configuration` class, guarded by `@ConditionalOnClass` and `@ConditionalOnMissingBean`, and wires up `@ConfigurationProperties`.
- `calculator-spring-boot-starter` is the "public face": the single dependency developers add.
- `calculator-starter-sample` proves the whole thing works: add the starter, set a property, inject the bean, done.

## Why this model is so important?

It might feel like overkill for a calculator (and honestly, for a calculator it is)! But this exact pattern is what every real Spring Boot starter uses (database drivers, messaging clients, security modules, etc...). Once you understand it with a toy example like this one, reading the source code of any real starter becomes much less intimidating.

## Conclusion

In this post, we implemented step by step how to develop your own Spring Boot Starter. Let's say you are a DevEx team, you will have to maintain internal tools for your developers.
As a conclusion here are some key point to keep in mind:

- Keep your core logic Spring-free when possible — it's easier to test and reuse.
- Auto-configuration classes should be conditional, so they don't fight with what the developer explicitly configures.
A good practice is to let the user of your starter decide if the application should load the dependencies by setting a condition.
Example:

`com.company.application.enabled=true`
```java
@ConditionalOnProperty(
    prefix = "com.company.application",
    name = "enabled",
    havingValue = "true",
    matchIfMissing = false
)
```
- The "starter" module itself can be almost empty; its whole purpose is to be a convenient, single dependency.

Happy coding, and enjoy building your own starters!

### Some useful links:
The source code on [github](https://github.com/HattoriHenzo/practice-springboot-autoconfiguration).
1. https://docs.spring.io/spring-boot/reference/features/developing-auto-configuration.html
2. https://www.youtube.com/watch?v=9m1bC57oWrc

