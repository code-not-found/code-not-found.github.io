---
title: "Spring WS - SOAPAction Header Example"
permalink: /2017/04/spring-ws-soapaction-header-example.html
excerpt: "A detailed step-by-step tutorial on how to set the SOAPAction header using Spring-WS and Spring Boot."
date: 2017-04-24
modified: 2017-04-24
categories: [Spring-WS]
tags: [Client, Endpoint, Example, Header, Maven, SOAPAction, Spring, Spring Boot, Spring Web Services, Spring-WS, Tutorial]
published: false
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/spring-logo.jpg" alt="spring logo" class="logo">
</figure>

According to the SOAP 1.1 specification, the [SOAPAction HTTP header field](https://www.w3.org/TR/2000/NOTE-SOAP-20000508/#_Toc478383528) can be used to indicate the intent of a request. There are no restrictions on the format and a client MUST use this header field when sending a SOAP HTTP request.

The below example illustrates how a client can set the SOAPAction header and how an endpoint leverages the `@SoapAction` annotation using Spring-WS, Spring Boot and Maven. 

Tools used:
* Spring-WS 2.4
* Spring Boot 1.5
* Maven 3.5

# General Project Setup

The setup of the project is based on a previous [Spring WS example]({{ site.url }}/2016/10/spring-ws-soap-web-service-consumer-provider-wsdl-example.html) in which we have swapped out the basic <var>helloworld.wsdl</var> for a more generic <var>ticketagent.wsdl</var> from the [W3C WSDL 1.1 specification](https://www.w3.org/TR/wsdl11elementidentifiers/#Iri-ref-ex).

``` xml

```

# Client Set SoapAction 

spring ws soapaction empty 

# Endpoint @SoapAction Annotation 

 the given soapaction does not match an operation 


---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-ws/tree/master/spring-ws-helloworld).
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This Spring WS example turned out a bit longer than expected but hopefully it helped to explain the core client and endpoint concepts.

Feel free to leave a comment if you enjoyed reading or in case you have any additional questions.
