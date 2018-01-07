---
title: "Spring WS - Log Client Server HTTP Headers Example"
permalink: /spring-ws-log-client-server-http-headers.html
excerpt: "A detailed step-by-step tutorial on how to log the client and server HTTP headers using Spring-WS and Spring Boot."
date: 2017-04-23
modified: 2017-04-23
header:
  teaser: "assets/images/teaser/spring-ws-teaser.png"
categories: [Spring-WS]
tags: [Client, Endpoint, Example, Headers, HTTP, Log, Logging, Maven, Spring, Spring Boot, Spring Web Services, Spring-WS, Tutorial]
redirect_from:
  - /2017/04/log-client-server-http-headers.html
  - /2017/04/spring-ws-log-client-server-http-headers.html
published: true
---

<figure>
    <img src="{{ site.url }}/assets/images/logo/spring-logo.png" alt="spring logo" class="logo">
</figure>

[Spring Web Services](http://projects.spring.io/spring-ws/){:target="_blank"}, contrary to a framework like for example [Apache CXF](http://cxf.apache.org/){:target="_blank"}, does not provide out-of-the box logging of HTTP headers. Reason for this is that Spring-WS tries to be transport-agnostic and as such only ships with [logging at SOAP message level](https://docs.spring.io/spring-ws/docs/2.4.2.RELEASE/reference/#logging){:target="_blank"}.

It is however still possible to log the client and server HTTP headers by creating a custom `Interceptor` which offers the possibility to add common pre- and postprocessing behavior without the need of modifying the core payload handling code.

The following example shows how to log the HTTP headers of messages that are being sent/received using Spring-WS, Spring Boot, and Maven.

If you want to learn more about Spring WS - head on over to the [Spring WS tutorials page]({{ site.url }}/spring-ws/).
{: .notice--primary}

# General Project Setup

Tools used:
* Spring-WS 2.4
* Spring Boot 1.5
* Maven 3.5

The setup of the project is based on a previous [Spring Web Services example]({{ site.url }}/spring-ws-soap-web-service-consumer-provider-wsdl-example.html) in which we have swapped out the basic <var>helloworld.wsdl</var> for a more generic <var>ticketagent.wsdl</var> from the [W3C WSDL 1.1 specification](https://www.w3.org/TR/wsdl11elementidentifiers/#Iri-ref-ex){:target="_blank"}.

In this example, we will get access to the HTTP headers by using the `writeTo()` method of the `WebServiceMessage` interface. This method writes the entire message to the given output stream and if the given stream is an instance of `TransportOutputStream`, [the corresponding headers will be written as well](http://docs.spring.io/spring-ws/site/apidocs/org/springframework/ws/WebServiceMessage.html#writeTo(java.io.OutputStream)){:target="_blank"}.

So the first thing to do is to extend the abstract `TransportOutputStream` class as there is no public implementation available that we can use. We implement the `addHeader()` method which writes a header that is being added to the `ByteArrayOutputStream`. In addition we also complete the `createOutputStream()` method with logic to create or reuse the class's `byteArrayOutputStream` variable.

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

Next, we create a small `HttpLoggingUtils` utility class that contains a single static `logMessage()` method that will be called from our custom client and endpoint interceptors.

The method takes as input a `WebServiceMessage` from which the content is written to above `ByteArrayTransportOutputStream` class. As we have extended `TransportOutputStream` the `writeTo()` method will output both the message and the HTTP headers. We then simply format the log message and pass it to our `LOGGER`.

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

We start with the client were we create a `LogHttpHeaderClientInterceptor` by implementing the `ClientInterceptor` interface. We call the `logMessage()` method of our `HttpLoggingUtils` utility class in the `handleRequest()` and `handleResponse()` methods. These are responsible for processing the outgoing request message and incoming response message respectively.

> Note that `true` needs to be returned on each handle method otherwise the processing is interrupted.

``` java
package com.codenotfound.ws.interceptor;

import org.springframework.ws.client.WebServiceClientException;
import org.springframework.ws.client.support.interceptor.ClientInterceptor;
import org.springframework.ws.context.MessageContext;

public class LogHttpHeaderClientInterceptor implements ClientInterceptor {

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

In order to enable the `LogHttpHeaderClientInterceptor` we define it on the `WebServiceTemplate`, using the `setInterceptors()` method.

``` java
package com.codenotfound.ws.client;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.oxm.jaxb.Jaxb2Marshaller;
import org.springframework.ws.client.core.WebServiceTemplate;
import org.springframework.ws.client.support.interceptor.ClientInterceptor;

import com.codenotfound.ws.interceptor.LogHttpHeaderClientInterceptor;

@Configuration
public class ClientConfig {

  @Value("${client.default-uri}")
  private String defaultUri;

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
    webServiceTemplate.setDefaultUri(defaultUri);

    // register the LogHttpHeaderClientInterceptor
    ClientInterceptor[] interceptors =
        new ClientInterceptor[] {new LogHttpHeaderClientInterceptor()};
    webServiceTemplate.setInterceptors(interceptors);

    return webServiceTemplate;
  }
}
```

# Adding Server HTTP Header Logging

For the server-side we create an `LogHttpHeaderEndpointInterceptor` which implements the `EndpointInterceptor` interface. We add the `logMessage()` method to the request and response processing flows respectively.

> Note that `true` needs to be returned on each handle method otherwise the processing is interrupted. 

``` java
package com.codenotfound.ws.interceptor;

import org.springframework.ws.context.MessageContext;
import org.springframework.ws.server.EndpointInterceptor;

public class LogHttpHeaderEndpointInterceptor implements EndpointInterceptor {

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

Now that our custom `EndpointInterceptor` is ready we need to tell Spring WS to use it. We do this by overriding the `addInterceptors()` method of the `WsConfigurerAdapter`. Simply add a new instance of the `LogHttpHeaderEndpointInterceptor` to the `interceptors` list in order to activate it.

``` java
package com.codenotfound.ws.endpoint;

import java.util.List;

import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.ClassPathResource;
import org.springframework.ws.config.annotation.EnableWs;
import org.springframework.ws.config.annotation.WsConfigurerAdapter;
import org.springframework.ws.server.EndpointInterceptor;
import org.springframework.ws.transport.http.MessageDispatcherServlet;
import org.springframework.ws.wsdl.wsdl11.SimpleWsdl11Definition;
import org.springframework.ws.wsdl.wsdl11.Wsdl11Definition;

import com.codenotfound.ws.interceptor.CustomEndpointInterceptor;

@EnableWs
@Configuration
public class WebServiceConfig extends WsConfigurerAdapter {

  @Bean
  public ServletRegistrationBean messageDispatcherServlet(ApplicationContext applicationContext) {

    MessageDispatcherServlet servlet = new MessageDispatcherServlet();
    servlet.setApplicationContext(applicationContext);

    return new ServletRegistrationBean(servlet, "/codenotfound/ws/*");
  }

  @Bean(name = "ticketagent")
  public Wsdl11Definition defaultWsdl11Definition() {
    SimpleWsdl11Definition wsdl11Definition = new SimpleWsdl11Definition();
    wsdl11Definition.setWsdl(new ClassPathResource("/wsdl/ticketagent.wsdl"));

    return wsdl11Definition;
  }

  @Override
  public void addInterceptors(List<EndpointInterceptor> interceptors) {
    // register the CustomEndpointInterceptor
    interceptors.add(new LogHttpHeaderEndpointInterceptor());
  }
}
```

# Testing the Logging of the HTTP Headers

Now that the interceptors are setup, let's use Maven to trigger the included unit test case in which the client makes a web service call to the endpoint.

``` plaintext
mvn test
```

The result is that both request and response messages are logged twice (once for the client and once for the server). And for all of them, the HTTP headers are included as shown below.

``` plaintext
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.5.9.RELEASE)

07:18:16.117 [main] INFO  c.c.ws.SpringWsApplicationTests - Starting SpringWsApplicationTests on cnf-pc with PID 2176 (started by CodeNotFound in c:\code\spring-ws\spring-ws-log-http-headers)
07:18:16.120 [main] INFO  c.c.ws.SpringWsApplicationTests - No active profile set, falling back to default profiles: default
07:18:18.729 [main] INFO  c.c.ws.SpringWsApplicationTests - Started SpringWsApplicationTests in 2.93 seconds (JVM running for 3.634)
07:18:18.962 [main] INFO  c.c.ws.interceptor.HttpLoggingUtils -
----------------------------
Client Request Message
----------------------------
Accept: text/xml, text/html, image/gif, image/jpeg, *; q=.2, */*; q=.2
SOAPAction: ""
Content-Type: text/xml; charset=utf-8
Content-Length: 219
<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/"><SOAP-ENV:Header/><SOAP-ENV:Body><ns3:listFlightsRequest xmlns:ns3="http://example.org/TicketAgent.xsd"/></SOAP-ENV:Body></SOAP-ENV:Envelope>

07:18:19.051 [http-nio-9090-exec-1] INFO  c.c.ws.interceptor.HttpLoggingUtils -
----------------------------
Server Request Message
----------------------------
accept-encoding: gzip
accept: text/xml
accept: text/html
accept: image/gif
accept: image/jpeg
accept: *; q=.2
accept: */*; q=.2
soapaction: ""
content-type: text/xml; charset=utf-8
cache-control: no-cache
pragma: no-cache
user-agent: Java/1.8.0_152
host: localhost:9090
connection: keep-alive
content-length: 219
<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/"><SOAP-ENV:Header/><SOAP-ENV:Body><ns3:listFlightsRequest xmlns:ns3="http://example.org/TicketAgent.xsd"/></SOAP-ENV:Body></SOAP-ENV:Envelope>

07:18:19.072 [http-nio-9090-exec-1] INFO  c.c.ws.interceptor.HttpLoggingUtils -
----------------------------
Server Response Message
----------------------------
Accept: text/xml, text/html, image/gif, image/jpeg, *; q=.2, */*; q=.2
SOAPAction: ""
Content-Type: text/xml; charset=utf-8
Content-Length: 277
<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/"><SOAP-ENV:Header/><SOAP-ENV:Body><ns3:listFlightsResponse xmlns:ns3="http://example.org/TicketAgent.xsd"><flightNumber>101</flightNumber></ns3:listFlightsResponse></SOAP-ENV:Body></SOAP-ENV:Envelope>

07:18:19.086 [main] INFO  c.c.ws.interceptor.HttpLoggingUtils -
----------------------------
Client Response Message
----------------------------
SOAPAction: ""
Accept: text/xml
Accept: text/html
Accept: image/gif
Accept: image/jpeg
Accept: *; q=.2
Accept: */*; q=.2
Content-Length: 277
Date: Sun
Date: 10 Dec 2017 06:18:19 GMT
Content-Type: text/xml; charset=utf-8
<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/"><SOAP-ENV:Header/><SOAP-ENV:Body><ns3:listFlightsResponse xmlns:ns3="http://example.org/TicketAgent.xsd"><flightNumber>101</flightNumber></ns3:listFlightsResponse></SOAP-ENV:Body></SOAP-ENV:Envelope>

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 3.397 sec - in com.codenotfound.ws.SpringWsApplicationTests

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 7.520 s
[INFO] Finished at: 2017-12-10T07:18:19+01:00
[INFO] Final Memory: 30M/276M
[INFO] ------------------------------------------------------------------------
```

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-ws/tree/master/spring-ws-log-http-headers){:target="_blank"}.
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

I created this post based on a [StackOverFlow question and answer](http://stackoverflow.com/q/28380277/4201470){:target="_blank"} on logging outgoing HTTP requests using Spring-WS.

Hopefully, it will help you out during the testing/debugging of your Spring-WS project.
