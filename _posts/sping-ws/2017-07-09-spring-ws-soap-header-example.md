---
title: "Spring WS - SOAP Header Example"
permalink: /spring-ws-soap-header-example.html
excerpt: "A detailed step-by-step tutorial on how to set and get a SOAP header using Spring-WS and Spring Boot."
date: 2017-07-09
modified: 2017-07-09
header:
  teaser: "assets/images/teaser/spring-ws-teaser.png"
categories: [Spring-WS]
tags: [Client, Endpoint, Example, Header, Maven, SOAP, Spring, Spring Boot, Spring Web Services, Spring-WS, Tutorial]
redirect_from:
  - /2017/07/spring-ws-soap-header-example.html
published: true
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/spring-logo.jpg" alt="spring logo" class="logo">
</figure>

The [SOAP header](https://www.w3.org/TR/2000/NOTE-SOAP-20000508/#_Toc478383497){:target="_blank"} is an optional sub-element of the SOAP envelope. It is used to pass application-related information that is processed by SOAP nodes along the message flow.

The below example details how a web service client can set a SOAP header on an outgoing request. It also illustrates how a server endpoint can then get the SOAP header from an incoming request. Both client and server are realized using Spring-WS, Spring Boot, and Maven.

If you want to learn more about Spring WS - head on over to the [Spring WS tutorials page]({{ site.url }}/spring-ws/).
{: .notice--primary}

# General Project Setup

Tools used:
* Spring-WS 2.4
* Spring Boot 1.5
* Maven 3.5

The setup of the project is based on a previous [Spring SOAP web service example]({{ site.url }}/spring-ws-soap-web-service-consumer-provider-wsdl-example.html) in which we have swapped out the basic <var>helloworld.wsdl</var> for a more generic <var>ticketagent.wsdl</var> from the [W3C WSDL 1.1 specification](https://www.w3.org/TR/wsdl11elementidentifiers/#Iri-ref-ex){:target="_blank"}.

As the sample ticketing WSDL does not contain any SOAP header we will add an <var>'clientId'</var> element in the context of this tutorial.

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

      <xs:element name="listFlightsSoapHeaders" type="xsTicketAgent:ListFlightsSoapHeaders" />
      <xs:complexType name="ListFlightsSoapHeaders">
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

  <wsdl:message name="listFlightsSoapHeaders">
    <wsdl:part name="header" element="xsTicketAgent:listFlightsSoapHeaders" />
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
        <soap:header use="literal" part="header"
          message="tns:listFlightsSoapHeaders"></soap:header>
        <soap:body parts="body" use="literal" />
      </wsdl:input>
      <wsdl:output>
        <soap:body parts="body" use="literal" />
      </wsdl:output>
    </wsdl:operation>
  </wsdl:binding>
</wsdl:definitions>
```

Running the below Maven command will generate the JAXB object for the added SOAP header.

``` plaintext
mvn generate-sources
```

The project is built using [Apache Maven](https://maven.apache.org/){:target="_blank"}. As we will create some Spring-WS unit test cases to verify the example, we also include the `spring-ws-test` dependency in the project POM file.

# Client Add SOAP Header

In order to set the SOAP header on the outgoing request, we need to get hold of the [SoapMessage](https://docs.spring.io/spring-ws/docs/2.4.2.RELEASE/reference/#soap-message){:target="_blank"} which has a SOAP-specific method `getSoapHeader()` for getting the SOAP Header.

The `SoapMessage` in turn can be obtained by casting the `WebServiceMessage` from the [WebServiceMessageCallback](https://docs.spring.io/spring-ws/docs/2.4.2.RELEASE/reference/#_code_webservicemessagecallback_code){:target="_blank"} interface that gives access to the message after it has been created, but before it is sent.

As a final step, create the SOAP header using the corresponding JAXB object and marshal it into the `SOAPHeader` as shown below.

> Note that if JAXB objects are not available because the SOAP headers have not been specified in the WSDL, you can [manually add them](https://stackoverflow.com/a/20904164/4201470){:target="_blank"}.

``` java
package com.codenotfound.ws.client;

import java.math.BigInteger;
import java.util.List;

import javax.xml.bind.JAXBContext;
import javax.xml.bind.JAXBElement;
import javax.xml.bind.Marshaller;

import org.example.ticketagent.ListFlightsSoapHeaders;
import org.example.ticketagent.ObjectFactory;
import org.example.ticketagent.TFlightsResponse;
import org.example.ticketagent.TListFlights;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import org.springframework.ws.WebServiceMessage;
import org.springframework.ws.client.core.WebServiceMessageCallback;
import org.springframework.ws.client.core.WebServiceTemplate;
import org.springframework.ws.soap.SoapHeader;
import org.springframework.ws.soap.SoapMessage;

@Component
public class TicketAgentClient {

  private static final Logger LOGGER = LoggerFactory.getLogger(TicketAgentClient.class);

  @Autowired
  private WebServiceTemplate webServiceTemplate;

  @SuppressWarnings("unchecked")
  public List<BigInteger> listFlights() {
    ObjectFactory factory = new ObjectFactory();
    TListFlights tListFlights = factory.createTListFlights();

    JAXBElement<TListFlights> request = factory.createListFlightsRequest(tListFlights);

    JAXBElement<TFlightsResponse> response = (JAXBElement<TFlightsResponse>) webServiceTemplate
        .marshalSendAndReceive(request, new WebServiceMessageCallback() {

          @Override
          public void doWithMessage(WebServiceMessage message) {
            try {
              // get the header from the SOAP message
              SoapHeader soapHeader = ((SoapMessage) message).getSoapHeader();

              // create the header element
              ObjectFactory factory = new ObjectFactory();
              ListFlightsSoapHeaders listFlightsSoapHeaders =
                  factory.createListFlightsSoapHeaders();
              listFlightsSoapHeaders.setClientId("abc123");

              JAXBElement<ListFlightsSoapHeaders> headers =
                  factory.createListFlightsSoapHeaders(listFlightsSoapHeaders);

              // create a marshaller
              JAXBContext context = JAXBContext.newInstance(ListFlightsSoapHeaders.class);
              Marshaller marshaller = context.createMarshaller();

              // marshal the headers into the specified result
              marshaller.marshal(headers, soapHeader.getResult());
            } catch (Exception e) {
              LOGGER.error("error during marshalling of the SOAP headers", e);
            }
          }
        });

    return response.getValue().getFlightNumber();
  }
}
```

# Server Soap Header Annotation

To access the SOAP header on the server side, we need to specify it as an additional parameter on the `Endpoint` handling method. [A handling method](https://docs.spring.io/spring-ws/docs/2.4.2.RELEASE/reference/#server-atEndpoint-methods){:target="_blank"} typically has one or more parameters that refer to various parts of the incoming XML message. Most commonly, the handling method will have a single parameter that will map to the payload of the message, but it is also possible to map to other parts of the request message, such as for example a SOAP header.

In this example, the `listFlights()` handling method has two parameters. The first is the request payload which is mapped to the JAXB `TListFlights` object. The second parameter is a `SoapHeaderElement` which needs to be used in combination with the `@SoapHeader` annotation in order to extract the correct element from the received SOAP header.

In order to do this, the `@SoapHeader` annotation has a `value` element which needs to specify the qualified name of the target SOAP header element. The format used is that of a `QName` (i.e. namespace URI + local part, where the namespace is optional).

> Note that it is also possible to specify the `SoapHeader` as a parameter of the handling method. You would then need to iterate over the available `SoapHeaderElement`(s) to get the one you need.

From the `SoapHeaderElement` we obtain the `Source` which is unmarshalled into a `ListFlightsSoapHeaders` instance. In a next step, the `clientId` value is retrieved and used in order to determine whether we return an extra flight in the response.

``` java
package com.codenotfound.ws.endpoint;

import java.math.BigInteger;

import javax.xml.bind.JAXBContext;
import javax.xml.bind.JAXBElement;
import javax.xml.bind.Unmarshaller;

import org.example.ticketagent.ListFlightsSoapHeaders;
import org.example.ticketagent.ObjectFactory;
import org.example.ticketagent.TFlightsResponse;
import org.example.ticketagent.TListFlights;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.ws.server.endpoint.annotation.Endpoint;
import org.springframework.ws.server.endpoint.annotation.PayloadRoot;
import org.springframework.ws.server.endpoint.annotation.RequestPayload;
import org.springframework.ws.server.endpoint.annotation.ResponsePayload;
import org.springframework.ws.soap.SoapHeaderElement;
import org.springframework.ws.soap.server.endpoint.annotation.SoapHeader;

@Endpoint
public class TicketAgentEndpoint {

  private static final Logger LOGGER = LoggerFactory.getLogger(TicketAgentEndpoint.class);

  @SuppressWarnings("unchecked")
  @PayloadRoot(namespace = "http://example.org/TicketAgent.xsd", localPart = "listFlightsRequest")
  @ResponsePayload
  public JAXBElement<TFlightsResponse> listFlights(
      @RequestPayload JAXBElement<TListFlights> request, @SoapHeader(
          value = "{http://example.org/TicketAgent.xsd}listFlightsSoapHeaders") SoapHeaderElement soapHeaderElement) {
    String clientId = "unknown";

    try {
      // create an unmarshaller
      JAXBContext context = JAXBContext.newInstance(ObjectFactory.class);
      Unmarshaller unmarshaller = context.createUnmarshaller();

      // unmarshal the header from the specified source
      JAXBElement<ListFlightsSoapHeaders> headers =
          (JAXBElement<ListFlightsSoapHeaders>) unmarshaller
              .unmarshal(soapHeaderElement.getSource());

      // get the header values
      ListFlightsSoapHeaders requestSoapHeaders = headers.getValue();
      clientId = requestSoapHeaders.getClientId();
    } catch (Exception e) {
      LOGGER.error("error during unmarshalling of the SOAP headers", e);
    }

    ObjectFactory factory = new ObjectFactory();
    TFlightsResponse tFlightsResponse = factory.createTFlightsResponse();
    tFlightsResponse.getFlightNumber().add(BigInteger.valueOf(101));

    // add an extra flightNumber in the case of a clientId == abc123
    if ("abc123".equals(clientId)) {
      LOGGER.info("clientId == abc123");
      tFlightsResponse.getFlightNumber().add(BigInteger.valueOf(202));
    }

    return factory.createListFlightsResponse(tFlightsResponse);
  }
}
```

# Testing the SOAP Header

Now that we are able to set and get a SOAP header in both client and server, let's write some unit test cases to verify the correct working of the example.

For the client, we will use a `MockWebServiceServer` in order to verify that the SOAP header has been set by the client. By configuring the `soapHeader()` method of the `RequestMatchers`, we expect the specified SOAP header to exist in the request message.

``` java
package com.codenotfound.ws.client;

import static org.assertj.core.api.Assertions.assertThat;
import static org.springframework.ws.test.client.RequestMatchers.payload;
import static org.springframework.ws.test.client.RequestMatchers.soapHeader;
import static org.springframework.ws.test.client.ResponseCreators.withPayload;

import java.math.BigInteger;
import java.util.List;

import javax.xml.namespace.QName;
import javax.xml.transform.Source;

import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.ws.client.core.WebServiceTemplate;
import org.springframework.ws.test.client.MockWebServiceServer;
import org.springframework.xml.transform.StringSource;

@RunWith(SpringRunner.class)
@SpringBootTest
public class TicketAgentClientTest {

  @Autowired
  private TicketAgentClient ticketAgentClient;

  @Autowired
  private WebServiceTemplate webServiceTemplate;

  private MockWebServiceServer mockWebServiceServer;

  @Before
  public void createServer() {
    mockWebServiceServer = MockWebServiceServer.createServer(webServiceTemplate);
  }

  @Test
  public void testListFlights() {
    Source requestPayload =
        new StringSource("<ns3:listFlightsRequest xmlns:ns3=\"http://example.org/TicketAgent.xsd\">"
            + "</ns3:listFlightsRequest>");

    Source responsePayload =
        new StringSource("<v1:listFlightsResponse xmlns:v1=\"http://example.org/TicketAgent.xsd\">"
            + "<flightNumber>101</flightNumber>" + "</v1:listFlightsResponse>");

    // check if the SOAP Header is present using the soapHeader matcher
    mockWebServiceServer
        .expect(
            soapHeader(new QName("http://example.org/TicketAgent.xsd", "listFlightsSoapHeaders")))
        .andExpect(payload(requestPayload)).andRespond(withPayload(responsePayload));

    List<BigInteger> flights = ticketAgentClient.listFlights();
    assertThat(flights.get(0)).isEqualTo(BigInteger.valueOf(101));

    mockWebServiceServer.verify();
  }
}
```

The endpoint setup is verified by creating a SOAP envelope request that contains the <var>'clientId'</var> SOAP header with a value equals to <var>'abc123'</var>. The response should then contain a list with two flight numbers instead of one.

``` java
package com.codenotfound.ws.endpoint;

import static org.springframework.ws.test.server.RequestCreators.withSoapEnvelope;
import static org.springframework.ws.test.server.ResponseMatchers.payload;

import javax.xml.transform.Source;

import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.context.ApplicationContext;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.ws.test.server.MockWebServiceClient;
import org.springframework.xml.transform.StringSource;

@RunWith(SpringRunner.class)
@SpringBootTest
public class TicketAgentEndpointTest {

  @Autowired
  private ApplicationContext applicationContext;

  private MockWebServiceClient mockClient;

  @Before
  public void createClient() {
    mockClient = MockWebServiceClient.createClient(applicationContext);
  }

  @Test
  public void testListFlights() {
    Source requestEnvelope = new StringSource(
        "<SOAP-ENV:Envelope xmlns:SOAP-ENV=\"http://schemas.xmlsoap.org/soap/envelope/\">"
            + "<SOAP-ENV:Header>"
            + "<ns3:listFlightsSoapHeaders xmlns:ns3=\"http://example.org/TicketAgent.xsd\">"
            + "<isGoldClubMember>true</isGoldClubMember>" + "<clientId>abc123</clientId>"
            + "</ns3:listFlightsSoapHeaders>" + "</SOAP-ENV:Header>" + "<SOAP-ENV:Body>"
            + "<ns3:listFlightsRequest xmlns:ns3=\"http://example.org/TicketAgent.xsd\">"
            + "</ns3:listFlightsRequest>" + "</SOAP-ENV:Body>" + "</SOAP-ENV:Envelope>");

    Source responsePayload =
        new StringSource("<v1:listFlightsResponse xmlns:v1=\"http://example.org/TicketAgent.xsd\">"
            + "<flightNumber>101</flightNumber>" + "<flightNumber>202</flightNumber>"
            + "</v1:listFlightsResponse>");

    mockClient.sendRequest(withSoapEnvelope(requestEnvelope)).andExpect(payload(responsePayload));
  }
}
```

Run the test cases by executing below Maven command.

``` plaintext
mvn test
```

The outcome should be a successful test run as shown below.

``` plaintext
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.5.9.RELEASE)

10:49:22.426 [main] INFO  c.c.ws.client.TicketAgentClientTest - Starting TicketAgentClientTest on cnf-pc with PID 4268 (started by CodeNotFound in c:\code\spring-ws\spring-ws-soap-header)
10:49:22.429 [main] INFO  c.c.ws.client.TicketAgentClientTest - No active profile set, falling back to default profiles: default
10:49:24.243 [main] INFO  c.c.ws.client.TicketAgentClientTest - Started TicketAgentClientTest in 2.071 seconds (JVM running for 2.716)
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 2.301 sec - in com.codenotfound.ws.client.TicketAgentClientTest
Running com.codenotfound.ws.endpoint.TicketAgentEndpointTest
10:49:24.460 [main] INFO  c.c.ws.endpoint.TicketAgentEndpoint - clientId == abc123
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.043 sec - in com.codenotfound.ws.endpoint.TicketAgentEndpointTest

Results :

Tests run: 2, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 4.612 s
[INFO] Finished at: 2017-07-18T10:49:24+02:00
[INFO] Final Memory: 19M/227M
[INFO] ------------------------------------------------------------------------
```

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-ws/tree/master/spring-ws-soap-header){:target="_blank"}.
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

In this tutorial, we showed how you can add and read a SOAP header using Spring WS.

If you found this example useful or if you have a question you would like to ask, feel free to leave a comment below.
