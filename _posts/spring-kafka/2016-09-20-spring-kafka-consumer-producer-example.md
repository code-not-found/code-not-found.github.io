---
title: Spring Kafka - Consumer & Producer Example
permalink: /2016/09/spring-kafka-consumer-producer-example.html
excerpt: A detailed step-by-step tutorial on how to implement an Apache Kafka Consumer and Producer using Spring Kafka and Spring Boot.
date: 2016-09-20 21:00
tags: [Apache Kafka, Consumer, Example, Hello World, Maven, Producer, Spring, Spring Boot, Spring Kafka, Tutorial]
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/spring-logo.png" alt="spring logo">
</figure>

The [Spring for Apache Kafka (spring-kafka) project](https://projects.spring.io/spring-kafka/) applies core Spring concepts to the development of Kafka-based messaging solutions. It provides a "template" as a high-level abstraction for sending messages. It also provides support for Message-driven POJOs with @KafkaListener annotations and a 'listener container'.

In the following tutorial we will configure, build and run a Hello World example in which we will send/receive messages to/from Apache Kafka using Spring Kafka, Spring Boot and Maven. Before running below code, make sure that [Apache Kafka is installed and started](http://www.source4code.info/2016/09/apache-kafka-download-installation.html).

> Spring Kafka 1.1 uses the Apache Kafka 0.10.x.x client. 

Tools used:
* Spring Kafka 1.1
* Spring Boot 1.4
* Maven 3

We start by defining a Maven POM file which contains the dependencies for the needed [Spring projects](https://spring.io/projects). The POM inherits from the `spring-boot-starter-parent` project and declares dependencies to `spring-boot-starter` and `spring-boot-starter-test` starters.

A dependency to `spring-kafka` is added in addition to a property that specifies the version. At the time of writing the latest stable release was _1.1.1.RELEASE_.

We also include the `spring-boot-maven-plugin` Maven plugin so that we can build a single, runnable "über-jar", which is convenient to execute and transport the written code. 

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.codenotfound</groupId>
    <artifactId>spring-kafka-helloworld-example</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <name>spring-kafka-helloworld-example</name>
    <description>Spring Kafka - Consumer & Producer Example</description>
    <url>http://www.codenotfound.com/2016/09/spring-kafka-consumer-producer-example.html</url>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.4.1.RELEASE</version>
        <relativePath /> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <java.version>1.8</java.version>
        <spring-kafka.version>1.1.1.RELEASE</spring-kafka.version>
    </properties>

    <dependencies>
        <!-- Spring Kafka -->
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
            <version>${spring-kafka.version}</version>
        </dependency>
        <!-- Spring Boot -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
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
            </plugin>
        </plugins>
    </build>

</project>
```

We will use Spring Boot in order to make a Spring Kafka example application that you can "just run". We start by creating an `SpringKafkaApplication` that contains the `main()` method that uses Spring Boot’s `SpringApplication.run()` method to launch an application. The `@SpringBootApplication` annotation is a convenience annotation that adds: `@Configuration`, `@EnableAutoConfiguration` and `@ComponentScan`.

