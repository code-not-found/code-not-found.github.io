---
title: Apache CXF - Logging SOAP Request Response Fault Messages Example
permalink: /apache-cxf-logging-soap-request-response-fault-messages-example.html
excerpt: A detailed step-by-step tutorial on how to setup CXF to log SOAP request, response and fault messages.
date: 2017-07-22
modified: 2017-07-22
header:
  teaser: "assets/images/apache-cxf-teaser.png"
categories: [Apache CXF - JAX-WS]
tags: [Apache CXF, CXF, Example, Fault, Feature, Interceptor, Logging, Maven, Request, Response, SOAP, Spring Boot, Tutorial]
published: true
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/apache-cxf-logo.png" alt="apache cxf logo" class="logo">
</figure>

[Since Apache CXF 3.1](http://cxf.apache.org/docs/message-logging.html){:target="_blank"}, the message logging code was moved into a separate module and gathered a number of new features.

In this tutorial we will demonstrate how to configure CXF to log the SOAP request, response and fault XML using a logging `Interceptor`. The example uses the [Logback logging framework](https://logback.qos.ch/) in addition to Apache CXF, Spring Boot, and Maven.

If you want to learn more about Apache CXF for JAX-WS - head on over to the [Apache CXF - JAX-WS tutorials page]({{ site.url }}/cxf-jaxws/).
{: .notice--primary}

# General Project Setup

Tools used:
* Apache CXF 3.1
* Spring Boot 1.5
* Maven 3.5

The setup of the project is based on a previous [CXF web service example]({{ site.url }}/apache-cxf-spring-boot-soap-web-service-client-server-example.html) in which we have swapped out the basic <var>helloworld.wsdl</var> for a more generic <var>ticketagent.wsdl</var> from the [W3C WSDL 1.1 specification](https://www.w3.org/TR/wsdl11elementidentifiers/#Iri-ref-ex){:target="_blank"}.

As the sample ticketing WSDL does not contain a SOAP fault we will add one in the context of this tutorial.

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

      <xs:element name="listFlightsFault" type="xsTicketAgent:tFlightsFault" />
      <xs:complexType name="tFlightsFault">
        <xs:sequence>
          <xs:element name="errorMessage" type="xs:string" />
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

  <wsdl:message name="listFlightsFault">
    <wsdl:part name="body" element="xsTicketAgent:listFlightsFault" />
  </wsdl:message>

  <wsdl:portType name="TicketAgent">
    <wsdl:operation name="listFlights">
      <wsdl:input message="tns:listFlightsRequest" />
      <wsdl:output message="tns:listFlightsResponse" />
      <wsdl:fault name="listFlightsFault" message="tns:listFlightsFault"></wsdl:fault>
    </wsdl:operation>
  </wsdl:portType>

  <wsdl:binding name="TicketAgentSoap" type="tns:TicketAgent">
    <soap:binding style="document"
      transport="http://schemas.xmlsoap.org/soap/http" />
    <wsdl:operation name="listFlights">
      <wsdl:input>
        <soap:body parts="body" use="literal" />
      </wsdl:input>
      <wsdl:output>
        <soap:body parts="body" use="literal" />
      </wsdl:output>
      <wsdl:fault name="listFlightsFault">
        <soap:fault name="listFlightsFault" use="literal" />
      </wsdl:fault>
    </wsdl:operation>
  </wsdl:binding>
</wsdl:definitions>
```

[Maven](https://maven.apache.org/){:target="_blank"} is used to build the project. As the CXF message logging code was moved into a separate package since version 3.1, we need to include it by adding the `cxf-rt-features-logging` dependency to the project POM file as shown below.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>cxf-jaxws-logging-logback</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>cxf-jaxws-logging-logback</name>
  <description>Apache CXF - Logging SOAP Request Response Fault Messages Example</description>
  <url>https://www.codenotfound.com/apache-cxf-logging-soap-request-response-fault-messages-example.html</url>

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
    <dependency>
      <groupId>org.apache.cxf</groupId>
      <artifactId>cxf-rt-features-logging</artifactId>
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

# Configure the CXF LoggingInInterceptor and CXF LoggingOutInterceptor

[Interceptors](https://cxf.apache.org/docs/interceptors.html){:target="_blank"} are the fundamental processing unit inside CXF. For more information checkout following post on the basic [CXF interceptor architecture]({{ site.url }}/2015/01/cxf-feature-vs-interceptor.html).

CXF ships with a `LoggingInInterceptor` that allows logging of the **received** SOAP XML messages. In addition, for logging **sent** SOAP XML messages, a `LoggingOutInterceptor` is provided. These interceptors can be added to one of the CXF interceptor providers (`Client`, `Endpoint`, `Service`, `Bus` or `Binding`) that implement the `InterceptorProvider` interface.

In order to demonstrate this we will add both `LoggingInInterceptor` and `LoggingOutInterceptor` to the TicketAgent client as illustrated below.

> Note that for SOAP faults, there will be separate error handling chains. In the case of a client this is an inbound error handling chain to which we also need to add an `LoggingInInterceptor`.

``` java
package com.codenotfound.client;


import org.apache.cxf.ext.logging.LoggingInInterceptor;
import org.apache.cxf.ext.logging.LoggingOutInterceptor;
import org.apache.cxf.jaxws.JaxWsProxyFactoryBean;
import org.example.ticketagent_wsdl11.TicketAgent;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class ClientConfig {

  @Value("${client.ticketagent.address}")
  private String address;

  @Bean(name = "ticketAgentProxy")
  public TicketAgent ticketAgentProxy() {
    JaxWsProxyFactoryBean jaxWsProxyFactoryBean = new JaxWsProxyFactoryBean();
    jaxWsProxyFactoryBean.setServiceClass(TicketAgent.class);
    jaxWsProxyFactoryBean.setAddress(address);

    // add an interceptor to log the outgoing request messages
    jaxWsProxyFactoryBean.getOutInterceptors().add(loggingOutInterceptor());
    // add an interceptor to log the incoming response messages
    jaxWsProxyFactoryBean.getInInterceptors().add(loggingInInterceptor());
    // add an interceptor to log the incoming fault messages
    jaxWsProxyFactoryBean.getInFaultInterceptors().add(loggingInInterceptor());

    return (TicketAgent) jaxWsProxyFactoryBean.create();
  }

  @Bean
  public LoggingOutInterceptor loggingOutInterceptor() {
    return new LoggingOutInterceptor();
  }

  @Bean
  public LoggingInInterceptor loggingInInterceptor() {
    return new LoggingInInterceptor();
  }
}
```

# Configure the CXF LoggingFeature

A [Feature](https://cxf.apache.org/docs/features.html){:target="_blank"} in CXF is a way of adding capabilities to a `Client`, `Server` or `Bus`. They provide a simple way to perform or configure a series of related tasks.

CXF includes a `LoggingFeature` which encapsulates the creation of the different logging interceptors and then subsequently adds them to all the different interceptor chains.

Let's add a `LoggingFeature` to the CXF `Bus` that hosts our TicketAgent `Endpoint`. First we create a new instance of the feature and enable formatting of the XML message by using `setPrettyLogging()` method. We then add the feature by using `setFeatures()` on the bus.

``` java
package com.codenotfound.endpoint;

import java.util.ArrayList;
import java.util.Arrays;

import javax.xml.ws.Endpoint;

import org.apache.cxf.Bus;
import org.apache.cxf.ext.logging.LoggingFeature;
import org.apache.cxf.jaxws.EndpointImpl;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class EndpointConfig {

  @Autowired
  private Bus cxfBus;

  @Bean
  public Endpoint endpoint() {
    EndpointImpl endpoint = new EndpointImpl(bus(), new TicketAgentImpl());
    endpoint.publish("/ticketagent");

    return endpoint;
  }

  @Bean
  public Bus bus() {
    cxfBus.setFeatures(new ArrayList<>(Arrays.asList(loggingFeature())));

    return cxfBus;
  }

  @Bean
  public LoggingFeature loggingFeature() {
    LoggingFeature loggingFeature = new LoggingFeature();
    loggingFeature.setPrettyLogging(true);

    return loggingFeature;
  }
}
```

> There is also an `@Features` annotation that can be used on either the service endpoint interface (SEI) or the SEI implementation class. If applied to the TicketAgent example we would need to annotate the `TicketAgentImpl` class as shown below.

``` java
package com.codenotfound.endpoint;

import java.math.BigInteger;

import org.example.ticketagent.ObjectFactory;
import org.example.ticketagent.TFlightsFault;
import org.example.ticketagent.TFlightsResponse;
import org.example.ticketagent.TListFlights;
import org.example.ticketagent_wsdl11.ListFlightsFault;
import org.example.ticketagent_wsdl11.TicketAgent;

@Features(features = "org.apache.cxf.ext.logging.LoggingFeature")
public class TicketAgentImpl implements TicketAgent {

  @Override
  public TFlightsResponse listFlights(TListFlights request) throws ListFlightsFault {
    ObjectFactory factory = new ObjectFactory();

    if ("XYZ".equals(request.getStartCity())) {
      TFlightsFault tFlightsFault = factory.createTFlightsFault();
      tFlightsFault.setErrorMessage("no city named XYZ");

      throw new ListFlightsFault("unknown city", tFlightsFault);
    }

    TFlightsResponse response = factory.createTFlightsResponse();
    response.getFlightNumber().add(BigInteger.valueOf(101));

    return response;
  }
}
```

# CXF Logging Configuration

Now that we have setup logging on both client and server we need to set the logging level of the <var>'org.apache.cxf.services'</var> logger to <var>'INFO'</var> in order to have the XML SOAP messages appear.

The `cxf-spring-boot-starter-jaxws` Spring Boot starter automatically includes the Logback, Log4J and SLF4J dependencies. As such we just have to place a <var>logback.xml</var> configuration file on the classpath in order to activate Logback.

You can use the logger name to fine tune which services you want to log. The logger name is <var>org.apache.cxf.services.<service name>.<type></var>. Where the service name is the name of the generate interface class (in this example this is equals to <kbd>"TicketAgent"</kbd>). The type can be one of the following depending on whether the message is sent (OUT) or received (IN):

* REQ_IN
* RESP_IN
* FAULT_IN
* REQ_OUT
* RESP_OUT
* FAULT_OUT

In this example we will avoid logging all messages that are received (by either client or endpoint) by setting them to WARN.

``` xml
<configuration>

  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
      <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n
      </pattern>
    </encoder>
  </appender>

  <logger name="com.codenotfound" level="INFO" />
  <logger name="org.apache.cxf" level="WARN" />
  <logger name="org.springframework" level="WARN" />

  <!-- INFO level needed to log the SOAP messages -->
  <logger name="org.apache.cxf.services" level="INFO" />

  <!-- fine tune individual service logging -->
  <logger name="org.apache.cxf.services.TicketAgent.REQ_IN"
    level="WARN" />
  <logger name="org.apache.cxf.services.TicketAgent.RESP_IN"
    level="WARN" />
  <logger name="org.apache.cxf.services.TicketAgent.FAULT_IN"
    level="WARN" />

  <root level="WARN">
    <appender-ref ref="STDOUT" />
  </root>

</configuration>
```

# Test That the CXF Logging Interceptor and Feature Are Enabled

As we also want to log faults, we extend the original `SpringCxfApplicationTests` class with an additional `testListFlightsFault()` unit test which triggers a SOAP fault to be returned in case an unknown city is supplied.

``` java
package com.codenotfound;

import static org.assertj.core.api.Assertions.assertThat;

import java.math.BigInteger;

import org.example.ticketagent_wsdl11.ListFlightsFault;
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
  public void testListFlights() throws ListFlightsFault {
    assertThat(ticketAgentClient.listFlights("LAX", "SFO").get(0))
        .isEqualTo(BigInteger.valueOf(101));
  }

  @Test
  public void testListFlightsFault() {
    try {
      ticketAgentClient.listFlights("XYZ", "SFO");
    } catch (Exception exception) {
      assertThat(exception.getMessage()).isEqualTo("unknown city");
    }
  }
}
```

Run the above test, by executing following Maven command in the projects root folder:

``` plaintext
mvn test
```

Maven will download the needed dependencies, compile the code and run the unit test cases during which the SOAP XML are logged as shown below:

``` plaintext
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.5.4.RELEASE)

07:27:54.753 [main] INFO  c.c.SpringCxfApplicationTests - Starting SpringCxfApplicationTests on cnf-pc with PID 1000 (started by CodeNotFound in c:\codenotfound\code\cxf-jaxws\cxf-jaxws-logging-logback)
07:27:54.755 [main] INFO  c.c.SpringCxfApplicationTests - No active profile set, falling back to default profiles: default
07:27:57.934 [main] INFO  c.c.SpringCxfApplicationTests - Started SpringCxfApplicationTests in 3.486 seconds (JVM running for 4.123)
07:27:58.082 [main] INFO  o.a.cxf.services.TicketAgent.REQ_OUT - REQ_OUT
    Address: http://localhost:9090/codenotfound/ws/ticketagent
    HttpMethod: POST
    Content-Type: text/xml
    ExchangeId: 2178e0db-4a64-4c9c-9467-dfdcfe1b3075
    ServiceName: TicketAgentService
    PortName: TicketAgentPort
    PortTypeName: TicketAgent
    Headers: {SOAPAction="", Accept=*/*}
    Payload: <soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"><soap:Body><ns2:listFlightsRequest xmlns:ns2="http://example.org/TicketAgent.xsd"><startCity>LAX</startCity><endCity>
SFO</endCity></ns2:listFlightsRequest></soap:Body></soap:Envelope>

07:27:58.211 [http-nio-9090-exec-1] INFO  o.a.c.services.TicketAgent.RESP_OUT - RESP_OUT
    Content-Type: text/xml
    ResponseCode: 200
    ExchangeId: 9633cf96-f3b7-44f0-ab32-948da11eeab3
    ServiceName: TicketAgentImplService
    PortName: TicketAgentImplPort
    PortTypeName: TicketAgent
    Headers: {}
    Payload: <soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
  <soap:Body>
    <ns2:listFlightsResponse xmlns:ns2="http://example.org/TicketAgent.xsd">
      <flightNumber>101</flightNumber>
    </ns2:listFlightsResponse>
  </soap:Body>
</soap:Envelope>


07:27:58.257 [main] INFO  o.a.cxf.services.TicketAgent.REQ_OUT - REQ_OUT
    Address: http://localhost:9090/codenotfound/ws/ticketagent
    HttpMethod: POST
    Content-Type: text/xml
    ExchangeId: f5f0520c-cc9c-479d-bb8f-188d7ad288ed
    ServiceName: TicketAgentService
    PortName: TicketAgentPort
    PortTypeName: TicketAgent
    Headers: {SOAPAction="", Accept=*/*}
    Payload: <soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"><soap:Body><ns2:listFlightsRequest xmlns:ns2="http://example.org/TicketAgent.xsd"><startCity>XYZ</startCity><endCity>
SFO</endCity></ns2:listFlightsRequest></soap:Body></soap:Envelope>

07:27:58.270 [http-nio-9090-exec-2] INFO  o.a.c.services.TicketAgent.FAULT_OUT - FAULT_OUT
    Content-Type: text/xml
    ResponseCode: 500
    ExchangeId: db05096f-9cb8-49c9-9c3f-d5030c2ba582
    ServiceName: TicketAgentImplService
    PortName: TicketAgentImplPort
    PortTypeName: TicketAgent
    Headers: {}
    Payload: <soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
  <soap:Body>
    <soap:Fault>
      <faultcode>soap:Server</faultcode>
      <faultstring>unknown city</faultstring>
      <detail>
        <ns2:listFlightsFault xmlns:ns2="http://example.org/TicketAgent.xsd">
          <errorMessage>no city named XYZ</errorMessage>
        </ns2:listFlightsFault>
      </detail>
    </soap:Fault>
  </soap:Body>
</soap:Envelope>


Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 3.919 sec - in com.codenotfound.SpringCxfApplicationTests

Results :

Tests run: 2, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 6.555 s
[INFO] Finished at: 2017-07-22T07:27:58+02:00
[INFO] Final Memory: 21M/227M
[INFO] ------------------------------------------------------------------------
```

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/cxf-jaxws/tree/master/cxf-jaxws-logging-logback).
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This concludes our example in which we programmatically added a CXF loggingininterceptor, loggingoutinterceptor and loggingfeature in order to log the sent/received SOAP messages.

Feel free to drop a comment in case of a question or if you just like the post.
