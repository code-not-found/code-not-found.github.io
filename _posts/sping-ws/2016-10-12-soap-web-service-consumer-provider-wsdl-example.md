---
title: Spring WS - SOAP Web Service Consumer & Provider WSDL Example
permalink: /2016/10/spring-ws-soap-web-service-consumer-provider-wsdl-example.html
excerpt: A detailed step-by-step tutorial on how to implement a Hello World web service starting from a WSDL and using Spring-WS and Spring Boot.
date: 2016-10-12 21:00
categories: [Spring-WS]
tags: [Client, Consumer, Endpoint, Example, Hello World, Maven, Provider, Spring, Spring Boot, Spring Web Services, Spring-WS, Tutorial, WSDL]
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/spring-logo.png" alt="spring logo">
</figure>

[Spring Web Services](http://projects.spring.io/spring-ws/) (Spring-WS) is a product of the Spring community focused on creating document-driven Web services. Spring-WS facilitates contract-first SOAP service development, allowing for a number of ways to manipulate XML payloads. The following step by step tutorial illustrates a basic example in which we will configure, build and run a Hello World contract first client and endpoint using a WSDL, Spring-WS, Spring Boot and Maven.

Tools used:
* Spring-WS 2.3
* Spring Boot 1.4
* Maven 3

The tutorial code is organized in such a way that you can choose to only run the [client]({{ site.url }}//2016/10/spring-ws-soap-web-service-consumer-provider-wsdl-example.html#creating-the-endpoint-provider) (consumer) or [endpoint]({{ site.url }}//2016/10/spring-ws-soap-web-service-consumer-provider-wsdl-example.html#creating-the-endpoint-provider) (provider) part. In the below example we will setup both parts and then make an end-to-end test in which the client calls the endpoint.

# General Project Setup

As Spring Web Services is contract first only, we need to start from a contract definition. In this tutorial we will use a Hello World service that is defined by below WSDL. The service takes as input a person's first and last name and returns a greeting.

``` xml
<?xml version="1.0"?>
<wsdl:definitions name="HelloWorld"

    targetNamespace="http://codenotfound.com/services/helloworld"
    xmlns:tns="http://codenotfound.com/services/helloworld" xmlns:types="http://codenotfound.com/types/helloworld"
    xmlns:soap="http://schemas.xmlsoap.org/wsdl/soap/" xmlns:wsdl="http://schemas.xmlsoap.org/wsdl/">

    <wsdl:types>
        <xsd:schema targetNamespace="http://codenotfound.com/types/helloworld"
            xmlns:xsd="http://www.w3.org/2001/XMLSchema"
            elementFormDefault="qualified" attributeFormDefault="unqualified"
            version="1.0">

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
            <soap:address
                location="http://localhost:9090/codenotfound/ws/helloworld" />
        </wsdl:port>
    </wsdl:service>

</wsdl:definitions>
```

We will be building and running our example using [Maven](https://maven.apache.org/). Shown below is the XML representation of our Maven project in a POM file. It contains the needed dependencies for compiling and running our example.

In order to expose the Hello World service endpoint we will use the [Spring Boot](https://projects.spring.io/spring-boot/) project that comes with an embedded Apache Tomcat server. To facilitate the management of the different Spring dependencies, [Spring Boot Starters](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters) are used which are a set of convenient dependency descriptors that you can include in your application.

The `spring-boot-starter-web-services` dependency includes the needed dependencies for using Spring Web Services. The `spring-boot-starter-test` includes the dependencies for testing Spring Boot applications with libraries that include <ins>JUnit</ins>, <ins>Hamcrest</ins> and <ins>Mockito</ins>.

To avoid having to manage the version compatibility of the different Spring dependencies, we will inherit the defaults from the `spring-boot-starter-parent` parent POM.

In the plugins section we included the `spring-boot-maven-plugin` Maven plugin so that we can build a single, runnable "über-jar". This will also allow us to start the web service via a Maven command.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.codenotfound</groupId>
    <artifactId>spring-ws-helloworld-example</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>spring-ws-helloworld-example</name>
    <description>Spring WS - Consume & Provide a Web Service Using WSDL</description>
    <url>http://www.codenotfound.com/2016/10/consume-provide-web-service-wsdl.html</url>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.4.1.RELEASE</version>
        <relativePath /> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>

        <maven-jaxb2-plugin.version>0.13.1</maven-jaxb2-plugin.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-ws</artifactId>
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

In order to directly use the <var>person</var> and <var>greeting</var> elements (defined in the <var>types</var> section of the Hello World WSDL) in our Java code, we will use JAXB to generate the corresponding Java classes. The above POM file configures the `maven-jaxb2-plugin` that will handle the generation.

The plugin will look into the defined <ins>&lt;schemaDirectory&gt;</ins> in order to find any WSDL files for which it needs to generate the the Java classes. In order to trigger the generation via Maven, executed following command:

``` plaintext
mvn generate-sources
```

This results in a number of generated classes amongst which the `Person` and `Greeting` that we will use when implementing the client and provider of the Hello World service.

<figure>
    <img src="{{ site.url }}/assets/images/spring-ws/hello-world-jaxb-generated-java-classes.png" alt="helloworld jaxb generated java classes">
</figure>

We start by creating an `SpringWsApplication` that contains a `main()` method that uses Spring Boot’s `SpringApplication.run()` method to bootstrap the application, starting Spring. For more information on Spring Boot we refer to the [Spring Boot getting started guide](https://spring.io/guides/gs/spring-boot/).
``` java
package com.codenotfound;

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

The server-side of Spring-WS is designed around a central class called `MessageDispatcher` that dispatches incoming XML messages to endpoints. For more detailed information checkout the [Spring Web Services reference documentation on the MessageDispatcher](http://docs.spring.io/spring-ws/docs/current/reference/htmlsingle/#message-dispatcher).

Spring Web Services supports multiple transport protocols. The most common is the HTTP transport, for which a custom `MessageDispatcherServlet` servlet is supplied. This is a standard `Servlet` which extends from the standard Spring Web `DispatcherServlet` ([=central dispatcher for HTTP request handlers/controllers](http://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/DispatcherServlet.html)), and wraps a `MessageDispatcher`. 

> In other words: the `MessageDispatcherServlet` combines the attributes of the `MessageDispatcher` and `DispatcherServlet` and as a result allows the handling of XML messages over HTTP.

In the below `WebServiceConfig` configuration class we use a `ServletRegistrationBean` to register the `MessageDispatcherServlet`. Note that it is important to inject and set the ApplicationContext to the `MessageDispatcherServlet`, otherwise it will not automatically detect other Spring Web Services related beans (such as the lower `Wsdl11Definition`). By naming this bean <var>messageDispatcherServlet</var>, it does [not replace Spring Boot’s default `DispatcherServlet` bean](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/htmlsingle/#howto-switch-off-the-spring-mvc-dispatcherservlet).

The servlet mapping URI pattern on the `ServletRegistrationBean` is set to <kbd>"/codenotfound/ws/*"<kbd>. The web container will use this path to map incoming HTTP requests to the servlet.

The `DefaultWsdl11Definition` exposes a standard WSDL 1.1 using the specified Hello World WSDL file. The URL location at which this WSDL is available is determined by it's `Bean` name in combination with the URI mapping of the `MessageDispatcherServlet`. For the example below this is: <ins>[host]</ins>=<kbd>http://localhost:9090</kbd>+<ins>[servlet mapping uri]</ins>=<kbd>/codenotfound/ws/</kbd>+<ins>[WsdlDefinition bean name]</ins>=<kbd>helloworld</kbd>+<ins>[WSDL postfix]</ins>=<kbd>.wsdl</kbd> or [http://localhost:9090/codenotfound/ws/helloworld.wsdl](http://localhost:9090/codenotfound/ws/helloworld.wsdl).

To enable the support for `@Endpoint` annotation that we will use in the next section we need to annotate our configuration class with `@EnableWs`.

``` java
package com.codenotfound.endpoint;

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
    public ServletRegistrationBean messageDispatcherServlet(
            ApplicationContext applicationContext) {

        MessageDispatcherServlet servlet = new MessageDispatcherServlet();
        servlet.setApplicationContext(applicationContext);

        return new ServletRegistrationBean(servlet,
                "/codenotfound/ws/*");
    }

    @Bean(name = "helloworld")
    public Wsdl11Definition defaultWsdl11Definition() {
        SimpleWsdl11Definition wsdl11Definition = new SimpleWsdl11Definition();
        wsdl11Definition.setWsdl(
                new ClassPathResource("/wsdl/helloworld.wsdl"));

        return wsdl11Definition;
    }
}
```

Now that our `MessageDispatcherServlet` is defined it will try to match incoming XML messages on the defined URI with one of the available handling methods. So all we need to do is setup an `Endpoint` that contains a handling method that matches the incoming request. This service endpoint can be a simple POJO with a number of Spring WS annotations as shown below.

The `HelloWorldEndpoint` POJO is annotated with the `@Endpoint` annotation which registers the class with Spring WS as a potential candidate for processing incoming SOAP messages. It contains a `sayHello()` method that receives a `Person` and returns a `Greeting`. Note that these are the Java classes that we generated earlier using JAXB.

To indicate what sort of messages a method can handle, it is annotated with the `@PayloadRoot` annotation that specifies a qualified name that is defined by a <ins>namespace</ins> and a local name (=<ins>localPart</ins>). Whenever a message comes in which has this qualified name for the payload root element, the method will be invoked.

The `@ResponsePayload` annotation makes Spring WS map the returned value to the response payload which in our example is the JAXB `Greeting` object.

The @RequestPayload annotation on the sayHello method parameter indicates that the incoming message will be mapped to the method’s request parameter. In our case this is the JAXB Person object.

The implementation of the sayHello service simply logs the name of the received Person and then uses this name to construct a Greeting that is also logged and then returned. 








