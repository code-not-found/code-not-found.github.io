---
title: "Spring JMS ActiveMQ Example"
permalink: /spring-jms-activemq-example.html
excerpt: "A detailed step-by-step tutorial on how to connect to ActiveMQ using Spring JMS and Spring Boot."
date: 2017-05-04
last_modified_at: 2018-12-12
header:
  teaser: "assets/images/spring-jms/spring-jms-activemq.png"
categories: [Spring JMS]
tags: [ActiveMQ, Apache ActiveMQ, Consumer, Example, Hello World, Maven, Producer, Spring, Spring Boot, Spring JMS, Tutorial]
redirect_from:
  - /2017/05/spring-jms-activemq-example.html
  - /2017/05/spring-jms-activemq-consumer-producer-example.html
  - /spring-jms-activemq-consumer-producer-example.html
published: true
---

<img src="{{ site.url }}/assets/images/spring-jms/spring-jms-activemq.png" alt="spring jms activemq" class="align-right title-image">

I'm going to show you exactly how to create a [Spring JMS](https://docs.spring.io/spring/docs/5.1.3.RELEASE/spring-framework-reference/integration.html#jms){:target="_blank"} _Hello World_ example that uses [Spring Boot](https://spring.io/projects/spring-boot){:target="_blank"}, [ActiveMQ](http://activemq.apache.org/){:target="_blank"}, and [Maven](https://maven.apache.org/){:target="_blank"}.

(Step-by-step)

So if you're a Spring JMS beginner, **you'll love this guide**.

Let's get this show on the road!

If you want to learn more about Spring JMS - head on over to the [Spring JMS tutorials]({{ site.url }}/spring-jms-tutorials) page.
{: .notice--primary}

## 1. What is Spring JMS?

