---
title: "Spring WS - SOAP Web Service Consumer Provider WSDL Example"
permalink: /spring-ws-soap-web-service-consumer-provider-wsdl-example.html
excerpt: "A detailed step-by-step tutorial on how to implement a Hello World web service starting from a WSDL and using Spring-WS and Spring Boot."
date: 2016-10-12
modified: 2017-04-18
header:
  teaser: "assets/images/header/spring-ws-teaser.png"
categories: [Spring-WS]
tags: [Client, Consumer, Endpoint, Example, Hello World, Maven, Provider, Spring, Spring Boot, Spring Web Services, Spring-WS, Tutorial, WSDL]
redirect_from:
  - /2016/10/consume-provide-web-service-wsdl.html
  - /2016/10/spring-ws-soap-web-service-consumer-provider-wsdl-example.html
published: true
---

<figure>
    <img src="{{ site.url }}/assets/images/logo/spring-logo.png" alt="spring logo" class="logo">
</figure>

[Spring Web Services](http://projects.spring.io/spring-ws/){:target="_blank"} (Spring-WS) is a product of the Spring community focused on creating document-driven Web services. Spring-WS facilitates contract-first SOAP service development, allowing for a number of ways to manipulate XML payloads.

The following step by step tutorial illustrates a basic example in which we will configure, build and run a Hello World contract first client and endpoint using a WSDL, Spring-WS, Spring Boot and Maven.

The tutorial code is organized in such a way that you can choose to only run the [client]({{ site.url }}/spring-ws-soap-web-service-consumer-provider-wsdl-example.html#creating-the-client-consumer) (consumer) or [endpoint]({{ site.url }}/spring-ws-soap-web-service-consumer-provider-wsdl-example.html#creating-the-endpoint-provider) (provider) part. In the below example we will setup both parts and then make an end-to-end test in which the client calls the endpoint.

If you want to learn more about Spring WS - head on over to the [Spring-WS tutorials page]({{ site.url }}/spring-ws/).
{: .notice--primary}

# General Project Setup

Tools used:
* Spring-WS 2.4
* Spring Boot 1.5
* Maven 3.5

As Spring Web Services is **contract first only**, we need to start from a contract definition. In this tutorial, we will use a Hello World service that is defined by below WSDL. The service takes as input a person's first and last name and returns a greeting.

``` xml
<?xml version="1.0"?>
<wsdl:definitions name="HelloWorld"
  targetNamespace="http://codenotfound.com/services/helloworld"
  xmlns:tns="http://codenotfound.com/services/helloworld" xmlns:types="http://codenotfound.com/types/helloworld"
  xmlns:soap="http://schemas.xmlsoap.org/wsdl/soap/" xmlns:wsdl="http://schemas.xmlsoap.org/wsdl/">

  <wsdl:types>
    <xsd:schema targetNamespace="http://codenotfound.com/types/helloworld"
      xmlns:xsd="http://www.w3.org/2001/XMLSchema" elementFormDefault="qualified"
      attributeFormDefault="unqualified" version="1.0">

      <xsd:element name="person">
        <xsd:complexType>
          <xsd:sequence>
            <xsd:element name="firstName" type="xsd:string" />
            <xsd:element name="lastName" type="xsd:string" />
          </xsd:sequence>
        </xsd:complexType>
      </xsd:element>

      <xsd:element name="greeting">
        <xsd:complexType>
          <xsd:sequence>
            <xsd:element name="greeting" type="xsd:string" />
          </xsd:sequence>
        </xsd:complexType>
      </xsd:element>
    </xsd:schema>
  </wsdl:types>

  <wsdl:message name="SayHelloInput">
    <wsdl:part name="person" element="types:person" />
  </wsdl:message>

  <wsdl:message name="SayHelloOutput">
    <wsdl:part name="greeting" element="types:greeting" />
  </wsdl:message>

  <wsdl:portType name="HelloWorld_PortType">
    <wsdl:operation name="sayHello">
      <wsdl:input message="tns:SayHelloInput" />
      <wsdl:output message="tns:SayHelloOutput" />
    </wsdl:operation>
  </wsdl:portType>

  <wsdl:binding name="HelloWorld_SoapBinding" type="tns:HelloWorld_PortType">
    <soap:binding style="document"
      transport="http://schemas.xmlsoap.org/soap/http" />
    <wsdl:operation name="sayHello">
      <soap:operation
        soapAction="http://codenotfound.com/services/helloworld/sayHello" />
      <wsdl:input>
        <soap:body use="literal" />
      </wsdl:input>
      <wsdl:output>
        <soap:body use="literal" />
      </wsdl:output>
    </wsdl:operation>
  </wsdl:binding>

  <wsdl:service name="HelloWorld_Service">
    <wsdl:documentation>Hello World service</wsdl:documentation>
    <wsdl:port name="HelloWorld_Port" binding="tns:HelloWorld_SoapBinding">
      <soap:address location="http://localhost:9090/codenotfound/ws/helloworld" />
    </wsdl:port>
  </wsdl:service>

</wsdl:definitions>
```

We will be building and running our example using [Apache Maven](https://maven.apache.org/){:target="_blank"}. Shown below is the XML representation of our Maven project in a POM file. It contains the needed dependencies for compiling and running our example.

In order to expose the Hello World service endpoint, we will use the [Spring Boot](https://projects.spring.io/spring-boot/){:target="_blank"} project that comes with an embedded Apache Tomcat server. To facilitate the management of the different Spring dependencies, [Spring Boot Starters](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters){:target="_blank"} are used which are a set of convenient dependency descriptors that you can include in your application.

The `spring-boot-starter-web-services` dependency includes the needed dependencies for using Spring Web Services. The `spring-boot-starter-test` includes the dependencies for testing Spring Boot applications with libraries that include [JUnit](http://junit.org/junit4/){:target="_blank"}, [Hamcrest](http://hamcrest.org/JavaHamcrest/){:target="_blank"} and [Mockito](http://site.mockito.org/){:target="_blank"}.

To avoid having to manage the version compatibility of the different Spring dependencies, we will inherit the defaults from the `spring-boot-starter-parent` parent POM.

In the plugins section, we included the `spring-boot-maven-plugin` Maven plugin so that we can build a single, runnable "über-jar". This will also allow us to start the web service via a Maven command.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>spring-ws-helloworld</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>spring-ws-helloworld</name>
  <description>Spring WS - SOAP Web Service Consumer &amp; Provider WSDL Example</description>
  <url>https://www.codenotfound.com/spring-ws-soap-web-service-consumer-provider-wsdl-example.html</url>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.9.RELEASE</version>
  </parent>

  <properties>
    <java.version>1.8</java.version>
    <maven-jaxb2-plugin.version>0.13.3</maven-jaxb2-plugin.version>
  </properties>

  <dependencies>
    <!-- spring-boot -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web-services</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
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
      <!-- maven-jaxb2-plugin -->
      <plugin>
        <groupId>org.jvnet.jaxb2.maven2</groupId>
        <artifactId>maven-jaxb2-plugin</artifactId>
        <version>${maven-jaxb2-plugin.version}</version>
        <executions>
          <execution>
            <goals>
              <goal>generate</goal>
            </goals>
          </execution>
        </executions>
        <configuration>
          <schemaDirectory>${project.basedir}/src/main/resources/wsdl</schemaDirectory>
          <schemaIncludes>
            <include>*.wsdl</include>
          </schemaIncludes>
        </configuration>
      </plugin>
    </plugins>
  </build>
</project>
```

In order to directly use the <var>'person'</var> and <var>'greeting'</var> elements (defined in the <var>'types'</var> section of the Hello World WSDL) in our Java code, we will use [JAXB](http://www.oracle.com/technetwork/articles/javase/index-140168.html){:target="_blank"} to generate the corresponding Java classes. The above POM file configures the [maven-jaxb2-plugin](https://github.com/highsource/maven-jaxb2-plugin){:target="_blank"} that will handle the generation.

The plugin will look into the defined <var>'&lt;schemaDirectory&gt;'</var> in order to find any WSDL files for which it needs to generate the Java classes. In order to trigger the generation via Maven, executed following command:

``` plaintext
mvn generate-sources
```

This results in a number of generated classes amongst which the `Person` and `Greeting` that we will use when implementing the client and provider of the Hello World service.

<figure>
    <img src="{{ site.url }}/assets/images/posts/spring-ws/hello-world-jaxb-generated-java-classes.png" alt="helloworld jaxb generated java classes">
</figure>

We start by creating an `SpringWsApplication` that contains a `main()` method that uses Spring Boot's `SpringApplication.run()` method to bootstrap the application, starting Spring. For more information on Spring Boot, we refer to the [Spring Boot getting started guide](https://spring.io/guides/gs/spring-boot/){:target="_blank"}.

``` java
package com.codenotfound.ws;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringWsApplication {

  public static void main(String[] args) {
    SpringApplication.run(SpringWsApplication.class, args);
  }
}
```

# Creating the Endpoint (Provider)

The server-side of Spring-WS is designed around a central class called `MessageDispatcher` that dispatches incoming XML messages to endpoints. For more detailed information check out the [Spring Web Services reference documentation on the MessageDispatcher](https://docs.spring.io/spring-ws/docs/2.4.2.RELEASE/reference/#_the_code_messagedispatcher_code){:target="_blank"}.

Spring Web Services supports multiple transport protocols. The most common is the HTTP transport, for which a custom `MessageDispatcherServlet` servlet is supplied. This is a standard `Servlet` which extends from the standard Spring Web `DispatcherServlet` ([=central dispatcher for HTTP request handlers/controllers](http://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/DispatcherServlet.html){:target="_blank"}), and wraps a `MessageDispatcher`.

> In other words: the `MessageDispatcherServlet` combines the attributes of the `MessageDispatcher` and `DispatcherServlet` and as a result allows the handling of XML messages over HTTP.

In the below `WebServiceConfig` configuration class we use a `ServletRegistrationBean` to register the `MessageDispatcherServlet`. Note that it is important to inject and set the ApplicationContext to the `MessageDispatcherServlet`, otherwise it will not automatically detect other Spring Web Services related beans (such as the lower `Wsdl11Definition`). By naming this bean <var>'messageDispatcherServlet'</var>, it does [not replace Spring Boot's default DispatcherServlet bean](http://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#howto-switch-off-the-spring-mvc-dispatcherservlet){:target="_blank"}.

The servlet mapping URI pattern on the `ServletRegistrationBean` is set to "<kbd>/codenotfound/ws/*</kbd>". The web container will use this path to map incoming HTTP requests to the servlet.

The `DefaultWsdl11Definition` exposes a standard WSDL 1.1 using the specified Hello World WSDL file. The URL location at which this WSDL is available is determined by it's `Bean` name in combination with the URI mapping of the `MessageDispatcherServlet`. For the example below this is: [host]="<kbd>http://localhost:9090</kbd>"+[servlet mapping uri]="<kbd>/codenotfound/ws/</kbd>"+[WsdlDefinition bean name]="<kbd>helloworld</kbd>"+[WSDL postfix]="<kbd>.wsdl</kbd>" or [http://localhost:9090/codenotfound/ws/helloworld.wsdl](http://localhost:9090/codenotfound/ws/helloworld.wsdl){:target="_blank"}.

To enable the support for `@Endpoint` annotation that we will use in the next section we need to annotate our configuration class with `@EnableWs`.

``` java
package com.codenotfound.ws.endpoint;

import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.ClassPathResource;
import org.springframework.ws.config.annotation.EnableWs;
import org.springframework.ws.config.annotation.WsConfigurerAdapter;
import org.springframework.ws.transport.http.MessageDispatcherServlet;
import org.springframework.ws.wsdl.wsdl11.SimpleWsdl11Definition;
import org.springframework.ws.wsdl.wsdl11.Wsdl11Definition;

@EnableWs
@Configuration
public class WebServiceConfig extends WsConfigurerAdapter {

  @Bean
  public ServletRegistrationBean messageDispatcherServlet(ApplicationContext applicationContext) {
    MessageDispatcherServlet servlet = new MessageDispatcherServlet();
    servlet.setApplicationContext(applicationContext);

    return new ServletRegistrationBean(servlet, "/codenotfound/ws/*");
  }

  @Bean(name = "helloworld")
  public Wsdl11Definition defaultWsdl11Definition() {
    SimpleWsdl11Definition wsdl11Definition = new SimpleWsdl11Definition();
    wsdl11Definition.setWsdl(new ClassPathResource("/wsdl/helloworld.wsdl"));

    return wsdl11Definition;
  }
}
```

Now that our `MessageDispatcherServlet` is defined it will try to match incoming XML messages on the defined URI with one of the available handling methods. So all we need to do is setup an `Endpoint` that contains a handling method that matches the incoming request. This service endpoint can be a simple POJO with a number of Spring WS annotations as shown below.

The `HelloWorldEndpoint` POJO is annotated with the `@Endpoint` annotation which registers the class with Spring WS as a potential candidate for processing incoming SOAP messages. It contains a `sayHello()` method that receives a `Person` and returns a `Greeting`. Note that these are the Java classes that we generated earlier using JAXB (both are annotated with `@XmlRoolElement`).

To indicate what sort of messages a method can handle, it is annotated with the `@PayloadRoot` annotation that specifies a qualified name that is defined by a <var>'namespace'</var> and a local name (=<var>'localPart'</var>). Whenever a message comes in which has this qualified name for the payload root element, the method will be invoked.

The `@ResponsePayload` annotation makes Spring WS map the returned value to the response payload which in our example is the JAXB `Greeting` object.

The `@RequestPayload` annotation on the `sayHello()` method parameter indicates that the incoming message will be mapped to the method's request parameter. In our case, this is the JAXB Person object.

The implementation of the `sayHello` service simply logs the name of the received `Person` and then uses this name to construct a `Greeting` that is also logged and then returned.

``` java
package com.codenotfound.ws.endpoint;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.ws.server.endpoint.annotation.Endpoint;
import org.springframework.ws.server.endpoint.annotation.PayloadRoot;
import org.springframework.ws.server.endpoint.annotation.RequestPayload;
import org.springframework.ws.server.endpoint.annotation.ResponsePayload;

import com.codenotfound.types.helloworld.Greeting;
import com.codenotfound.types.helloworld.ObjectFactory;
import com.codenotfound.types.helloworld.Person;

@Endpoint
public class HelloWorldEndpoint {

  private static final Logger LOGGER = LoggerFactory.getLogger(HelloWorldEndpoint.class);

  @PayloadRoot(namespace = "http://codenotfound.com/types/helloworld", localPart = "person")
  @ResponsePayload
  public Greeting sayHello(@RequestPayload Person request) {
    LOGGER.info("Endpoint received person[firstName={},lastName={}]", request.getFirstName(),
        request.getLastName());

    String greeting = "Hello " + request.getFirstName() + " " + request.getLastName() + "!";

    ObjectFactory factory = new ObjectFactory();
    Greeting response = factory.createGreeting();
    response.setGreeting(greeting);

    LOGGER.info("Endpoint sending greeting='{}'", response.getGreeting());
    return response;
  }
}
```

# Creating the Client (Consumer)

The `WebServiceTemplate` is the core class for client-side Web service access in Spring-WS. It contains methods for sending requests and receiving response messages. Additionally, it can marshal objects to XML before sending them across a transport, and unmarshal any response XML into an object again.

As we will use JAXB to marshal our `Person` to a request XML and in turn unmarshal the response XML to our `Greeting` we need an instance of Spring's `Jaxb2Marshaller`. This class requires a context path to operate, which you can set using the <var>'contextPath'</var> property. The context path is a list of colon (:) separated Java package names that contain schema derived classes. In our example this is the package name of the generated Person and `Greeting` classes which is: <var>'com.codenotfound.types.helloworld'</var>.

The below `ClientConfig` configuration class specifies the `WebServiceTemplate` bean that uses the above `Jaxb2Marshaller` for marshaling and unmarshalling. We also set the default service URI using a `defaultUri` property loaded from the <var>application.yml</var> properties file shown below.

``` yaml
client:
  default-uri: http://localhost:9090/codenotfound/ws/helloworld
```

> The below class is annotated with `@Configuration` which indicates that the class can be used by the Spring IoC container as a source of bean definitions.

> Note that the <var>'helloworld'</var> at the end can actually be omitted as we had specified "<kbd>/codenotfound/ws/*</kbd>" as URI of our endpoint servlet.

``` java
package com.codenotfound.ws.client;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.oxm.jaxb.Jaxb2Marshaller;
import org.springframework.ws.client.core.WebServiceTemplate;

@Configuration
public class ClientConfig {

  @Value("${client.default-uri}")
  private String defaultUri;

  @Bean
  Jaxb2Marshaller jaxb2Marshaller() {
    Jaxb2Marshaller jaxb2Marshaller = new Jaxb2Marshaller();
    jaxb2Marshaller.setContextPath("com.codenotfound.types.helloworld");

    return jaxb2Marshaller;
  }

  @Bean
  public WebServiceTemplate webServiceTemplate() {
    WebServiceTemplate webServiceTemplate = new WebServiceTemplate();
    webServiceTemplate.setMarshaller(jaxb2Marshaller());
    webServiceTemplate.setUnmarshaller(jaxb2Marshaller());
    webServiceTemplate.setDefaultUri(defaultUri);

    return webServiceTemplate;
  }
}
```

The client code is specified in the `HelloWorldClient` class. The `sayHello()` method creates a Person object based on the <var>'firstname'</var> and <var>'lastname'</var> input parameters.

The auto-wired `WebServiceTemplate` is used to marshal and send a person XML request towards the Hello World service. The result is unmarshalled to a `Greeting` object which is logged.

The `@Component` annotation will cause Spring to automatically import this bean into the container if automatic component scanning is enabled (adding the `@SpringBootApplication` annotation to the main `SpringWsApplication` class [is equivalent](http://docs.spring.io/autorepo/docs/spring-boot/current/reference/html/using-boot-using-springbootapplication-annotation.html){:target="_blank"} to using `@ComponentScan`).

``` java
package com.codenotfound.ws.client;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import org.springframework.ws.client.core.WebServiceTemplate;

import com.codenotfound.types.helloworld.Greeting;
import com.codenotfound.types.helloworld.ObjectFactory;
import com.codenotfound.types.helloworld.Person;

@Component
public class HelloWorldClient {

  private static final Logger LOGGER = LoggerFactory.getLogger(HelloWorldClient.class);

  @Autowired
  private WebServiceTemplate webServiceTemplate;

  public String sayHello(String firstName, String lastName) {
    ObjectFactory factory = new ObjectFactory();
    Person person = factory.createPerson();

    person.setFirstName(firstName);
    person.setLastName(lastName);

    LOGGER.info("Client sending person[firstName={},lastName={}]", person.getFirstName(),
        person.getLastName());

    Greeting greeting = (Greeting) webServiceTemplate.marshalSendAndReceive(person);

    LOGGER.info("Client received greeting='{}'", greeting.getGreeting());
    return greeting.getGreeting();
  }
}
```

# Testing the Client & Endpoint

We will create a basic unit test case in which the above client is used to send a request to the Hello World endpoint. We then verify if the response is equal to the expected Hello World greeting.

The `@RunWith` and `@SpringBootTest` testing annotations, [that were introduced with Spring Boot 1.4](https://spring.io/blog/2016/04/15/testing-improvements-in-spring-boot-1-4#spring-boot-1-4-simplifications){:target="_blank"}, are used to tell JUnit to run using Spring's testing support and bootstrap with Spring Boot's support.

By setting the `DEFINED_PORT` web environment variable, a real HTTP server is started on the the <var>'server.port'</var> property defined in the <var>application.properties</var> file.

``` java
package com.codenotfound.ws;

import static org.assertj.core.api.Assertions.assertThat;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.SpringBootTest.WebEnvironment;
import org.springframework.test.context.junit4.SpringRunner;

import com.codenotfound.ws.client.HelloWorldClient;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = WebEnvironment.DEFINED_PORT)
public class SpringWsApplicationTests {

  @Autowired
  private HelloWorldClient helloWorldClient;

  @Test
  public void testSayHello() {
    assertThat(helloWorldClient.sayHello("John", "Doe")).isEqualTo("Hello John Doe!");
  }
}
```

The above test case can be triggered by opening a command prompt in the projects root folder and executing following Maven command:

``` plaintext
mvn test
```

The result should be a successful build during which the embedded Tomcat is started and a service call is made to the Hello World service:

``` plaintext
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.5.9.RELEASE)

21:37:53.519 [main] INFO  c.c.ws.SpringWsApplicationTests - Starting SpringWsApplicationTests on cnf-pc with PID 3276 (started by CodeNotFound in c:\code\spring-ws\spring-ws-helloworld)
21:37:53.522 [main] INFO  c.c.ws.SpringWsApplicationTests - No active profile set, falling back to default profiles: default
21:37:55.819 [main] INFO  c.c.ws.SpringWsApplicationTests - Started SpringWsApplicationTests in 2.587 seconds (JVM running for 3.29)
21:37:55.849 [main] INFO  c.c.ws.client.HelloWorldClient - Client sending person[firstName=John,lastName=Doe]
21:37:56.077 [http-nio-9090-exec-1] INFO  c.c.ws.endpoint.HelloWorldEndpoint - Endpoint received person[firstName=John,lastName=Doe]
21:37:56.077 [http-nio-9090-exec-1] INFO  c.c.ws.endpoint.HelloWorldEndpoint - Endpoint sending greeting='Hello John Doe!'
21:37:56.092 [main] INFO  c.c.ws.client.HelloWorldClient - Client received greeting='Hello John Doe!'
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 2.954 sec - in com.codenotfound.ws.SpringWsApplicationTests

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 6.603 s
[INFO] Finished at: 2017-12-08T21:37:56+01:00
[INFO] Final Memory: 29M/282M
[INFO] ------------------------------------------------------------------------
```

If you just want to start Spring Boot so that the endpoint is up and running, execute following Maven command:

``` plaintext
mvn spring-boot:run
```

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-ws/tree/master/spring-ws-helloworld).
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This Spring WS example turned out a bit longer than expected but hopefully, it helped to explain the core client and endpoint concepts.

If you found this sample useful or have a question you would like to ask, drop a line below!
