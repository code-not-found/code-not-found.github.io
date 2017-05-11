---
title: "Spring Kafka - Integration Example"
permalink: /2017/05/spring-kafka-integration-example.html
excerpt: "A detailed step-by-step tutorial on how to connect Apache Kafka to a Spring Integration Channel using Spring Kafka and Spring Boot."
date: 2017-05-11
modified: 2017-05-11
header:
  teaser: "assets/images/spring-kafka-teaser.jpg"
categories: [Spring Kafka]
tags: [Apache Kafka, Consumer, Example, Hello World, Maven, Producer, Spring, Spring Boot, Spring Integration, Spring Kafka, Tutorial]
published: true
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/spring-logo.jpg" alt="spring logo" class="logo">
</figure>

[Spring Integration](https://projects.spring.io/spring-integration/) extends the Spring programming model to support the well-known [Enterprise Integration Patterns](http://www.enterpriseintegrationpatterns.com/). It enables lightweight messaging within Spring-based applications and supports integration with external systems via declarative adapters.

The [Spring Integration Kafka](https://github.com/spring-projects/spring-integration-kafka) extension project provides inbound and outbound channel adapters specifically for [Apache Kafka](https://kafka.apache.org/). In this tutorial we will configure, build and run a Hello World example in which we will send/receive messages to/from Apache Kafka using Spring Integration Kafka, Spring Boot and Maven.

Tools used:
* Spring Kafka 1.2
* Spring Integration 2.1
* Spring Boot 1.5
* Maven 3.5

# General Project Setup

Building of the project will be automated using [Maven](https://maven.apache.org/). We include the needed Spring Integration dependencies using the `spring-boot-starter-integration` [Spring Boot starter](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters). For testing support we also include the `spring-boot-starter-test` starter.

As we will be using the Spring Integration Kafka extension we add the corresponding `spring-integration-kafka` dependency. Starting from version 2.0 this project is a complete rewrite based on the [Spring for Apache Kafka](https://projects.spring.io/spring-kafka/) project which uses the pure java Producer and Consumer clients provided by Kafka. As such we also add the dependencies for `spring-kafka` for core functionality as well as `spring-kafka-test` in order to have access to an embedded Kafka broker when running our unit test.

The `spring-boot-maven-plugin` Maven plugin is added so that we can build a single, runnable JAR, which is convenient to execute and transport our written code.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>spring-kafka-integration-helloworld</artifactId>
  <version>0.0.1-SNAPSHOT</version>

  <name>spring-kafka-integration-helloworld</name>
  <description>Spring Kafka - Consumer Producer Example</description>
  <url>https://www.codenotfound.com/2016/09/spring-kafka-consumer-producer-example.html</url>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.3.RELEASE</version>
  </parent>

  <properties>
    <java.version>1.8</java.version>

    <spring-integration-kafka.version>2.1.0.RELEASE</spring-integration-kafka.version>
    <spring-kafka.version>1.2.1.RELEASE</spring-kafka.version>
  </properties>

  <dependencies>
    <!-- spring-boot -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-integration</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
    <!-- spring-integration -->
    <dependency>
      <groupId>org.springframework.integration</groupId>
      <artifactId>spring-integration-kafka</artifactId>
      <version>${spring-integration-kafka.version}</version>
    </dependency>
    <!-- spring-kafka -->
    <dependency>
      <groupId>org.springframework.kafka</groupId>
      <artifactId>spring-kafka</artifactId>
      <version>${spring-kafka.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework.kafka</groupId>
      <artifactId>spring-kafka-test</artifactId>
      <version>${spring-kafka.version}</version>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <!-- spring-boot-maven-plugin -->
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>
</project>
```

We create a `SpringKafkaIntegrationApplication` class that takes care of some basic setup and which also allows to launch the application.

``` java
package com.codenotfound.kafka.integration;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringKafkaIntegrationApplication {

  public static void main(String[] args) {
    SpringApplication.run(SpringKafkaIntegrationApplication.class, args);
  }
}
```

# Spring Integration Kafka Consumer Channel



# Spring Integration Kafka Producer Channel



# Spring Integration Kafka Test


---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-kafka/tree/master/spring-kafka-integration-helloworld).
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

???
