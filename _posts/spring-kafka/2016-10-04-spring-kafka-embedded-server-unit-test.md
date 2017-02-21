---
title: Spring Kafka - Embedded Server Unit Test
permalink: /2016/10/spring-kafka-embedded-server-unit-test.html
excerpt: A detailed step-by-step tutorial on how to test your application using an embedded Apache Kafka server together with Spring Kafka and Spring Boot.
date: 2016-10-04 21:00
categories: [Spring Kafka]
tags: [Apache Kafka, Embedded, Example, Maven, Server, Spring Boot, Spring Kafka, Test, Testing, Tutorial, Unit  ]
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/spring-logo.png" alt="spring logo">
</figure>

The [Spring Kafka project](https://projects.spring.io/spring-kafka/) comes with a `spring-kafka-test` JAR that contains a number of useful utilities to assist you with your application testing. These include: an embedded Kafka server, some static methods to setup consumers/producers and some utility methods to fetch results.

Let's demonstrate how you can use these utilities with some runnable code. We will reuse the [Spring Kafka Hello World project](http://www.source4code.info/2016/09/spring-kafka-consumer-producer-example.html) from a previous post in which we created a consumer and producer using Spring Kafka, Spring Boot and Maven.


Tools used:
* Spring Kafka 1.1
* Spring Boot 1.4
* Maven 3

We need to add the `spring-kafka-test` dependency to the Maven POM file in addition to the Spring Kafka and Spring Boot dependencies. In the plugins section we included the `maven-surefire-plugin` to trigger a `AllSpringKafkaTests` test suite class that will be used to start the embedded server only once for the different unit test cases in our project.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.codenotfound</groupId>
    <artifactId>spring-kafka-embedded-test</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <name>spring-kafka-embedded-test</name>
    <description>Spring-Kafka - Embedded Kafka Test Example</description>
    <url>http://www.codenotfound.com/2016/09/spring-kafka-embedded-kafka-test-example.html</url>

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
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka-test</artifactId>
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

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.19.1</version>
                <configuration>
                    <includes>
                        <include>AllSpringKafkaTests.java</include>
                    </includes>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

The message consumer and producer classes from the Hello World example are unchanged so we won't go into detail explaining them. You can checkout the [Spring Boot Kafka example]({{ site.url }}/2016/09/spring-kafka-consumer-producer-example.html) from a previous post for more details. 

# Unit Testing with an Embedded Kafka Server




















