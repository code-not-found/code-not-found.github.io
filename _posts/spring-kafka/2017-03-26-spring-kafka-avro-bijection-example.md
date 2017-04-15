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

We base this example on a previous [Spring Kafka Avro serializer/deserializer example]({{ site.url }}//2017/03/spring-kafka-apache-avro-example.html) in which we used the Avro API's to serialize and deserialize objects. For this tutorial we will be using the Bijection APIs which are a bit easier to use as we will see further down below.

Starting point is again the <var>user.avsc</var> schema from the [Avro getting started guide](https://avro.apache.org/docs/current/gettingstartedjava.html#Defining+a+schema). It describes the fields and their types of a `User` type.

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

We setup our project using [Maven](https://maven.apache.org/). In the POM file we add the `bijection-avro_2.11` dependency. The <var>artifactId</var> suffix of the dependency (in this case ** _2.11**) highlights the [Scala](https://www.scala-lang.org/) version used to compile the JAR.

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

Generation of the Avro `User` class is done by executed below Maven command. The result is a `User` class that contains the schema and `Builder` methods.

``` plaintext
mvn generate-sources
```

<figure>
    <img src="{{ site.url }}/assets/images/bijection-avro-generated-java-classes.png" alt="bijection avro generated java classes">
</figure>

# Producing Avro Messages to a Kafka Topic

Serializing an Avro message to a `byte[]` array using Bijection can be achieved in just two lines of code as shown below. We first create an `Injection` which is an object that can make the conversion in one way or the other.


``` java
package com.codenotfound.kafka.serializer;

import java.util.Map;

import javax.xml.bind.DatatypeConverter;

import org.apache.avro.generic.GenericRecord;
import org.apache.avro.specific.SpecificRecordBase;
import org.apache.kafka.common.serialization.Serializer;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import com.twitter.bijection.Injection;
import com.twitter.bijection.avro.GenericAvroCodecs;

public class AvroSerializer<T extends SpecificRecordBase> implements Serializer<T> {

  private static final Logger LOGGER = LoggerFactory.getLogger(AvroSerializer.class);

  @Override
  public void close() {
    // No-op
  }

  @Override
  public void configure(Map<String, ?> arg0, boolean arg1) {
    // No-op
  }

  @Override
  public byte[] serialize(String topic, T data) {
    LOGGER.debug("data to serialize='{}'", data);

    Injection<GenericRecord, byte[]> genericRecordInjection =
        GenericAvroCodecs.toBinary(data.getSchema());
    byte[] result = genericRecordInjection.apply(data);

    LOGGER.debug("serialized data='{}'", DatatypeConverter.printHexBinary(result));
    return result;
  }
}
```


# Consuming Avro Messages from a Kafka Topic



# Test Sending and Receiving Avro Messages on Kafka




---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-kafka/tree/master/spring-kafka-avro-bijection).
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This concludes the example on how to send/receive Avro messages using Twitter Bijection and Spring Kafka. I created this blog post based on a user request so if you found this tutorial useful or would like to see another variation, let me know.