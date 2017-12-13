---
title: "Spring JMS - ActiveMQ Boot Example"
permalink: /spring-jms-activemq-boot-example.html
excerpt: "A detailed step-by-step tutorial on how to autoconfigure ActiveMQ and Spring JMS using Spring Boot."
date: 2017-05-08
modified: 2017-05-08
header:
  teaser: "assets/images/header/spring-jms-teaser.png"
categories: [Spring JMS]
tags: [Autoconfig, Autoconfiguration, ActiveMQ, Apache ActiveMQ, Example, Maven, Spring, Spring Boot, Spring JMS, Tutorial]
redirect_from:
  - /2017/04/spring-jms-boot-example.html
  - /2017/05/spring-jms-activemq-boot-example.html
published: true
---

<figure>
    <img src="{{ site.url }}/assets/images/logo/spring-logo.png" alt="spring logo" class="logo">
</figure>

[Spring Boot auto-configuration](http://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-auto-configuration.html){:target="_blank"} will try to automatically configure your Spring application based on the JAR dependencies that are available. In other words, if the `spring-jms` and `activemq-broker` dependencies are on the classpath and you have not manually configured any `ConnectionFactory`, `JmsTemplate` or `JmsListenerContainerFactory` beans, then Spring Boot will auto-configure them for you using default values.

To illustrate this behavior we will start from a previous [Spring JMS tutorial]({{ site.url }}/spring-jms-activemq-consumer-producer-example.html) in which we send/receive messages to/from an Apache ActiveMQ destination using Spring JMS. The original code will be reduced to a bare minimum in order to demonstrate Spring Boot's autoconfiguration capabilities.

# General Project Setup

If you want to learn more about Spring JMS - head on over to the [Spring JMS tutorials page]({{ site.url }}/spring-jms/).
{: .notice--primary}

Tools used:
* ActiveMQ 5.14
* Spring JMS 4.3
* Spring Boot 1.5
* Maven 3.5

The example project is managed using [Maven](https://maven.apache.org/){:target="_blank"}. Needed dependencies like [Spring Boot](https://projects.spring.io/spring-boot/){:target="_blank"} and [Spring JMS](http://projects.spring.io/spring-framework/){:target="_blank"} are included by declaring the `spring-boot-starter-activemq` [Spring Boot starter](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters){:target="_blank"} in the POM file as shown below.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>spring-jms-activemq-boot</artifactId>
  <version>0.0.1-SNAPSHOT</version>

  <name>spring-jms-activemq-boot</name>
  <description>Spring JMS - Spring Boot ActiveMQ Example</description>
  <url>https://www.codenotfound.com/spring-jms-activemq-boot-example.html</url>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.9.RELEASE</version>
  </parent>

  <properties>
    <java.version>1.8</java.version>
  </properties>

  <dependencies>
    <!-- spring-boot -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-activemq</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <!-- spring-boot-maven-plugin -->
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>
</project>
```

The `SpringJmsApplication` remains untouched. Note that in order for the auto-configuration to work we need to opt-in by adding the `@EnableAutoConfiguration` or `@SpringBootApplication` (which is same as adding `@Configuration` `@EnableAutoConfiguration` `@ComponentScan`) annotation to one of our `@Configuration` classes.

> Only ever add one `@EnableAutoConfiguration` annotation. It is recommended to add it to your primary `@Configuration` class.

``` java
package com.codenotfound.jms;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringJmsApplication {

  public static void main(String[] args) {
    SpringApplication.run(SpringJmsApplication.class, args);
  }
}
```

# Autoconfigure the Spring JMS Message Producer

The setup and creation of the `JmsTemplate` and `ConnectionFactory` beans are automatically done by Spring Boot. We just need to autowire the `JmsTemplate` and use it in the `send()` method.

> By annotating the `Sender` class with `@Component`, Spring will instantiate this class as a bean that we will use in a below test case. In order for this to work, we also need the `@EnableAutoConfiguration` which was indirectly specified on `SpringKafkaApplication` by using the `@SpringBootApplication` annotation.

``` java
package com.codenotfound.jms.producer;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jms.core.JmsTemplate;
import org.springframework.stereotype.Component;

@Component
public class Sender {

  private static final Logger LOGGER = LoggerFactory.getLogger(Sender.class);

  @Autowired
  private JmsTemplate jmsTemplate;

  public void send(String destination, String message) {
    LOGGER.info("sending message='{}' to destination='{}'", message, destination);
    jmsTemplate.convertAndSend(destination, message);
  }
}
```

# Autoconfigure the Spring Kafka Message Consumer

Similar to the `Sender`, the setup and creation of the `ConnectionFactory` and `JmsListenerContainerFactory` bean is automatically done by Spring Boot. The `@JmsListener` annotation creates a message listener container for the annotated `receive()` method. The destination name is specified using the <var>'${destination.boot}'</var> placeholder for which the value will be automatically fetched from the <var>application.yml</var> properties file.

``` java
package com.codenotfound.jms.consumer;

import java.util.concurrent.CountDownLatch;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.jms.annotation.JmsListener;
import org.springframework.stereotype.Component;

@Component
public class Receiver {

  private static final Logger LOGGER = LoggerFactory.getLogger(Receiver.class);

  private CountDownLatch latch = new CountDownLatch(1);

  public CountDownLatch getLatch() {
    return latch;
  }

  @JmsListener(destination = "${queue.boot}")
  public void receive(String message) {
    LOGGER.info("received message='{}'", message);
    latch.countDown();
  }
}
```

Using application properties we can further fine tune the different settings of the `ConnectionFactory`, `JmsTemplate` and `JmsListenerContainerFactory` beans. Scroll down to <var># ACTIVEMQ</var> and <var># JMS</var> sections in the following link in order to get a [complete overview on all the available ActiveMQ and JMS properties](https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html) 

In this example, we use default values and only specify the destination and broker URL in the included <var>application.yml</var> properties file.

``` yaml
spring:
  activemq:
    broker-url: tcp://localhost:61616

queue:
  boot: boot.q
```

# Testing the Sender and Receiver

In order to verify that our code works, a simple `SpringJmsApplicationTest` test case is used. It contains a `testReceive()` unit test case that uses the `Sender` to send a message to the <var>'boot.q'</var> queue on the ActiveMQ broker. We then use the `CountDownLatch` from the `Receiver` to verify that a message was successfully received.

We include a dedicated <var>application.yml</var> properties file for testing under <var>src/test/resources</var> that does not contain a broker URL. If Spring Boot does not find a broker URL, auto-configuration will automatically start an embedded ActiveMQ broker instance.

``` java
package com.codenotfound.jms;

import static org.assertj.core.api.Assertions.assertThat;

import java.util.concurrent.TimeUnit;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import com.codenotfound.jms.consumer.Receiver;
import com.codenotfound.jms.producer.Sender;

@RunWith(SpringRunner.class)
@SpringBootTest
@DirtiesContext
public class SpringJmsApplicationTest {

  @Autowired
  private Sender sender;

  @Autowired
  private Receiver receiver;

  @Test
  public void testReceive() throws Exception {
    sender.send("boot.q", "Hello Boot!");

    receiver.getLatch().await(10000, TimeUnit.MILLISECONDS);
    assertThat(receiver.getLatch().getCount()).isEqualTo(0);
  }
}
```

Let's run the test case by executing following Maven command at the command prompt: 

``` plaintext
mvn test
```

The test case will be triggered resulting in following log statements:

``` plaintext
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.5.9.RELEASE)

20:44:45.934 [main] INFO  c.c.jms.SpringJmsApplicationTest - Starting SpringJmsApplicationTest on cnf-pc with PID 3756 (started by CodeNotFound in c:\code\spring-jms\spring-jms-activemq-boot)
20:44:45.937 [main] INFO  c.c.jms.SpringJmsApplicationTest - No active profile set, falling back to default profiles: default
20:44:47.073 [main] WARN  o.a.activemq.broker.BrokerService - Temporary Store limit is 51200 mb (current store usage is 0 mb). The data directory: c:\code\st\spring-jms\spring-jms-activemq-boot only has 14451 mb of usable space. - resetting to maximum available disk space: 14451 mb
20:44:47.139 [main] INFO  c.c.jms.SpringJmsApplicationTest - Started SpringJmsApplicationTest in 1.461 seconds (JVM running for 2.271)
20:44:47.183 [main] INFO  com.codenotfound.jms.producer.Sender - sending message='Hello Boot!' to destination='boot.q'
20:44:47.206 [DefaultMessageListenerContainer-1] INFO  c.codenotfound.jms.consumer.Receiver - received message='Hello Boot!'
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 1.633 sec - in com.codenotfound.jms.SpringJmsApplicationTest
20:44:47.260 [DefaultMessageListenerContainer-1] WARN  o.s.j.l.DefaultMessageListenerContainer - Setup of JMS message listener invoker failed for destination 'boot.q' - trying to recover. Cause: peer(vm://localhost#1) stopped.

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 4.263 s
[INFO] Finished at: 2017-05-08T20:44:47+02:00
[INFO] Final Memory: 18M/226M
[INFO] ------------------------------------------------------------------------
```

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-jms/tree/master/spring-jms-activemq-boot){:target="_blank"}.
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

In above example, we were able to autoconfigure JMS using Spring Boot and a couple of lines of code.

Drop a comment in case you thought the example was helpful or if you found something was missing.
