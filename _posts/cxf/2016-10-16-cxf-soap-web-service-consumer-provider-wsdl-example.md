---
title: CXF - SOAP Web Service Consumer & Provider WSDL Example
excerpt: How to handle element missing @XmlRootElement annotation errors when trying to marshal a Java object using JAXB.
date: 2016-10-16 21:00
tags: [Apache CXF, Client, Consumer, Contract First, CXF, Endpoint, Example, Hello World, Maven, Provider, Spring Boot, Tutorial, WSDL]
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/jaxb-logo.png" alt="jaxb logo">
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
~~~ xml
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
~~~

Maven is used to build and run the example. The Hello World service endpoint will be hosted on an embedded Apache Tomcat server that ships directly with [Spring Boot](https://projects.spring.io/spring-boot/). To facilitate the management of the different Spring dependencies, [Spring Boot Starters](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters) are used which are a set of convenient dependency descriptors that you can include in your application.

To avoid having to manage the version compatibility of the different Spring dependencies, we will inherit the defaults from the `spring-boot-starter-parent` parent POM.

There is actually a [Spring Boot starter specifically for CXF](http://cxf.apache.org/docs/springboot.html) that takes care of importing the needed Spring Boot dependencies. In addition it automatically registers `CXFServlet` with a '/services/*' URL pattern for serving CXF JAX-WS endpoints and it offers some properties for configuration of the `CXFServlet`. In order to use the starter we declare a dependency to `cxf-spring-boot-starter-jaxws` in our Maven POM file.

For Unit testing our Spring Boot application we also include the `spring-boot-starter-test` dependency.

To take advantage of Spring Boot's capability to create a single, runnable "über-jar", we also include the `spring-boot-maven-plugin` Maven plugin. This also allows quickly starting the web service via a Maven command.