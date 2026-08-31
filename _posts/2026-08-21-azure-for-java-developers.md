---
layout: post
title:  "Azure for Java Developers"
---

In this article, I will progressively introduce you to the benefits of using Java on Azure.

![alt text](/assets/image_java_on_azure.png)

## What Is Microsoft Azure?

Simply put, **Microsoft Azure** is a **cloud computing platform** provided by Microsoft. It allows organizations and individuals to build, deploy, run, and manage applications and IT infrastructure using Microsoft’s global network of data centers, organized into [regions](https://learn.microsoft.com/en-us/azure/reliability/regions-overview) and [availability zones](https://learn.microsoft.com/en-us/azure/reliability/availability-zones-overview?tabs=azure-cli).

Azure provides hundreds of cloud services covering areas such as **computing, storage, databases, networking, security, artificial intelligence, analytics, DevOps, and containers**.

Azure supports three major types of services:

* **SaaS (Software as a Service):** Microsoft-hosted applications and services consumed directly by users.
* **PaaS (Platform as a Service):** Services such as Azure App Service, Azure SQL Database, and Azure Functions.
* **IaaS (Infrastructure as a Service):** Services such as virtual machines, virtual networks, disks, and load balancers.

So, rather than asking, *"What servers should I buy to run my application?"*, Azure lets you ask, *"Which cloud services should I combine to run my application securely, reliably, and at scale?"*

We cannot cover every aspect of Microsoft Azure in one article. The subject is broad and would require hundreds of pages to cover fully.

## Types of Applications to Deploy

Microsoft Azure can host many kinds of applications, depending on your requirements and goals.

### Monolith

A monolith is an application deployed as a single unit. Examples include a single *JAR*, *WAR*, or *EAR* file.

You can deploy a monolith on Azure App Service, Azure Virtual Machines, Azure Container Apps, or Azure Kubernetes Service (AKS).

### Microservices

Microservices are applications decomposed into independently deployable services that communicate through APIs or messaging to accomplish tasks. Instead of having many features in one deployable package, features are deployed as independent units.

You can deploy microservices on Azure Kubernetes Service (AKS), Azure Container Apps, Azure API Management, Azure Service Bus, and Azure Event Hubs.

### Batch

Batch applications process large amounts of data or execute jobs asynchronously, usually on a schedule or in response to a trigger.

With Azure, you can deploy batch workloads on Azure Batch, Azure Functions, Azure Container Apps Jobs, and Azure Data Factory.

### Serverless

With serverless computing, your code is executed on demand while Azure manages the underlying infrastructure and scaling.

Azure supports serverless computing through services such as Azure Functions and Azure Container Apps.

## Java Technologies

As one of the most widely used programming languages, [Java](https://www.oracle.com/ca-en/java/) was initially built to support many kinds of applications and to run on different operating systems and platforms, including Windows, Linux, macOS, Android, and embedded systems. Azure allows you to deploy and run supported Java technologies.

- **Java SE:** Java Standard Edition is the core computing platform used to develop and deploy portable, secure applications on desktops, servers, and embedded systems.

- **Jakarta EE:** Initially known as Java Enterprise Edition (Java EE), Jakarta EE is a set of specifications that extends Java Standard Edition (Java SE) with tools for building large-scale, scalable, and secure enterprise applications. Jakarta EE applications typically run on application servers such as Apache Tomcat, Oracle WebLogic Server, Oracle GlassFish, Red Hat JBoss, WildFly, and IBM WebSphere Application Server.

- **Spring Framework:** Spring Framework is a free, open-source Java application framework designed to simplify enterprise software development. It serves as an infrastructural "plumbing" layer that handles repetitive tasks such as object creation, configuration, and security. This allows developers to focus on business logic. Its core capabilities include *dependency injection*, *aspect-oriented programming*, and *business abstraction*. Spring Framework has many subprojects, including Spring Boot, Spring Data, Spring Security, Spring Cloud, Spring Batch, and Spring AI.

Even if they are not explicitly mentioned, other important technologies include frameworks for microservice-oriented architectures, such as [Quarkus](https://quarkus.io/) and [Micronaut](https://micronaut.io/).

## Conclusion

If you are a Java developer looking for a robust, user-friendly, and secure cloud platform, Azure is a strong choice. As one of the top three cloud platforms, Azure provides a comprehensive environment for building, modernizing, deploying, and scaling Java applications. Java developers can run everything from traditional Spring Boot monoliths and batch applications to microservices and serverless architectures.

With services such as Azure App Service, Azure Container Apps, Azure Kubernetes Service (AKS), Azure Functions, and Azure databases, developers can focus on application development while Azure handles much of the infrastructure, scalability, security, and availability.