Spring provides a [JMS integration framework](https://docs.spring.io/spring/docs/4.3.13.RELEASE/spring-framework-reference/html/jms.html){:target="_blank"} that simplifies the use of the JMS API much like Spring's integration does for the JDBC API.

The Spring Framework will take care of some low-level details when working with the [JMS API](http://docs.oracle.com/javaee/6/tutorial/doc/bncdr.html){:target="_blank"}.

In this tutorial we will create a Hello World example in which we will send/receive a message to/from Apache ActiveMQ using Spring JMS, Spring Boot, and Maven.

## 2. General Project Overview

We will use the following tools/frameworks:
* Spring JMS 5.1
* Spring Boot 2.1
* ActiveMQ 5.14
* Maven 3.5

Our project has the following directory structure:

<img src="{{ site.url }}/assets/images/spring-jms/spring-jms-activemq-hello-world-maven-project.png" alt="spring jms activemq hello world maven project">

## 3. Maven Setup

We build and run our example using **Apache Maven**. Shown below is the XML representation of our Maven project in a POM file. It contains the needed dependencies for compiling and running our example.

In order to run the example we will use the [Spring Boot](https://projects.spring.io/spring-boot/){:target="_blank"} project that makes it easy to create stand-alone, production-grade Spring based Applications.

To facilitate the management of the different Spring dependencies, [Spring Boot Starters](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters){:target="_blank"} are used which are a set of convenient dependency descriptors that you can include in your application.

The `spring-boot-starter-activemq` dependency includes the needed dependencies for using Spring JMS in combination with ActiveMQ.

The `spring-boot-starter-test` includes the dependencies for testing Spring Boot applications with libraries that include [JUnit](http://junit.org/junit4/){:target="_blank"}, [Hamcrest](http://hamcrest.org/JavaHamcrest/){:target="_blank"} and [Mockito](http://site.mockito.org/){:target="_blank"}.

To avoid having to manage the version compatibility of the different Spring dependencies, we will inherit the defaults from the `spring-boot-starter-parent` parent POM.

A dependency to `activemq-junit` is also added as we will include a basic unit test case that verifies our setup using an embedded ActiveMQ broker.

> You can find back the Spring Boot dependency version in Appendix F of the reference documentation. For [Spring Boot 2.1.1 the ActiveMQ dependency was version '5.15.8'](https://docs.spring.io/spring-boot/docs/1.5.9.RELEASE/reference/html/appendix-dependency-versions.html#appendix-dependency-versions){:target="_blank"}.

In the plugins section, we included the `spring-boot-maven-plugin` Maven plugin so that we can build a single, runnable "uber-jar". This will also allow us to start the example via a Maven command.

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>spring-jms-activemq-hello-world</artifactId>
  <version>0.0.1-SNAPSHOT</version>

  <name>spring-jms-activemq-hello-world</name>
  <description>Spring JMS ActiveMQ Example</description>
  <url>https://codenotfound.com/spring-jms-activemq-example.html</url>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.1.RELEASE</version>
    <relativePath /><!-- lookup parent from repository -->
  </parent>

  <properties>
    <java.version>1.8</java.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-activemq</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.apache.activemq.tooling</groupId>
      <artifactId>activemq-junit</artifactId>
      <version>${activemq.version}</version>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>
</project>
{% endhighlight %}

## 4. Spring Boot Setup

Spring Boot is used in order to make a Spring JMS example application that you can "just run".

We start by creating a `SpringJmsApplication` which contains the `main()` method that uses Spring Boot's `SpringApplication.run()` method to launch the application.

> The `@SpringBootApplication` annotation is a convenience annotation that adds: `@Configuration`, `@EnableAutoConfiguration` and `@ComponentScan`.

For more information on Spring Boot check out the [Spring Boot getting started guide](https://spring.io/guides/gs/spring-boot/){:target="_blank"}.

{% highlight java %}
package com.codenotfound;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringJmsApplication {

  public static void main(String[] args) {
    SpringApplication.run(SpringJmsApplication.class, args);
  }
}
{% endhighlight %}

> The below sections will detail how to create a sender and receiver together with their respective configurations. Note that it is also possible to have [Spring Boot autoconfigure Spring JMS]({{ site.url }}/spring-jms-activemq-boot-example.html) using default values so that actual code that needs to be written is reduced to a bare minimum.

# 5. Create a Spring JMS Message Producer

For sending messages we will be using the `JmsTemplate` which requires a reference to a `ConnectionFactory` and provides convenience methods which handle the creation and release of resources when sending or synchronously receiving messages.

> Note that instances of the JmsTemplate class are thread-safe once configured.

In the below `Sender` class, the `JmsTemplate` is autowired as the actual creation of the `Bean` will be done in a separate `SenderConfig` class.

In this example we will use the `convertAndSend()` method which sends the given object to the specified destination, converting the object to a JMS message.

The [type of JMS message]({{ site.url }}/jms-message-types-properties-overview.html#jms-message-body) depends on the type of the object being passed. In the case of a `String`, a JMS `TextMessage` will be created.

{% highlight java %}
package com.codenotfound.jms;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jms.core.JmsTemplate;

public class Sender {

  private static final Logger LOGGER =
      LoggerFactory.getLogger(Sender.class);

  @Autowired
  private JmsTemplate jmsTemplate;

  public void send(String message) {
    LOGGER.info("sending message='{}'", message);
    jmsTemplate.convertAndSend("helloworld.q", message);
  }
}
{% endhighlight %}

The creation of the `JmsTemplate` and `Sender` is handled in the `SenderConfig` class. This class is annotated with `@Configuration` which indicates that the class can be used by the Spring IoC container as a source of bean definitions.

In order to be able to use the Spring JMS template, we need to provide a reference to a `ConnectionFactory` which is used to [create connections with the JMS provider](http://docs.oracle.com/javaee/6/tutorial/doc/bnceh.html){:target="_blank"}. In addition, it encapsulates various configuration parameters, many of which are vendor specific. In the case of ActiveMQ, we use the `ActiveMQConnectionFactory`.

On the `ActiveMQConnectionFactory` we set the broker URL which is fetched from the <var>application.yml</var> properties file using the `@Value` annotation.

{% highlight yaml %}
activemq:
  broker-url: tcp://localhost:61616
{% endhighlight %}

The `JmsTemplate` was originally designed to be used in combination with a J2EE container where the container would provide the necessary pooling of the JMS resources. As we are running this example on Spring Boot, we will wrap `ActiveMQConnectionFactory` using Spring's `CachingConnectionFactory` in order to still have the benefit of caching of sessions, connections, and producers as well as automatic connection recovery.

{% highlight java %}
package com.codenotfound.jms;

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
  public ActiveMQConnectionFactory senderActiveMQConnectionFactory() {
    ActiveMQConnectionFactory activeMQConnectionFactory =
        new ActiveMQConnectionFactory();
    activeMQConnectionFactory.setBrokerURL(brokerUrl);

    return activeMQConnectionFactory;
  }

  @Bean
  public CachingConnectionFactory cachingConnectionFactory() {
    return new CachingConnectionFactory(
        senderActiveMQConnectionFactory());
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
{% endhighlight %}

# 6. Create a Spring JMS Message Consumer

Like with any messaging-based application, you need to create a receiver that will handle the messages that have been sent. The below `Receiver` is nothing more than a simple POJO that defines a method for receiving messages. In this example we named the method `receive()`, but you can name it anything you like.

The `@JmsListener` annotation creates a message listener container behind the scenes for each annotated method, using a `JmsListenerContainerFactory`. By default, a bean with name <var>'jmsListenerContainerFactory'</var> is expected that we will setup in the next section.

Using the `destination` element, we specify the destination for this listener. In the below example we set the destination to <var>helloworld.q</var>.

> For testing convenience, we added a `CountDownLatch`. This allows the POJO to signal that a message is received. This is something you are not likely to implement in a production application.

{% highlight java %}
package com.codenotfound.jms;

import java.util.concurrent.CountDownLatch;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.jms.annotation.JmsListener;

public class Receiver {

  private static final Logger LOGGER =
      LoggerFactory.getLogger(Receiver.class);

  private CountDownLatch latch = new CountDownLatch(1);

  public CountDownLatch getLatch() {
    return latch;
  }

  @JmsListener(destination = "helloworld.q")
  public void receive(String message) {
    LOGGER.info("received message='{}'", message);
    latch.countDown();
  }
}
{% endhighlight %}

The creation and configuration of the different Spring Beans needed for the `Receiver` POJO are grouped in the `ReceiverConfig` class. Note that we need to add the `@EnableJms` annotation to enable support for the `@JmsListener` annotation that was used on the `Receiver`.

The `jmsListenerContainerFactory()` is expected by the `@JmsListener` annotation from the `Receiver`. We set the concurrency between 3 and 10. This means that listener container will always hold on to the minimum number of consumers and will slowly scale up to the maximum number of consumers in case of increasing load.

Contrary to the `JmsTemplate` [ideally don't use Spring's CachingConnectionFactory with a message listener container at all](http://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/jms/listener/DefaultMessageListenerContainer.html){:target="_blank"}. Reason for this is that it is generally preferable to let the listener container itself handle appropriate caching within its lifecycle.

As we are connecting to ActiveMQ, an `ActiveMQConnectionFactory` is created and passed in the constructor of the `DefaultJmsListenerContainerFactory`.

{% highlight java %}
package com.codenotfound.jms;

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
  public ActiveMQConnectionFactory receiverActiveMQConnectionFactory() {
    ActiveMQConnectionFactory activeMQConnectionFactory =
        new ActiveMQConnectionFactory();
    activeMQConnectionFactory.setBrokerURL(brokerUrl);

    return activeMQConnectionFactory;
  }

  @Bean
  public DefaultJmsListenerContainerFactory jmsListenerContainerFactory() {
    DefaultJmsListenerContainerFactory factory =
        new DefaultJmsListenerContainerFactory();
    factory
        .setConnectionFactory(receiverActiveMQConnectionFactory());
    factory.setConcurrency("3-10");

    return factory;
  }

  @Bean
  public Receiver receiver() {
    return new Receiver();
  }
}
{% endhighlight %}

# 7. Testing the Spring JMS Template & Listener

In order to verify that we are able to send and receive a message to and from ActiveMQ, a basic `SpringJmsApplicationTest` test case is used. It contains a `testReceive()` unit test case that uses the `Sender` to send a message to the <var>'helloworld.q'</var> queue on the ActiveMQ message broker. We then use the `CountDownLatch` from the `Receiver` to verify that a message was received.

An embedded ActiveMQ broker is automatically started by using an [EmbeddedActiveMQBroker JUnit Rule](http://activemq.apache.org/how-to-unit-test-jms-code.html#HowToUnitTestJMSCode-UsingTheEmbeddedActiveMQBrokerJUnitRule(ActiveMQ5.13)){:target="_blank"}. We have added a dedicated <var>application.yml</var> properties file for testing under <var>src/test/resources</var> that contains the VM URI to connect with the embedded broker.

> Note that as the embedded broker gets shutdown once the unit test cases are finished, we need to stop our `Sender` and `Receiver` before this happens in order to avoid connection errors. This achieved by using the `@DirtiesContext` annotation which closes the `ApplicationContext` after each test.

Below test case can also be executed after you [install Apache ActiveMQ]({{ site.url }}/2014/01/jms-apache-activemq-installation.html) on your local system. You need to comment out the lines annotated with `@ClassRule` to avoid the embedded broker gets created. In addition you need to change the <var>'activemq:broker-url'</var> property to point to <var>'tcp://localhost:61616'</var> in case the broker is running on the default URL value.

{% highlight java %}
package com.codenotfound;

import static org.assertj.core.api.Assertions.assertThat;
import java.util.concurrent.TimeUnit;
import org.apache.activemq.junit.EmbeddedActiveMQBroker;
import org.junit.ClassRule;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.annotation.DirtiesContext;
import org.springframework.test.context.junit4.SpringRunner;
import com.codenotfound.jms.Receiver;
import com.codenotfound.jms.Sender;

@RunWith(SpringRunner.class)
@SpringBootTest
@DirtiesContext
public class SpringJmsApplicationTest {

  @ClassRule
  public static EmbeddedActiveMQBroker broker =
      new EmbeddedActiveMQBroker();

  @Autowired
  private Sender sender;

  @Autowired
  private Receiver receiver;

  @Test
  public void testReceive() throws Exception {
    sender.send("Hello Spring JMS ActiveMQ!");

    receiver.getLatch().await(10000, TimeUnit.MILLISECONDS);
    assertThat(receiver.getLatch().getCount()).isEqualTo(0);
  }
}
{% endhighlight %}

In order to execute above example open a command prompt and run following Maven command:

{% highlight plaintext %}
mvn test
{% endhighlight %}

Maven will download the dependencies, compile the code and run the unit test case. The result should be a successful build as shown below:

{% highlight plaintext %}
.   ____          _            __ _ _
/\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
\\/  ___)| |_)| | | | | || (_| |  ) ) ) )
 '  |____| .__|_| |_|_| |_\__, | / / / /
=========|_|==============|___/=/_/_/_/
:: Spring Boot ::        (v2.1.1.RELEASE)

2018-12-10 13:12:53.646  INFO 10208 --- [           main] c.codenotfound.SpringJmsApplicationTest  : Starting SpringJmsApplicationTest on DESKTOP-2RB3C1U with PID 10208 (started by Codenotfound in C:\Users\Codenotfound\repos\spring-jms\spring-jms-activemq-hello-world)
2018-12-10 13:12:53.646  INFO 10208 --- [           main] c.codenotfound.SpringJmsApplicationTest  : No active profile set, falling back to default profiles: default
2018-12-10 13:12:54.670  INFO 10208 --- [           main] c.codenotfound.SpringJmsApplicationTest  : Started SpringJmsApplicationTest in 1.331 seconds (JVM running for 2.796)
2018-12-10 13:12:54.883  INFO 10208 --- [           main] com.codenotfound.jms.Sender              : sending message='Hello Spring JMS ActiveMQ!'
2018-12-10 13:12:54.911  INFO 10208 --- [enerContainer-3] com.codenotfound.jms.Receiver            : received message='Hello Spring JMS ActiveMQ!'
2018-12-10 13:12:55.922  INFO 10208 --- [           main] o.a.a.junit.EmbeddedActiveMQBroker       : Stopping Embedded ActiveMQ Broker: embedded-broker
2018-12-10 13:12:55.922  INFO 10208 --- [           main] o.a.activemq.broker.TransportConnector   : Connector vm://embedded-broker stopped
2018-12-10 13:12:55.922  INFO 10208 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.8 (embedded-broker, ID:DESKTOP-2RB3C1U-60944-1544443972991-0:1) is shutting down
2018-12-10 13:12:55.938  INFO 10208 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.8 (embedded-broker, ID:DESKTOP-2RB3C1U-60944-1544443972991-0:1) uptime 3.047 seconds
2018-12-10 13:12:55.938  INFO 10208 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.8 (embedded-broker, ID:DESKTOP-2RB3C1U-60944-1544443972991-0:1) is shutdown
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 3.702 s - in com.codenotfound.SpringJmsApplicationTest
[INFO]
[INFO] Results:
[INFO]
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 9.689 s
[INFO] Finished at: 2018-12-10T13:12:56+01:00
[INFO] ------------------------------------------------------------------------
{% endhighlight %}

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-jms/tree/master/spring-jms-activemq-hello-world){:target="_blank"}.
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This concludes our example in which we used a Spring JMS and Spring Boot in order to connect to ActiveMQ.

If you found this sample useful or have a question you would like to ask.

Leave a comment below!
