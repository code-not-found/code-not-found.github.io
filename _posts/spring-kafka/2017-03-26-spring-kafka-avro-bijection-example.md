---
title: Spring Kafka - Avro Bijection Example
permalink: /2017/03/spring-kafka-avro-bijection-example.html
excerpt: A detailed step-by-step tutorial on how to implement an Avro Serializer &amp; Deserializer using Twitter Bijection, Spring Kafka and Spring Boot.
date: 2017-03-26 21:00
categories: [Spring Kafka]
tags: [Apache Kafka, Apache Avro, Avro, Bijection, Deserializer, Example, Maven, Serializer, Spring, Spring Boot, Spring Kafka, Tutorial, Twitter Bijection]
published: false
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/spring-logo.png" alt="spring logo">
</figure>

[Twitter Bijection](https://github.com/twitter/bijection) is an invertible function library that converts back and forth between two types. It supports a number of types including Apache Avro. In the following tutorial we will configure, build and run an example in which we will send/receive an Avro message to/from Apache Kafka using Bijection, Apache Avro, Spring Kafka, Spring Boot and Maven.

Tools used:
* Twitter Bijection 0.9
* Apache Avro 1.8
* Spring Kafka 1.1
* Spring Boot 1.5
* Maven 3

We base this example on a previous [Spring Kafka Avro serializer/deserializer example]({{ site.url }}//2017/03/spring-kafka-apache-avro-example.html) in which we used the Avro API's to serialize and deserialize objects. As we will see further down below, Bijection allows to convert back and forth in only a couple of lines of code.

The <var>user.avsc</var> schema from the [Avro getting started guide](https://avro.apache.org/docs/current/gettingstartedjava.html#Defining+a+schema) describes the fields and their types of our `User` type.

``` json
{"namespace": "example.avro",
  "type": "record",
  "name": "User",
  "fields": [
    {"name": "name", "type": "string"},
    {"name": "favorite_number",  "type": ["int", "null"]},
    {"name": "favorite_color", "type": ["string", "null"]}
  ]
}
```

In the Maven POM file we add the `bijection-avro_2.11` dependency. The <var>artifactId</var> suffix of the dependency highlights the scala version used to compile the JAR.

> Note that we choose the <var>2.11</var> version of `bijection-avro` since `spring-kafka-test` includes a dependency on the <var>2.11</var> version of the `scala-library`.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>spring-kafka-avro-bijection</artifactId>
  <version>0.0.1-SNAPSHOT</version>

  <name>spring-kafka-avro-bijection</name>
  <description>Spring Kafka - Avro Bijection Example</description>
  <url>https://www.codenotfound.com/2017/03/spring-kafka-avro-bijection-example.html</url>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.2.RELEASE</version>
  </parent>

  <properties>
    <java.version>1.8</java.version>

    <spring-kafka.version>1.1.2.RELEASE</spring-kafka.version>
    <avro.version>1.8.1</avro.version>
    <bijection.version>0.9.5</bijection.version>
  </properties>

  <dependencies>
    <!-- spring-boot -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
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
    <!-- avro -->
    <dependency>
      <groupId>org.apache.avro</groupId>
      <artifactId>avro</artifactId>
      <version>${avro.version}</version>
    </dependency>
    <dependency>
      <groupId>com.twitter</groupId>
      <artifactId>bijection-avro_2.11</artifactId>
      <version>${bijection.version}</version>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <!-- spring-boot-maven-plugin -->
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
      <!-- avro-maven-plugin -->
      <plugin>
        <groupId>org.apache.avro</groupId>
        <artifactId>avro-maven-plugin</artifactId>
        <version>${avro.version}</version>
        <executions>
          <execution>
            <phase>generate-sources</phase>
            <goals>
              <goal>schema</goal>
            </goals>
            <configuration>
              <sourceDirectory>${project.basedir}/src/main/resources/avro/</sourceDirectory>
              <outputDirectory>${project.build.directory}/generated/avro</outputDirectory>
            </configuration>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
</project>
```

# Producing Avro Messages to a Kafka Topic



# Consuming Avro Messages from a Kafka Topic



# Test Sending and Receiving Avro Messages on Kafka




---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-kafka/tree/master/spring-kafka-avro-bijection).
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This concludes the example on how to send/receive Avro messages using Twitter Bijection and Spring Kafka. I created this blog post based on a user request so if you found this tutorial useful or would like to see another variation, let me know.