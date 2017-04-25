---
title: "Spring WS - SOAPAction Header Example"
permalink: /2017/04/spring-ws-soapaction-header-example.html
excerpt: "A detailed step-by-step tutorial on how to set the SOAPAction header using Spring-WS and Spring Boot."
date: 2017-04-24
modified: 2017-04-24
categories: [Spring-WS]
tags: [Client, Endpoint, Example, Header, Maven, SOAPAction, Spring, Spring Boot, Spring Web Services, Spring-WS, Tutorial]
published: true
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/spring-logo.jpg" alt="spring logo" class="logo">
</figure>

According to the SOAP 1.1 specification, the [SOAPAction HTTP header field](https://www.w3.org/TR/2000/NOTE-SOAP-20000508/#_Toc478383528) can be used to indicate the intent of a request. There are no restrictions on the format and a client MUST use this header field when sending a SOAP HTTP request.

The below example illustrates how a client can set the SOAPAction header and how a server endpoint can leverage the `@SoapAction` annotation to receive the request using Spring-WS, Spring Boot and Maven. 

Tools used:
* Spring-WS 2.4
* Spring Boot 1.5
* Maven 3.5

# General Project Setup

The setup of the project is based on a previous [Spring WS example]({{ site.url }}/2016/10/spring-ws-soap-web-service-consumer-provider-wsdl-example.html) in which we have swapped out the basic <var>helloworld.wsdl</var> for a more generic <var>ticketagent.wsdl</var> from the [W3C WSDL 1.1 specification](https://www.w3.org/TR/wsdl11elementidentifiers/#Iri-ref-ex).

As the WSDL is missing a SOAPAction we will add it in the context of this tutorial.

> Note that Spring-WS will not automatically extract the SOAPAction value from the WSDL file, it needs to be programmed manually as we will see further below.

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
    </xs:schema>
  </wsdl:types>

  <wsdl:message name="listFlightsRequest">
    <wsdl:part name="body" element="xsTicketAgent:listFlightsRequest" />
  </wsdl:message>

  <wsdl:message name="listFlightsResponse">
    <wsdl:part name="body" element="xsTicketAgent:listFlightsResponse" />
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
      <soap:operation soapAction="http://example.com/TicketAgent/listFlights" />
      <wsdl:input>
        <soap:body parts="body" use="literal" />
      </wsdl:input>
      <wsdl:output>
        <soap:body parts="body" use="literal" />
      </wsdl:output>
    </wsdl:operation>
  </wsdl:binding>
</wsdl:definitions>
```

# Client SoapActionCallback

Spring WS by default sends an empty SOAPAction header. In order to set the value we need to configure it on the `WebServiceTemplate` by passing a `WebServiceMessageCallback` which gives access to the message after it has been created, but before it is sent.

There is a dedicated `SoapActionCallback` class which already implements a `WebServiceMessageCallback` that sets the SOAPAction header. Just pass a new instance to the `WebServiceTemplate` in order to set it up.

Alternatively [you can implement](http://docs.spring.io/spring-ws/docs/2.4.0.RELEASE/reference/htmlsingle/#d5e1912) your own `WebServiceMessageCallback` class and set the SOAPAction on the `SoapMessage` using the `setSoapAction()` method.

``` java
package com.codenotfound.ws.client;

import java.math.BigInteger;
import java.util.List;

import javax.xml.bind.JAXBElement;

import org.example.ticketagent.ObjectFactory;
import org.example.ticketagent.TFlightsResponse;
import org.example.ticketagent.TListFlights;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import org.springframework.ws.client.core.WebServiceTemplate;
import org.springframework.ws.soap.client.core.SoapActionCallback;

@Component
public class TicketAgentClient {

  @Autowired
  private WebServiceTemplate webServiceTemplate;

  @SuppressWarnings("unchecked")
  public List<BigInteger> listFlights() {

    ObjectFactory factory = new ObjectFactory();
    TListFlights tListFlights = factory.createTListFlights();

    JAXBElement<TListFlights> request = factory.createListFlightsRequest(tListFlights);

    // use SoapActionCallback to add the SOAPAction
    JAXBElement<TFlightsResponse> response =
        (JAXBElement<TFlightsResponse>) webServiceTemplate.marshalSendAndReceive(request,
            new SoapActionCallback("http://example.com/TicketAgent/listFlights"));

    return response.getValue().getFlightNumber();
  }
}
```

# Endpoint @SoapAction Annotation

The [endpoint mapping](http://docs.spring.io/spring-ws/docs/2.4.0.RELEASE/reference/htmlsingle/#server-endpoint-mapping) is responsible for mapping incoming messages to appropriate endpoints. The `@SoapAction` annotation marks methods with a particular SOAP Action. Whenever a message comes in which has this SOAPAction header, the method will be invoked.

In this example instead of the `@PayloadRoot` mapping, we will use `@SoapAction` to trigger the `listFlights()` method of our `TicketAgentEndpoint` class. The annotation takes as `value` the SOAPAction string.

The value of the SOAPAction header can be accessed within the endpoint logic by calling the `getSoapAction()` method on the request `SoapMessage`. Just add the `MessageContext` as input parameter in order to retrieve it.

``` java
package com.codenotfound.ws.endpoint;

import java.math.BigInteger;

import javax.xml.bind.JAXBElement;

import org.example.ticketagent.ObjectFactory;
import org.example.ticketagent.TFlightsResponse;
import org.example.ticketagent.TListFlights;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.ws.WebServiceMessage;
import org.springframework.ws.context.MessageContext;
import org.springframework.ws.server.endpoint.annotation.Endpoint;
import org.springframework.ws.server.endpoint.annotation.RequestPayload;
import org.springframework.ws.server.endpoint.annotation.ResponsePayload;
import org.springframework.ws.soap.SoapMessage;
import org.springframework.ws.soap.server.endpoint.annotation.SoapAction;

@Endpoint
public class TicketAgentEndpoint {

  private static final Logger LOGGER = LoggerFactory.getLogger(TicketAgentEndpoint.class);

  // map a message to this endpoint based on the SOAPAction
  @SoapAction(value = "http://example.com/TicketAgent/listFlights")
  @ResponsePayload
  public JAXBElement<TFlightsResponse> listFlights(
      @RequestPayload JAXBElement<TListFlights> request, MessageContext messageContext) {

    // access the SOAPAction value
    WebServiceMessage webServiceMessage = messageContext.getRequest();
    SoapMessage soapMessage = (SoapMessage) webServiceMessage;
    LOGGER.info("SOAPAction: '{}'", soapMessage.getSoapAction());

    ObjectFactory factory = new ObjectFactory();
    TFlightsResponse tFlightsResponse = factory.createTFlightsResponse();
    tFlightsResponse.getFlightNumber().add(BigInteger.valueOf(101));

    return factory.createListFlightsResponse(tFlightsResponse);
  }
}
```

# Testing the SOAPAction Header

Now that we have setup the SOAPAction in both client and server let's write some unit test cases to test the correct working.

For the client we will use a `MockWebServiceServer` in combination with a custom `SoapActionMatcher` in order to verify that the SOAPAction header has been set. Based on following [Stack Overflow example](http://stackoverflow.com/a/31202927/4201470) we implment the `RequestMatcher` inteface and assert that an expected SOAPAction is present.

``` java
package com.codenotfound.ws.client;

import static org.assertj.core.api.Assertions.assertThat;

import java.io.IOException;
import java.net.URI;

import org.springframework.ws.WebServiceMessage;
import org.springframework.ws.soap.SoapMessage;
import org.springframework.ws.soap.support.SoapUtils;
import org.springframework.ws.test.client.RequestMatcher;

public class SoapActionMatcher implements RequestMatcher {

  private final String expectedSoapAction;

  public SoapActionMatcher(String expectedSoapAction) {
    this.expectedSoapAction = SoapUtils.escapeAction(expectedSoapAction);
  }

  @Override
  public void match(URI uri, WebServiceMessage webServiceMessage)
      throws IOException, AssertionError {
    assertThat(webServiceMessage).isInstanceOf(SoapMessage.class);

    SoapMessage soapMessage = (SoapMessage) webServiceMessage;
    assertThat(soapMessage.getSoapAction()).isEqualTo(expectedSoapAction);
  }
}
```

In the test case we add the `SoapActionMatcher` as expected to the `MockWebServiceServer` and check the result by calling the `verify()` method. 

``` java
package com.codenotfound.ws.client;

import static org.assertj.core.api.Assertions.assertThat;
import static org.springframework.ws.test.client.RequestMatchers.payload;
import static org.springframework.ws.test.client.ResponseCreators.withPayload;

import java.math.BigInteger;
import java.util.List;

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

    // check if the SOAPAction is present using the custom matcher
    mockWebServiceServer.expect(new SoapActionMatcher("http://example.com/TicketAgent/listFlights"))
        .andExpect(payload(requestPayload)).andRespond(withPayload(responsePayload));

    List<BigInteger> flights = ticketAgentClient.listFlights();
    assertThat(flights.get(0)).isEqualTo(BigInteger.valueOf(101));

    mockWebServiceServer.verify();
  }
}
```

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-ws/tree/master/spring-ws-soapaction-header).
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This Spring WS example turned out a bit longer than expected but hopefully it helped to explain the core client and endpoint concepts.

Feel free to leave a comment if you enjoyed reading or in case you have any additional questions.
