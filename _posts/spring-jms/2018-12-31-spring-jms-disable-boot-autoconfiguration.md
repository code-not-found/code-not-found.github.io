---
title: "Spring JMS - ActiveMQ Boot Example"
permalink: /2017/05/spring-jms-activemq-boot-example.html
excerpt: "A detailed step-by-step tutorial on how to autoconfigure ActiveMQ and Spring JMS using Spring Boot."
date: 2017-05-08
modified: 2017-05-08
header:
  teaser: "assets/images/teaser/spring-jms-teaser.jpg"
categories: [Spring JMS]
tags: [Autoconfig, Autoconfiguration, ActiveMQ, Apache ActiveMQ, Example, Maven, Spring, Spring Boot, Spring JMS, Tutorial]
published: false
---

<figure>
    <img src="{{ site.url }}/assets/images/logo/spring-logo.png" alt="spring logo" class="logo">
</figure>

[Spring Boot auto-configuration](http://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-auto-configuration.html) will try to automatically configure your Spring application based on the JAR dependencies that are available. In other words if the `spring-jms` and `activemq-broker` dependencies are on the classpath and you have not manually configured any `ConnectionFactory`, `JmsTemplate` or `JmsListenerContainerFactory` beans, then Spring Boot will auto-configure them for you using default values.

# General Project Setup

If you want to learn more about Spring JMS - head on over to the [Spring JMS tutorials page]({{ site.url }}/spring-jms/).
{: .notice--primary}

Tools used:
* ActiveMQ 5.14
* Spring JMS 4.3
* Spring Boot 1.5
* Maven 3.5

To illustrate this behavior we will start from a previous [Spring JMS tutorial]({{ site.url }}/2017/05/spring-jms-activemq-consumer-producer-example.html) in which we send/receive messages to/from an Apache ActiveMQ destination using Spring JMS. The original code will be reduced to a bare minimum in order to demonstrate Spring Boot's autoconfiguration capabilities.

# Disable JMS Autoconfiguration

If you want Spring Boot to skip the autoconfiguring of ActiveMQ, you can do so by adding the `exclude` parameter to the `@SpringBootApplication` (or the `@EnableAutoConfiguration` annotation) annotation. Specify the `ActiveMQAutoConfiguration` class as shown below and JMS autoconfiguration for ActiveMQ is turned off.

``` java
package com.codenotfound.jms;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.autoconfigure.jms.activemq.ActiveMQAutoConfiguration;

@SpringBootApplication(exclude = {ActiveMQAutoConfiguration.class})
public class SpringJmsApplication {

  public static void main(String[] args) {
    SpringApplication.run(SpringJmsApplication.class, args);
  }
}
```

> Note that if Spring Boot finds any beans of type `ConnectionFactory`, the entire `ActiveMQAutoConfiguration` will be switched off.

In this example we are able to autoconfigure JMS using Spring Boot and a couple of lines of code.

Drop a comment in case you thought the example was helpful or if you found something was missing.
