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

Next is the Maven POM file which contains the needed dependencies. At the bottom of the list we find the CXF dependencies. The 'cxf-rt-transports-http-jetty' dependency is only needed in case the CFXServlet is not used. As the example includes a JUnit test that runs without CXFServlet we need to add this dependency.

CXF supports the Spring 2.0 XML syntax, which makes it easy to declare endpoints which are backed by Spring and inject clients into application code. It is also possible to use CXF without Spring but it may take a little extra effort. A number of things like policies and annotation processing are not wired in without Spring. To take advantage of those you would need to do some pre-setup coding. The example below will use Spring to create both requester (client) and provider (service) as such the needed Spring dependencies are included.

In addition to a unit test case we will also create an integration test case for which an instance of Jetty will be started that will host the above Hello World service. In order to achieve this the 'jetty-maven-plugin' has been added which is configured to be started/stopped, before/after the integration-test phase of Maven. The '<daemon>true</daemon>' configuration option forces Jetty to execute only while Maven is running, instead of running indefinitely.















