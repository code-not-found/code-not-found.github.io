---
title: "JSON API - Spring Boot Katharsis Example"
permalink: /json-api-spring-boot-katharsis-example.html
excerpt: "A detailed step-by-step tutorial on how to provide and consume a RESTful Hello World JSON API using Katharsis and Spring Boot."
date: 2017-04-21
modified: 2017-12-16
header:
  teaser: "assets/images/header/katharsis-teaser.png"
categories: [JSON API]
tags: [Example, Hello World, JSON API, Katharsis, Maven, Spring, Spring Boot, Tutorial]
redirect_from:
  - /2017/04/json-api-katharsis-spring-boot-example.html
  - /2017/04/json-api-spring-boot-katharsis-example.html
published: true
---

<figure>
    <img src="{{ site.url }}/assets/images/logo/katharsis-logo.png" alt="spring logo" class="logo">
</figure>

[JSON API](http://jsonapi.org/){:target="_blank"} is a specification for building APIs using JSON. It details how clients should request resources to be fetched or modified, and how servers should respond to those requests. JSON API is designed to minimize both the number of requests and the amount of data transmitted between clients and servers.

[Katharsis](http://katharsis.io){:target="_blank"} is a Java library that implements the JSON API specification. It is an additional layer that can be plugged on top of existing server-side Java implementations in order to provide easy [HATEOAS](https://en.wikipedia.org/wiki/HATEOAS){:target="_blank"} support. Katharsis defines resources which can be shared over a RESTful interface and a repository for handling them.

The below tutorial details how to configure and build a RESTful Hello World API that complies with the JSON API specification. Frameworks used are Katharsis, OkHttp, and Spring Boot.

Tools used:
* Katharsis 3.0
* OkHttp 3.6
* Spring Boot 1.5
* Maven 3.5

# General Project Setup

The example will be built and run using [Apache Maven](https://maven.apache.org/){:target="_blank"}. In order to use the Katharsis framework, we need to include the `katharsis-core` dependency. As Spring Boot will be used for running the server part we also need to include `katharsis-spring`. For implementing the client part the `katharsis-client` dependency is needed.

Katharsis aims to have as little dependencies as possible, as such a HTTP client library is not included by default. It needs to be [provided on the classpath where it will be automatically picked up](http://katharsis-jsonapi.readthedocs.io/en/latest/user-docs.html#client){:target="_blank"} by the framework. Both [OkHttp](https://square.github.io/okhttp){:target="_blank"} and [Apache Http Client](https://hc.apache.org/httpcomponents-client-ga/index.html){:target="_blank"} are supported. For this example we will use the `okhttp` library.

Running and testing of the example is based on the `spring-boot-starter` and `spring-boot-starter-test` [Spring Boot starters](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters){:target="_blank"}. The `spring-boot-maven-plugin` is used to spin up an embedded Tomcat server that will host the RESTful Hello World API.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>json-api-katharsis-helloworld</artifactId>
  <version>0.0.1-SNAPSHOT</version>

  <name>json-api-katharsis-helloworld</name>
  <description>JSON API - Spring Boot Katharsis Example</description>
  <url>https://www.codenotfound.com/json-api-katharsis-spring-boot-example.html</url>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.9.RELEASE</version>
  </parent>

  <properties>
    <java.version>1.8</java.version>
    <katharsis.version>3.0.2</katharsis.version>
    <okhttp.version>3.9.1</okhttp.version>
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

The `@JsonApiResource` annotation [defines a resource](http://katharsis-jsonapi.readthedocs.io/en/latest/user-docs.html#jsonapiresource){:target="_blank"}. It requires the type parameter to be set which will be used to form the URI and populate the [type field](http://jsonapi.org/format/#document-resource-objects) which is passed as part of the resource object.

> According to JSON API standard, the name defined in type can be either plural or singular. However, the same value should be used consistently throughout an implementation.

The `@JsonApiId` [defines a field which will be used as an identifier](http://katharsis-jsonapi.readthedocs.io/en/latest/user-docs.html#jsonapiid){:target="_blank"} of the Greeting resource. Each resource requires this annotation to be present on a field which is of primitive type or a type that implements the `Serializable` interface.

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

To allow Katharsis to operate on defined resources, a special type of class called [repository](http://katharsis-jsonapi.readthedocs.io/en/latest/user-docs.html#repositories){:target="_blank"} needs to be created. Katharsis will scan for these classes and using annotations it will discover the available methods.

For our `Greeting` resource we will create a `GreetingRepositoryImpl` which extends the `ResourceRepositoryBase` implementation of the `ResourceRepositoryV2` repository interface. 
The [ResourceRepositoryBase](https://github.com/katharsis-project/katharsis-framework/blob/master/katharsis-core/src/main/java/io/katharsis/repository/ResourceRepositoryBase.java){:target="_blank"} is a base class that takes care of some boiler-plate code like for example implementing `findOne()` and `findAll()`.

> Note that when extending the `ResourceRepositoryBase` only `findAll()` needs to be implemented to have a working read-only repository.

In this example, we will store the greeting resources in a simple `HashMap` and add a "Hello World!" greeting as a resource via the constructor.

Katharsis passes JSON API query parameters to repositories through a `QuerySpec` parameter. The implementation of the `findAll()` uses the `apply()` method which evaluates the querySpec against the provided list and returns the result.

> Note that the `@Component` annotation is needed so that Katharsis is able to find the repository.

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

    greetings.put(1L, new Greeting(1L, "Hello World!"));
  }

  @Override
  public synchronized ResourceList<Greeting> findAll(QuerySpec querySpec) {
    return querySpec.apply(greetings.values());
  }
}
```

# Setting up the Server

Katharsis comes with [out-of-the-box support for Spring Boot](http://katharsis-jsonapi.readthedocs.io/en/latest/user-docs.html#spring-integration){:target="_blank"}. The entry point is a `KatharsisConfig` class which configures Katharsis using Spring properties. Additionally, we have to make sure that each repository is annotated with `@Component` (as we did with the above `GreetingRepositoryImpl`).

Spring's `@RestController` annotation is used to mark the `KatharsisController` class as a controller for handling HTTP requests. The `KatharsisConfigV3` import will setup and expose the resource endpoints based on auto-scanning for specific annotations in addition to some properties which we will see further below.

Katharsis also ships with a `ResourceRegistry` which holds information on all the registered repositories. We expose this information on <var>/resources-info</var> by iterating over all the entries in the auto-wired `ResourceRegistry`.

``` java
package com.codenotfound.katharsis.server;

import java.util.HashMap;
import java.util.Map;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Import;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import io.katharsis.resource.registry.RegistryEntry;
import io.katharsis.resource.registry.ResourceRegistry;
import io.katharsis.spring.boot.v3.KatharsisConfigV3;

@RestController
@Import({KatharsisConfigV3.class})
public class KatharsisController {

  @Autowired
  private ResourceRegistry resourceRegistry;

  @RequestMapping("/resources-info")
  public Map<String, String> getResources() {
    Map<String, String> result = new HashMap<>();
    // Add all resources
    for (RegistryEntry entry : resourceRegistry.getResources()) {
      result.put(entry.getResourceInformation().getResourceType(),
          resourceRegistry.getResourceUrl(entry.getResourceInformation()));
    }

    return result;
  }
}
```

In addition to the above auto-configuration, a number of items are configured using the <var>application.yml</var> properties file:

* The <var>katharsis:resourcePackage</var> property specifies which package(s) should be searched to find models and repositories used by the core and exception mappers. Multiple packages can be passed by specifying a comma-separated string of packages.
* The <var>katharsis:domainName</var> property specifies the domain name as well as protocol and optionally port number to be used when building links objects in responses. The value must not end with a backslash (/).
* The <var>katharsis:pathPrefix</var> property defines the default prefix of the URL path used when building link objects in responses and when performing method matching.

``` yaml
katharsis:
  resourcePackage: com.codenotfound.katharsis.domain
  domainName: http://localhost:9090/codenotfound
  pathPrefix: /api

server:
  context-path: /codenotfound
  port: 9090
```

# Setting up the Client

Katharsis includes a [Java client](http://katharsis-jsonapi.readthedocs.io/en/latest/user-docs.html#client){:target="_blank"} which allows to communicate with JSON-API compliant servers. To start using the client just create an instance of `KatharsisClient` and pass the service URL. Then use the client to create a repository that gives access to the different resource CRUD operations.

In below `GreetingClient` we have created a `findOne()` method that returns a single `Greeting` based on the identifier.

``` java
package com.codenotfound.katharsis.client;

import javax.annotation.PostConstruct;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;

import com.codenotfound.katharsis.domain.model.Greeting;

import io.katharsis.client.KatharsisClient;
import io.katharsis.queryspec.QuerySpec;
import io.katharsis.repository.ResourceRepositoryV2;

@Component
public class GreetingClient {

  private static final Logger LOGGER = LoggerFactory.getLogger(GreetingClient.class);

  private KatharsisClient katharsisClient =
      new KatharsisClient("http://localhost:9090/codenotfound/api");
  private ResourceRepositoryV2<Greeting, Long> resourceRepositoryV2;

  @PostConstruct
  public void init() {
    resourceRepositoryV2 = katharsisClient.getRepositoryForType(Greeting.class);
  }

  public Greeting findOne(long id) {
    Greeting result = resourceRepositoryV2.findOne(id, new QuerySpec(Greeting.class));

    LOGGER.info("found {}", result.toString());
    return result;
  }
}
```

# Testing the Hello World JSON API

Let's test our greeting resource! If you are running Google Chrome you can simply do this from the browser. Make sure you start the application by starting up Spring Boot:

``` plaintext
mvn spring-boot:run
```

Next open following URL which should show all our registered resources (in this example the greetings resource).

``` plaintext
http://localhost:9090/codenotfound/resources-info
```

<figure>
    <img src="{{ site.url }}/assets/images/posts/json-api/hello-world-resource-registry.png" alt="hello world resource registry">
</figure>

Go ahead and use the greetings endpoint as next URL. This will show us all the available greetings.

``` plaintext
http://localhost:9090/codenotfound/api/greetings
```

<figure>
    <img src="{{ site.url }}/assets/images/posts/json-api/hello-world-greetings.png" alt="hello world greetings">
</figure>

For the details of our Hello World resource we enter below URL.

``` plaintext
http://localhost:9090/codenotfound/api/greetings/1
```

<figure>
    <img src="{{ site.url }}/assets/images/posts/json-api/hello-world-greeting-1.png" alt="hello world greeting 1">
</figure>

> Notice that above flow is fully driven using hypermedia!

To wrap up we will also add a simple `SpringKatharsisApplicationTest` unit test case in which we lookup the Hello World greeting resource using its identifier.

``` java
package com.codenotfound.katharsis;

import static org.assertj.core.api.Assertions.assertThat;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.SpringBootTest.WebEnvironment;
import org.springframework.test.context.junit4.SpringRunner;

import com.codenotfound.katharsis.client.GreetingClient;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = WebEnvironment.DEFINED_PORT)
public class SpringKatharsisApplicationTest {

  @Autowired
  GreetingClient greetingClient;

  @Test
  public void testFindOne() {
    assertThat(greetingClient.findOne(1L).getContent()).isEqualTo("Hello World!");
  }
}
```

Triggering the test case is done using following maven command.

``` plaintext
mvn test
```

The output should be as shown below.

``` plaintext
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.5.9.RELEASE)

09:35:41.763 [main] INFO  c.c.k.SpringKatharsisApplicationTest - Starting SpringKatharsisApplicationTest on cnf-pc with PID 1740 (started by CodeNotFound in c:\code\json-api\json-api-katharsis-helloworld)
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
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/json-api/tree/master/json-api-katharsis-helloworld){:target="_blank"}.
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This wraps up the Katharsis JSON API tutorial in which we developed a simple Hello World example from scratch and exposed it via a RESTful API.

If you liked this tutorial or if you would like other Katharsis topics to be covered please leave a comment below.
