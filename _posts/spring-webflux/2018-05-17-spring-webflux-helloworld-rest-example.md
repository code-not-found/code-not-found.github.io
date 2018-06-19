---
title: "Spring WebFlux - Hello World REST Example"
permalink: /spring-webflux-hello-world-rest-example.html
excerpt: "A detailed step-by-step tutorial on how to implement a Hello World WebFlux REST example using Spring Boot and Maven."
date: 2018-05-17
last_modified_at: 2018-05-17
header:
  teaser: "assets/images/teaser/spring-teaser.png"
categories: [Spring WebFlux]
tags: [Example, Hello World, Maven, REST, Spring Boot, Spring WebFlux, Tutorial]
published: false
---

<figure>
  <img src="{{ site.url }}/assets/images/logo/spring-logo.png" alt="spring logo" class="logo">
</figure>

Spring Framework 5 includes a new [spring-webflux](https://docs.spring.io/spring/docs/5.0.6.RELEASE/spring-framework-reference/web-reactive.html#webflux){:target="_blank"} module which contains support for reactive HTTP and WebSocket clients as well as reactive server web applications including REST style interactions.

The Spring Framework uses [Reactor](https://projectreactor.io/){:target="_blank"} which is a Reactive library for building non-blocking applications based on the [Reactive Streams Specification](http://www.reactive-streams.org/){:target="_blank"}. 

In the following tutorial, we will configure, build and run a WebFlux REST API using Spring Boot, and Maven.

# General Project Setup

Tools used:
* Reactor 3.1
* Spring Boot 2.0
* Maven 3.5

We will be building and running our example using [Apache Maven](https://maven.apache.org/){:target="_blank"}. Shown below is the XML representation of our Maven project in a POM file. It contains the needed dependencies for compiling and running the example.

In order to configure and expose the Hello World gRPC service endpoint, we will use the [Spring Boot](https://projects.spring.io/spring-boot/){:target="_blank"} project.

To facilitate the management of the different Spring dependencies, [Spring Boot Starters](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters){:target="_blank"} are used which are a set of convenient dependency descriptors that you can include in your application. We include the `spring-boot-starter-web` dependency which automatically sets up an embedded Apache Tomcat that will host our gRPC service endpoint.

 The `spring-boot-starter-test` includes the dependencies for testing Spring Boot applications with libraries that include [JUnit](http://junit.org/junit4/){:target="_blank"}, [Hamcrest](http://hamcrest.org/JavaHamcrest/){:target="_blank"} and [Mockito](http://site.mockito.org/){:target="_blank"}.

The [Spring boot starter for gRPC framework](https://github.com/LogNet/grpc-spring-boot-starter){:target="_blank"} auto-configures and runs an embedded gRPC server with `@GRpcService` enabled Beans as part of a Spring Boot application. The starter supports both Spring Boot version 1.5.X and 2.X.X applications. We enable it by including the `grpc-spring-boot-starter` dependency.

Protocol buffers support generated code in a number of [programming languages](https://developers.google.com/protocol-buffers/docs/tutorials){:target="_blank"}. This tutorial focuses on Java.

There are multiple ways to generate the protobuf-based code and for this example we will use the [protobuf-maven-plugin](https://github.com/xolstice/protobuf-maven-plugin){:target="_blank"} as documented on the [grpc-java](https://github.com/grpc/grpc-java){:target="_blank"} GitHub page.

We also include the [os-maven-plugin](https://github.com/trustin/os-maven-plugin){:target="_blank"} extension that generates various useful platform-dependent project properties. This information is needed as the Protocol Buffer compiler is native code. In other words, the `protobuf-maven-plugin` needs to fetch the correct compiler for the platform it is running on.

Finally, the plugins section includes the `spring-boot-maven-plugin` Maven plugin so that we can build a single, runnable "uber-jar". This will also allow us to startup our gRPC server via a Maven command.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>grpc-spring-boot-helloworld</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>grpc-spring-boot-helloworld</name>
  <description>gRPC - Spring Boot Hello World Example</description>
  <url>https://www.codenotfound.com/grpc-spring-boot-example.html</url>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.1.RELEASE</version>
    <relativePath /> <!-- lookup parent from repository -->
  </parent>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>1.8</java.version>

    <grpc-spring-boot-starter.version>2.3.2</grpc-spring-boot-starter.version>

    <os-maven-plugin.version>1.6.0</os-maven-plugin.version>
    <protobuf-maven-plugin.version>0.5.1</protobuf-maven-plugin.version>
  </properties>

  <repositories>
    <repository>
      <id>jcenter</id>
      <url>https://jcenter.bintray.com/</url>
    </repository>
  </repositories>

  <dependencies>
    <!-- spring-boot -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.lognet</groupId>
      <artifactId>grpc-spring-boot-starter</artifactId>
      <version>${grpc-spring-boot-starter.version}</version>
    </dependency>
  </dependencies>

  <build>
    <extensions>
      <!-- os-maven-plugin -->
      <extension>
        <groupId>kr.motd.maven</groupId>
        <artifactId>os-maven-plugin</artifactId>
        <version>${os-maven-plugin.version}</version>
      </extension>
    </extensions>
    <plugins>
      <!-- spring-boot-maven-plugin -->
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
      <!-- protobuf-maven-plugin -->
      <plugin>
        <groupId>org.xolstice.maven.plugins</groupId>
        <artifactId>protobuf-maven-plugin</artifactId>
        <version>${protobuf-maven-plugin.version}</version>
        <configuration>
          <protocArtifact>com.google.protobuf:protoc:3.5.1-1:exe:${os.detected.classifier}</protocArtifact>
          <pluginId>grpc-java</pluginId>
          <pluginArtifact>io.grpc:protoc-gen-grpc-java:1.11.0:exe:${os.detected.classifier}</pluginArtifact>
        </configuration>
        <executions>
          <execution>
            <goals>
              <goal>compile</goal>
              <goal>compile-custom</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
</project>
```

The `protobuf-maven-plugin` will generate Java artifacts for the <var>HelloWorld.proto</var> file located in <var>src/main/proto/</var> (this is the default location the plugin uses).

Execute following Maven command, and the different message and service classes should be generated under <var>target/generated-sources/protobuf/</var> as shown below.

``` plaintext
mvn compile
```

<figure>
  <img src="{{ site.url }}/assets/images/posts/grpc/protobuf-generated-classes.png" alt="protobuf generated classes">
</figure>

We also create a `SpringGRPCApplication` that contains a `main()` method that uses Spring Boot's `SpringApplication.run()` method to bootstrap the application, starting Spring.

For more information on Spring Boot, we refer to the [Spring Boot getting started guide](https://spring.io/guides/gs/spring-boot/){:target="_blank"}.

``` java
package com.codenotfound.grpc;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringGRPCApplication {

  public static void main(String[] args) {
    SpringApplication.run(SpringGRPCApplication.class, args);
  }
}
```

# Creating the WebFlux REST Controller

The service implementation is defined in the `HelloWorldServiceImpl` POJO that implements the `HelloWorldServiceImplBase` class that was generated from the <var>HelloWorld.proto</var> file. We override the `sayHello()` method and generate a `Greeting` response based on the first and last name of the `Person` passed in the request.

> Note that the response is a `StreamObserver` object. In other words, the service is by default asynchronous and whether you want to block or not when receiving the response(s) is the decision of the client as we will see further below.

We use the response observer's `onNext()` method to return the `Greeting` and then call the response observer's `onCompleted()` method to tell gRPC that we've finished writing responses.

The `HelloWorldServiceImpl` POJO is annotated with `@GRpcService` which auto-configures the specified gRPC service to be exposed on port <var>6565</var>.

``` java
package com.codenotfound.grpc.server;

import org.lognet.springboot.grpc.GRpcService;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import com.codenotfound.grpc.helloworld.Greeting;
import com.codenotfound.grpc.helloworld.HelloWorldServiceGrpc;
import com.codenotfound.grpc.helloworld.Person;

import io.grpc.stub.StreamObserver;

@GRpcService
public class HelloWorldServiceImpl
    extends HelloWorldServiceGrpc.HelloWorldServiceImplBase {

  private static final Logger LOGGER =
      LoggerFactory.getLogger(HelloWorldServiceImpl.class);

  @Override
  public void sayHello(Person request,
      StreamObserver<Greeting> responseObserver) {
    LOGGER.info("server received {}", request);

    String message = "Hello " + request.getFirstName() + " "
        + request.getLastName() + "!";
    Greeting greeting =
        Greeting.newBuilder().setMessage(message).build();
    LOGGER.info("server responded {}", greeting);

    responseObserver.onNext(greeting);
    responseObserver.onCompleted();
  }
}
```

# Creating the Client

To call gRPC service methods, we first need to create a stub. There are two types of stubs available: a **blocking/synchronous stub** that will wait for the server to respond and a **non-blocking/asynchronous stub** that makes non-blocking calls to the server, where the response is returned asynchronously.

The client code is specified in the `HelloWorldClient` class.

In order to transport messages, gRPC uses http/2 and some abstraction layers in between. This complexity is hidden behind a `MessageChannel` that handles the connectivity. The general recommendation is to use one channel per application and share it among service stubs.

We use an `init()` method annotated with `@PostConstruct` in order to build a new `MessageChannel` right after the after the bean has been initialized. The channel is then used to create the `helloWorldServiceBlockingStub` stub.

> gRPC by default uses a secure connection mechanism such as TLS. As this is a simple development test will use `usePlaintext()` in order to avoid having to setup the different security artifacts such as key/trust stores.

 The `sayHello()` method creates a Person object using the Builder pattern on which we set the <var>'firstname'</var> and <var>'lastname'</var> input parameters.

The `helloWorldServiceBlockingStub` is then used to send the request towards the Hello World gRPC service. The result is a `Greeting` object from which we return the containing message.

We annotate the client with `@Component` which will cause Spring to automatically create and import below bean into the container if automatic component scanning is enabled (adding the `@SpringBootApplication` annotation to the main `SpringWsApplication` class [is equivalent](https://docs.spring.io/autorepo/docs/spring-boot/2.0.1.RELEASE/reference/html/using-boot-using-springbootapplication-annotation.html){:target="_blank"} to using `@ComponentScan`).

``` java
package com.codenotfound.grpc.client;

import javax.annotation.PostConstruct;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;

import com.codenotfound.grpc.helloworld.Greeting;
import com.codenotfound.grpc.helloworld.HelloWorldServiceGrpc;
import com.codenotfound.grpc.helloworld.Person;

import io.grpc.ManagedChannel;
import io.grpc.ManagedChannelBuilder;

@Component
public class HelloWorldClient {

  private static final Logger LOGGER =
      LoggerFactory.getLogger(HelloWorldClient.class);

  private HelloWorldServiceGrpc.HelloWorldServiceBlockingStub helloWorldServiceBlockingStub;

  @PostConstruct
  private void init() {
    ManagedChannel managedChannel = ManagedChannelBuilder
        .forAddress("localhost", 6565).usePlaintext().build();

    helloWorldServiceBlockingStub =
        HelloWorldServiceGrpc.newBlockingStub(managedChannel);
  }

  public String sayHello(String firstName, String lastName) {
    Person person = Person.newBuilder().setFirstName(firstName)
        .setLastName(lastName).build();
    LOGGER.info("client sending {}", person);

    Greeting greeting =
        helloWorldServiceBlockingStub.sayHello(person);
    LOGGER.info("client received {}", greeting);

    return greeting.getMessage();
  }

}
```

# Testing the gRPC Spring Boot Example

Let's wrap up by creating a basic unit test case in which the above client is used to send a request to the gRPC Hello World service endpoint. We then verify if the response is equal to the expected greeting.

The `@RunWith` and `@SpringBootTest` testing annotations, [that were introduced with Spring Boot 1.4](https://spring.io/blog/2016/04/15/testing-improvements-in-spring-boot-1-4#spring-boot-1-4-simplifications){:target="_blank"}, are used to tell JUnit to run using Spring's testing support and bootstrap with Spring Boot's support.

The `HelloWorldClient` Bean is autowired so we can use it in the test case. The service itself is automatically started by the `@SpringBootTest` annotation. All that is left to do is to compare the received result to the expected greeting message using an assert statement.

``` java
package com.codenotfound.grpc;

import static org.assertj.core.api.Assertions.assertThat;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import com.codenotfound.grpc.client.HelloWorldClient;

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringGRPCApplicationTests {

  @Autowired
  private HelloWorldClient helloWorldClient;

  @Test
  public void testSayHello() {
    assertThat(helloWorldClient.sayHello("John", "Doe"))
        .isEqualTo("Hello John Doe!");
  }
}
```

Run the above test case by opening a command prompt in the projects root folder and executing following Maven command:

``` plaintext
mvn test
```

The result should be a successful build during which the gRPC server is started and a call is made to the Hello World service.

``` plaintext
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.0.1.RELEASE)

2018-05-15 05:14:45.942  INFO 5972 --- [           main] c.c.grpc.SpringGRPCApplicationTests      : Starting SpringGRPCApplicationTests on cnf-pc with PID 5972 (started by CodeNotFound in c:\blogs\codenotfound\code\grpc\grpc-spring-boot-helloworld)
2018-05-15 05:14:45.944  INFO 5972 --- [           main] c.c.grpc.SpringGRPCApplicationTests      : No active profile set, falling back to default profiles: default
2018-05-15 05:14:45.983  INFO 5972 --- [           main] o.s.w.c.s.GenericWebApplicationContext   : Refreshing org.springframework.web.context.support.GenericWebApplicationContext@4c60d6e9: startup date [Tue May 15 05:14:45 CEST 2018]; root of context hierarchy
2018-05-15 05:14:47.874  INFO 5972 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**/favicon.ico] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2018-05-15 05:14:48.180  INFO 5972 --- [           main] s.w.s.m.m.a.RequestMappingHandlerAdapter : Looking for @ControllerAdvice: org.springframework.web.context.support.GenericWebApplicationContext@4c60d6e9: startup date [Tue May 15 05:14:45 CEST 2018]; root of context hierarchy
2018-05-15 05:14:48.275  INFO 5972 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/error]}" onto public org.springframework.http.ResponseEntity<java.util.Map<java.lang.String, java.lang.Object>> org.springframework.boot.autoconfigure.web.servlet.error.BasicErrorController.error(javax.servlet.http.HttpServletRequest)
2018-05-15 05:14:48.277  INFO 5972 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/error],produces=[text/html]}" onto public org.springframework.web.servlet.ModelAndView org.springframework.boot.autoconfigure.web.servlet.error.BasicErrorController.errorHtml(javax.servlet.http.HttpServletRequest,javax.servlet.http.HttpServletResponse)
2018-05-15 05:14:48.310  INFO 5972 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/webjars/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2018-05-15 05:14:48.310  INFO 5972 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2018-05-15 05:14:48.598  INFO 5972 --- [           main] c.c.grpc.SpringGRPCApplicationTests      : Started SpringGRPCApplicationTests in 2.89 seconds (JVM running for 3.8)
2018-05-15 05:14:48.601  INFO 5972 --- [           main] o.l.springboot.grpc.GRpcServerRunner     : Starting gRPC Server ...
2018-05-15 05:14:48.622  INFO 5972 --- [           main] o.l.springboot.grpc.GRpcServerRunner     : 'com.codenotfound.grpc.server.HelloWorldServiceImpl' service has been registered.
2018-05-15 05:14:48.791  INFO 5972 --- [           main] o.l.springboot.grpc.GRpcServerRunner     : gRPC Server started, listening on port 6565.
2018-05-15 05:14:48.929  INFO 5972 --- [           main] c.c.grpc.client.HelloWorldClient         : client sending first_name: "John"
last_name: "Doe"

2018-05-15 05:14:49.284  INFO 5972 --- [ault-executor-0] c.c.grpc.server.HelloWorldServiceImpl    : server received first_name: "John"
last_name: "Doe"

2018-05-15 05:14:49.286  INFO 5972 --- [ault-executor-0] c.c.grpc.server.HelloWorldServiceImpl    : server responded message: "Hello John Doe!"

2018-05-15 05:14:49.298  INFO 5972 --- [           main] c.c.grpc.client.HelloWorldClient         : client received message: "Hello John Doe!"

[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 4.487 s - in com.codenotfound.grpc.SpringGRPCApplicationTests
2018-05-15 05:14:49.598  INFO 5972 --- [       Thread-2] o.s.w.c.s.GenericWebApplicationContext   : Closing org.springframework.web.context.support.GenericWebApplicationContext@4c60d6e9: startup date
[Tue May 15 05:14:45 CEST 2018]; root of context hierarchy
2018-05-15 05:14:49.600  INFO 5972 --- [       Thread-2] o.l.springboot.grpc.GRpcServerRunner     : Shutting down gRPC server ...
2018-05-15 05:14:49.602  INFO 5972 --- [       Thread-2] o.l.springboot.grpc.GRpcServerRunner     : gRPC server stopped.
[INFO]
[INFO] Results:
[INFO]
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 8.902 s
[INFO] Finished at: 2018-05-15T05:14:49+02:00
[INFO] ------------------------------------------------------------------------
```

If you just want to start Spring Boot so that the service endpoint is up and running, execute following Maven command.

``` plaintext
mvn spring-boot:run
```

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/grpc/tree/master/grpc-spring-boot-helloworld){:target="_blank"}.
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This concludes our tutorial on how to implement a gRPC service and the corresponding client using a Spring Boot starter and Maven.

If you liked this example or have a question you would like to ask, leave a comment below!
