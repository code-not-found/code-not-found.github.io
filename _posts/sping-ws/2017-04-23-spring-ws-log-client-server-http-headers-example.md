---
title: "Spring WS - Log Client Server HTTP Headers Example"
permalink: /2017/04/log-client-server-http-headers.html
excerpt: "A detailed step-by-step tutorial on how to log the client and server HTTP headers using Spring-WS and Spring Boot."
date: 2017-04-23
modified: 2017-04-23
categories: [Spring-WS]
tags: [Client, Consumer, Endpoint, Example, Headers, HTTP, Log, Logging, Maven, Provider, Spring, Spring Boot, Spring Web Services, Spring-WS, Tutorial]
published: true
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/spring-logo.jpg" alt="spring logo" class="logo">
</figure>

[Spring Web Services](http://projects.spring.io/spring-ws/), contrary to a framework like for example [Apache CXF](http://cxf.apache.org/), does not provide out-of-the box logging of HTTP headers. Reason for this is that Spring-WS tries to be transport-agnostic and as such only ships with [logging at SOAP message level](http://docs.spring.io/spring-ws/docs/current/reference/htmlsingle/#logging).

It is however still possible to log the client and server HTTP headers by creating a custom `Interceptor` which offers the possibility to add common pre- and postprocessing behavior without the need of modifying the core payload handling code.

The following example shows how to log the HTTP headers of messages that are being sent/received using Spring-WS, Spring Boot and Maven.

Tools used:
* Spring-WS 2.4
* Spring Boot 1.5
* Maven 3.5

# General Project Setup

The setup of the project is based on a previous [Spring Web Services example]({{ site.url }}/2016/10/spring-ws-soap-web-service-consumer-provider-wsdl-example.html) in which we have swapped out the basic <var>helloworld.wsdl</var> for a more generic <var>ticketagent.wsdl</var> from the [W3C WSDL 1.1 specification](https://www.w3.org/TR/wsdl11elementidentifiers/#Iri-ref-ex).

In this example we will get access to the HTTP headers by using the `writeTo()` method from the `WebServiceMessage` interface. This method writes the entire message to the given output stream and if the given stream is an instance of `TransportOutputStream`, [the corresponding headers will be written as well](http://docs.spring.io/spring-ws/site/apidocs/org/springframework/ws/WebServiceMessage.html#writeTo(java.io.OutputStream)).

So first thing to do is to extend the abstract `TransportOutputStream` class as there is no public implementation available that we could use. We implement the `addHeader()` method which writes a header that is being added to the `ByteArrayOutputStream`. In addition we also complete the `createOutputStream()` method with logic to create or reuse the classes `ByteArrayOutputStream` variable.

``` java
package com.codenotfound.ws.interceptor;

import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.io.OutputStream;

import org.springframework.ws.transport.TransportOutputStream;

public class ByteArrayTransportOutputStream extends TransportOutputStream {

  private static final String NEW_LINE = System.getProperty("line.separator");

  private ByteArrayOutputStream byteArrayOutputStream;

  @Override
  public void addHeader(String name, String value) throws IOException {
    createOutputStream();
    String header = name + ": " + value + NEW_LINE;
    byteArrayOutputStream.write(header.getBytes());
  }

  @Override
  protected OutputStream createOutputStream() throws IOException {
    if (byteArrayOutputStream == null) {
      byteArrayOutputStream = new ByteArrayOutputStream();
    }
    return byteArrayOutputStream;
  }

  public byte[] toByteArray() {
    return byteArrayOutputStream.toByteArray();
  }
}
```

Next we create a small `HttpLoggingUtils` utility class that contains a single static `logMessage()` method that will be called from our custom client and endpoint interceptors.

The method takes as input a `WebServiceMessage` from which the content is written to our `ByteArrayTransportOutputStream` class. As we have extended `TransportOutputStream` the `writeTo()` method will output both the message and the HTTP headers. We then simply format the log message and pass it to our `LOGGER`.

``` java
package com.codenotfound.ws.interceptor;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.ws.WebServiceMessage;
import org.springframework.xml.transform.TransformerObjectSupport;

public class HttpLoggingUtils extends TransformerObjectSupport {

  private static final Logger LOGGER = LoggerFactory.getLogger(HttpLoggingUtils.class);

  private static final String NEW_LINE = System.getProperty("line.separator");

  private HttpLoggingUtils() {}

  public static void logMessage(String id, WebServiceMessage webServiceMessage) {
    try {
      ByteArrayTransportOutputStream byteArrayTransportOutputStream =
          new ByteArrayTransportOutputStream();
      webServiceMessage.writeTo(byteArrayTransportOutputStream);

      String httpMessage = new String(byteArrayTransportOutputStream.toByteArray());
      LOGGER.info(NEW_LINE + "----------------------------" + NEW_LINE + id + NEW_LINE
          + "----------------------------" + NEW_LINE + httpMessage + NEW_LINE);
    } catch (Exception e) {
      LOGGER.error("Unable to log HTTP message.", e);
    }
  }
}
```

# Adding Client HTTP Header Logging

We create a `CustomClientInterceptor` by implementing the `ClientInterceptor` interface. We call the `logMessage()` method of our `HttpLoggingUtils` utility class in the `handleRequest()` and `handleResponse()` methods. These are responsible for processing the outgoing request message and incoming response message respectively.

``` java
package com.codenotfound.ws.interceptor;

import org.springframework.ws.client.WebServiceClientException;
import org.springframework.ws.client.support.interceptor.ClientInterceptor;
import org.springframework.ws.context.MessageContext;

public class CustomClientInterceptor implements ClientInterceptor {

  @Override
  public void afterCompletion(MessageContext arg0, Exception arg1)
      throws WebServiceClientException {
    // No-op
  }

  @Override
  public boolean handleFault(MessageContext messageContext) throws WebServiceClientException {
    // No-op
    return true;
  }

  @Override
  public boolean handleRequest(MessageContext messageContext) throws WebServiceClientException {
    HttpLoggingUtils.logMessage("Client Request Message", messageContext.getRequest());

    return true;
  }

  @Override
  public boolean handleResponse(MessageContext messageContext) throws WebServiceClientException {
    HttpLoggingUtils.logMessage("Client Response Message", messageContext.getResponse());

    return true;
  }
}
```

In order to enable the `CustomClientInterceptor` we define it on the `WebServiceTemplate`, using the `setInterceptors()` method.

``` java
package com.codenotfound.ws.client;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.oxm.jaxb.Jaxb2Marshaller;
import org.springframework.ws.client.core.WebServiceTemplate;
import org.springframework.ws.client.support.interceptor.ClientInterceptor;

import com.codenotfound.ws.interceptor.CustomClientInterceptor;

@Configuration
public class ClientConfig {

  @Bean
  Jaxb2Marshaller jaxb2Marshaller() {

    Jaxb2Marshaller jaxb2Marshaller = new Jaxb2Marshaller();
    jaxb2Marshaller.setContextPath("org.example.ticketagent");
    return jaxb2Marshaller;
  }

  @Bean
  public WebServiceTemplate webServiceTemplate() {

    WebServiceTemplate webServiceTemplate = new WebServiceTemplate();
    webServiceTemplate.setMarshaller(jaxb2Marshaller());
    webServiceTemplate.setUnmarshaller(jaxb2Marshaller());
    webServiceTemplate.setDefaultUri("http://localhost:9090/codenotfound/ws/ticketagent");

    // register the CustomEndpointInterceptor
    ClientInterceptor[] interceptors = new ClientInterceptor[] {new CustomClientInterceptor()};
    webServiceTemplate.setInterceptors(interceptors);

    return webServiceTemplate;
  }
}
```

# Adding Server HTTP Header Logging



``` java
package com.codenotfound.ws.interceptor;

import org.springframework.ws.context.MessageContext;
import org.springframework.ws.server.EndpointInterceptor;

public class CustomEndpointInterceptor implements EndpointInterceptor {

  @Override
  public void afterCompletion(MessageContext arg0, Object arg1, Exception arg2) throws Exception {
    // No-op
  }

  @Override
  public boolean handleFault(MessageContext messageContext, Object arg1) throws Exception {
    // No-op
    return true;
  }

  @Override
  public boolean handleRequest(MessageContext messageContext, Object arg1) throws Exception {
    HttpLoggingUtils.logMessage("Server Request Message", messageContext.getRequest());

    return true;
  }

  @Override
  public boolean handleResponse(MessageContext messageContext, Object arg1) throws Exception {
    HttpLoggingUtils.logMessage("Server Response Message", messageContext.getResponse());

    return true;
  }
}
```

#Testing the Logging of the Headers





---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-ws/tree/master/spring-ws-helloworld).
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This Spring WS example turned out a bit longer than expected but hopefully it helped to explain the core client and endpoint concepts.

Feel free to leave a comment if you enjoyed reading or in case you have any additional questions.
