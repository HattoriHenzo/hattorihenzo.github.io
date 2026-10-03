---

layout: post  
title: "Migrating from Spring Boot 2.7.3 and Java 11 to Spring Boot 4.x and Java 21 with OpenRewrite and GenAI"  
categories: [java,devex,ai]

---

![alt text](/assets/image_spring_boot_java_openrewrite.png)

## Overview

In this post, I am going to show you how to upgrade your Spring Boot application. The hard part is not the idea of moving from Java 11 to Java 21, or from Spring Boot 2.7 to Spring Boot 4. The hard part is the amount of repetitive code, the dependency realignment, and the small breaking changes that only show up after the first compile.

For this migration, I used a combination that works very well in practice: **OpenRewrite** for the large, mechanical refactoring work, and **GenAI models** for the reasoning-heavy parts such as recipe selection, error triage, and understanding framework changes that require human judgment.

The target in this case is an application called **Person Manager**, moving from:

- **Java 11** to **Java 21**
- **Spring Boot 2.7.3** to **Spring Boot 4.0.8**, on the path to the **4.1 line**

That combination gave me a migration process that was faster than doing everything manually, but still controlled enough to validate each step.

## Why OpenRewrite and GenAI work well together

To simply say:

- OpenRewrite is **deterministic**. If you run it, you will get the same result.
- GenAI is **probabilistic**, meaning that you may get different results for the same prompt. Its real strength is its ability to provide fairly accurate responses.
- Another important reason why we generally use both is the cost of tokens 💸. In an enterprise context, with hundreds or thousands of applications and a mix of many languages, frameworks, and technologies, using GenAI alone can become expensive.

As mentioned above, OpenRewrite is excellent at broad, deterministic changes across a codebase. If you need to replace imports, apply framework migration recipes, modernize testing annotations, or move many files to a new API baseline, it saves a lot of time.

GenAI is useful in a different way. It helps before and after the automated refactor:

- Before the migration, it helps identify which recipes are worth running.
- During the migration, it helps explain compile errors and dependency conflicts.
- After the migration, it helps you reason about behavior changes that cannot be fixed by a simple search-and-replace.

## Migration step by step

Here I describe the different steps used to migrate the Spring Boot application. It is important to follow these steps because this is one of the best approaches you can use to help yourself through the process.

### Step 1: Talk to your GenAI tool

I used GitHub Copilot, and you can use any IDE of your choice, such as IntelliJ or VS Code. I will not go deeply into the choice of model. You can choose Claude, GPT, or Gemini. It all depends on your use case and your preference. For this step, each model is able to help you.

Type the following prompt:

```text
You are an expert in application modernization and you know the tool OpenRewrite. My context is the current project implemented with Spring Boot 2.7.3 and Java 11.
I want you to migrate to Spring Boot 4.x and Java 21. I want you to follow these steps:
• explore the project to understand the dependencies
• describe the strategy to do the migration
• give me the list of OpenRewrite available recipes I have to use to migrate Spring Boot 4.x and Java 21 including all dependencies like JUnit, Lombok
• generate the OpenRewrite maven plugin to apply in the project
```

This is the kind of prompt you can submit to your model, and in response you will get an OpenRewrite plugin that looks like this:

```xml
<plugin>
  <groupId>org.openrewrite.maven</groupId>
  <artifactId>rewrite-maven-plugin</artifactId>
  <version>6.46.1</version>
  <configuration>
    <activeRecipes>
      <recipe>org.openrewrite.java.migrate.UpgradeToJava21</recipe>
      <recipe>org.openrewrite.java.spring.boot3.UpgradeSpringBoot_3_3</recipe>
      <recipe>org.openrewrite.java.spring.boot4.UpgradeSpringBoot_4_0</recipe>
      <recipe>org.openrewrite.java.testing.junit5.JUnit4to5Migration</recipe>
    </activeRecipes>
  </configuration>
  <dependencies>
    <dependency>
      <groupId>org.openrewrite.recipe</groupId>
      <artifactId>rewrite-spring</artifactId>
      <version>6.37.1</version>
    </dependency>
    <dependency>
      <groupId>org.openrewrite.recipe</groupId>
      <artifactId>rewrite-migrate-java</artifactId>
      <version>3.42.1</version>
    </dependency>
    <dependency>
      <groupId>org.openrewrite.recipe</groupId>
      <artifactId>rewrite-testing-frameworks</artifactId>
      <version>3.44.0</version>
    </dependency>
  </dependencies>
</plugin>
```

### ⚠️ Warning

*While your model is able to generate an OpenRewrite plugin section for you, you may face some small issues concerning the recipes. There is one important thing to understand about OpenRewrite and Moderne, the company backing it. You have free recipes provided by the community, and some recipes are available only through the paid SaaS offering from Moderne. At the time of writing this post, the most recent recipe available for Spring Boot is for version `4.0.8`.*

### Step 2: Execute the OpenRewrite plugin

