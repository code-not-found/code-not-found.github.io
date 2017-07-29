---
title: Apache CXF - Custom SOAP Header Example
permalink: /apache-cxf-custom-soap-header-example.html
excerpt: "A detailed step-by-step tutorial on how to add and get a custom SOAP header using Apache CXF and Spring Boot."
date: 2017-07-24
modified: 2017-07-24
header:
  teaser: "assets/images/apache-cxf-teaser.png"
categories: [Apache CXF - JAX-WS]
tags: [Apache CXF, Client, CXF, Endpoint, Example, Header, Maven, SOAP, Spring Boot, Tutorial]
published: false
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/apache-cxf-logo.png" alt="apache cxf logo" class="logo">
</figure>

The [SOAP header](https://www.w3.org/TR/2000/NOTE-SOAP-20000508/#_Toc478383497){:target="_blank"} is an optional sub-element of the SOAP envelope. It is used to pass application related information that is processed by SOAP nodes along the message flow.

This example details how a web service client can add a SOAP header on an outgoing request. It also illustrates how a server endpoint can then get the SOAP header from an incoming request. Both client and server are realized using Apache CXF, Spring Boot, and Maven.

If you want to learn more about Apache CXF for JAX-WS - head on over to the [Apache CXF - JAX-WS tutorials page]({{ site.url }}/cxf-jaxws/).
{: .notice--primary}

# General Project Setup

Tools used:
* Apache CXF 3.1
* Spring Boot 1.5
* Maven 3.5

The configuration of this project is based on a previous [CXF example project ]({{ site.url }}/apache-cxf-spring-boot-soap-web-service-client-server-example.html) in which we have swapped out the basic <var>helloworld.wsdl</var> for a more generic <var>ticketagent.wsdl</var> from the [W3C WSDL 1.1 specification](https://www.w3.org/TR/wsdl11elementidentifiers/#Iri-ref-ex){:target="_blank"}.

There are two different ways to define the use of SOAP header fields in a Web service, namely [implicit and explicit headers](https://www.ibm.com/developerworks/library/ws-tip-headers/index.html){:target="_blank"}.

A header definition is called explicit if it is part of the service <var>'&lt;portType&gt;'</var>. If the message part that is transferred in the header does not show up anywhere in the <var>'&lt;portType&gt;'</var> element, then it is an implicit header.

In the context of this code sample, we will tackle implicit SOAP headers. The reason for this is that with explicit headers there is no specific configuration needed in order to have CXF process the headers and make them available as part of the generated service interface. For those interested, there is an [explicit SOAP header CXF project](https://github.com/code-not-found/cxf-jaxws/tree/master/cxf-jaxws-soap-header-explicit) available on the CXF JAX-WS GitHub repository.

As the sample ticketing WSDL does not contain a SOAP header we will add an <var>'clientId'</var> element. Note that the header is only defined on the binding, as such it is implicit.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<wsdl:definitions targetNamespace="http://example.org/TicketAgent.wsdl11"
  xmlns:tns="http://example.org/TicketAgent.wsdl11" xmlns:wsdl="http://schemas.xmlsoap.org/wsdl/"
  xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:xsTicketAgent="http://example.org/TicketAgent.xsd"
  xmlns:soap="http://schemas.xmlsoap.org/wsdl/soap/">

  <wsdl:types>
    <xs:schema xmlns:xsTicketAgent="http://example.org/TicketAgent.xsd"
      targetNamespace="http://example.org/TicketAgent.xsd">

      <xs:element name="listFlightsRequest" type="xsTicketAgent:tListFlights" />
      <xs:complexType name="tListFlights">
        <xs:sequence>
          <xs:element name="travelDate" type="xs:date" />
          <xs:element name="startCity" type="xs:string" />
          <xs:element name="endCity" type="xs:string" />
        </xs:sequence>
      </xs:complexType>

      <xs:element name="listFlightsResponse" type="xsTicketAgent:tFlightsResponse" />
      <xs:complexType name="tFlightsResponse">
        <xs:sequence>
          <xs:element name="flightNumber" type="xs:integer"
            minOccurs="0" maxOccurs="unbounded" />
        </xs:sequence>
      </xs:complexType>

      <xs:element name="listFlightsRequestHeader" type="xsTicketAgent:tListFlightsHeader" />
      <xs:complexType name="tListFlightsHeader">
        <xs:sequence>
          <xs:element name="clientId" type="xs:string" />
        </xs:sequence>
      </xs:complexType>
    </xs:schema>
  </wsdl:types>

  <wsdl:message name="listFlightsRequest">
    <wsdl:part name="body" element="xsTicketAgent:listFlightsRequest" />
  </wsdl:message>

  <wsdl:message name="listFlightsResponse">
    <wsdl:part name="body" element="xsTicketAgent:listFlightsResponse" />
  </wsdl:message>

  <wsdl:message name="listFlightsRequestHeader">
    <wsdl:part name="header" element="xsTicketAgent:listFlightsRequestHeader" />
  </wsdl:message>

  <wsdl:portType name="TicketAgent">
    <wsdl:operation name="listFlights">
      <wsdl:input message="tns:listFlightsRequest" />
      <wsdl:output message="tns:listFlightsResponse" />
    </wsdl:operation>
  </wsdl:portType>

  <wsdl:binding name="TicketAgentSoap" type="tns:TicketAgent">
    <soap:binding style="document"
      transport="http://schemas.xmlsoap.org/soap/http" />
    <wsdl:operation name="listFlights">
      <wsdl:input>
        <soap:header message="tns:listFlightsRequestHeader"
          part="header" use="literal"></soap:header>
        <soap:body parts="body" use="literal" />
      </wsdl:input>
      <wsdl:output>
        <soap:body parts="body" use="literal" />
      </wsdl:output>
    </wsdl:operation>
  </wsdl:binding>
</wsdl:definitions>
```

Typically in the case of implicit headers, extra code must be written (or generated) to deal with the header information that is not part of the portType. When using the [wsdl2java](http://cxf.apache.org/docs/wsdl-to-java.html){:target="_blank"} goal of the `cxf-codegen-plugin` plugin, there is an <var>'-exsh'</var> option that enables or disables processing of implicit SOAP headers. The default value is false.

When using Maven, add the <var>&lt;extendedSoapHeaders&gt;</var> element with a value equals to <var>'true'</var> in order to have CXF process the implicit header.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>cxf-jaxws-soap-header</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>cxf-jaxws-soap-header</name>
  <description>Spring WS - SOAP Header Example</description>
  <url>https://www.codenotfound.com/apache-cxf-soap-header-example.html</url>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.4.RELEASE</version>
  </parent>

  <properties>
    <java.version>1.8</java.version>

    <cxf.version>3.1.12</cxf.version>
  </properties>

  <dependencies>
    <!-- cxf -->
    <dependency>
      <groupId>org.apache.cxf</groupId>
      <artifactId>cxf-spring-boot-starter-jaxws</artifactId>
      <version>${cxf.version}</version>
    </dependency>
    <!-- spring-boot -->
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
                  <wsdl>${project.basedir}/src/main/resources/wsdl/ticketagent.wsdl</wsdl>
                  <wsdlLocation>classpath:wsdl/ticketagent.wsdl</wsdlLocation>
                  <!-- enables processing of implicit SOAP headers, default is false -->
                  <extendedSoapHeaders>true</extendedSoapHeaders>
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

Running the below Maven command will generate the JAXB objects, including the `ListFlightsSoapHeaders` SOAP header.

``` plaintext
mvn generate-sources
```

<figure>
    <img src="{{ site.url }}/assets/images/cxf/jaxws/cxf-codegen-plugin-soap-header-generated-class.png" alt="cxf-codegen-plugin soap header generated class">
</figure>

# CXF Add SOAP Header

The `cxf-codegen-plugin` did most of the heavy lifting. Simply use the `ObjectFactory` in order to generate a `TListFlightsHeader` object. Set the client ID and pass the header to the `listFlights()` method of the TickectAgent proxy.

``` java
package com.codenotfound.client;

import java.math.BigInteger;
import java.util.List;

import org.example.ticketagent.ObjectFactory;
import org.example.ticketagent.TFlightsResponse;
import org.example.ticketagent.TListFlights;
import org.example.ticketagent.TListFlightsHeader;
import org.example.ticketagent_wsdl11.TicketAgent;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class TicketAgentClient {

  @Autowired
  private TicketAgent ticketAgentProxy;

  public List<BigInteger> listFlights(String clientId) {
    ObjectFactory factory = new ObjectFactory();
    TListFlights tListFlights = factory.createTListFlights();

    // create the SOAP header
    TListFlightsHeader tListFlightsHeader = factory.createTListFlightsHeader();
    tListFlightsHeader.setClientId(clientId);

    TFlightsResponse response = ticketAgentProxy.listFlights(tListFlights, tListFlightsHeader);

    return response.getFlightNumber();
  }
}
```

# CXF Get Soap Header 

Accessing the SOAP header is again straight forward as the interface method already contains the `TListFlightsHeader` as a parameter. In case the `getClientId()` method returns a specific value an extra flight is returned in the response.

> Note that we have used the `@Features` annotation in order to log the received request message. This allows us to visualize the added SOAP header. 

``` java
package com.codenotfound.endpoint;

import java.math.BigInteger;

import org.apache.cxf.feature.Features;
import org.example.ticketagent.ObjectFactory;
import org.example.ticketagent.TFlightsResponse;
import org.example.ticketagent.TListFlights;
import org.example.ticketagent.TListFlightsHeader;
import org.example.ticketagent_wsdl11.TicketAgent;

@Features(features = "org.apache.cxf.feature.LoggingFeature")
public class TicketAgentImpl implements TicketAgent {

  @Override
  public TFlightsResponse listFlights(TListFlights body, TListFlightsHeader header) {
    ObjectFactory factory = new ObjectFactory();
    TFlightsResponse response = factory.createTFlightsResponse();
    response.getFlightNumber().add(BigInteger.valueOf(101));

    // add an extra flightNumber in the case of a clientId == abc123
    if ("abc123".equals(header.getClientId())) {
      response.getFlightNumber().add(BigInteger.valueOf(202));
    }

    return response;
  }
}
```

# Testing the SOAP Header

Now that we are able to add and get a SOAP header in both client and server, let's add a unit test case to verify the above code. We simply call the endpoint and verify if a second flight number is returned.

``` java
package com.codenotfound;

import static org.assertj.core.api.Assertions.assertThat;

import java.math.BigInteger;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.SpringBootTest.WebEnvironment;
import org.springframework.test.context.junit4.SpringRunner;

import com.codenotfound.client.TicketAgentClient;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = WebEnvironment.DEFINED_PORT)
public class SpringCxfApplicationTests {

  @Autowired
  private TicketAgentClient ticketAgentClient;

  @Test
  public void testListFlights() {
    assertThat(ticketAgentClient.listFlights("abc123").get(1)).isEqualTo(BigInteger.valueOf(202));
  }
}
```

Fire up the test case by executing below Maven command.

``` plaintext
mvn test
```

The outcome should be a successful test run in which the request and response messages are logged. As expected, the client ID has been set on the request.

``` plaintext
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.5.4.RELEASE)

11:27:50.992 [main] INFO  c.c.SpringCxfApplicationTests - Starting SpringCxfApplicationTests on cnf-pc with PID 5708 (started by CodeNotFound in c:\codenotfound\code\cxf-jaxws\cxf-jaxws-soap-header)
11:27:50.994 [main] INFO  c.c.SpringCxfApplicationTests - No active profile set, falling back to default profiles: default
11:27:54.175 [main] INFO  c.c.SpringCxfApplicationTests - Started SpringCxfApplicationTests in 3.47 seconds (JVM running for 4.142)
11:27:54.398 [http-nio-9090-exec-1] INFO  o.a.c.s.T.T.TicketAgent - Inbound Message
----------------------------
ID: 1
Address: http://localhost:9090/codenotfound/ws/ticketagent
Encoding: UTF-8
Http-Method: POST
Content-Type: text/xml; charset=UTF-8
Headers: {Accept=[*/*], cache-control=[no-cache], connection=[keep-alive], Content-Length=[343], content-type=[text/xml; charset=UTF-8], host=[localhost:9090], pragma=[no-cache], SOAPAction=[""], user-agent=[Apache-CXF/3.1.12]}
Payload: <soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"><soap:Header><ns2:listFlightsRequestHeader xmlns:ns2="http://example.org/TicketAgent.xsd"><clientId>abc123</clientId></ns2:listFlightsRequestHeader></soap:Header><soap:Body><ns2:listFlightsRequest xmlns:ns2="http://example.org/TicketAgent.xsd"/></soap:Body></soap:Envelope>
--------------------------------------
11:27:54.467 [http-nio-9090-exec-1] INFO  o.a.c.s.T.T.TicketAgent - Outbound Message
---------------------------
ID: 1
Response-Code: 200
Encoding: UTF-8
Content-Type: text/xml
Headers: {}
Payload: <soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"><soap:Body><ns2:listFlightsResponse xmlns:ns2="http://example.org/TicketAgent.xsd"><flightNumber>101</flightNumber><flightNumber>202</flightNumber></ns2:listFlightsResponse></soap:Body></soap:Envelope>
--------------------------------------
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 3.872 sec - in com.codenotfound.SpringCxfApplicationTests

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 6.535 s
[INFO] Finished at: 2017-07-23T11:27:54+02:00
[INFO] Final Memory: 21M/227M
[INFO] ------------------------------------------------------------------------
```

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-ws/tree/master/spring-ws-soap-header).
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

In this tutorial, we illustrated how you can add and get a SOAP header using Apache CXF.

If you found this example useful or if something is not clear, feel free to leave a comment.
