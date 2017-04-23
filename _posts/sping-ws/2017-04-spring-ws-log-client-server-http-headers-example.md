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

Contrary to a framework like for example [Apache CXF](http://cxf.apache.org/), [Spring Web Services](http://projects.spring.io/spring-ws/) does not provide out-of-the box logging of HTTP headers. Reason for this is that Spring-WS tries to be transport-agnostic and as such only ships with [logging at SOAP message level](http://docs.spring.io/spring-ws/docs/current/reference/htmlsingle/#logging).

It is however still possible to log the client or server HTTP headers by creating a custom `Interceptor` which offers the possibility to add common pre- and postprocessing behavior without the need of modifying the payload handling code. The following example shows how to log the HTTP headers of messages that are being sent/received using Spring-WS, Spring Boot and Maven.

Tools used:
* Spring-WS 2.4
* Spring Boot 1.5
* Maven 3.5

# General Project Setup

The setup of the project is based on a previous [Spring Web Services example]({{ site.url }}/2016/10/spring-ws-soap-web-service-consumer-provider-wsdl-example.html) in which we have swapped out the basic <var>helloworld.wsdl</var> for a more generic <var>ticketagent.wsdl</var> from the W3C WSDL specification.








---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-ws/tree/master/spring-ws-helloworld).
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This Spring WS example turned out a bit longer than expected but hopefully it helped to explain the core client and endpoint concepts.

Feel free to leave a comment if you enjoyed reading or in case you have any additional questions.