You are now ready to execute the plugin and migrate your application. A good practice is to do a dry run first, so you can preview the changes that will be applied.

```bash
./mvnw rewrite:dryRun
```

If you are OK with those changes, you can run the plugin:

```bash
./mvnw rewrite:run
```

Good job! Your code has been migrated.

### Step 3: What changed in the codebase

The project moved its Java baseline from `11` to `21`, and its Spring Boot parent from `2.7.3` to `4.0.8`. That also pulled in a different set of starters and test modules aligned with the newer Spring Boot platform.

At the `pom.xml` level, the shift looked like this:

```diff
- <parent>
-   <groupId>org.springframework.boot</groupId>
-   <artifactId>spring-boot-starter-parent</artifactId>
-   <version>2.7.3</version>
- </parent>
+ <parent>
+   <groupId>org.springframework.boot</groupId>
+   <artifactId>spring-boot-starter-parent</artifactId>
+   <version>4.0.8</version>
+ </parent>

- <java.version>11</java.version>
+ <java.version>21</java.version>
```

For Java 21 builds, an update to the Lombok annotation processor configuration was made so the compiler setup stayed predictable.

The second major change was the Jakarta migration. Any code still importing `javax.*` packages had to move to `jakarta.*`. For JPA entities, that means changes like this:

```diff
- import javax.persistence.*;
+ import jakarta.persistence.*;
```

This is one of the places where OpenRewrite is very effective because the change is broad, repetitive, and usually safe when the recipe is correct.

The third layer was test modernization. Spring Boot 4 reorganizes some test packages and annotations, so test classes needed updates such as:

```diff
- import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
- import org.springframework.boot.test.mock.mockito.MockBean;
+ import org.springframework.boot.webmvc.test.autoconfigure.WebMvcTest;
+ import org.springframework.test.context.bean.override.mockito.MockitoBean;

- @MockBean
+ @MockitoBean
```

On the other hand, a mutable static URI string became a `URI_TEMPLATE` constant plus an instance variable initialized in `@BeforeEach`. Also, the local server port field was changed from `String` to `int` because the newer Spring Boot test setup expects the correct type for binding.

That change looked like this:

```diff
+ import org.springframework.boot.webtestclient.autoconfigure.AutoConfigureWebTestClient;

+ @AutoConfigureWebTestClient
  class PersonControllerITest {
-   private static String URI = "http://localhost:%s";
+   private static final String URI_TEMPLATE = "http://localhost:%s";
+   private String uri;

-   private String localServerPort;
+   private int localServerPort;

    @BeforeEach
    void setUp() {
-     URI = String.format(URI, localServerPort);
+     uri = String.format(URI_TEMPLATE, localServerPort);
    }
```

Another example was repository testing, where `DataJpaTest` moved to a new package:

```diff
- import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
+ import org.springframework.boot.data.jpa.test.autoconfigure.DataJpaTest;
```

This is typical of framework upgrades: a large part of the work is mechanical, but a smaller part depends on how your application actually uses the framework.

### Step 4: Run the tests

When the migration is finished, you have to run your unit tests to check whether everything went well. Of course, migration projects are not always easy, and you will very likely face some issues. Most of the time, you will have to finish some parts of your migration by hand.

## Where OpenRewrite helps most, and where it does not

OpenRewrite is strongest when the change is repetitive and well understood:

- `javax` to `jakarta`
- known deprecations
- package moves with established recipes
- large-scale dependency-aligned source rewrites

It is weaker when the migration depends on application-specific meaning:

- semantic behavior changes
- custom framework extensions
- dependency incompatibilities
- configuration restructuring
- runtime-only failures
- for those use cases, you can also write your own custom recipes

This is exactly why combining it with GenAI works well. Let OpenRewrite do the bulk transformation. Use GenAI to shorten the investigation cycle around the remaining failures. Then let the compiler, tests, and application runtime decide whether the migration is actually done.

## Conclusion

If you approach a migration like this as a fully manual rewrite, it is slow and error-prone. If you treat OpenRewrite as magic, you will also get into trouble. The better approach is to use each tool for what it does best.

Use **OpenRewrite** to remove repetitive migration work. Use **GenAI** to analyze the gaps, explain the failures, and speed up the manual fixes. Then verify everything with a real build and real tests.

That is the balance that makes upgrades to Java 21 and Spring Boot 4.x feel manageable instead of chaotic.

### Some useful links

1. The source code is available on [GitHub](https://github.com/HattoriHenzo/person-spring-openrewrite). Check out the branch **feature/migration-to-spring-boot-4-and-java-21** to see the changes.
2. [OpenRewrite Documentation](https://docs.openrewrite.org/)
3. [Spring Boot 4.0 Migration Guide](https://spring.io/blog/2024/07/29/spring-boot-4-0-is-here/)
4. [Java 21 Features](https://www.oracle.com/java/technologies/javase/21-relnotes.html)
5. [Jakarta EE 10](https://jakarta.ee/releases/10/)
