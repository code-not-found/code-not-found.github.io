---
title: JAX-WS - CXF Contract First Hello World Webservice Tutorial
permalink: /2014/08/jaxws-cxf-contract-first-hello-world-webservice-tutorial.html
excerpt: A detailed step-by-step tutorial on how to implement a Hello World web service starting from a WSDL and using Apache CXF and Spring Boot.
date: 2014-08-23 21:00
categories: [Apache CXF]
tags: [Apache CXF, Contract First, CXF, Example, Hello World, JAX-WS, Jetty, Maven, SOAP, Spring, Tutorial, Web service, Webservice]
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/apache-cxf-logo.png" alt="apache cxf logo">
</figure>

[Apache CXF](https://cxf.apache.org/) is an open source services framework. CXF helps to build and develop services using front end programming APIs like JAX-WS and JAX-RS. These services can speak a variety of protocols such as SOAP, XML/HTTP or RESTful HTTP and work over a variety of transports such as HTTP or JMS. The following tutorial illustrates a basic example in which we will configure, build and run a Hello World contract first client and web service using CXF, Spring , Maven and Jetty.

> An updated version of this blog post has been created in which the [Hello World CXF SOAP service is created using Spring JavaConfig and Spring Boot]({{ site.url }}/2016/10/cxf-soap-web-service-consumer-provider-wsdl-example.html).

Tools used:
* CXF 3.1
* Spring 4.3
* Maven 3
* Jetty 9

A web service can be developed using one of two approaches:
1. **Start with a WSDL contract** and generate Java objects to implement the service.
2. **Start with a Java object** and service enable it using annotations.

For new development the preferred approach is to first design your services using the Web Services Description Language (WSDL) and then generate the code to implement them. This approach enforces the concept that a service is an abstract entity that is implementation neutral. For the following example we will use a Hello World service that is defined by the WSDL shown below. It takes as input a persons first and last name and as result returns a greeting.

``` xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<wsdl:definitions targetNamespace="http://codenotfound.com/services/helloworld"
    xmlns:soap="http://schemas.xmlsoap.org/wsdl/soap/" xmlns:tns="http://codenotfound.com/services/helloworld"
    xmlns:wsdl="http://schemas.xmlsoap.org/wsdl/" xmlns:xsd="http://www.w3.org/2001/XMLSchema"
    name="HelloWorld">

    <wsdl:types>
        <schema targetNamespace="http://codenotfound.com/services/helloworld"
            xmlns="http://www.w3.org/2001/XMLSchema" xmlns:tns="http://codenotfound.com/services/helloworld"
            elementFormDefault="qualified" attributeFormDefault="unqualified"
            version="1.0">

            <element name="person">
                <complexType>
                    <sequence>
                        <element name="firstName" type="xsd:string" />
                        <element name="lastName" type="xsd:string" />
                    </sequence>
                </complexType>
            </element>

            <element name="greeting">
                <complexType>
                    <sequence>
                        <element name="text" type="xsd:string" />
                    </sequence>
                </complexType>
            </element>
        </schema>
    </wsdl:types>

    <wsdl:message name="sayHelloRequest">
        <wsdl:part name="person" element="tns:person"></wsdl:part>
    </wsdl:message>

    <wsdl:message name="sayHelloResponse">
        <wsdl:part name="greeting" element="tns:greeting"></wsdl:part>
    </wsdl:message>

    <wsdl:portType name="HelloWorld_PortType">
        <wsdl:operation name="sayHello">
            <wsdl:input message="tns:sayHelloRequest"></wsdl:input>
            <wsdl:output message="tns:sayHelloResponse"></wsdl:output>
        </wsdl:operation>
    </wsdl:portType>

    <wsdl:binding name="HelloWorld_Binding" type="tns:HelloWorld_PortType">
        <soap:binding style="document"
            transport="http://schemas.xmlsoap.org/soap/http" />
        <wsdl:operation name="sayHello">
            <wsdl:input>
                <soap:body use="literal" />
            </wsdl:input>
            <wsdl:output>
                <soap:body use="literal" />
            </wsdl:output>
        </wsdl:operation>
    </wsdl:binding>

    <wsdl:service name="HelloWorld_Service">
        <wsdl:port name="HelloWorld_Port" binding="tns:HelloWorld_Binding">
            <soap:address
                location="http://localhost:9090/codenotfound/services/helloworld" />
        </wsdl:port>
    </wsdl:service>
</wsdl:definitions>
```

Next is the Maven POM file which contains the needed dependencies. At the bottom of the list we find the [CXF dependencies](https://cxf.apache.org/docs/using-cxf-with-maven.html). The <var>'cxf-rt-transports-http-jetty'</var> dependency is only needed in case the `CFXServlet` is not used. As the example includes a JUnit test that runs without `CXFServlet` we need to add this dependency.

CXF supports the Spring 2.0 XML syntax, which makes it easy to declare endpoints which are backed by Spring and inject clients into application code. It is also possible to use CXF without Spring but it may take a little extra effort. A number of things like policies and annotation processing are not wired in without Spring. To take advantage of those you would need to do some pre-setup coding. The example below will use Spring to create both requester (client) and provider (service) as such the needed Spring dependencies are included.

In addition to a unit test case we will also create an integration test case for which an instance of Jetty will be started that will host the above Hello World service. In order to achieve this the <var>'jetty-maven-plugin'</var> has been added which is configured to be started/stopped, before/after the <var>'integration-test'</var> phase of Maven. The <var>'&lt;daemon&gt;true&lt;/daemon&gt;'<var>/ configuration option forces Jetty to execute only while Maven is running, instead of running indefinitely.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.codenotfound</groupId>
    <artifactId>jaxws-cxf-helloworld</artifactId>
    <version>1.0</version>
    <packaging>war</packaging>

    <name>JAX-WS - CXF Contract First Hello World Webservice Tutorial</name>
    <url>https://codenotfound.com/2014/08/jaxws-cxf-contract-first-hello-world-webservice-tutorial.html</url>

    <prerequisites>
        <maven>3.0</maven>
    </prerequisites>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>

        <slf4j.version>1.7.21</slf4j.version>
        <logback.version>1.1.7</logback.version>
        <junit.version>4.12</junit.version>
        <cxf.version>3.1.7</cxf.version>
        <spring.version>4.3.2.RELEASE</spring.version>

        <jetty-maven-plugin.version>9.4.0.M0</jetty-maven-plugin.version>
        <maven-compiler-plugin.version>3.5.1</maven-compiler-plugin.version>
        <maven-failsafe-plugin.version>2.19.1</maven-failsafe-plugin.version>
    </properties>

    <dependencies>
        <!-- Logging -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>jcl-over-slf4j</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>${logback.version}</version>
        </dependency>
        <!-- JUnit -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>
        <!-- CXF -->
        <dependency>
            <groupId>org.apache.cxf</groupId>
            <artifactId>cxf-rt-frontend-jaxws</artifactId>
            <version>${cxf.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.cxf</groupId>
            <artifactId>cxf-rt-transports-http</artifactId>
            <version>${cxf.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.cxf</groupId>
            <!-- Jetty is needed if you're are not using the CXFServlet -->
            <artifactId>cxf-rt-transports-http-jetty</artifactId>
            <version>${cxf.version}</version>
        </dependency>
        <!-- Spring -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>${spring.version}</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>${maven-compiler-plugin.version}</version>
                <configuration>
                    <source>${java.version}</source>
                    <target>${java.version}</target>
                </configuration>
            </plugin>
            <!-- jetty-maven-plugin -->
            <plugin>
                <groupId>org.eclipse.jetty</groupId>
                <artifactId>jetty-maven-plugin</artifactId>
                <version>${jetty-maven-plugin.version}</version>
                <configuration>
                    <httpConnector>
                        <port>9090</port>
                    </httpConnector>
                    <stopPort>8005</stopPort>
                    <stopKey>STOP</stopKey>
                    <daemon>true</daemon>
                </configuration>
                <executions>
                    <execution>
                        <id>start-jetty</id>
                        <phase>pre-integration-test</phase>
                        <goals>
                            <goal>start</goal>
                        </goals>
                    </execution>
                    <execution>
                        <id>stop-jetty</id>
                        <phase>post-integration-test</phase>
                        <goals>
                            <goal>stop</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <!-- maven-failsafe-plugin -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-failsafe-plugin</artifactId>
                <version>${maven-failsafe-plugin.version}</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>integration-test</goal>
                            <goal>verify</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <!-- cxf-codegen-plugin -->
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

CXF includes a Maven plugin called <var>'cxf-codegen-plugin'</var> which can generate Java artifacts from a WSDL. In the above POM the <var>'wsdl2java'<var> goal is configured to run in the generate-sources phase. By running the below Maven command, CXF will generate the Java artifacts. Each <var>'&lt;wsdlOption&gt;'</var> element corresponds to a WSDL that needs generated artifacts.

``` plaintext
mvn generate-sources
```

> In order to avoid a hard-coded absolute path towards the configured WSDL in the generated Java artifacts, specify a <var>'&lt;wsdlLocation&gt;'</var> element using the classpath reference as shown above.

After running the <var>'mvn generate-sources'</var> command you should be able to find back a number of auto generated classes amongst which the `HelloWorldPort` interface as shown below. In the next sections we will use this interface to implement both the client and service.

<figure>
    <img src="{{ site.url }}/assets/images/logos/cxf-plugin-generated-classes.png" alt="cxf plugin generated classes">
</figure>

# Creating The Requester (Client)

Creating a CXF client using Spring is done by specifying a `jaxws:client` bean as shown in the below <var>cxf-requester.xml</var>. In addition to a bean name, the service interface and the service address (or URL) need to be specified. The result of below configuration is a bean that will be created with the specified name, implementing the service interface and invoking the remote SOAP service under the covers.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:jaxws="http://cxf.apache.org/jaxws" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
http://cxf.apache.org/jaxws http://cxf.apache.org/schemas/jaxws.xsd">

    <!-- allows for ${} replacement in a spring xml configuration file 
        from a '.properties' file on the classpath -->
    <context:property-placeholder
        location="classpath:cxf.properties" />

    <jaxws:client id="helloWorldRequesterBean"
        serviceClass="com.codenotfound.services.helloworld.HelloWorldPortType"
        address="${helloworld.address}">
    </jaxws:client>

</beans>
```

For this example the CXF Spring configuration is kept in a separate file that is imported in the main <var>context-requester.xml</var> shown below. In addition to this, annotation based configuration is enabled and a <var>cxf.properties</var> file is loaded which contains the address at which the Hello World service was made available in the previous section.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- import the CXF requester configuration -->
    <import resource="classpath:META-INF/spring/cxf-requester.xml" />

    <!-- HelloWorldClient bean -->
    <bean id="helloWorldClientBean"
        class="com.codenotfound.soap.http.cxf.HelloWorldClient" />

</beans>
```

The actual client code is specified in the `HelloWorldClient` class which exposes a `sayHello()` method that takes as input a `Person` object. The `helloworldRequesterBean` previously defined in the <var>cxf-requester.xml</var> is auto wired using the corresponding annotation. 

``` java
package com.codenotfound.soap.http.cxf;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;

import com.codenotfound.services.helloworld.Greeting;
import com.codenotfound.services.helloworld.HelloWorldPortType;
import com.codenotfound.services.helloworld.Person;

public class HelloWorldClient {

    private static final Logger LOGGER = LoggerFactory
            .getLogger(HelloWorldClient.class);

    @Autowired
    private HelloWorldPortType helloWorldRequesterBean;

    public String sayHello(Person person) {
        Greeting greeting = helloWorldRequesterBean.sayHello(person);

        String result = greeting.getText();
        LOGGER.info("result={}", result);
        return result;
    }
}
```


















