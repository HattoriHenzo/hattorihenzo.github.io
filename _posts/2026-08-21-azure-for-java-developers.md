---
layout: post
title:  "Azure for Java Developers"
---

In this article I will introduce you progressively on how beautiful it is to use Java on Azure.

## What is Microsoft Azure?

To simply say, **Microsoft Azure** is a **cloud computing platform** provided by Microsoft. It allows organizations and individuals to build, deploy, run, and manage applications and IT infrastructure using Microsoft’s global network of data centers spread all around the world, between [Regions](https://learn.microsoft.com/en-us/azure/reliability/regions-overview) and [Availability Zones](https://learn.microsoft.com/en-us/azure/reliability/availability-zones-overview?tabs=azure-cli).

Azure provides hundreds of cloud services covering areas such as **computing, storage, databases, networking, security, artificial intelligence, analytics, DevOps, and containers**.

Azure supports the three major types of services: 

* **SaaS (Software as a Service):** Microsoft-hosted applications and services consumed directly by users.
* **PaaS (Platform as a Service):** Azure App Service, Azure SQL Database, Azure Functions, etc.
* **IaaS (Infrastructure as a Service):** Virtual Machines, Virtual Networks, disks, load balancers, etc.

So rather than asking, *"What servers should I buy to run my application?"*, Azure lets you ask, *"Which cloud services should I combine to run my application securely, reliably, and at scale?"*

We can not cover all the different aspects concerning Microsoft Azure in one article. The subject is very wide and it will need hundreds of pages to cover it.

## Type of Application to deploy

Microsoft Azure can host many kind of applications depending on your requirement and your goals.

### Monolith
A monolith is an application that is deployed as a single unit. Exemple of a single *jar*, *war* or *ear*.
You can deploy a Monolith on: Azure App Service, Azure Virtual Machines, Azure Container App, Azure Kubernetes Service (AKS).

### Microservices
Application decomposed into independently deployable services communicating together throught API or messaging to accomplish tasks. Here, instead of having many feature in one deployable package, package are deployed into independent unit.
You can deploy Micorservices on: Azure Kubernetes Service (AKS), Azure Container Apps, Azure API Management, Azure Service Bus, Azure Event Hubs.

### Batch
Are defined as application processing large amounts of data or jobs executing asynchronously, usually on a schedule or trigger.
With Azure, you can deploy your batch on Azure Batch, Azure Functions, Azure Container Apps Jobs, Azure Data Factory.

### Serverless
With Serverless your code is executed on demand while Azure Manages the underlying infrastructure and scaling.
Azure support Serverless through those services: Azure Functions, Azure Container Apps.

## Java Technologies

As the #1 programming language, you may know that [Java](https://www.oracle.com/ca-en/java/) has been built initially to support many kind of application and to run on any Operating System/Platform (Windows/PC, Linux/PC, macOS/Mac Computer, Android/Embedded etc...). Azure allows you to deploy and run all the supported Java technologies.

 - **Java SE:** it stands for Java Standard Edition. This is the core computing platform used to develop and deploy portable, secure applications on desktops, servers, and embedded systems.

 - **Jakarta EE:** initially called Java Enterprise Edition (Java EE), is a set of specifications that extends the core Java Standard Edition (Java SE) with tools for building large-scale, scalable, and secure enterprise applications. Jakarta EE applications run exclively on Applications Servers: Apache Tomcat, Oracle Weblogic Server, Oracle Glassfish, Red Hat JBoss, Wildfly, IBM WebSphere Application Server etc...

 - **Spring Framework:**  is a free, open-source Java application framework designed to simplify enterprise software development. It serves as an infrastructural "plumbing" layer that handles repetitive tasks like object creation, configuration, and security. This allows developers to focus purely on business logic. Its core functionalities are the following: *Dependency injection*, *Aspect-oriented programming*, *Business abstraction*. Spring Framework has many sub-projects such as: Spring Boot, Spring Data, Spring Security, Spring Cloud, Spring Batch, Spring AI.

 Even if not explicitly mentionned, there are other very important techonlogies like other frameworks for microservice-oriented architectures: [Quarkus](https://quarkus.io/), [Micronaut](https://micronaut.io/).

## Conclusion

You are a Java Developers and looking for a robust, user friendly and, secure cloud plaform, Azure is your choice. Among the top 3 Cloud platform, Azure provides a complete cloud platform for building, modernizing, deploying, and scaling Java applications. Java developers can run everything from traditional Spring Boot monoliths and batch applications to microservices and serverless architectures.

With services such as Azure App Service, Azure Container Apps, Azure Kubernetes Service (AKS), Azure Functions, and Azure databases, developers can focus on application development while Azure handles much of the infrastructure, scalability, security, and availability.