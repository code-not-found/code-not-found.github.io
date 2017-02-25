---
title: JAX-WS - CXF Log Request/Response SOAP Messages Logback
permalink: /2014/09/jaxws-cxf-log-request-response-soap-messages-logback.html
excerpt: A detailed example on how to setup CXF to log request and response SOAP messages using the Logback logging framework.
date: 2014-09-21 21:00
categories: [Apache CXF]
tags: [Apache CXF, CXF, Feature, Interceptor, JAX-WS, Log, logback, logging, Messages, Request, Response, SOAP]
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/apache-cxf-logo.png" alt="apache cxf logo">
</figure>

CXF uses Java SE Logging for both client- and server-side logging of SOAP requests and responses. Logging is activated by use of separate in/out interceptors that can be attached to the requester and/or provider as required. These interceptors can be specified either programmatically (via Java code and/or annotations) or via use of configuration files.

The following tutorial shows how to configure CXF logging interceptors using Logback for the [contract first Hello World web service example]({{ site.url }}/2014/08/jaxws-cxf-contract-first-hello-world-jetty-maven.html) from a previous post.

Tools used:
* CXF 3.1
* Spring 4.3
* Logback 1.1
* Jetty 9
* Maven 3

Specifying the interceptors via configuration files offers two benefits over programmatic configuration:
1. Logging requirements **can be altered without needing to recompile** the code
2. **No Apache CXF-specific APIs need to be added** to your code, which helps it remain interoperable with other JAX-WS compliant web service stacks

For this example [Logback](http://logback.qos.ch/) will be used which is a successor to the [Log4j](https://logging.apache.org/log4j/1.2/) project. As a best practice the CXF `java.util.logging` calls will first be redirected to [SLF4J (Simple Logging Facade for Java)](http://www.slf4j.org/) as described [here](https://cxf.apache.org/docs/general-cxf-logging.html#GeneralCXFLogging-UsingSLF4JInsteadofjava.util.logging(since2.2.8)). In other words a <var>META-INF/cxf/org.apache.cxf.Logger</var> file needs to be created on the classpath containing the following:

``` plaintext
org.apache.cxf.common.logging.Slf4jLogger
```

As the Hello World example uses Spring, the commons-logging calls from the Spring framework will also be redirected to SLF4J using jcl-over-slf4j. Now that all logging calls of both CXF and Spring are redirected to SLF4J, Logback will receive them automatically as it natively implements SLF4J. The picture below illustrates the described approach. 































