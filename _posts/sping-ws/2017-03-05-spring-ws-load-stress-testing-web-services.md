---
title: Spring WS - Load Testing Web Services
permalink: /2017/03/spring-ws-load-stress-testing-web-services.html
excerpt: A detailed step-by-step tutorial on how to implement a mock object for Spring's WebServiceTemplate or Web Service Endpoint.
date: 2017-02-28 21:00
categories: [Spring-WS]
tags: [Client, Consumer, Endpoint, Example, Integration Test, JUnit, Maven, Mock, Provider, Spring, Spring Boot, Spring Web Services, Spring-WS, Tutorial, WSDL, Unit Test, Testing]
published: false
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/spring-logo.png" alt="spring logo">
</figure>

[Spring Web Services](http://projects.spring.io/spring-ws/) (Spring-WS) framework 


The following step by step tutorial illustrates a basic example in which we will configure, build and run a Hello World contract first client and endpoint using a WSDL, Spring-WS, Spring Boot and Maven.

Tools used:
* Spring-WS 2.4
* Spring Boot 1.5
* Maven 3


# General Project Setup

The below example is based on a previous [step by step tutorial in which we built a Spring Web Services client and server starting from a Hello World WSDL file]({{ site.url }}/2016/10/spring-ws-soap-web-service-consumer-provider-wsdl-example.html). Instead of reusing <var>helloworld.wsdl</var> we will use an <var>order.wsdl</var> which mimics a more  real life example WSDL.

As input we pass an order that consists out of customer information in addition to a list of order line items. The output is a order confirmation identifier that allows tracking the order or an order error in case something went wrong.

``` xml
<?xml version="1.0"?>
<wsdl:definitions name="Order"
    targetNamespace="http://codenotfound.com/services/order" xmlns:tns="http://codenotfound.com/services/order"
    xmlns:types="http://codenotfound.com/types/order" xmlns:soap="http://schemas.xmlsoap.org/wsdl/soap/"
    xmlns:wsdl="http://schemas.xmlsoap.org/wsdl/" xmlns:p="http://www.w3.org/2001/XMLSchema">

    <wsdl:types>
        <xsd:schema targetNamespace="http://codenotfound.com/types/order"
            xmlns:tns="http://codenotfound.com/types/order" xmlns:xsd="http://www.w3.org/2001/XMLSchema"
            elementFormDefault="qualified" attributeFormDefault="unqualified"
            version="1.0">

            <xsd:element name="order">
                <xsd:complexType>
                    <xsd:sequence>
                        <xsd:element name="customer" type="tns:customerType" />
                        <xsd:element name="lineItems"
                            type="tns:lineItemsType" />
                    </xsd:sequence>
                </xsd:complexType>
            </xsd:element>

            <xsd:element name="orderConfirmation">
                <xsd:complexType>
                    <xsd:sequence>
                        <xsd:element name="confirmationId"
                            type="xsd:string" />
                    </xsd:sequence>
                </xsd:complexType>
            </xsd:element>

            <xsd:element name="orderError">
                <xsd:complexType>
                    <xsd:sequence>
                        <xsd:element name="code" type="xsd:string" />
                        <xsd:element name="description"
                            type="xsd:string" />
                    </xsd:sequence>
                </xsd:complexType>
            </xsd:element>

            <xsd:complexType name="customerType">
                <xsd:sequence>
                    <xsd:element name="firstName" type="xsd:string" />
                    <xsd:element name="lastName" type="xsd:string" />
                    <xsd:element name="street" type="xsd:string" />
                    <xsd:element name="city" type="xsd:string" />
                    <xsd:element name="state" type="xsd:string" />
                    <xsd:element name="zip" type="xsd:string" />
                    <xsd:element name="country" type="xsd:string" />
                </xsd:sequence>
            </xsd:complexType>

            <xsd:complexType name="lineItemsType">
                <xsd:sequence>
                    <xsd:element name="lineItem" type="tns:lineItemType"
                        minOccurs="1" maxOccurs="unbounded" />
                </xsd:sequence>
            </xsd:complexType>

            <xsd:complexType name="lineItemType">
                <xsd:sequence>
                    <xsd:element name="product" type="tns:productType" />
                    <xsd:element name="quantity" type="xsd:integer" />
                </xsd:sequence>
            </xsd:complexType>

            <xsd:complexType name="productType">
                <xsd:sequence>
                    <xsd:element name="id" type="xsd:string" />
                    <xsd:element name="name" type="xsd:string" />
                    <xsd:element name="description" type="xsd:string" />
                </xsd:sequence>
            </xsd:complexType>
        </xsd:schema>

    </wsdl:types>

    <wsdl:message name="OrderRequest">
        <wsdl:part name="order" element="types:order" />
    </wsdl:message>

    <wsdl:message name="OrderResponse">
        <wsdl:part name="orderConfirmation" element="types:orderConfirmation" />
    </wsdl:message>

    <wsdl:message name="OrderFault">
        <wsdl:part name="orderFault" element="types:orderError"></wsdl:part>
    </wsdl:message>

    <wsdl:portType name="Order_PortType">
        <wsdl:operation name="createOrder">
            <wsdl:input message="tns:OrderRequest" />
            <wsdl:output message="tns:OrderResponse" />
            <wsdl:fault name="fault" message="tns:OrderFault"></wsdl:fault>
        </wsdl:operation>
    </wsdl:portType>

    <wsdl:binding name="Order_SoapBinding" type="tns:Order_PortType">
        <soap:binding style="document"
            transport="http://schemas.xmlsoap.org/soap/http" />

        <wsdl:operation name="createOrder">
            <soap:operation
                soapAction="http://codenotfound.com/services/order/createOrder" />
            <wsdl:input>
                <soap:body use="literal" />
            </wsdl:input>
            <wsdl:output>
                <soap:body use="literal" />
            </wsdl:output>
            <wsdl:fault name="fault">
                <soap:fault use="literal" name="fault" />
            </wsdl:fault>
        </wsdl:operation>
    </wsdl:binding>

    <wsdl:service name="Order_Service">
        <wsdl:documentation>Order service</wsdl:documentation>
        <wsdl:port name="Order_Port" binding="tns:Order_SoapBinding">
            <soap:address location="http://localhost:9090/codenotfound/ws/order" />
        </wsdl:port>
    </wsdl:service>

</wsdl:definitions>
```

To build and run the example we will be using [Apache Maven](https://maven.apache.org/). In addition to the `spring-boot-starter-web-services` and `spring-boot-starter-test` dependencies we need to add the `spring-ws-test` dependency which  contains support for both server side and client side integration testing as we will see further down below.

The [Mockito](http://site.mockito.org/) mocking framework will be used to mock some classes when creating Unit test cases for the for the client implementation. For this we will include the `mockito-all` dependency as shown below. The rest of the configuration is identical to the hello world example.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.codenotfound</groupId>
    <artifactId>spring-ws-test</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>spring-ws-test</name>
    <description>Spring WS - Unit &amp; Integration Test by Mocking the Client and Web Service</description>
    <url>https://www.codenotfound.com/2017/02/spring-ws-unit-integration-test-mocking-client-web-service.html</url>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.1.RELEASE</version>
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>

        <mockito-all.version>2.0.2-beta</mockito-all.version>

        <maven-jaxb2-plugin.version>0.13.1</maven-jaxb2-plugin.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web-services</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!-- spring-ws-test -->
        <dependency>
            <groupId>org.springframework.ws</groupId>
            <artifactId>spring-ws-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!-- mockito -->
        <dependency>
            <groupId>org.mockito</groupId>
            <artifactId>mockito-all</artifactId>
            <version>${mockito-all.version}</version>
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

# Testing the Endpoint (Provider)

The Spring WS reference documentation contains [a detailed section on how to test the server side part](http://docs.spring.io/spring-ws/docs/2.4.0.RELEASE/reference/htmlsingle/#d5e1577). When it comes to testing Web service endpoints, there are two possible approaches:
1. Write Unit Tests, where you provide (mock) arguments for your 'Endpoint' to consume.
2. Write Integrations Tests, which do test the contents of the message.



# Testing the Client (Consumer)


---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-ws/tree/master/springws-helloworld-example).
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This Spring WS example turned out a bit longer than expected but hopefully it helped to explain the core client and endpoint concepts. Feel free to leave a comment if you enjoyed reading or in case you have any additional questions.



How do I create a mock object for Spring's WebServiceTemplate?
Testing Spring web services end point at server side?
Junit test for spring ws endpoint interceptor
How do I create a mock object for Spring's WebServiceTemplate?
How Should I Unit Test WebServiceTemplate (SpringWS)
Mockito pattern for a Spring web service call
How do I create a mock object for Spring's WebServiceTemplate?
Mockito pattern for a Spring web service call










