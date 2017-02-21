---
title: CXF - SOAP Web Service Consumer & Provider WSDL Example
permalink: /2016/10/cxf-soap-web-service-consumer-provider-wsdl-example.html
excerpt: A detailed step-by-step tutorial on how to implement a Hello World web service starting from a WSDL and using Apache CXF and Spring Boot.
date: 2016-10-16 21:00
categories: [Apache CXF]
tags: [Apache CXF, Client, Consumer, Contract First, CXF, Endpoint, Example, Hello World, Maven, Provider, Spring Boot, Tutorial, WSDL]
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/apache-cxf-logo.png" alt="apache cxf logo">
</figure>

[Apache CXF](http://cxf.apache.org/) is an open source services framework that helps build and develop services using frontend programming APIs, like JAX-WS. In this tutorial we will take a look at how we can integrate CXF with Spring Boot in order to build and run a Hello World SOAP service. Throughout the example, we will be creating a contract first client and endpoint using a WSDL, CXF, Spring Boot and Maven.

Tools used:
* Apache CXF 3.1
* Spring Boot 1.4
* Maven 3

This tutorial is actually a newer version of an previous [example in which we used Jetty to host a Hello World CXF web service](http://www.source4code.info/2014/08/jaxws-cxf-contract-first-hello-world-webservice-tutorial.html). This time we will be using the embedded Tomcat server that ships with Spring Boot as a runtime for the service.

The below code is organized in such a way that you can choose to only run the client (consumer) or endpoint (provider) part. In the below example we will setup both parts and then make an end-to-end test in which the client calls the endpoint.

# General Project Setup

In this example we will start from an existing WSDL file (contract-first) which is shown below. The content represent a SOAP service in which a person is sent as input and a greeting is received as a response.

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

Maven is used to build and run the example. The Hello World service endpoint will be hosted on an embedded Apache Tomcat server that ships directly with [Spring Boot](https://projects.spring.io/spring-boot/). To facilitate the management of the different Spring dependencies, [Spring Boot Starters](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters) are used which are a set of convenient dependency descriptors that you can include in your application.

To avoid having to manage the version compatibility of the different Spring dependencies, we will inherit the defaults from the `spring-boot-starter-parent` parent POM.

There is actually a [Spring Boot starter specifically for CXF](http://cxf.apache.org/docs/springboot.html) that takes care of importing the needed Spring Boot dependencies. In addition it automatically registers `CXFServlet` with a <var>/services/</var> URL pattern for serving CXF JAX-WS endpoints and it offers some properties for configuration of the `CXFServlet`. In order to use the starter we declare a dependency to `cxf-spring-boot-starter-jaxws` in our Maven POM file.

For Unit testing our Spring Boot application we also include the `spring-boot-starter-test` dependency.

To take advantage of Spring Boot's capability to create a single, runnable "über-jar", we also include the `spring-boot-maven-plugin` Maven plugin. This also allows quickly starting the web service via a Maven command.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.codenotfound</groupId>
    <artifactId>jaxws-cxf-helloworld-example</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>jaxws-cxf-helloworld-example</name>
    <description>CXF - SOAP Web Service Consumer & Provider WSDL Example</description>
    <url>http://</url>

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

        <cxf.version>3.1.7</cxf.version>
    </properties>

    <dependencies>
        <!-- Apache CXF -->
        <dependency>
            <groupId>org.apache.cxf</groupId>
            <artifactId>cxf-spring-boot-starter-jaxws</artifactId>
            <version>${cxf.version}</version>
        </dependency>
        <!-- Spring Boot -->
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
                <groupId>org.apache.cxf</groupId>
                <artifactId>cxf-codegen-plugin</artifactId>
                <version>${cxf.version}</version>
                <executions>
                    <execution>
                        <id>generate-sources</id>
                        <phase>generate-sources</phase>
                        <configuration>
                            <sourceRoot>${project.build.directory}/generated/cxf</sourceRoot>
                            <wsdlOptions>
                                <wsdlOption>
                                    <wsdl>${project.basedir}/src/main/resources/wsdl/helloworld.wsdl</wsdl>
                                    <wsdlLocation>classpath:wsdl/helloworld.wsdl</wsdlLocation>
                                </wsdlOption>
                            </wsdlOptions>
                        </configuration>
                        <goals>
                            <goal>wsdl2java</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

</project>
```

CXF includes a Maven `cxf-codegen-plugin plugin` which can [generate java artifacts from a WSDL file](http://cxf.apache.org/docs/maven-cxf-codegen-plugin-wsdl-to-java.html). In the above plugin configuration we're running the <var>wsdl2java</var> goal in the <var>generate-sources</var> phase. When executing following Maven command, CXF will generate artifacts in the <ins>&lt;sourceRoot&gt;</ins> directory that we have specified. 

``` powershell
mvn generate-sources
```

After running above command you should be able to find back a number of auto generated classes among which the `HelloWorldPortType` interface in addition to `Person` and `Greeting` as shown below. 

<figure>
    <img src="{{ site.url }}/assets/images/cxf/hello-world-jaxb-generated-java-classes.png" alt="helloworld jaxb generated java classes">
</figure>

Next we create a `SpringCxfApplication` class. It contain a `main()` method that delegates to Spring Boot’s `SpringApplication` class by calling run. `SpringApplication` will bootstrap our application, starting Spring which will in turn start the auto-configured Tomcat web server.

 The `@SpringBootApplication` is a convenience annotation that adds all of the following: `@Configuration`, `@EnableAutoConfiguration` and `@ComponentScan`. Checkout the [Spring Boot getting started guide](https://spring.io/guides/gs/spring-boot/) for more details.

``` java
package com.codenotfound;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringCxfApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringCxfApplication.class, args);
    }
}
```

# Creating the Endpoint (Provider)

In order for the CXF framework to be able to process incoming SOAP request over HTTP, we need to setup a `CXFServlet`. In our previous [CXF SOAP Web Service tutorial](http://www.source4code.info/2014/08/jaxws-cxf-contract-first-hello-world-webservice-tutorial.html) we did this by using a deployment descriptor file ('web.xml' file under the 'WEB-INF' directory) or an alternative with Spring is to use a `ServletRegistrationBean`. In this example there is nothing to be done as the `cxf-spring-boot-starter-jaxws` automatically register the `CXFServlet` for us, great!

 In this example we want the `CXFServlet` to listen for incoming requests on the following URI: "<kbd>/codenotfound/ws</kbd>", instead of the default value which is: "<kbd>/services/*</kbd>". This can be achieved by setting the <ins>cxf.path</ins> property in the <ins>application.properties</ins> file located under the <ins>src/main/resources</ins> folder.

``` properties
# server HTTP port
server.port=9090

# set the CXFServlet URL pattern
cxf.path=/codenotfound/ws

# hello world service address
helloworld.service.address=http://localhost:9090/codenotfound/ws/helloworld
```

Next we create a configuration file that contains the definition of our `Endpoint` at which our Hello World SOAP service will be exposed. The `Endpoint` gets created by passing the [CXF bus, which is the backbone of the CXF architecture](http://cxf.apache.org/docs/cxf-architecture.html#CXFArchitecture-Bus) that manages the respective inbound and outbound message and fault interceptor chains for all client and server endpoints. We use the default CXF bus and get a reference to it via Spring's `@Autowired` annotation.

In addition to the bus we also specify the `HelloWorldImpl` class which contains the actual implementation of the service. Finally we set the URI on which the endpoint will be exposed to <ins>/helloworld</ins>. Together with the <ins>cxf.path</ins> configuration in the <ins>application.properties</ins> file this result into following URL that clients will need to call: [http://localhost:9090/codenotfound/ws/helloworld](http://localhost:9090/codenotfound/ws/helloworld).















