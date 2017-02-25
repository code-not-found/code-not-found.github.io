---
title: JAX-WS - CXF logging request and response SOAP messages using Log4j2
permalink: /2015/01/jaxws-cxf-logging-request-response-soap-messages-log4j2.html
excerpt: A code sample which shows how to configure CXF to log the request and response SOAP messages using Log4j2.
date: 2015-01-10 21:00
categories: [Apache CXF]
tags: [Apache CXF, Code Sample, CXF, JAX-WS, Jetty, log4j2, logging, Maven, SOAP message]
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/apache-cxf-logo.png" alt="apache cxf logo">
</figure>

CXF uses Java SE Logging for both client- and server-side logging of SOAP requests and responses. Logging is activated by use of separate in/out interceptors that can be attached to the requester and/or provider as required. These interceptors can be specified either programmatically (via Java code and/or annotations) or via use of configuration files. The following code sample shows how to configure CXF interceptors using Log4j2 for the Hello World web service from a previous [post]({{ site.url }}/2014/08/jaxws-cxf-contract-first-hello-world-jetty-maven.html).

Tools used:
* CXF 3.0
* Spring 4.1
* Log4j2 2.1
* Jetty 8
* Maven 3

Specifying the interceptors via configuration files offers two benefits over programmatic configuration:
1. Logging requirements **can be altered without needing to recompile the code**.
2. **No Apache CXF-specific APIs need to be added to your code**, which helps it remain interoperable with other JAX-WS compliant web service stacks.

For this example [Log4j2](https://logging.apache.org/log4j/2.x/) will be used which is an upgrade of the [Log4j](https://logging.apache.org/log4j/1.2/) project. As a best practice the CXF `java.util.logging` calls will first be redirected to [SLF4J](http://www.slf4j.org/) (Simple Logging Facade for Java) as described [here](https://cxf.apache.org/docs/debugging-and-logging.html#DebuggingandLogging-UsingSLF4JInsteadofjava.util.logging%28since2.2.8%29). In other words a <var>META-INF/cxf/org.apache.cxf.Logger</var> file will be created on the classpath containing the following:

``` plaintext
org.apache.cxf.common.logging.Slf4jLogger
```

As the Hello World example uses Spring, the commons-logging calls from the Spring framework will also be redirected to SLF4J using [jcl-over-slf4j](http://www.slf4j.org/legacy.html). Now that all logging calls of both CXF and Spring are redirected to SLF4J, Log4j2 will be plugged into SLF4J using the [Log4j 2 SLF4J Binding](https://logging.apache.org/log4j/2.0/log4j-slf4j-impl/index.html). The picture below illustrates the described approach. 

<figure>
    <img src="{{ site.url }}/assets/images/logos/cxf-logging-using-log4j2.png" alt="cxf logging using log4j2">
</figure>

The below Maven POM file contains the needed dependencies for the SLF4 bridge (<var>'jcl-over-slf4j'</var>), Log4j 2 SLF4J Binding (<var>'log4j-slf4j-impl'</var>) and Log4j2 (<var>'log4j-core'</var>). In addition it contains all other needed dependencies and plugins needed in order to run the example.

``` xml

```


















