---
title: "Spring Batch Admin Example"
permalink: /spring-batch-admin-example.html
excerpt: "A detailed step-by-step tutorial on how to use a Spring Boot admin UI to manage Spring Batch jobs."
date: 2018-11-11
last_modified_at: 2018-11-11
header:
  teaser: "assets/images/spring-batch/spring-batch-admin.png"
categories: [Spring Batch]
tags: [Admin, Example, Maven, Spring Batch, Spring Boot, Tutorial, UI]
published: true
---

<img src="{{ site.url }}/assets/images/spring-batch/spring-batch-admin.png" alt="spring batch admin" class="align-right title-image">

Looking for a [Spring Batch Admin](https://docs.spring.io/spring-batch-admin/trunk/){:target="_blank"} UI tutorial?

You might be surprised to learn it is no longer supported.

But don't worry as in this post **I'll show you the recommended replacement**. And how to set it up.

Let's dive right in.

## What is Spring Batch Admin?

Spring Batch Admin provides a web-based user interface (UI) that allows you to manage [Spring Batch](https://spring.io/projects/spring-batch){:target="_blank"} jobs. The project however is _end of life since December 31, 2017_.

[Spring Cloud Data Flow](https://cloud.spring.io/spring-cloud-dataflow/){:target="_blank"} is now the [recommended replacement](https://github.com/spring-projects/spring-batch-admin#note-this-project-is-being-moved-to-the-spring-attic-and-is-not-recommended-for-new-projects--spring-cloud-data-flow-is-the-recommended-replacement-for-managing-and-monitoring-spring-batch-jobs-going-forward--you-can-read-more-about-migrating-to-spring-cloud-data-flow-here){:target="_blank"} for managing and monitoring Spring Batch jobs. It is one of the [main Spring Projects](https://spring.io/projects){:target="_blank"}.

Let's demonstrate how you can configure Spring Cloud Data Flow to run a batch job.

We re-use the [Spring Batch capitalize names](https://github.com/code-not-found/spring-batch/tree/master/spring-batch-capitalize-names){:target="_blank"} project. It contains a batch job that converts person names from lower case into upper case.

We then start a Spring Cloud Data Flow server and configure the batch job. Using the web-based UI we launch the job and monitor the status.

## General Project Setup

We will use the following tools/frameworks:
* _Spring Batch 3.0_
* _Spring Boot 1.5_
* _Maven 3.5_

> [Spring Cloud Data Flow](https://github.com/spring-cloud/spring-cloud-dataflow){:target="_blank"} does not yet support Spring Boot 2.0. That is why this example uses Spring Boot 1.5 and Spring Batch 3.0.

We will create two Maven projects (one for the batch job and one for Spring Cloud Data Flow) that have the following structure:

<img src="{{ site.url }}/assets/images/spring-batch/spring-batch-admin-maven-projects.png" alt="spring batch admin maven projects">

## Creating a Spring Batch Task

Spring Cloud Data Flow is **a toolkit for building data processing pipelines**. The pipelines consist of Spring Boot applications. This means we can run a Spring Boot batch job using a Data Flow server.

All we need to do is annotate our existing `SpringBatchApplication` with `@EnableTask` as shown below.

This class-level annotation tells Spring Cloud Task to bootstrap it's functionality. It enables a `TaskConfigurer` that registers the application in a `TaskRepository`.

Spring Cloud Task will also [associate the execution of a batch job with a task's execution](https://docs.spring.io/spring-cloud-task/docs/1.3.0.RELEASE/reference/htmlsingle/#batch-association){:target="_blank"} so that one can be traced back to the other. This association is by default in any context that has both a Spring Batch Job configured and the `spring-cloud-task-batch` JAR available within the classpath.

We can now define a [Spring Cloud Task](https://spring.io/projects/spring-cloud-task){:target="_blank"} as we will see in the next section.

{% highlight xml %}
package com.codenotfound.batch;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.task.configuration.EnableTask;

@EnableTask
@SpringBootApplication
public class SpringBatchApplication {

  public static void main(String[] args) {
    SpringApplication.run(SpringBatchApplication.class, args);
  }
}
{% endhighlight %}

Spring Cloud Task uses a datasource for storing the results of task executions. When running on Spring Cloud Data Flow we need to make sure that [common datasource settings are shared among both](http://docs.spring.io/spring-cloud-dataflow/docs/1.7.0.RELEASE/reference/htmlsingle/#_task_database_configuration){:target="_blank"}.

By default Spring Cloud Data Flow uses an in-memory instance of [H2](http://www.h2database.com/html/main.html){:target="_blank"} with the following URL: <var>jdbc:h2:tcp://localhost:19092/mem:dataflow</var>.

We set the same datasource on our Spring Batch Task using the <var>application.yml</var> properties file located in <var>src/main/resources</var>.

{% highlight yml %}
spring:
  datasource:
    url: jdbc:h2:tcp://localhost:19092/mem:dataflow
    driverClassName: org.h2.Driver
{% endhighlight %}

 To enable above configuration changes we need to add additional dependencies in the Maven <var>pom.xml</var> file.

The `spring-cloud-starter-task` starter includes the dependencies for testing Spring Boot applications. It imports libraries that include [JUnit](http://junit.org/junit4/){:target="_blank"}, [Hamcrest](http://hamcrest.org/JavaHamcrest/){:target="_blank"} and [Mockito](https://site.mockito.org/){:target="_blank"}.

We also declare a dependency on `h2`. Spring Boot will take care of the [auto-configuration of the datasource](https://docs.spring.io/spring-boot/docs/1.5.7.RELEASE/reference/html/using-boot-auto-configuration.html){:target="_blank"} when it finds the H2 library on the classpath.

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>spring-batch-task</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>spring-batch-task</name>
  <description>Spring Batch Task Example</description>
  <url>https://www.codenotfound.com/spring-batch-admin-example.html</url>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.17.RELEASE</version>
    <relativePath /> <!-- lookup parent from repository -->
  </parent>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>1.8</java.version>
    <spring-cloud-starter-task.version>1.3.0.RELEASE</spring-cloud-starter-task.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-batch</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-task</artifactId>
      <version>${spring-cloud-starter-task.version}</version>
    </dependency>

    <dependency>
      <groupId>com.h2database</groupId>
      <artifactId>h2</artifactId>
      <scope>runtime</scope>
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

## Running a Spring Cloud Data Flow Server




---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-batch/tree/master/spring-batch-hello-world){:target="_blank"}.
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

In this getting started tutorial you learned how to create a simple Spring Batch example with Spring Boot and Maven.

Let me know if you liked this post.

Leave a comment below.

Thanks!
