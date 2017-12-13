---
title: "Spring JMS - ActiveMQ Consumer Producer Example"
permalink: /spring-jms-activemq-consumer-producer-example.html
excerpt: "A detailed step-by-step tutorial on how to implement an ActiveMQ Consumer and Producer using Spring JMS and Spring Boot."
date: 2017-05-04
modified: 2017-05-04
header:
  teaser: "assets/images/header/spring-jms-teaser.png"
categories: [Spring JMS]
tags: [ActiveMQ, Apache ActiveMQ, Consumer, Example, Hello World, Maven, Producer, Spring, Spring Boot, Spring JMS, Tutorial]
redirect_from:
  - /2017/05/spring-jms-activemq-example.html
  - /2017/05/spring-jms-activemq-consumer-producer-example.html
published: true
---

<figure>
    <img src="{{ site.url }}/assets/images/logo/spring-logo.png" alt="spring logo" class="logo">
</figure>

Spring provides a [JMS integration framework](https://docs.spring.io/spring/docs/4.3.13.RELEASE/spring-framework-reference/html/jms.html){:target="_blank"} that simplifies the use of the JMS API much like Spring's integration does for the JDBC API. The Spring Framework will take care of some low-level details when working with the [JMS API](http://docs.oracle.com/javaee/6/tutorial/doc/bncdr.html){:target="_blank"}.

The below tutorial illustrates how to build and run a Hello World example in which we will send/receive messages to/from Apache ActiveMQ using Spring JMS, Spring Boot and Maven.

# General Project Setup

If you want to learn more about Spring JMS - head on over to the [Spring JMS tutorials page]({{ site.url }}/spring-jms/).
{: .notice--primary}

Tools used:
  * ActiveMQ 5.14
  * Spring JMS 4.3
  * Spring Boot 1.5
  * Maven 3.5

We will be building and running our example using [Apache Maven](https://maven.apache.org/){:target="_blank"}. Shown below is the XML representation of our Maven project in a POM file. It contains the needed dependencies for compiling and running our example.

In order to run the example we will use the [Spring Boot](https://projects.spring.io/spring-boot/){:target="_blank"} project that makes it easy to create stand-alone, production-grade Spring based Applications. To facilitate the management of the different Spring dependencies, [Spring Boot Starters](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters){:target="_blank"} are used which are a set of convenient dependency descriptors that you can include in your application.

The `spring-boot-starter-activemq` dependency includes the needed dependencies for using Spring JMS in combination with ActiveMQ. The `spring-boot-starter-test` includes the dependencies for testing Spring Boot applications with libraries that include [JUnit](http://junit.org/junit4/){:target="_blank"}, [Hamcrest](http://hamcrest.org/JavaHamcrest/){:target="_blank"} and [Mockito](http://site.mockito.org/){:target="_blank"}.

To avoid having to manage the version compatibility of the different Spring dependencies, we will inherit the defaults from the `spring-boot-starter-parent` parent POM.

A dependency to `activemq-junit` is also added as we will include a basic unit test case that verifies our setup using an embedded ActiveMQ broker. The version of the dependency needs to match with the ActiveMQ version supported by the `spring-boot-starter-activemq` starter. For [Spring Boot 1.5.9 the ActiveMQ dependency was version '5.14.5'](https://docs.spring.io/spring-boot/docs/1.5.9.RELEASE/reference/html/appendix-dependency-versions.html#appendix-dependency-versions).

> You can find back the Spring Boot dependencies in Appendix F of the reference documentation.

In the plugins section, we included the `spring-boot-maven-plugin` Maven plugin so that we can build a single, runnable "uber-jar". This will also allow us to start the example via a Maven command.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>spring-jms-activemq-helloworld</artifactId>
  <version>0.0.1-SNAPSHOT</version>

  <name>spring-jms-activemq-helloworld</name>
  <description>Spring JMS - ActiveMQ Consumer Producer Example</description>
  <url>https://www.codenotfound.com/spring-jms-activemq-consumer-producer-example.html</url>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.9.RELEASE</version>
  </parent>

  <properties>
    <java.version>1.8</java.version>
    <activemq.version>5.14.5</activemq.version>
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
    <!-- activemq -->
    <dependency>
      <groupId>org.apache.activemq.tooling</groupId>
      <artifactId>activemq-junit</artifactId>
      <version>${activemq.version}</version>
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

Spring Boot is used in order to make a Spring JMS example application that you can "just run".

We start by creating an `SpringJmsApplication` which contains the `main()` method that uses Spring Boot's `SpringApplication.run()` method to launch the application. The `@SpringBootApplication` annotation is a convenience annotation that adds: `@Configuration`, `@EnableAutoConfiguration` and `@ComponentScan`.

For more information on Spring Boot check out the [Spring Boot getting started guide](https://spring.io/guides/gs/spring-boot/){:target="_blank"}.

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

> The below sections will detail how to create a sender and receiver together with their respective configurations. Note that it is also possible to have [Spring Boot autoconfigure Spring JMS]({{ site.url }}/spring-jms-activemq-boot-example.html) using default values so that actual code that needs to be written is reduced to a bare minimum.

# Create a Spring JMS Message Producer

For sending messages we will be using the `JmsTemplate` which requires a reference to a `ConnectionFactory` and provides convenience methods which handles the creation and release of resources when sending or synchronously receiving messages.

> Note that instances of the JmsTemplate class are thread-safe once configured.

In the below `Sender` class, the `JmsTemplate` is autowired as the actual creation of the `Bean` will be done in a separate `SenderConfig` class.

In this example we will use the `convertAndSend()` method which sends the given object to the specified destination, converting the object to a JMS message. The [type of JMS message]({{ site.url }}/2014/01/jms-message-structure-overview.html#jms-message-body) depends on the type of the object being passed. In the case of a `String` a JMS `TextMessage` will be created.

``` java
package com.codenotfound.jms.producer;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jms.core.JmsTemplate;

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

The creation of the `JmsTemplate` and `Sender` is handled in the `SenderConfig` class. This class is annoted with `@Configuration` which indicates that the class can be used by the Spring IoC container as a source of bean definitions.

In order to be able to use the Spring JMS template we need to provide a reference to a `ConnectionFactory` which is used to [create connections with the JMS provider](http://docs.oracle.com/javaee/6/tutorial/doc/bnceh.html){:target="_blank"}. In addition it encapsulates various configuration parameters, many of which are vendor specific. In the case of ActiveMQ we use the `ActiveMQConnectionFactory`.

On the `ActiveMQConnectionFactory` we set the broker URL which is fetched from the <var>application.yml</var> properties file using the `@Value` annotation.

``` yaml
activemq:
  broker-url: tcp://localhost:61616
```

The `JmsTemplate` was originally designed to be used in combination with a J2EE container where the container would provide the necessary pooling of the JMS resources. As we are running this example on Spring Boot, we will wrap `ActiveMQConnectionFactory` using Spring's `CachingConnectionFactory` in order to still have the benefit of caching of sessions, connections and producers as well as automatic connection recovery.

``` java
package com.codenotfound.jms.producer;

import org.apache.activemq.ActiveMQConnectionFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.jms.connection.CachingConnectionFactory;
import org.springframework.jms.core.JmsTemplate;

@Configuration
public class SenderConfig {

  @Value("${activemq.broker-url}")
  private String brokerUrl;

  @Bean
  public ActiveMQConnectionFactory activeMQConnectionFactory() {
    ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory();
    activeMQConnectionFactory.setBrokerURL(brokerUrl);

    return activeMQConnectionFactory;
  }

  @Bean
  public CachingConnectionFactory cachingConnectionFactory() {
    return new CachingConnectionFactory(activeMQConnectionFactory());
  }

  @Bean
  public JmsTemplate jmsTemplate() {
    return new JmsTemplate(cachingConnectionFactory());
  }

  @Bean
  public Sender sender() {
    return new Sender();
  }
}
```

# Create a Spring JMS Message Consumer

Like with any messaging-based application, you need to create a receiver that will handle the messages that have been sent. The below `Receiver` is nothing more than a simple POJO that defines a method for receiving messages. In this example we named the method `receive()`, but you can name it anything you like.

The `@JmsListener` annotation creates a message listener container behind the scenes for each annotated method, using a `JmsListenerContainerFactory`. By default, a bean with name <var>'jmsListenerContainerFactory'</var> is expected that we will setup in the next section.

Using the `destination` element, we specify the destination for this listener. In the below example we load the destination <var>'helloworld.q'</var> from the <var>application.yml</var> properties file. This is done using the '${   }' placeholder which Spring will automatically resolve.

``` yaml
queue:
  helloworld: helloworld.q
```

> For testing convenience, we added a `CountDownLatch`. This allows the POJO to signal that a message is received. This is something you are not likely to implement in a production application. 

``` java
package com.codenotfound.jms.consumer;

import java.util.concurrent.CountDownLatch;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.jms.annotation.JmsListener;

public class Receiver {

  private static final Logger LOGGER = LoggerFactory.getLogger(Receiver.class);

  private CountDownLatch latch = new CountDownLatch(1);

  public CountDownLatch getLatch() {
    return latch;
  }

  @JmsListener(destination = "${activemq.queue.helloworld}")
  public void receive(String message) {
    LOGGER.info("received message='{}'", message);
    latch.countDown();
  }
}
```

The creation and configuration of the different Spring Beans needed for the `Receiver` POJO are grouped in the `ReceiverConfig` class. Note that we need to add the `@EnableJms` annotation to enable support for the `@JmsListener` annotation that was used on the `Receiver`.

The `jmsListenerContainerFactory()` is expected by the `@JmsListener` annotation from the `Receiver`. We set the concurrency between 3 and 10. This means that listener container will always hold on to the minimum number of consumers and will slowly scale up to the maximum number of consumers in case of increasing load.

Contrary to the `JmsTemplate` [ideally don't use Spring's CachingConnectionFactory with a message listener container at all](http://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/jms/listener/DefaultMessageListenerContainer.html){:target="_blank"}. Reason for this is that it is generally preferable to let the listener container itself handle appropriate caching within its lifecycle.

As we are connecting to ActiveMQ, an `ActiveMQConnectionFactory` is created and passed in the constructor of the `CachingConnectionFactory`.

``` java
package com.codenotfound.jms.consumer;

import org.apache.activemq.ActiveMQConnectionFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.jms.annotation.EnableJms;
import org.springframework.jms.config.DefaultJmsListenerContainerFactory;

@Configuration
@EnableJms
public class ReceiverConfig {

  @Value("${activemq.broker-url}")
  private String brokerUrl;

  @Bean
  public ActiveMQConnectionFactory activeMQConnectionFactory() {
    ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory();
    activeMQConnectionFactory.setBrokerURL(brokerUrl);

    return activeMQConnectionFactory;
  }

  @Bean
  public DefaultJmsListenerContainerFactory jmsListenerContainerFactory() {
    DefaultJmsListenerContainerFactory factory = new DefaultJmsListenerContainerFactory();
    factory.setConnectionFactory(activeMQConnectionFactory());
    factory.setConcurrency("3-10");

    return factory;
  }

  @Bean
  public Receiver receiver() {
    return new Receiver();
  }
}
```

# Testing the Spring JMS Template & Listener

In order to verify that we are able to send and receive a message to and from ActiveMQ, a basic `SpringJmsApplicationTest` test case is used. It contains a `testReceive()` unit test case that uses the `Sender` to send a message to the <var>'helloworld.q'</var> queue on the ActiveMQ message broker. We then use the `CountDownLatch` from the `Receiver` to verify that a message was received.

An embedded ActiveMQ broker is automatically started by using an [EmbeddedActiveMQBroker JUnit Rule](http://activemq.apache.org/how-to-unit-test-jms-code.html#HowToUnitTestJMSCode-UsingTheEmbeddedActiveMQBrokerJUnitRule(ActiveMQ5.13)){:target="_blank"}. We have added a dedicated <var>application.yml</var> properties file under <var>src/test/resources</var> that contains the VM URI to connect with the broker.

> Note that as the embedded broker gets shutdown once the unit test cases are finished, we need to stop our `Sender` and `Receiver` before this happens in order to avoid connection errors. This is done by calling a `close()` on the `ApplicationContext` using the `@AfterClass` annotation.

Below test case can also be executed after you [install Apache ActiveMQ]({{ site.url }}/2014/01/jms-apache-activemq-installation.html) on your local system. You need to comment out the lines annotated with `@ClassRule` to avoid the embedded broker gets created. In addition you need to change the <var>'activemq:broker-url'</var> property to point to <var>'tcp://localhost:61616'</var> in case the broker is running on the default URL value.

``` java
package com.codenotfound.jms;

import static org.assertj.core.api.Assertions.assertThat;

import java.util.concurrent.TimeUnit;

import org.apache.activemq.junit.EmbeddedActiveMQBroker;
import org.junit.AfterClass;
import org.junit.ClassRule;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.test.context.junit4.SpringRunner;

import com.codenotfound.jms.consumer.Receiver;
import com.codenotfound.jms.producer.Sender;

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringJmsApplicationTest {

  private static ApplicationContext applicationContext;

  @Autowired
  void setContext(ApplicationContext applicationContext) {
    SpringJmsApplicationTest.applicationContext = applicationContext;
  }

  @AfterClass
  public static void afterClass() {
    ((ConfigurableApplicationContext) applicationContext).close();
  }

  @ClassRule
  public static EmbeddedActiveMQBroker broker = new EmbeddedActiveMQBroker();

  @Autowired
  private Sender sender;

  @Autowired
  private Receiver receiver;

  @Test
  public void testReceive() throws Exception {
    sender.send("helloworld.q", "Hello Spring JMS ActiveMQ!");

    receiver.getLatch().await(10000, TimeUnit.MILLISECONDS);
    assertThat(receiver.getLatch().getCount()).isEqualTo(0);
  }
}
```

In order to execute above example open a command prompt and run following Maven command: 

``` plaintext
mvn test
```

Maven will download the dependencies, compile the code and run the unit test case. The result should be a successful build as shown below:

``` plaintext
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.5.9.RELEASE)

21:26:38.419 [main] INFO  c.c.jms.SpringJmsApplicationTest - Starting SpringJmsApplicationTest on cnf-pc with PID 5124 (started by CodeNotFound in c:\code\spring-jms\spring-jms-activemq-helloworld)
21:26:38.419 [main] INFO  c.c.jms.SpringJmsApplicationTest - No active profile set, falling back to default profiles: default
21:26:39.187 [main] INFO  c.c.jms.SpringJmsApplicationTest - Started SpringJmsApplicationTest in 1.025 seconds (JVM running for 2.021)
21:26:39.233 [main] INFO  com.codenotfound.jms.producer.Sender - sending message='Hello Spring JMS ActiveMQ!' to destination='helloworld.q'
21:26:39.253 [DefaultMessageListenerContainer-2] INFO  c.codenotfound.jms.consumer.Receiver - received message='Hello Spring JMS ActiveMQ!'
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 2.541 sec - in com.codenotfound.jms.SpringJmsApplicationTest

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 4.915 s
[INFO] Finished at: 2017-05-04T21:26:40+02:00
[INFO] Final Memory: 17M/226M
[INFO] ------------------------------------------------------------------------
```

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-jms/tree/master/spring-jms-activemq-helloworld){:target="_blank"}.
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This concludes our example in which we used a Spring JMS template to create a producer and Spring JMS listener to create a consumer.

If you found this sample useful or have a question you would like to ask, leave a comment below!
