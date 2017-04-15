---
title: "JSON API - Spring Boot &amp; Katharsis Example"
permalink: /2017/04/json-api-spring-boot-katharsis-example.html
excerpt: "A detailed step-by-step tutorial on how to provide and consume a RESTful Hello World JSON API using Katharsis and Spring Boot."
date: 2017-04-15
modified: 2017-04-15
categories: [Spring Kafka]
tags: [Example, Hello World, JSON API, Katharsis, Maven, Spring, Spring Boot, Tutorial]
published: false
---

{% include figure image_path="/assets/images/logos/katharsis-logo.jpg" alt="katharsis logo" %}

[JSON API](http://jsonapi.org/) is a specification for building APIs in JSON. It details how clients should request resources to be fetched or modified, and how servers should respond to those requests. JSON API is designed to minimize both the number of requests and the amount of data transmitted between clients and servers.

[Katharsis](http://katharsis.io) is a Java library that implements the JSON API specification. It is an additional layer that can be plugged on top of existing server side Java implementations in order to provide easy [HATEOAS](https://en.wikipedia.org/wiki/HATEOAS) support. Katharsis defines resources which can be shared over a RESTful interface and a repository for handling them.

The below tutorial details how to configure and build a RESTful Hello World API that complies to the JSON API specification. Frameworks used are Katharsis, OkHttp, Spring Boot and Maven.

Tools used:
* Katharsis 3.0
* OkHttp 3.6
* Spring Boot 1.5
* Maven 3.5

# General Project Setup

The example will be built and run using [Apache Maven](https://maven.apache.org/). In order to use the Katharsis framework we need to include the `katharsis-core` dependency. As Spring Boot will be used for running the server part we also need to include `katharsis-spring`. For the client part the `katharsis-client` dependency is needed.

Katharsis aims to have as little dependencies as possible, as such a HTTP client library needs to be provided on the classpath that will be automatically picked up. Both [OkHttp](https://square.github.io/okhttp) and [Apache Http Client](https://hc.apache.org/httpcomponents-client-ga/index.html) are supported. For this example we will use `okhttp`.

The running and testing of the example is based on the `spring-boot-starter` and `spring-boot-starter-test` [Spring Boot starters](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters). The `spring-boot-maven-plugin` is used to spin up an embedded Tomcat server that will host the RESTful Hello World API.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>json-api-katharsis-helloworld</artifactId>
  <version>0.0.1-SNAPSHOT</version>

  <name>json-api-katharsis-helloworld</name>
  <description>JSON API - Katharsis Spring Boot Hello World Example</description>
  <url>https://www.codenotfound.com/2017/04/json-api-katharsis-spring-boot-example.html</url>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.2.RELEASE</version>
  </parent>

  <properties>
    <java.version>1.8</java.version>

    <katharsis.version>3.0.1</katharsis.version>
    <okhttp.version>3.6.0</okhttp.version>
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
    <!-- katharsis -->
    <dependency>
      <groupId>io.katharsis</groupId>
      <artifactId>katharsis-core</artifactId>
      <version>${katharsis.version}</version>
    </dependency>
    <dependency>
      <groupId>io.katharsis</groupId>
      <artifactId>katharsis-spring</artifactId>
      <version>${katharsis.version}</version>
    </dependency>
    <dependency>
      <groupId>io.katharsis</groupId>
      <artifactId>katharsis-client</artifactId>
      <version>${katharsis.version}</version>
    </dependency>
    <!-- okhttp -->
    <dependency>
      <groupId>com.squareup.okhttp3</groupId>
      <artifactId>okhttp</artifactId>
      <version>${okhttp.version}</version>
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



``` java
package com.codenotfound.katharsis;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringKatharsisApplication {

  public static void main(String[] args) {
    SpringApplication.run(SpringKatharsisApplication.class, args);
  }
}
```

# Creating the Domain Model

We start by defining a simple model that will represent our `Greeting` resource. It contains an `id` in addition to the actual greeting `content`. We also define the constructors and getters/setters for the two fields.

The @JsonApiResource annotation defines a resource. It requires the type parameter to be defined which will be used to form the URI and populate the type field which is passed as part of the JSON message. According to JSON API standard, the name defined in type can be either plural or singular

The @JsonApiId defines a field which will be used as an identifier of the Greeting resource. Each resource requires this annotation to be present on a field which is of primitive type or which type implements the Serializable interface.

---

Modelled resources must be complemented by a corresponding repository implementation. This can be achieved by implementing one of those two repository interfaces:
1. ResourceRepositoryV2 for a resource
2. RelationshipRepositoryV2 resp. BulkRelationshipRepositoryV2 for resource relationships

For our Greeting resource we will create a GreetingRepositoryImpl which extends the ResourceRepositoryBase implmentation of the ResourceRepositoryV2 repository interface. This base repository will be used by Katharsis to operate on the resource. The implementation consists of five basic methods which provide CRUD operation for a resource and two parameters: the first is a type of a resource and the second is the type of the resource's identifier.

For this example we will use a HashMap to store the greetings and in the constructor we already populate the Map with a greeting that contains <var>'Hello World!</var> as content.

The ResourceRepositoryBase is a base class that takes care of some boiler-plate, like implementing findOne with findAll. An implementation can then look as simple as:











---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-kafka/tree/master/spring-kafka-boot).
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

Using Spring Boot's autoconfiguration we were able to setup a sender and receiver using only a couple of lines of code. Hopefully this example will kick-start your Spring Kafka development. Drop a line below in case something was not clear or just to let me know if everything worked.
