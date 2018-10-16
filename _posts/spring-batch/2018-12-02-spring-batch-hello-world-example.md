---
title: "Spring Batch Hello World Example"
permalink: /spring-batch-hello-world-example.html
excerpt: "A detailed step-by-step tutorial on how to implement a Hello World Spring Batch job using Spring Boot and Maven."
date: 2018-10-16
last_modified_at: 2018-10-16
header:
  teaser: "assets/images/spring-batch/spring-batch-hello-world-example.png"
categories: [Spring Batch]
tags: [Example, Maven, Spring Batch, Spring Boot, Tutorial ]
published: false
---

<img src="{{ site.url }}/assets/images/spring-batch/spring-batch-hello-world-example.png" alt="spring batch hello world example" class="align-right title-image">

I’m going to show you exactly how to create a [Spring Batch](https://spring.io/projects/spring-batch){:target="_blank"} Hello World example that uses [Maven](https://maven.apache.org/){:target="_blank"} and [Spring Boot](https://spring.io/projects/spring-boot){:target="_blank"}.

(Step-by-step)

So if you’re a Spring Batch beginner, **you’ll love this guide**.

Ready?

## How Does the Spring Batch Framework Work?

Before we dive into the code let's look at the Spring Batch framework. It contains following key building blocks:

<img src="{{ site.url }}/assets/images/spring-batch/spring-batch-framework.png" alt="spring batch framework">

A batch process consists out of a `Job`. This is an entity that encapsulates the entire batch process.

A `Job` can consist out of one or more `Step`s. In most cases, a step will read data (`ItemReader`), process it (`ItemProcessor`) and then write it (`ItemWriter`).

The `JobLauncher` is responsible for launching a `Job`.

And finally the `JobRepository` stores metadata about configured and executed `Job`s.

## General Project Setup

To demonstrate how Spring Batch works let's build a simple Hello World batch job.

In the example, we read person data from a CSV file. From this data, a greeting is generated. This greeting is then written to a text file.

Our Maven project has the following structure:

<img src="{{ site.url }}/assets/images/spring-batch/spring-batch-hello-world-maven-project.png" alt="spring batch hello world maven project">

It uses the following tools/frameworks:
* Spring Batch 4.0
* Spring Boot 2.0
* Maven 3.5

We will build and run our example using **Maven**. If not already the case make sure to [download and install Apache Maven](https://downlinko.com/download-install-apache-maven-windows.html){:target="_blank"}.

Shown below is the XML representation of our Maven project. It is stored in the <var>pom.xml</var> file. It contains the needed dependencies to compile and run the example.

To run the batch job, we will use **Spring Boot**. We also use [Spring Boot Starters](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters){:target="_blank"} so that we do not have to manage the different Spring dependencies.

The `spring-boot-starter-batch` starter imports the Spring Boot and Spring Batch dependencies.

The `spring-boot-starter-test` starter includes the dependencies for testing Spring Boot applications. It imports libraries that include [JUnit](http://junit.org/junit4/){:target="_blank"}, [Hamcrest](http://hamcrest.org/JavaHamcrest/){:target="_blank"} and [Mockito](https://site.mockito.org/){:target="_blank"}.

We also declare a dependency on `spring-batch-test`. This library contains some helper classes that will help test our batch job.

In the plugins section, we define the [Spring Boot Maven Plugin](https://docs.spring.io/spring-boot/docs/current/reference/html/build-tool-plugins-maven-plugin.html){:target="_blank"}: `spring-boot-maven-plugin`. This allows us to build a single, runnable "uber-jar" which is a convenient way to execute and transport our code. In addition, the plugin allows us to start the example via a Maven command.

{% highlight xml linenos %}
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>spring-batch-hello-world</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>spring-batch-hello-world</name>
  <description>Spring Batch Hello World Example</description>
  <url>https://www.codenotfound.com/spring-batch-hello-world-example.html</url>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.6.RELEASE</version>
    <relativePath /> <!-- lookup parent from repository -->
  </parent>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>1.8</java.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-batch</artifactId>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework.batch</groupId>
      <artifactId>spring-batch-test</artifactId>
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
{% endhighlight %}

We use Spring Boot so that we have a Spring Batch application that you can "just run". Start by creating a `SpringBatchApplication` class. It contains the `main()` method that uses Spring Boot's `SpringApplication.run()` to launch the application.

> Note that `@SpringBootApplication` is a convenience annotation that adds: `@Configuration`, `@EnableAutoConfiguration`, and `@ComponentScan`.

For more information on Spring Boot, check the [Spring Boot getting started guide](https://spring.io/guides/gs/spring-boot/){:target="_blank"}.

Spring Batch by default uses a database to store metadata on the configured batch jobs. In this example, we will **run Spring Batch without a database**. Instead, an in-memory `Map` based repository is used.

For this to work we need to specify <var>exclude = {DataSourceAutoConfiguration.class}</var> (_line 7_). This prevents Spring Boot from auto-configuring a `DataSource` to connect to a stand-alone database.

{% highlight java linenos %}
package com.codenotfound.batch;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration;

@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
public class SpringBatchApplication {

  public static void main(String[] args) {
    SpringApplication.run(SpringBatchApplication.class, args);
  }
}
{% endhighlight %}
