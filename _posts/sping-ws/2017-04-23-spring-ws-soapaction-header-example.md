---
title: "Spring WS - SOAPAction Header Example"
permalink: /2017/04/spring-ws-soapaction-header-example.html
excerpt: "A detailed step-by-step tutorial on how to set a SOAPAction header on a web service request using Spring-WS and Spring Boot."
date: 2017-04-21
modified: 2017-04-21
categories: [Spring-WS]
tags: [Client, Consumer, Endpoint, Example, Header, Maven, Provider, SOAPAction, Spring, Spring Boot, Spring Web Services, Spring-WS, Tutorial]
published: false
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/spring-logo.jpg" alt="spring logo" class="logo">
</figure>

According to the SOAP 1.1 specification, the [SOAPAction HTTP header field](https://www.w3.org/TR/2000/NOTE-SOAP-20000508/#_Toc478383528) can be used to indicate the intent of the SOAP HTTP request. SOAP places no restrictions on the format or specificity of the URI or that it is resolvable. An HTTP client MUST use this header field when issuing a SOAP HTTP Request.

Tools used:
* Spring-WS 2.4
* Spring Boot 1.5
* Maven 3.5

# General Project Setup





---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-ws/tree/master/spring-ws-helloworld).
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This Spring WS example turned out a bit longer than expected but hopefully it helped to explain the core client and endpoint concepts.

Feel free to leave a comment if you enjoyed reading or in case you have any additional questions.
