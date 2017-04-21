---
title: "JSON API - Spring Boot &amp; Katharsis Example"
permalink: /2017/04/json-api-spring-boot-katharsis-example.html
excerpt: "A detailed step-by-step tutorial on how to provide and consume a RESTful Hello World JSON API using Katharsis and Spring Boot."
date: 2017-04-21
modified: 2017-04-21
categories: [Spring Kafka]
tags: [Example, Hello World, JSON API, Katharsis, Maven, Spring, Spring Boot, Tutorial]
published: true
---

{% include figure image_path="/assets/images/logos/katharsis-logo.jpg" alt="katharsis logo" %}

[JSON API](http://jsonapi.org/) is a specification for building APIs using JSON. It details how clients should request resources to be fetched or modified, and how servers should respond to those requests. JSON API is designed to minimize both the number of requests and the amount of data transmitted between clients and servers.

[Katharsis](http://katharsis.io) is a Java library that implements the JSON API specification. It is an additional layer that can be plugged on top of existing server side Java implementations in order to provide easy [HATEOAS](https://en.wikipedia.org/wiki/HATEOAS) support. Katharsis defines resources which can be shared over a RESTful interface and a repository for handling them.

The below tutorial details how to configure and build a RESTful Hello World API that complies to the JSON API specification. Frameworks used are Katharsis, OkHttp, Spring Boot and Maven.

Tools used:
* Katharsis 3.0
* OkHttp 3.6
* Spring Boot 1.5
* Maven 3.5

# General Project Setup

The example will be built and run using [Apache Maven](https://maven.apache.org/). In order to use the Katharsis framework we need to include the `katharsis-core` dependency. As Spring Boot will be used for running the server part we also need to include `katharsis-spring`. For implementing the client part the `katharsis-client` dependency is needed.

Katharsis aims to have as little dependencies as possible, as such a HTTP client library is not included by default. It needs to be [provided on the classpath where it will be automatically picked up](http://katharsis-jsonapi.readthedocs.io/en/latest/user-docs.html#client) by the framework. Both [OkHttp](https://square.github.io/okhttp) and [Apache Http Client](https://hc.apache.org/httpcomponents-client-ga/index.html) are supported. For this example we will use the `okhttp` library.

Running and testing of the example is based on the `spring-boot-starter` and `spring-boot-starter-test` [Spring Boot starters](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters). The `spring-boot-maven-plugin` is used to spin up an embedded Tomcat server that will host the RESTful Hello World API.

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

Spring Boot is used in order to make a stand-alone Katharsis example application that we can "just run". The `SpringKatharsisApplication` class contains the `main()` method that uses Spring Boot's `SpringApplication.run()` method to launch the application.

The `@SpringBootApplication` annotation is a convenience annotation that adds: `@Configuration`, `@EnableAutoConfiguration` and `@ComponentScan`.

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

We start by defining a simple model that will represent a `Greeting` resource. It contains an `id` in addition to the actual greeting `content`. We also need to define the constructors and getters/setters for the two fields.

The `@JsonApiResource` annotation [defines a resource](http://katharsis-jsonapi.readthedocs.io/en/latest/user-docs.html#jsonapiresource). It requires the type parameter to be set which will be used to form the URI and populate the [type field](http://jsonapi.org/format/#document-resource-objects) which is passed as part of the resource object.

> According to JSON API standard, the name defined in type can be either plural or singular. However, the same value should be used consistently throughout an implementation.

The `@JsonApiId` [defines a field which will be used as an identifier](http://katharsis-jsonapi.readthedocs.io/en/latest/user-docs.html#jsonapiid) of the Greeting resource. Each resource requires this annotation to be present on a field which is of primitive type or a type that implements the `Serializable` interface.

``` java
package com.codenotfound.katharsis.domain.model;

import io.katharsis.resource.annotations.JsonApiId;
import io.katharsis.resource.annotations.JsonApiResource;

@JsonApiResource(type = "greetings")
public class Greeting {

  @JsonApiId
  private long id;

  private String content;

  public Greeting() {
    super();
  }

  public Greeting(long id, String content) {
    this.id = id;
    this.content = content;
  }

  public long getId() {
    return id;
  }

  public void setId(long id) {
    this.id = id;
  }

  public String getContent() {
    return content;
  }

  public void setContent(String content) {
    this.content = content;
  }
}
```

# Creating the Repository

To allow Katharsis to operate on defined resources, a special type of class called [repository](http://katharsis-jsonapi.readthedocs.io/en/latest/user-docs.html#repositories) needs to be created. Katharsis will scan for these classes and using annotations it will find out the available methods.

For our `Greeting` resource we will create a `GreetingRepositoryImpl` which extends the `ResourceRepositoryBase` implementation of the `ResourceRepositoryV2` repository interface. 
The [ResourceRepositoryBase](https://github.com/katharsis-project/katharsis-framework/blob/master/katharsis-core/src/main/java/io/katharsis/repository/ResourceRepositoryBase.java) is a base class that takes care of some boiler-plate, like for example implementing `findOne()` with `findAll()`.




will be used by Katharsis to operate on the resource. The implementation consists of five basic methods which provide CRUD operations for a resource and two parameters: the first is a type of a resource and the second is the type of the resource's identifier.



``` java
package com.codenotfound.katharsis.domain.repository;

import java.util.HashMap;
import java.util.Map;

import org.springframework.stereotype.Component;

import com.codenotfound.katharsis.domain.model.Greeting;

import io.katharsis.queryspec.QuerySpec;
import io.katharsis.repository.ResourceRepositoryBase;
import io.katharsis.resource.list.ResourceList;

@Component
public class GreetingRepositoryImpl extends ResourceRepositoryBase<Greeting, Long> {

  private Map<Long, Greeting> greetings = new HashMap<>();

  public GreetingRepositoryImpl() {
    super(Greeting.class);
    save(new Greeting(123L, "Hello World!"));
  }

  @Override
  public synchronized <S extends Greeting> S save(S greeting) {
    greetings.put(greeting.getId(), greeting);
    return greeting;
  }

  @Override
  public synchronized ResourceList<Greeting> findAll(QuerySpec querySpec) {
    return querySpec.apply(greetings.values());
  }
}
```



 This can be achieved by implementing one of two repository interfaces:
1. ResourceRepositoryV2 for a resource
2. RelationshipRepositoryV2 resp. BulkRelationshipRepositoryV2 for resource relationships



For this example we will use a HashMap to store the greetings and in the constructor we already populate the Map with a greeting that contains <var>'Hello World!</var> as content.

The ResourceRepositoryBase is a base class that takes care of some boiler-plate, like implementing findOne with findAll. An implementation can then look as simple as:






``` plaintext
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.5.2.RELEASE)

09:35:41.763 [main] INFO  c.c.k.SpringKatharsisApplicationTest - Starting SpringKatharsisApplicationTest on cnf-pc with PID 1740 (started by CodeNotFound in c:\code\st\json-api\json-api-katharsis-helloworld)
09:35:41.765 [main] INFO  c.c.k.SpringKatharsisApplicationTest - No active profile set, falling back to default profiles: default
09:35:44.476 [main] INFO  c.c.k.SpringKatharsisApplicationTest - Started SpringKatharsisApplicationTest in 3.02 seconds (JVM running for 3.693)
09:35:44.900 [main] INFO  c.c.katharsis.client.GreetingClient - found com.codenotfound.katharsis.domain.model.Greeting@a52ca2e
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 3.574 sec - in com.codenotfound.katharsis.SpringKatharsisApplicationTest

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 6.963 s
[INFO] Finished at: 2017-04-21T09:35:45+02:00
[INFO] Final Memory: 28M/262M
[INFO] ------------------------------------------------------------------------
```

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/json-api/tree/master/json-api-katharsis-helloworld).
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

Using Spring Boot's autoconfiguration we were able to setup a sender and receiver using only a couple of lines of code. Hopefully this example will kick-start your Spring Kafka development. Drop a line below in case something was not clear or just to let me know if everything worked.
