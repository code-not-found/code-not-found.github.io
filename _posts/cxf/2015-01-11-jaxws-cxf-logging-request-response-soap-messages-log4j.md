---
title: JAX-WS - CXF logging request and response SOAP messages using Log4j
permalink: /2015/01/jaxws-cxf-logging-request-response-soap-messages-log4j.html
excerpt: A code sample which shows how to configure CXF to log the request and response SOAP messages using Log4j.
date: 2015-01-11 21:00
categories: [Apache CXF]
tags: [Apache CXF, Code Sample, CXF, JAX-WS, Jetty, log4j, logging, Maven, SOAP message]
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/apache-cxf-logo.png" alt="apache cxf logo">
</figure>

CXF uses Java SE Logging for both client- and server-side logging of SOAP requests and responses. Logging is activated by use of separate in/out interceptors that can be attached to the requester and/or provider as required. These interceptors can be specified either programmatically (via Java code and/or annotations) or via use of configuration files. The following code sample shows how to configure CXF interceptors using Log4j for the Hello World web service from a previous post.

Tools used:
* CXF 3.0
* Spring 4.1
* Log4j 1.2
* Jetty 8
* Maven 3

Specifying the interceptors via configuration files offers two benefits over programmatic configuration:
1. Logging requirements **can be altered without needing to recompile the code**.
2. **No Apache CXF-specific APIs need to be added to your code**, which helps it remain interoperable with other JAX-WS compliant web service stacks.











