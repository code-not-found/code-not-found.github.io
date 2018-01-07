---
title: Spring WS - Unit Test by Mocking the Client &amp; Web Service Endpoint
permalink: /2017/02/spring-ws-unit-test-mocking-client-web-service-endpoint.html
excerpt: A detailed step-by-step tutorial on how to implement a unit test by mocking Spring's WebServiceTemplate and Web Service Endpoint.
date: 2017-02-28 21:00
categories: [Spring-WS]
tags: [Client, Consumer, Endpoint, Example, JUnit, Maven, Mock, Mockito, Provider, Spring, Spring Boot, Spring Web Services, Spring-WS, Tutorial, Unit Test, Testing]
published: false
---

<figure>
    <img src="{{ site.url }}/assets/images/logo/spring-logo.png" alt="spring logo" class="logo">
</figure>

[Unit testing](https://en.wikipedia.org/wiki/Unit_testing) is a software testing method by which **individual units of source code** are tested to determine whether they are fit for proper operation. Unit testing can be done manually but by preference it is automated. In this example we'll show how to unit test a Spring-WS web service client and endpoint by using Mockito, Spring WS Test, Spring Boot and Maven.

Tools used:
* Spring-WS 2.4
* Spring-WS Test 2.4
* Spring Boot 1.5
* Maven 3

# General Project Setup

The below example is based on a previous [tutorial in which we built a Spring Web Services client and server starting from a Hello World WSDL file]({{ site.url }}/2016/10/spring-ws-soap-web-service-consumer-provider-wsdl-example.html). Instead of reusing the <var>helloworld.wsdl</var> we will use below <var>order.wsdl</var> which mimics a more real life example.

As input we pass an order that consists out of customer information and a list of order line items. The output is either an order confirmation identifier that allows tracking the order or an order error in case something went wrong.

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

To build and run the example we will be using [Apache Maven](https://maven.apache.org/). In addition to the `spring-boot-starter-web-services` and `spring-boot-starter-test` dependencies we need to add the `spring-ws-test` dependency which  contains support for both Spring-WS server and client side testing as we will see further down below.

> Note that [Mockito](http://site.mockito.org/) is included in the `spring-boot-starter-web-services` dependency so we don't need to add it separately.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.codenotfound</groupId>
    <artifactId>spring-ws-unit-test</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>spring-ws-unit-test</name>
    <description>Spring WS - Unit Test by Mocking the Client &amp; Web Service Endpoint</description>
    <url>https://www.codenotfound.com/2017/02/spring-ws-unit-test-mocking-client-web-service-endpoint.html</url>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.2.RELEASE</version>
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
    </dependencies>

    <build>
        <plugins>
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

            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

# Testing the Endpoint (Provider)

Testing the `Endpoint` is actually straight forward as you can see in below `CreateOrderEndpointTest` test class. We just call the `createOrder()` method and pass an `Order` object that we prepare before the actual test is run. As the `confirmationId` is a unique ID that gets generated for each call, we just test that the returned value is not blank. 

``` java
package com.codenotfound.order.endpoint;

import static org.assertj.core.api.Assertions.assertThat;

import java.math.BigInteger;

import org.junit.Before;
import org.junit.Test;

import com.codenotfound.order.endpoint.CreateOrderEndpoint;
import com.codenotfound.types.order.CustomerType;
import com.codenotfound.types.order.LineItemType;
import com.codenotfound.types.order.LineItemsType;
import com.codenotfound.types.order.ObjectFactory;
import com.codenotfound.types.order.Order;
import com.codenotfound.types.order.ProductType;

public class CreateOrderEndpointTest {

    private CreateOrderEndpoint createOrderEndpoint = new CreateOrderEndpoint();

    private Order order;

    @Before
    public void setUp() throws Exception {
        ObjectFactory factory = new ObjectFactory();

        CustomerType customer = factory.createCustomerType();
        customer.setFirstName("John");
        customer.setLastName("Doe");

        ProductType product1 = factory.createProductType();
        product1.setId("1");
        product1.setName("batman action figure");

        LineItemType lineItem1 = factory.createLineItemType();
        lineItem1.setProduct(product1);
        lineItem1.setQuantity(BigInteger.valueOf(1));

        LineItemsType lineItems = factory.createLineItemsType();
        lineItems.getLineItem().add(lineItem1);

        order = factory.createOrder();
        order.setCustomer(customer);
        order.setLineItems(lineItems);
    }

    @Test
    public void testCreateOrder() {
        assertThat(createOrderEndpoint.createOrder(order)
                .getConfirmationId()).isNotBlank();
    }
}
```

In addition to the above unit test we will also demonstrate how you can write an `Endpoint` test case using the test features introduced in Spring Web Services 2.0. Although the [Spring WS reference documentation refers to this way of testing as integration testing](http://docs.spring.io/spring-ws/docs/2.4.0.RELEASE/reference/htmlsingle/#d5e1577), this does not really match with [the definition given by Maven which states that integration tests are run after the package phase](https://cwiki.apache.org/confluence/display/MAVENOLD/Testing+Strategies).

Spring WS Test ships with a `MockWebServiceClient` that allows sending actual XML messages towards a Spring Web Service endpoint. In an `@Before` method, we create a new `mockClient` by calling the static `createClient` method that takes the Spring `ApplicationContext` as input parameter.

In the `testCreateOrder()` test method we first create an `requestPayload` `Source` based on an XML `String`. This request is then sent to the endpoint using the `sendRequest()` method of the `mockClient` which expects an `RequestCreator` object as input. The static `withPayload()` method of the `RequestCreators` interface is used to convert the `Source` `requestPayload` to an `RequestCreator` object.

``` java
package com.codenotfound.order.endpoint;

import static org.springframework.ws.test.server.RequestCreators.withPayload;

import java.util.Collections;
import java.util.Map;

import javax.xml.transform.Source;
import javax.xml.xpath.XPathExpressionException;

import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.context.ApplicationContext;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.ws.test.server.MockWebServiceClient;
import org.springframework.ws.test.server.ResponseMatchers;
import org.springframework.xml.transform.StringSource;

@RunWith(SpringRunner.class)
@SpringBootTest
public class CreateOrderEndpointMockTest {

    @Autowired
    private ApplicationContext applicationContext;

    private MockWebServiceClient mockClient;

    @Before
    public void setUp() throws Exception {
        // create the mock client
        mockClient = MockWebServiceClient
                .createClient(applicationContext);
    }

    @Test
    public void testCreateOrder() throws XPathExpressionException {
        Source requestPayload = new StringSource(
                "<ns2:order xmlns:ns2=\"http://codenotfound.com/types/order\">"
                        + "<ns2:customer><ns2:firstName>John</ns2:firstName>"
                        + "<ns2:lastName>Doe</ns2:lastName>"
                        + "</ns2:customer><ns2:lineItems><ns2:lineItem>"
                        + "<ns2:product>" + "<ns2:id>2</ns2:id>"
                        + "<ns2:name>batman action figure</ns2:name>"
                        + "</ns2:product>"
                        + "<ns2:quantity>1</ns2:quantity>"
                        + "</ns2:lineItem>" + "</ns2:lineItems>"
                        + "</ns2:order>");

        Map<String, String> namespaces = Collections.singletonMap("ns1",
                "http://codenotfound.com/types/order");

        mockClient.sendRequest(withPayload(requestPayload))
                .andExpect(ResponseMatchers
                        .xpath("/ns1:orderConfirmation/ns1:confirmationId",
                                namespaces)
                        .exists());
    }
}
```






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










