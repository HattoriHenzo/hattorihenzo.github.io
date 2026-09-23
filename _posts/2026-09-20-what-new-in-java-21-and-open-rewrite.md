---

layout: post  
title: "Create a Spring Boot Starter"  
categories: [java,devex]

---

![alt text](/assets/image_java_21.png)

Hello friend! If you are maintaining older Java applications, Java 21 is a very reasonable target if you are not willing to move to Java 25. Beside of being a Long Term Support (LTS) version, it gives you a modern baseline without forcing you to do a risky migration. Talking about migration, tools like [OpenRewrite](https://docs.openrewrite.org/) make the transition more easier than ever.

## 1\. Better performance and efficiency

Every Java release does not magically make an application twice faster, but moving to a newer LTS usually gives you a healthier JVM to run on. Java 21 continues that trend with runtime and Just-In-Time (JIT)optimization that can improve throughput, reduce latency, and make better use of current hardware.

## 2\. Modern language features that reduce boilerplate

One of the best reasons to upgrade is simply that - modern Java is nicer to read. Java 21 gives you language features that remove noise and the verbosity Java syntax used to suffer.

Below you have some syntax improvement:

```java
String name = "Simba";
String message = STR."[AFTER] Hello \{name}, I am your father!";
```

That is much easier on the eyes than older format-based code such as:

```java
String name = "Simba";
String message = String.format("[BEFORE] Hello, %s! Welcome to Java 21.", name);
```

The difference may look small in isolation, but these little improvements add up quickly in real codebases.

## 3\. Improved safety and clearer intent in code

Java 21 also helps you write code that says exactly what it is doing. Pattern matching, record patterns reduce the amount of casting, and defensive plumbing you have to write by hand.

That matters most in code that handles structured data: domain objects, events, DTOs, and request payloads. Instead of fighting the syntax, you can focus more directly on the shape of the data and the behavior you want.

## 4\. Access to a modern Java ecosystem

At some point, staying on an older Java version starts to create a gap between you and the improvement made by community. Libraries move on. Frameworks are optimizes for newer baselines. Tooling support gets better on current LTS releases and more awkward on old ones.

That is why upgrading to Java 21 is not only about new syntax. It is also about getting back onto the main road of the ecosystem instead of maintaining your own side path.

## 5\. Long-term maintainability and security

For most teams, the real value of Java 21 is stability. It is an LTS release, which means it is a strong place to land if you want to modernize once and then build on top of that foundation for a while.

From a maintenance perspective, that gives you a cleaner future path: fewer compatibility surprises, better vendor support, and less accumulated upgrade debt.

# Java 21 new features

The examples in this repository are intentionally small. The goal is not to show every corner of Java 21, but to make a few improvements easy to see without a lot of setup.

### 1\. String Templates

Here, you can find a comparison with the old and the new syntax:

```java
String name = "Simba";
String message = String.format("[BEFORE] Hello, %s! Welcome to Java 21.", name);
```

versus

```java
String name = "Simba";
String message = STR."[AFTER] Hello \{name}, I am your father!";
```

This kind of syntax reads closer to the final output, which is exactly what you want for messages, logs, and generated text.

### 2\. Pattern Matching for `switch`

The example below shows how branching logic becomes easier to follow when `switch` can work with richer type information. Instead of stacking `instanceof` checks and casts, the code can express its cases more directly:

```java
private static void before(Shape shape) {
    if (shape instanceof Rectangle r) {
        Message.displayMessage(String.format("[BEFORE] Area: %.2f", r.computeArea()));
    } else if (shape instanceof Square s) {
        Message.displayMessage(String.format("[BEFORE] Area: %.2f", s.computeArea()));
    } else if (shape instanceof Circle c) {
        Message.displayMessage(String.format("[BEFORE] Area: %.2f", c.computeArea()));
    } else {
        throw new IllegalArgumentException(String.format("Unknown shape: %s", shape));
    }
}
```

versus

```java
private static void afterExpression(Shape shape) {
    double area = switch (shape) {
        case Rectangle r -> r.computeArea();
        case Square s -> s.computeArea();
        case Circle c -> c.computeArea();
        default -> throw new IllegalArgumentException(String.format("Unknown shape: %s", shape));
    };
    Message.displayMessage(String.format("[AFTER] Area: %.2f", area));
}

private static double afterStatement(Shape shape) {
    switch (shape) {
        case Rectangle r: return r.computeArea();
        case Square s: return s.computeArea();
        case Circle c: return c.computeArea();
        default: throw new IllegalArgumentException(String.format("Unknown shape: %s", shape));
    }
}
```

That makes a real difference once a code path starts handling several possible input shapes.

### 3\. Record Patterns

Pattern matching is an interesting new feature:

```java
private static void before(Object object) {
    if  (object instanceof Point(float x, float y)) {
        Message.displayMessage(String.format("[BEFORE] X = %f, and Y = %f", x, y));
    }
}
```

versus

```java
private static void after(Object object) {
    if  (object instanceof Point point) {
        Message.displayMessage(String.format("[AFTER] X = %f, and Y = %f", point.x, point.y));
    }
}
```

You can destructure data closer to the point where you use it, instead of writing repetitive extraction code around it.

It is a small improvement syntactically, but it makes record-based code feel much more natural.

### 4\. Virtual Threads

Virtual threads are lightweight threads managed by the JVM rather than mapped one-to-one to operating system threads. They make it much easier to write concurrent code in the familiar thread-per-task style without paying the same cost in memory and scheduling overhead as traditional platform threads:

```java
private static void before() {
    Thread thread = new Thread(() -> Message.displayMessage("[BEFORE] Hello from a virtual thread!"));
    thread.start();
}
```

versus

```java
private static void after() {
    Thread thread = Thread.ofVirtual().start(() -> Message.displayMessage("[AFTER] Hello from a virtual thread!"));
    try {
        thread.join();
    } catch (InterruptedException e) {
        Message.displayMessage(e.getMessage());
    }
}
```

That matters most for applications that spend a lot of time waiting on I/O, such as web APIs, messaging consumers, and integration services. Instead of redesigning everything around complex asynchronous code, you can often keep a straightforward programming model while still scaling to many more concurrent tasks.

# A quick overview of Java 21 migration with OpenRewrite

This is where OpenRewrite becomes genuinely useful. Most of the time migration if diffcult because the work is repetitive, spread across many files, and easy to do inconsistently by hand. We all know that doing things by hand introduce many human errors. With OpenRewrite, the main concept is around recipes. And what is a [recipe](https://docs.openrewrite.org/concepts-and-explanations/recipes)?

> _A recipe represents a group of search and refactoring operations that can be applied to a_ [_Lossless Semantic Tree_](https://docs.openrewrite.org/concepts-and-explanations/lossless-semantic-trees)_. A recipe can represent a single, stand-alone operation or it can be linked together with other recipes to accomplish a larger goal such as a framework migration._

OpenRewrite helps with exactly that. You apply a recipe and it is applied all around your source code or your repositories, if you have the SaaS version (Moderne).A typical migration looks like this:

1.  Upgrade the Java version in the build tool.
2.  Run targeted OpenRewrite recipes to modernize APIs and syntax.
3.  Compile and test the application.
4.  Automate anything that remains manual.

### 1\. Using OpenRewrite with the Maven plugin

If your project already builds with Maven, the Maven plugin is usually the most straightforward starting point. It fits naturally into an existing build (in you IDE) and works well in Continous Integration (CI).

Example of configuration:

```xml
<plugin>
  <groupId>org.openrewrite.maven</groupId>
  <artifactId>rewrite-maven-plugin</artifactId>
  <version>latest</version>
  <configuration>
    <activeRecipes>
      <recipe>org.openrewrite.java.migrate.UpgradeToJava21</recipe>
    </activeRecipes>
  </configuration>
</plugin>
```

Then run:

```
mvn rewrite:run
```

And Voila! You have your migration to Java 21 done in just one command.

### 2\. Using the IntelliJ plugin

If you prefer to inspect changes interactively, the IntelliJ [plugin](https://plugins.jetbrains.com/plugin/23814-openrewrite) is a good option. It lets you run recipes from the IDE, review what will change, and apply updates incrementally. It is important to note that it is only available for the Ultimate version.

The workflow is generally:

*   open the project in IntelliJ,
*   install the OpenRewrite plugin,
*   choose a recipe set such as Java 21 migration recipes,
*   review the suggested code changes,
*   apply and validate the results.

This is a comfortable workflow when you want visibility first and automation second.

### 3\. Using custom OpenRewrite implementations

Built-in recipes will get you far, but not all the way in every codebase. If your organization has custom wrappers, internal frameworks, or very specific conventions, you may need your own recipes.

OpenRewrite supports custom Java-based recipes that can be packaged with your project or run in a build pipeline.

This approach is best when:

*   the project has legacy patterns not covered by built-in recipes,
*   the team wants codified modernization rules,
*   migration must be repeated across several repositories.

In practice, that hybrid approach is often the most effective one: start with the standard recipes, then fill the remaining gaps with custom rules.

## OpenRewrite recipe strategy for Java 21

If your goal is to move quickly without making the migration sloppy, start with the recipes that already exist. They are not a substitute for testing, but they are excellent at removing tedious mechanical work.

These recipes can help with:

*   upgrading language levels,
*   updating deprecated API usage,
*   modernizing collections or APIs,
*   preparing project code for new language constructs.

That saves time and, just as importantly, keeps the migration consistent across modules and repositories.

## Conclusion

Java 21 is a solid upgrade target because it improves both the experience of writing Java and the practicality of running it in production. You get a current LTS, cleaner language features, and a platform that aligns much better with where the ecosystem already is.

And if migration effort is the main thing holding you back, OpenRewrite removes a lot of the grind. It will not do all of the thinking for you, but it can do a large part of the repetitive work, which is usually the part we don't really like.

If you are still running on an older Java version, Java 21 is you next migration project.

### Some useful links:

The source code available on [github](https://github.com/HattoriHenzo/what-new-java-21).

1. [Java 21 changes](https://docs.oracle.com/en/java/javase/21/language/java-language-changes-release.html)
2. [OpenRewrite](https://docs.openrewrite.org/)