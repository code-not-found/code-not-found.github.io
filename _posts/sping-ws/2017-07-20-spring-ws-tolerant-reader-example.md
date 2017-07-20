---
title: Spring WS - SOAP Tolerant Reader Example
permalink: /2017/07/spring-ws-soap-tolerant-reader-example.html
excerpt: A detailed step-by-step tutorial on how to implement a soap tolerant reader using Spring-WS and Spring Boot.
date: 2017-07-20
modified: 2017-07-20
header:
  teaser: "assets/images/spring-ws-teaser.jpg"
categories: [Spring-WS]
tags: [Design Pattern, Example, SOAP, Spring, Spring Boot, Spring Web Services, Spring-WS, Tolerant Reader, Tutorial]
published: true
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/spring-logo.jpg" alt="spring logo" class="logo">
</figure>

The [tolerant reader pattern](https://martinfowler.com/bliki/TolerantReader.html){:target="_blank"} was coined by Martin Fowler as a way to reduce the coupling between the consumer and provider of a SOAP web service. The pattern tries to reduce the impact on existing consumers in case the service contract changes.

Fowler highlights three main points when working with XML:
* Only take the elements you need
* Make minimum assumptions about the structure
* Wrap the data payload behind a convenient object

The following example will apply the tolerant reader design pattern to both consumer and provider of a SOAP web service implemented using Spring-WS, Spring Boot, and Maven.

If you want to learn more about Spring WS - head on over to the [Spring WS tutorials page]({{ site.url }}/spring-ws/).
{: .notice--primary}

# General Project Setup

Tools used:
* Spring-WS 2.4
* Spring Boot 1.5
* Maven 3.5

When describing the tolerant reader design pattern, Fowler uses the example of an order history service. As Spring-WS is contract first only, we need to start by creating an Order History service WSDL file. The service has a single <var>'getOrderHistory'</var> operation that takes as input a user ID and returns the full history of that user.

``` xml
<?xml version="1.0"?>
<wsdl:definitions name="OrderHistory"
  targetNamespace="http://codenotfound.com/services/orderhistory"
  xmlns:tns="http://codenotfound.com/services/orderhistory" xmlns:types="http://codenotfound.com/types/orderhistory"
  xmlns:soap="http://schemas.xmlsoap.org/wsdl/soap/" xmlns:wsdl="http://schemas.xmlsoap.org/wsdl/">

  <wsdl:types>
    <xsd:schema targetNamespace="http://codenotfound.com/types/orderhistory"
      xmlns:tns="http://codenotfound.com/types/orderhistory" xmlns:xsd="http://www.w3.org/2001/XMLSchema"
      elementFormDefault="qualified" attributeFormDefault="unqualified"
      version="1.0">

      <xsd:element name="getOrderHistoryRequest">
        <xsd:complexType>
          <xsd:sequence>
            <xsd:element name="userId" type="xsd:string" />
          </xsd:sequence>
        </xsd:complexType>
      </xsd:element>

      <xsd:element name="getOrderHistoryResponse">
        <xsd:complexType>
          <xsd:sequence>
            <xsd:element name="orderHistory" type="types:OrderHistoryType" />
          </xsd:sequence>
        </xsd:complexType>
      </xsd:element>

      <xsd:complexType name="OrderHistoryType">
        <xsd:sequence>
          <xsd:element name="orderList" type="tns:OrderListType" />
        </xsd:sequence>
      </xsd:complexType>

      <xsd:complexType name="OrderListType">
        <xsd:sequence>
          <xsd:element name="order" type="tns:OrderType"
            minOccurs="0" maxOccurs="unbounded" />
        </xsd:sequence>
      </xsd:complexType>

      <xsd:complexType name="OrderType">
        <xsd:sequence>
          <xsd:element name="orderId" type="xsd:string" />
        </xsd:sequence>
      </xsd:complexType>
    </xsd:schema>
  </wsdl:types>

  <wsdl:message name="GetOrderHistoryRequest">
    <wsdl:part name="GetOrderHistoryRequest" element="types:getOrderHistoryRequest" />
  </wsdl:message>

  <wsdl:message name="GetOrderHistoryResponse">
    <wsdl:part name="GetOrderHistoryResponse" element="types:getOrderHistoryResponse" />
  </wsdl:message>

  <wsdl:portType name="OrderHistory_PortType">
    <wsdl:operation name="getOrderHistory">
      <wsdl:input message="tns:GetOrderHistoryRequest" />
      <wsdl:output message="tns:GetOrderHistoryResponse" />
    </wsdl:operation>
  </wsdl:portType>

  <wsdl:binding name="OrderHistory_SoapBinding" type="tns:OrderHistory_PortType">
    <soap:binding style="document"
      transport="http://schemas.xmlsoap.org/soap/http" />
    <wsdl:operation name="getOrderHistory">
      <soap:operation soapAction="http://codenotfound.com/services/getOrderHistory" />
      <wsdl:input>
        <soap:body use="literal" />
      </wsdl:input>
      <wsdl:output>
        <soap:body use="literal" />
      </wsdl:output>
    </wsdl:operation>
  </wsdl:binding>

  <wsdl:service name="OrderHistory_Service">
    <wsdl:documentation>Order History service</wsdl:documentation>
    <wsdl:port name="OrderHistory_Port" binding="tns:OrderHistory_SoapBinding">
      <soap:address location="http://localhost:9090/codenotfound/ws/orderhistory" />
    </wsdl:port>
  </wsdl:service>

</wsdl:definitions>
```

The main setup of the project is based on a previous [Spring WS step by step example]({{ site.url }}/2016/10/spring-ws-soap-web-service-consumer-provider-wsdl-example.html). As such we won't go into details the basic configuration of Spring-WS in combination with Spring Boot and Maven.

# Wrap the Data Payload Behind a Convenient Object

We will first create two simple objects that will wrap the received order history. This will reduce coupling with the rest of the application code and make it shield it from future changes to the order history service.

First object is a simple `Order` [POJO](https://en.wikipedia.org/wiki/Plain_old_Java_object){:target="_blank"} that contains an order id.

``` java
package com.codenotfound.ws.model;

public class Order {

  private String orderId;

  public Order(String orderId) {
    this.orderId = orderId;
  }

  public String getOrderId() {
    return orderId;
  }

  public void setOrderId(String orderId) {
    this.orderId = orderId;
  }

  @Override
  public String toString() {
    return "Order[orderId=" + orderId + "]";
  }
}
```

The second `OrderHistory` class wraps the above `Order` in a list.

``` java
package com.codenotfound.ws.model;

import java.util.List;

public class OrderHistory {

  private List<Order> orders;

  public List<Order> getOrders() {
    return orders;
  }

  public void setOrders(List<Order> orders) {
    this.orders = orders;
  }
}
```

# Make Minimum Assumptions About the Structure

As we are working with XML we can use [XPath](https://en.wikipedia.org/wiki/XPath){:target="_blank"}'s recursive descent operator in order to search for a specified element. This way we can extract the orders from the response message without specifying the full path. This way our client will not break in case the service provider changes the location of the orders in the XML. We will also illustrate this with a unit test case further below.

Not only will we apply this principle to extract the orders but also the needed attributes from an order (in this example the order id) are obtained using XPath. Both queries are defined in the <var>application.yml</var> as shown below.

In order to select the needed element we use the <var>'local-name()'</var> function which ignores the namespace and returns the query results as if the XML did not have any namespace. This way of working increases the tolerance in case the namespace of the response would change (for example if it contains a version number).

> Note the use of the "." (dot) in the second expression as we want to search for the orderId in the [current document](https://www.w3schools.com/xml/xpath_syntax.asp){:target="_blank"}.

``` yaml
client:
  default-uri: http://localhost:9090/codenotfound/ws/orderhistory
  xpath:
    order: //*[local-name()='order']
    order-id: .//*[local-name()='orderId']

server:
  port: 9090
```

We will apply the above-defined XPath queries using `XPathExpression` from Spring-WS. This is an abstraction over a [compiled XPath expression](http://docs.spring.io/spring-ws/docs/current/reference/htmlsingle/#xpath-expression){:target="_blank"}.

The expressions are first loaded via the `@Value` annotation and are then used to create the <var>'orderIdXPath'</var> and <var>'orderIdXPath'</var> `XPathExpression` beans.

```
package com.codenotfound.ws.client;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.oxm.jaxb.Jaxb2Marshaller;
import org.springframework.ws.client.core.WebServiceTemplate;
import org.springframework.xml.xpath.XPathExpression;
import org.springframework.xml.xpath.XPathExpressionFactory;

@Configuration
public class ClientConfig {

  @Value("${client.default-uri}")
  private String defaultUri;

  @Value("${client.xpath.order}")
  private String orderXPath;

  @Value("${client.xpath.order-id}")
  private String orderIdXPath;

  @Bean
  public XPathExpression orderXPath() {
    return XPathExpressionFactory.createXPathExpression(orderXPath);
  }

  @Bean
  public XPathExpression orderIdXPath() {
    return XPathExpressionFactory.createXPathExpression(orderIdXPath);
  }

  @Bean
  Jaxb2Marshaller jaxb2Marshaller() {
    Jaxb2Marshaller jaxb2Marshaller = new Jaxb2Marshaller();
    jaxb2Marshaller.setContextPath("com.codenotfound.types.orderhistory");

    return jaxb2Marshaller;
  }

  @Bean
  public WebServiceTemplate webServiceTemplate() {
    WebServiceTemplate webServiceTemplate = new WebServiceTemplate();
    webServiceTemplate.setMarshaller(jaxb2Marshaller());
    webServiceTemplate.setUnmarshaller(jaxb2Marshaller());
    webServiceTemplate.setDefaultUri(defaultUri);

    return webServiceTemplate;
  }
}
```

# Only Take the Elements You Need

Now that we have created our XPath expressions we will use them in the client code. First, we create the request message that contains the user ID for which the order history needs to be retrieved. Note that for the creation we can use the JAXB generated objects as the tolerant reader pattern only applies to messages that are read (received).

As we want to apply the `XPathExpression`s we need to use the `sendSourceAndReceiveToResult()` method on the `WebServiceTemplate` which will provide us with a `Result`. We get the `Source` from our JAXB `GetOrderHistoryRequest` object by marshaling it using the `MarshallingUtils` provided by Spring-WS.

Once we have received the request we only fetch the orders and ignore everything else. For each order, we use a `NodeMapper` to perform the actual work of mapping each node to a corresponding `Order` object. Again we only take the <var>'orderId'</var> element from each node and we ignore the rest.

``` java
package com.codenotfound.ws.client;

import java.io.IOException;
import java.util.List;

import javax.xml.transform.dom.DOMResult;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import org.springframework.ws.WebServiceMessage;
import org.springframework.ws.client.core.WebServiceTemplate;
import org.springframework.ws.support.MarshallingUtils;
import org.springframework.xml.xpath.NodeMapper;
import org.springframework.xml.xpath.XPathExpression;
import org.w3c.dom.Node;

import com.codenotfound.types.orderhistory.GetOrderHistoryRequest;
import com.codenotfound.types.orderhistory.ObjectFactory;
import com.codenotfound.ws.model.Order;
import com.codenotfound.ws.model.OrderHistory;

@Component
public class OrderHistoryClient {

  private static final Logger LOGGER = LoggerFactory.getLogger(OrderHistoryClient.class);

  @Autowired
  private XPathExpression orderXPath;
  @Autowired
  private XPathExpression orderIdXPath;

  @Autowired
  private WebServiceTemplate webServiceTemplate;

  public OrderHistory getOrderHistory(String userId) throws IOException {
    // create the request
    ObjectFactory factory = new ObjectFactory();
    GetOrderHistoryRequest getOrderHistoryRequest = factory.createGetOrderHistoryRequest();
    getOrderHistoryRequest.setUserId(userId);

    // marshal the request
    WebServiceMessage request = webServiceTemplate.getMessageFactory().createWebServiceMessage();
    MarshallingUtils.marshal(webServiceTemplate.getMarshaller(), getOrderHistoryRequest, request);

    // call the service
    DOMResult responseResult = new DOMResult();
    webServiceTemplate.sendSourceAndReceiveToResult(request.getPayloadSource(), responseResult);

    // extract the needed elements
    List<Order> orders = orderXPath.evaluate(responseResult.getNode(), new NodeMapper<Order>() {

      @Override
      public Order mapNode(Node node, int nodeNum) {
        // get the orderId
        String orderId = orderIdXPath.evaluateAsString(node);
        // create an order
        Order order = new Order(orderId);
        LOGGER.info("found " + order.toString());

        return order;
      }
    });

    OrderHistory result = new OrderHistory();
    result.setOrders(orders);

    return result;
  }
}
```

For the implementation of the `Endpoint`, we will also apply the above principles to the received messages.

By annotating a parameter of the handling method with `@XPathParam` we can bind it to the [evaluation of an XPath expression](http://docs.spring.io/spring-ws/docs/current/reference/htmlsingle/#server-xpath-param){:target="_blank"}. In other words we only need to specify the elements we need as parameters of the handling method annotated with `@XPathParam`. This way we only get what we need with a minimal assumption on the received structure.

In the `OrderHistoryEndpoint` we define the XPath expression to extract the user ID and then define it as a parameter of the `getOrderHistory()` handling method. In this tutorial we simply log the received <var>'userId'</var> and then in a real-life example typically a database lookup would be performed to retrieve the user's orders. In the below code we simply generate a fix response that contains three orders.

``` java
package com.codenotfound.ws.endpoint;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.ws.server.endpoint.annotation.Endpoint;
import org.springframework.ws.server.endpoint.annotation.Namespace;
import org.springframework.ws.server.endpoint.annotation.PayloadRoot;
import org.springframework.ws.server.endpoint.annotation.ResponsePayload;
import org.springframework.ws.server.endpoint.annotation.XPathParam;

import com.codenotfound.types.orderhistory.GetOrderHistoryResponse;
import com.codenotfound.types.orderhistory.ObjectFactory;
import com.codenotfound.types.orderhistory.OrderHistoryType;
import com.codenotfound.types.orderhistory.OrderListType;
import com.codenotfound.types.orderhistory.OrderType;

@Endpoint
public class OrderHistoryEndpoint {

  private static final Logger LOGGER = LoggerFactory.getLogger(OrderHistoryEndpoint.class);

  private static final String userIdXPath = "//*[local-name()='userId']";

  @PayloadRoot(namespace = "http://codenotfound.com/types/orderhistory",
      localPart = "getOrderHistoryRequest")
  @Namespace(prefix = "oh", uri = "http://codenotfound.com/types/orderhistory")
  @ResponsePayload
  public GetOrderHistoryResponse getOrderHistory(@XPathParam(userIdXPath) String userId) {
    LOGGER.info("received request for order history of userId={}", userId);

    // fetch the order history for the received user
    ObjectFactory factory = new ObjectFactory();
    OrderListType orderListType = factory.createOrderListType();

    for (int i = 0; i < 3; i++) {
      OrderType orderType = factory.createOrderType();
      orderType.setOrderId("order" + i);
      orderListType.getOrder().add(orderType);
    }

    OrderHistoryType orderHistoryType = factory.createOrderHistoryType();
    orderHistoryType.setOrderList(orderListType);

    GetOrderHistoryResponse result = factory.createGetOrderHistoryResponse();
    result.setOrderHistory(orderHistoryType);

    return result;
  }
}
```

# Testing the Tolerant Reader Setup

In order to test whether our example complies with the tolerant reader principles, we will be using the `MockWebServiceServer` and `MockWebServiceClient` classes provided by Spring-WS as these allow us to freely define the XML messages that are sent/received to/from our client and server implementations.

`OrderHistoryClientTest` contains a number of unit tests for the client. The first `testGetOrderHistory()` test case verifies the correct working of the client code.

The second `testGetOrderHistoryOnlyNeededElements()` test case simulates the scenario in which the service has evolved. The orders that are being returned now contain an <var>'orderName'</var> element in addition to the <var>'orderId'</var> element. As we are only extracting the elements that we need, our client code should not break.

Finally, the `testGetOrderHistoryMinimumAssumptions()` test case illustrates that even when the order history is moved into a <var>'newWrapper'</var> element, the client code continues to work as it searches for the orders as opposed to using an absolute path.

``` java
package com.codenotfound.ws.client;

import static org.assertj.core.api.Assertions.assertThat;
import static org.springframework.ws.test.client.RequestMatchers.payload;
import static org.springframework.ws.test.client.ResponseCreators.withPayload;

import java.io.IOException;

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

import com.codenotfound.ws.model.OrderHistory;

@RunWith(SpringRunner.class)
@SpringBootTest
public class OrderHistoryClientTest {

  @Autowired
  private OrderHistoryClient orderHistoryClient;

  @Autowired
  private WebServiceTemplate webServiceTemplate;

  private MockWebServiceServer mockWebServiceServer;

  @Before
  public void createServer() {
    mockWebServiceServer = MockWebServiceServer.createServer(webServiceTemplate);
  }

  @Test
  public void testGetOrderHistory() throws IOException {
    Source requestPayload = new StringSource(
        "<ns1:getOrderHistoryRequest xmlns:ns1=\"http://codenotfound.com/types/orderhistory\">"
            + "<ns1:userId>abc123</ns1:userId>" + "</ns1:getOrderHistoryRequest>");

    Source responsePayload = new StringSource(
        "<ns1:getOrderHistoryResponse xmlns:ns1=\"http://codenotfound.com/types/orderhistory\">"
            + "<ns1:orderHistory>" + "<ns1:orderList>"
            + "<ns1:order><ns1:orderId>order1</ns1:orderId></ns1:order>"
            + "<ns1:order><ns1:orderId>order2</ns1:orderId></ns1:order>"
            + "<ns1:order><ns1:orderId>order3</ns1:orderId></ns1:order>" + "</ns1:orderList>"
            + "</ns1:orderHistory>" + "</ns1:getOrderHistoryResponse>");

    mockWebServiceServer.expect(payload(requestPayload)).andRespond(withPayload(responsePayload));

    OrderHistory orderHistory = orderHistoryClient.getOrderHistory("abc123");
    assertThat(orderHistory.getOrders().get(2).getOrderId()).isEqualTo("order3");

    mockWebServiceServer.verify();
  }

  @Test
  public void testGetOrderHistoryOnlyNeededElements() throws IOException {
    Source requestPayload = new StringSource(
        "<ns1:getOrderHistoryRequest xmlns:ns1=\"http://codenotfound.com/types/orderhistory\">"
            + "<ns1:userId>def456</ns1:userId>" + "</ns1:getOrderHistoryRequest>");

    Source responsePayload = new StringSource(
        "<ns1:getOrderHistoryResponse xmlns:ns1=\"http://codenotfound.com/types/orderhistory\">"
            + "<ns1:orderHistory>" + "<ns1:orderList>"
            + "<ns1:order><ns1:orderId>order4</ns1:orderId><ns1:orderName>order-name-1</ns1:orderName></ns1:order>"
            + "<ns1:order><ns1:orderId>order5</ns1:orderId><ns1:orderName>order-name-2</ns1:orderName></ns1:order>"
            + "<ns1:order><ns1:orderId>order6</ns1:orderId><ns1:orderName>order-name-3</ns1:orderName></ns1:order>"
            + "</ns1:orderList>" + "</ns1:orderHistory>" + "</ns1:getOrderHistoryResponse>");

    mockWebServiceServer.expect(payload(requestPayload)).andRespond(withPayload(responsePayload));

    OrderHistory orderHistory = orderHistoryClient.getOrderHistory("def456");
    assertThat(orderHistory.getOrders().get(2).getOrderId()).isEqualTo("order6");

    mockWebServiceServer.verify();
  }

  @Test
  public void testGetOrderHistoryMinimumAssumptions() throws IOException {
    Source requestPayload = new StringSource(
        "<ns1:getOrderHistoryRequest xmlns:ns1=\"http://codenotfound.com/types/orderhistory\">"
            + "<ns1:userId>ghi789</ns1:userId>" + "</ns1:getOrderHistoryRequest>");

    Source responsePayload = new StringSource(
        "<ns1:getOrderHistoryResponse xmlns:ns1=\"http://codenotfound.com/types/orderhistory\">"
            + "<ns1:anotherElement>" + "<ns1:orderHistory>" + "<ns1:orderList>"
            + "<ns1:order><ns1:orderId>order7</ns1:orderId></ns1:order>"
            + "<ns1:order><ns1:orderId>order8</ns1:orderId></ns1:order>"
            + "<ns1:order><ns1:orderId>order9</ns1:orderId></ns1:order>" + "</ns1:orderList>"
            + "</ns1:orderHistory>" + "</ns1:anotherElement>" + "</ns1:getOrderHistoryResponse>");

    mockWebServiceServer.expect(payload(requestPayload)).andRespond(withPayload(responsePayload));

    OrderHistory orderHistory = orderHistoryClient.getOrderHistory("ghi789");
    assertThat(orderHistory.getOrders().get(2).getOrderId()).isEqualTo("order9");

    mockWebServiceServer.verify();
  }
}
```

For test the server part, the `TicketAgentEndpointTest` will be used. Again the first `testGetOrderHistory()` test case verifies the correct working of the service endpoint.

The second `testGetOrderHistoryOnlyNeededElements()` test case illustrates the scenario in which a client is sending additional information which is no longer supported by the service. If the server is able to extract the information needed to execute the request (in this case the <var>'userId'</var> element), it simply ignores the <var>'userName'</var> element that is no longer supported.

A third `testGetOrderHistoryMinimumAssumptions()` test case simulates the scenario where the service definition has been updated. The <var>'userId'</var> element is no longer wrapped in an <var>'oldWrapper'</var> element. The older clients don't need to change as the endpoint is able to handle the requests of both old and new service version because there is no assumption on the absolute location of the <var>'userId'</var> element in the received XML.

``` java
package com.codenotfound.ws.endpoint;

import static org.springframework.ws.test.server.RequestCreators.withPayload;
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
  public void testGetOrderHistory() {
    Source requestPayload = new StringSource(
        "<ns1:getOrderHistoryRequest xmlns:ns1=\"http://codenotfound.com/types/orderhistory\">"
            + "<ns1:userId>jkl123</ns1:userId>" + "</ns1:getOrderHistoryRequest>");

    Source responsePayload = new StringSource(
        "<ns2:getOrderHistoryResponse xmlns:ns2=\"http://codenotfound.com/types/orderhistory\">"
            + "<ns2:orderHistory>" + "<ns2:orderList>"
            + "<ns2:order><ns2:orderId>order0</ns2:orderId></ns2:order>"
            + "<ns2:order><ns2:orderId>order1</ns2:orderId></ns2:order>"
            + "<ns2:order><ns2:orderId>order2</ns2:orderId></ns2:order>" + "</ns2:orderList>"
            + "</ns2:orderHistory>" + "</ns2:getOrderHistoryResponse>");

    mockClient.sendRequest(withPayload(requestPayload)).andExpect(payload(responsePayload));
  }

  @Test
  public void testGetOrderHistoryOnlyNeededElements() {
    Source requestPayload = new StringSource(
        "<ns1:getOrderHistoryRequest xmlns:ns1=\"http://codenotfound.com/types/orderhistory\">"
            + "<ns1:userId>mno123</ns1:userId>" + "<ns1:userName>user-name</ns1:userName>"
            + "</ns1:getOrderHistoryRequest>");

    Source responsePayload = new StringSource(
        "<ns2:getOrderHistoryResponse xmlns:ns2=\"http://codenotfound.com/types/orderhistory\">"
            + "<ns2:orderHistory>" + "<ns2:orderList>"
            + "<ns2:order><ns2:orderId>order0</ns2:orderId></ns2:order>"
            + "<ns2:order><ns2:orderId>order1</ns2:orderId></ns2:order>"
            + "<ns2:order><ns2:orderId>order2</ns2:orderId></ns2:order>" + "</ns2:orderList>"
            + "</ns2:orderHistory>" + "</ns2:getOrderHistoryResponse>");

    mockClient.sendRequest(withPayload(requestPayload)).andExpect(payload(responsePayload));
  }

  @Test
  public void testGetOrderHistoryMinimumAssumptions() {
    Source requestPayload = new StringSource(
        "<ns1:getOrderHistoryRequest xmlns:ns1=\"http://codenotfound.com/types/orderhistory\">"
            + "<ns1:anotherElement>" + "<ns1:userId>pqr123</ns1:userId>" + "</ns1:anotherElement>"
            + "</ns1:getOrderHistoryRequest>");

    Source responsePayload = new StringSource(
        "<ns2:getOrderHistoryResponse xmlns:ns2=\"http://codenotfound.com/types/orderhistory\">"
            + "<ns2:orderHistory>" + "<ns2:orderList>"
            + "<ns2:order><ns2:orderId>order0</ns2:orderId></ns2:order>"
            + "<ns2:order><ns2:orderId>order1</ns2:orderId></ns2:order>"
            + "<ns2:order><ns2:orderId>order2</ns2:orderId></ns2:order>" + "</ns2:orderList>"
            + "</ns2:orderHistory>" + "</ns2:getOrderHistoryResponse>");

    mockClient.sendRequest(withPayload(requestPayload)).andExpect(payload(responsePayload));
  }
}
```

In order to run the example, open up a command prompt and execute following Maven command:

``` plaintext
mvn test
```

This should result in a successful test run as shown below.

``` plaintext
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.5.4.RELEASE)

22:22:50.714 [main] INFO  c.c.ws.client.OrderHistoryClientTest - Starting OrderHistoryClientTest on cnf-pc with PID 3132 (started by CodeNotFound in c:\codenotfound\spring-ws\spring-ws-tolerant-reader)
22:22:50.717 [main] INFO  c.c.ws.client.OrderHistoryClientTest - No active profile set, falling back to default profiles: default
22:22:52.495 [main] INFO  c.c.ws.client.OrderHistoryClientTest - Started OrderHistoryClientTest in 2.033 seconds (JVM running for 2.716)
22:22:52.611 [main] INFO  c.c.ws.client.OrderHistoryClient - found '3' orders for userId='abc123'
22:22:52.654 [main] INFO  c.c.ws.client.OrderHistoryClient - found '3' orders for userId='ghi789'
22:22:52.663 [main] INFO  c.c.ws.client.OrderHistoryClient - found '3' orders for userId='def456'
Tests run: 3, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 2.268 sec - in com.codenotfound.ws.client.OrderHistoryClientTest
Running com.codenotfound.ws.endpoint.TicketAgentEndpointTest
22:22:52.694 [main] INFO  c.c.ws.endpoint.OrderHistoryEndpoint - received request for order history of userId='jkl123'
22:22:52.711 [main] INFO  c.c.ws.endpoint.OrderHistoryEndpoint - received request for order history of userId='pqr123'
22:22:52.721 [main] INFO  c.c.ws.endpoint.OrderHistoryEndpoint - received request for order history of userId='mno123'
Tests run: 3, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.037 sec - in com.codenotfound.ws.endpoint.TicketAgentEndpointTest

Results :

Tests run: 6, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 4.647 s
[INFO] Finished at: 2017-07-19T22:22:52+02:00
[INFO] Final Memory: 19M/222M
[INFO] ------------------------------------------------------------------------
```

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-ws/tree/master/spring-ws-tolerant-reader).
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

In this tutorial, we illustrated the tolerant reader pattern using a concrete example and then implemented it in Java using the Spring Boot microservices framework in combination with Spring WS.

Any other techniques you know of on how to make a SOAP service consumer or provider more tolerant? Let us know by leaving a comment below!
