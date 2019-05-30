---
title: "Spring JMS Integration Example"
permalink: /spring-jms-integration-example.html
excerpt: "A detailed step-by-step tutorial on how to connect to an ActiveMQ JMS broker using Spring Integration and Spring Boot."
date: 2018-12-17
last_modified_at: 2019-05-30
header:
  teaser: "assets/images/spring-jms/spring-jms-integration.png"
categories: [Spring JMS]
tags: [ActiveMQ, Apache ActiveMQ, Example, Integration, Maven, Spring, Spring Boot, Spring Integration, Spring JMS, Tutorial]
published: true
---

<img src="{{ site.url }}/assets/images/spring-jms/spring-jms-integration.png" alt="spring jms integration" class="align-right title-image">

In this post you're going to learn how to use [Spring Integration](https://spring.io/projects/spring-integration){:target="_blank"} components to connect to an [ActiveMQ](http://activemq.apache.org/){:target="_blank"} message broker.

In fact:

We will build an example that covers both the sending and receiving of JMS messages.

Let's dive right in.

If you want to learn more about Spring JMS - head on over to the [Spring JMS tutorials]({{ site.url }}/spring-jms-tutorials) page.
{: .notice--primary}

## 1. What is Spring Integration?

Spring Integration extends the Spring programming model to support the well-known [Enterprise Integration Patterns](http://www.enterpriseintegrationpatterns.com/){:target="_blank"}.

It **enables lightweight messaging** within Spring-based applications. It also **supports integration** with external systems via declarative adapters. Some of the supported integrations are FTP, HTTP, JMS, and email.

Spring Integration is a main Spring project. It is developed and maintained by [Pivotal Software](https://pivotal.io/){:target="_blank"}.

## 2. Example Setup

Spring Integration uses the concept of a **Message Channel** to pass along information from one component to another. It represents the "pipe" of a [pipes-and-filters architecture](http://www.enterpriseintegrationpatterns.com/patterns/messaging/PipesAndFilters.html){:target="_blank"}.

You can use a [Channel Adapter](https://www.enterpriseintegrationpatterns.com/patterns/messaging/ChannelAdapter.html){:target="_blank"} to connect an application to a messaging system so that it can send and receive messages.

An [Outbound Channel Adapter](https://docs.spring.io/spring-integration/docs/5.1.5.RELEASE/reference/html/#jms-outbound-channel-adapter){:target="_blank"} is capable of converting Spring Integration messages to JMS messages and then sending them to a JMS destination.

A [Message-driven Channel Adapter](https://docs.spring.io/spring-integration/docs/5.1.5.RELEASE/reference/html/#jms-message-driven-channel-adapter) receives JMS messages from a JMS destination and converts them into Spring Integration messages.

There are two JMS-based inbound Channel Adapters. The first uses Springs `JmsTemplate` to receive based on a polling period. The second is "message-driven" and relies on a Spring `MessageListener` container. We will use the latter in this example.

<img src="{{ site.url }}/assets/images/spring-jms/spring-jms-integration-example-overview.png" alt="spring jms integration example overview">

Let's build a Spring JMS integration example using ActiveMQ. It consists of two channels as shown in the above diagram.

The first _ProducingChannel_ will have a `JmsSendingMessageHandler` that subscribes to the channel and writes all received messages to an <var>integration.q</var> queue.

A second _ConsumingChannel_ will connect to the same queue using a `JmsMessageDrivenEndpoint`. A custom `CountDownLatchHandler` subscribes to this second channel and lowers a `CountDownLatch`.

## 3. General Project Overview

We will use the following tools/frameworks:
* Spring JMS 5.1
* Spring Integration 5.1
* Spring Boot 2.1
* ActiveMQ 5.15
* Maven 3.6

Our project has the following directory structure:

<img src="{{ site.url }}/assets/images/spring-jms/spring-jms-integration-maven-project.png" alt="spring jms integration maven project">

## 4. Maven Setup

We start from a previous [Spring JMS Boot example]({{ site.url }}/spring-jms-annotations-example.html) for the setup of our project.

To use the different Spring Integration JMS components we need to add the `spring-integration-jms` dependency.

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>spring-jms-integration</artifactId>
  <version>0.0.1-SNAPSHOT</version>

  <name>spring-jms-integration</name>
  <description>Spring JMS Integration Example</description>
  <url>https://codenotfound.com/spring-jms-integration-example.html</url>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.5.RELEASE</version>
    <relativePath /><!-- lookup parent from repository -->
  </parent>

  <properties>
    <java.version>11</java.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-activemq</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.integration</groupId>
      <artifactId>spring-integration-jms</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
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

## 5. Create a JmsSendingMessageHandler

Create a `ProducingChannelConfig` class and annotate it with `@Configuration`.

Define a _ProducingChannel_ as a `DirectChannel` bean. This is the default channel provided by the framework, but you can use any of the [message channels Spring Integration provides](https://docs.spring.io/spring-integration/docs/5.1.5.RELEASE/reference/html/#messaging-channels-section){:target="_blank"}.

Next, we create the `JmsSendingMessageHandler` that will send messages received from the _ProducingChannel_ towards a destination. The constructor requires a `JmsTemplate` to be passed as a parameter. We simply inject the template that is auto-configured by Spring Boot.

Using the `@Value` annotation we load the name of the destination from the <var>application.yml</var> properties file under <var>src/main/resources</var>. We then set it using `setDestinationName()`.

The `JmsSendingMessageHandler` is attached to the _ProducingChannel_ using the `@ServiceActivator` annotation. As `inputChannel` we need to specify the <var>producingChannel</var> as a key/value pair in order to make the link.

{% highlight java %}
package com.codenotfound.jms;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.integration.annotation.ServiceActivator;
import org.springframework.integration.channel.DirectChannel;
import org.springframework.integration.jms.JmsSendingMessageHandler;
import org.springframework.jms.core.JmsTemplate;
import org.springframework.messaging.MessageHandler;

@Configuration
public class ProducingChannelConfig {

  @Value("${destination.integration}")
  private String integrationDestination;

  @Bean
  public DirectChannel producingChannel() {
    return new DirectChannel();
  }

  @Bean
  @ServiceActivator(inputChannel = "producingChannel")
  public MessageHandler jmsMessageHandler(JmsTemplate jmsTemplate) {
    JmsSendingMessageHandler handler =
        new JmsSendingMessageHandler(jmsTemplate);
    handler.setDestinationName(integrationDestination);

    return handler;
  }
}
{% endhighlight %}

## 6. Create a JmsMessageDrivenEndpoint

Similar to the _ProducingChannel_, we specify a _ConsumingChannel_ using the `DirectChannel` channel type.

We then create a `JmsMessageDrivenEndpoint` that can receive JMS messages. The constructor takes a `MessageListenerContainer` and `ChannelPublishingJmsMessageListener` as an input parameters.

> For details on the container we refer to the [Spring JMS listener example]({{ site.url }}//spring-jms-listener-example.html).

The `ChannelPublishingJmsMessageListener` creates a listener that converts a JMS Message into a Spring Integration Message and sends that message to a channel.

We connect the endpoint to the _ConsumingChannel_ by using the `setOutputChannel()` method.

In order to test our setup, a `CountDownLatchHandler` bean is specified that is linked to the _ConsumingChannel_ using the `@ServiceActivator` annotation.

{% highlight java %}
package com.codenotfound.jms;

import javax.jms.ConnectionFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.integration.annotation.ServiceActivator;
import org.springframework.integration.channel.DirectChannel;
import org.springframework.integration.jms.ChannelPublishingJmsMessageListener;
import org.springframework.integration.jms.JmsMessageDrivenEndpoint;
import org.springframework.jms.listener.SimpleMessageListenerContainer;

@Configuration
public class ConsumingChannelConfig {

  @Value("${destination.integration}")
  private String integrationDestination;

  @Bean
  public DirectChannel consumingChannel() {
    return new DirectChannel();
  }

  @Bean
  public JmsMessageDrivenEndpoint jmsMessageDrivenEndpoint(
      ConnectionFactory connectionFactory) {
    JmsMessageDrivenEndpoint endpoint = new JmsMessageDrivenEndpoint(
        simpleMessageListenerContainer(connectionFactory),
        channelPublishingJmsMessageListener());
    endpoint.setOutputChannel(consumingChannel());

    return endpoint;
  }

  @Bean
  public SimpleMessageListenerContainer simpleMessageListenerContainer(
      ConnectionFactory connectionFactory) {
    SimpleMessageListenerContainer container =
        new SimpleMessageListenerContainer();
    container.setConnectionFactory(connectionFactory);
    container.setDestinationName(integrationDestination);
    return container;
  }

  @Bean
  public ChannelPublishingJmsMessageListener channelPublishingJmsMessageListener() {
    return new ChannelPublishingJmsMessageListener();
  }

  @Bean
  @ServiceActivator(inputChannel = "consumingChannel")
  public CountDownLatchHandler countDownLatchHandler() {
    return new CountDownLatchHandler();
  }
}
{% endhighlight %}

The `CountDownLatchHandler` class allows us to verify the correct working of our two connected channels.

It implements the `handleMessage()` method of the `MessageHandler` interface.

Messages from the attached _ConsumingChannel_ are logged and a `CountDownLatch` is lowered per message.

{% highlight java %}
package com.codenotfound.jms;

import java.util.concurrent.CountDownLatch;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.messaging.Message;
import org.springframework.messaging.MessageHandler;

public class CountDownLatchHandler implements MessageHandler {

  private static final Logger LOGGER =
      LoggerFactory.getLogger(CountDownLatchHandler.class);

  private CountDownLatch latch = new CountDownLatch(10);

  public CountDownLatch getLatch() {
    return latch;
  }

  @Override
  public void handleMessage(Message<?> message) {
    LOGGER.info("received message='{}'", message);
    latch.countDown();
  }
}
{% endhighlight %}

## 7. Testing Spring JMS Integration Example

We change the existing test case to check if different Spring Integration JMS components work.

To get hold of the _ProducingChannel_, we auto-wire the `ApplicationContext` and use the `getBean()` method.

We then create a for loop in which we sent 10 messages to the <var>integration.q</var> queue using the channel's `send()` method. Set the destination by adding a message header `Map` which contains the `JmsHeaders.DESTINATION` value which corresponds to the destination name.

The messages should arrive via the _ConsumingChannel_ in the `CountDownLatchHandler` where the `CountDownLatch` is lowered from its initial value of <var>10</var>.

We check if all the messages have been received by asserting that the `CountDownLatch` value equals to <var>0</var>.

{% highlight java %}
package com.codenotfound.jms;

import static org.assertj.core.api.Assertions.assertThat;
import java.util.Collections;
import java.util.Map;
import java.util.concurrent.TimeUnit;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.context.ApplicationContext;
import org.springframework.jms.support.JmsHeaders;
import org.springframework.messaging.MessageChannel;
import org.springframework.messaging.support.GenericMessage;
import org.springframework.test.annotation.DirtiesContext;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
@DirtiesContext
public class SpringJmsApplicationTest {

  private static final Logger LOGGER =
      LoggerFactory.getLogger(SpringJmsApplicationTest.class);

  @Value("${destination.integration}")
  private String integrationDestination;

  @Autowired
  private ApplicationContext applicationContext;

  @Autowired
  private CountDownLatchHandler countDownLatchHandler;

  @Test
  public void testIntegration() throws Exception {
    MessageChannel producingChannel = applicationContext
        .getBean("producingChannel", MessageChannel.class);

    Map<String, Object> headers = Collections.singletonMap(
        JmsHeaders.DESTINATION, integrationDestination);

    LOGGER.info("sending 10 messages");
    for (int i = 0; i < 10; i++) {
      GenericMessage<String> message = new GenericMessage<>(
          "Hello Spring Integration JMS " + i + "!", headers);
      producingChannel.send(message);
      LOGGER.info("sent message='{}'", message);
    }

    countDownLatchHandler.getLatch().await(10000,
        TimeUnit.MILLISECONDS);
    assertThat(countDownLatchHandler.getLatch().getCount())
        .isEqualTo(0);
  }
}
{% endhighlight %}

Now it's time to run the test case. Open a command prompt in the root directory and enter the following command:

{% highlight plaintext %}
mvn test
{% endhighlight %}

In the output logs, we see that 10 messages arrive.

{% highlight plaintext %}
 .   ____          _            __ _ _
/\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
\\/  ___)| |_)| | | | | || (_| |  ) ) ) )
'  |____| .__|_| |_|_| |_\__, | / / / /
=========|_|==============|___/=/_/_/_/
:: Spring Boot ::        (v2.1.5.RELEASE)

2019-05-30 17:00:06.652  INFO 12020 --- [           main] c.c.jms.SpringJmsApplicationTest         : Starting SpringJmsApplicationTest on DESKTOP-2RB3C1U with PID 12020 (started by Codenotfound in C:\Users\Codenotfound\repos\spring-jms\spring-jms-integration)
2019-05-30 17:00:06.654  INFO 12020 --- [           main] c.c.jms.SpringJmsApplicationTest         : No active profile set, falling back to default profiles: default
2019-05-30 17:00:07.444  INFO 12020 --- [           main] faultConfiguringBeanFactoryPostProcessor : No bean named 'errorChannel' has been explicitly defined. Therefore, a default PublishSubscribeChannel will be created.
2019-05-30 17:00:07.451  INFO 12020 --- [           main] faultConfiguringBeanFactoryPostProcessor : No bean named 'taskScheduler' has been explicitly defined. Therefore, a default ThreadPoolTaskScheduler will be created.
2019-05-30 17:00:07.464  INFO 12020 --- [           main] faultConfiguringBeanFactoryPostProcessor : No bean named 'integrationHeaderChannelRegistry' has been explicitly defined. Therefore, a default DefaultHeaderChannelRegistry will be created.
2019-05-30 17:00:07.559  INFO 12020 --- [           main] trationDelegate$BeanPostProcessorChecker : Bean 'org.springframework.integration.config.IntegrationManagementConfiguration' of type [org.springframework.integration.config.IntegrationManagementConfiguration$$EnhancerBySpringCGLIB$$730d8ffc] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
2019-05-30 17:00:07.607  INFO 12020 --- [           main] trationDelegate$BeanPostProcessorChecker : Bean 'integrationDisposableAutoCreatedBeans' of type [org.springframework.integration.config.annotation.Disposables] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
2019-05-30 17:00:08.399  INFO 12020 --- [           main] o.apache.activemq.broker.BrokerService   : Using Persistence Adapter: MemoryPersistenceAdapter
2019-05-30 17:00:08.476  INFO 12020 --- [  JMX connector] o.a.a.broker.jmx.ManagementContext       : JMX consoles can connect to service:jmx:rmi:///jndi/rmi://localhost:1099/jmxrmi
2019-05-30 17:00:08.572  INFO 12020 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.9 (localhost, ID:DESKTOP-2RB3C1U-59524-1559228408439-0:1) is starting
2019-05-30 17:00:08.586  INFO 12020 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.9 (localhost, ID:DESKTOP-2RB3C1U-59524-1559228408439-0:1) started
2019-05-30 17:00:08.586  INFO 12020 --- [           main] o.apache.activemq.broker.BrokerService   : For help or more information please see: http://activemq.apache.org
2019-05-30 17:00:08.616  INFO 12020 --- [           main] o.a.activemq.broker.TransportConnector   : Connector vm://localhost started
2019-05-30 17:00:09.030  INFO 12020 --- [           main] o.s.s.c.ThreadPoolTaskScheduler          : Initializing ExecutorService 'taskScheduler'
2019-05-30 17:00:09.385  INFO 12020 --- [           main] o.s.i.endpoint.EventDrivenConsumer       : Adding {logging-channel-adapter:_org.springframework.integration.errorLogger} as a subscriber to the 'errorChannel' channel
2019-05-30 17:00:09.385  INFO 12020 --- [           main] o.s.i.channel.PublishSubscribeChannel    : Channel 'application.errorChannel' has 1 subscriber(s).
2019-05-30 17:00:09.386  INFO 12020 --- [           main] o.s.i.endpoint.EventDrivenConsumer       : started _org.springframework.integration.errorLogger
2019-05-30 17:00:09.386  INFO 12020 --- [           main] o.s.i.endpoint.EventDrivenConsumer       : Adding {message-handler:consumingChannelConfig.countDownLatchHandler.serviceActivator} as a subscriber to the 'consumingChannel' channel
2019-05-30 17:00:09.386  INFO 12020 --- [           main] o.s.integration.channel.DirectChannel    : Channel 'application.consumingChannel' has 1 subscriber(s).
2019-05-30 17:00:09.386  INFO 12020 --- [           main] o.s.i.endpoint.EventDrivenConsumer       : started consumingChannelConfig.countDownLatchHandler.serviceActivator
2019-05-30 17:00:09.387  INFO 12020 --- [           main] o.s.i.endpoint.EventDrivenConsumer       : Adding {message-handler:producingChannelConfig.jmsMessageHandler.serviceActivator} as a subscriber to the 'producingChannel' channel
2019-05-30 17:00:09.387  INFO 12020 --- [           main] o.s.integration.channel.DirectChannel    : Channel 'application.producingChannel' has 1 subscriber(s).
2019-05-30 17:00:09.388  INFO 12020 --- [           main] o.s.i.endpoint.EventDrivenConsumer       : started producingChannelConfig.jmsMessageHandler.serviceActivator
2019-05-30 17:00:09.388  INFO 12020 --- [           main] ishingJmsMessageListener$GatewayDelegate : started org.springframework.integration.jms.ChannelPublishingJmsMessageListener$GatewayDelegate@5099c59b
2019-05-30 17:00:09.389  INFO 12020 --- [           main] o.s.i.jms.JmsMessageDrivenEndpoint       : started jmsMessageDrivenEndpoint
2019-05-30 17:00:09.403  INFO 12020 --- [           main] c.c.jms.SpringJmsApplicationTest         : Started SpringJmsApplicationTest in 3.187 seconds (JVM running for 4.311)
2019-05-30 17:00:09.800  INFO 12020 --- [           main] c.c.jms.SpringJmsApplicationTest         : sending 10 messages
2019-05-30 17:00:09.836  INFO 12020 --- [           main] c.c.jms.SpringJmsApplicationTest         : sent message='GenericMessage [payload=Hello Spring Integration JMS 0!, headers={jms_destination=integration.q, id=726d2ef1-bd27-ac02-138e-6adda629de4c, timestamp=1559228409801}]'
2019-05-30 17:00:09.838  INFO 12020 --- [           main] c.c.jms.SpringJmsApplicationTest         : sent message='GenericMessage [payload=Hello Spring Integration JMS 1!, headers={jms_destination=integration.q, id=9c0a5332-4899-241a-64c9-abc67edc199c, timestamp=1559228409836}]'
2019-05-30 17:00:09.839  INFO 12020 --- [           main] c.c.jms.SpringJmsApplicationTest         : sent message='GenericMessage [payload=Hello Spring Integration JMS 2!, headers={jms_destination=integration.q, id=61cfc4ba-55d5-6bab-8285-e4ae2995c50b, timestamp=1559228409838}]'
2019-05-30 17:00:09.847  INFO 12020 --- [           main] c.c.jms.SpringJmsApplicationTest         : sent message='GenericMessage [payload=Hello Spring Integration JMS 3!, headers={jms_destination=integration.q, id=00673fad-ec44-e555-c8f4-e924f82147e4, timestamp=1559228409840}]'
2019-05-30 17:00:09.849  INFO 12020 --- [ Session Task-1] c.c.jms.CountDownLatchHandler            : received message='GenericMessage [payload=Hello Spring Integration JMS 0!, headers={jms_redelivered=false, jms_destination=queue://integration.q, id=a2824337-13f0-a6e5-09e1-6add09ac934d, priority=4, jms_timestamp=1559228409824, jms_messageId=ID:DESKTOP-2RB3C1U-59524-1559228408439-4:1:2:1:1, timestamp=1559228409849}]'
2019-05-30 17:00:09.853  INFO 12020 --- [           main] c.c.jms.SpringJmsApplicationTest         : sent message='GenericMessage [payload=Hello Spring Integration JMS 4!, headers={jms_destination=integration.q, id=337d03ab-0c41-23a9-9d4c-5ba563857cb9, timestamp=1559228409848}]'
2019-05-30 17:00:09.856  INFO 12020 --- [ Session Task-1] c.c.jms.CountDownLatchHandler            : received message='GenericMessage [payload=Hello Spring Integration JMS 1!, headers={jms_redelivered=false, jms_destination=queue://integration.q, id=04ccd784-40b7-4b5d-28e7-a08a2256e6ec, priority=4, jms_timestamp=1559228409836, jms_messageId=ID:DESKTOP-2RB3C1U-59524-1559228408439-4:1:2:1:2, timestamp=1559228409856}]'
2019-05-30 17:00:09.862  INFO 12020 --- [           main] c.c.jms.SpringJmsApplicationTest         : sent message='GenericMessage [payload=Hello Spring Integration JMS 5!, headers={jms_destination=integration.q, id=9cc4335e-6215-f685-8ae5-e444f5fe98d2, timestamp=1559228409853}]'
2019-05-30 17:00:09.863  INFO 12020 --- [ Session Task-1] c.c.jms.CountDownLatchHandler            : received message='GenericMessage [payload=Hello Spring Integration JMS 2!, headers={jms_redelivered=false, jms_destination=queue://integration.q, id=19851b82-5cf7-a76b-bca2-2dfab84c86d0, priority=4, jms_timestamp=1559228409838, jms_messageId=ID:DESKTOP-2RB3C1U-59524-1559228408439-4:1:2:1:3, timestamp=1559228409863}]'
2019-05-30 17:00:09.865  INFO 12020 --- [ Session Task-1] c.c.jms.CountDownLatchHandler            : received message='GenericMessage [payload=Hello Spring Integration JMS 3!, headers={jms_redelivered=false, jms_destination=queue://integration.q, id=977ed741-2b0d-d3ca-a1f2-9425eea07c4d, priority=4, jms_timestamp=1559228409840, jms_messageId=ID:DESKTOP-2RB3C1U-59524-1559228408439-4:1:2:1:4, timestamp=1559228409865}]'
2019-05-30 17:00:09.868  INFO 12020 --- [ Session Task-1] c.c.jms.CountDownLatchHandler            : received message='GenericMessage [payload=Hello Spring Integration JMS 4!, headers={jms_redelivered=false, jms_destination=queue://integration.q, id=ca3dc880-3376-3c19-6c30-8878c5a90a30, priority=4, jms_timestamp=1559228409848, jms_messageId=ID:DESKTOP-2RB3C1U-59524-1559228408439-4:1:2:1:5, timestamp=1559228409867}]'
2019-05-30 17:00:09.869  INFO 12020 --- [ Session Task-1] c.c.jms.CountDownLatchHandler            : received message='GenericMessage [payload=Hello Spring Integration JMS 5!, headers={jms_redelivered=false, jms_destination=queue://integration.q, id=99fb58ae-5662-2078-be1b-5e206760dce9, priority=4, jms_timestamp=1559228409853, jms_messageId=ID:DESKTOP-2RB3C1U-59524-1559228408439-4:1:2:1:6, timestamp=1559228409869}]'
2019-05-30 17:00:09.894  INFO 12020 --- [           main] c.c.jms.SpringJmsApplicationTest         : sent message='GenericMessage [payload=Hello Spring Integration JMS 6!, headers={jms_destination=integration.q, id=8937d229-febe-12ae-c753-c82b400f3013, timestamp=1559228409862}]'
2019-05-30 17:00:09.899  INFO 12020 --- [ Session Task-1] c.c.jms.CountDownLatchHandler            : received message='GenericMessage [payload=Hello Spring Integration JMS 6!, headers={jms_redelivered=false, jms_destination=queue://integration.q, id=153cabb6-9174-fd62-941d-55665db0d4fe, priority=4, jms_timestamp=1559228409862, jms_messageId=ID:DESKTOP-2RB3C1U-59524-1559228408439-4:1:2:1:7, timestamp=1559228409899}]'
2019-05-30 17:00:09.903  INFO 12020 --- [           main] c.c.jms.SpringJmsApplicationTest         : sent message='GenericMessage [payload=Hello Spring Integration JMS 7!, headers={jms_destination=integration.q, id=751e2a92-f51a-ddd7-78bc-bc5a4521f18a, timestamp=1559228409895}]'
2019-05-30 17:00:09.905  INFO 12020 --- [ Session Task-1] c.c.jms.CountDownLatchHandler            : received message='GenericMessage [payload=Hello Spring Integration JMS 7!, headers={jms_redelivered=false, jms_destination=queue://integration.q, id=c83b7022-139e-571a-ef81-3bcb41ae1154, priority=4, jms_timestamp=1559228409895, jms_messageId=ID:DESKTOP-2RB3C1U-59524-1559228408439-4:1:2:1:8, timestamp=1559228409905}]'
2019-05-30 17:00:09.906  INFO 12020 --- [           main] c.c.jms.SpringJmsApplicationTest         : sent message='GenericMessage [payload=Hello Spring Integration JMS 8!, headers={jms_destination=integration.q, id=6902e79a-b536-463e-3161-1fd2babdad9d, timestamp=1559228409903}]'
2019-05-30 17:00:09.906  INFO 12020 --- [ Session Task-1] c.c.jms.CountDownLatchHandler            : received message='GenericMessage [payload=Hello Spring Integration JMS 8!, headers={jms_redelivered=false, jms_destination=queue://integration.q, id=6b629edd-ed5c-e2e8-7d55-0cf7a9ac457d, priority=4, jms_timestamp=1559228409903, jms_messageId=ID:DESKTOP-2RB3C1U-59524-1559228408439-4:1:2:1:9, timestamp=1559228409906}]'
2019-05-30 17:00:09.909  INFO 12020 --- [           main] c.c.jms.SpringJmsApplicationTest         : sent message='GenericMessage [payload=Hello Spring Integration JMS 9!, headers={jms_destination=integration.q, id=1e773b94-b6bd-811d-6fa4-6732cb46914f, timestamp=1559228409906}]'
2019-05-30 17:00:09.910  INFO 12020 --- [ Session Task-1] c.c.jms.CountDownLatchHandler            : received message='GenericMessage [payload=Hello Spring Integration JMS 9!, headers={jms_redelivered=false, jms_destination=queue://integration.q, id=330fac90-f8cb-421f-546b-0fc11b1bc759, priority=4, jms_timestamp=1559228409906, jms_messageId=ID:DESKTOP-2RB3C1U-59524-1559228408439-4:1:2:1:10, timestamp=1559228409910}]'
2019-05-30 17:00:09.998  INFO 12020 --- [           main] ishingJmsMessageListener$GatewayDelegate : stopped org.springframework.integration.jms.ChannelPublishingJmsMessageListener$GatewayDelegate@5099c59b
2019-05-30 17:00:09.998  INFO 12020 --- [           main] o.s.i.jms.JmsMessageDrivenEndpoint       : stopped jmsMessageDrivenEndpoint
2019-05-30 17:00:09.998  INFO 12020 --- [           main] o.s.i.endpoint.EventDrivenConsumer       : Removing {logging-channel-adapter:_org.springframework.integration.errorLogger} as a subscriber to the 'errorChannel' channel
2019-05-30 17:00:10.000  INFO 12020 --- [           main] o.s.i.channel.PublishSubscribeChannel    : Channel 'application.errorChannel' has 0 subscriber(s).
2019-05-30 17:00:10.000  INFO 12020 --- [           main] o.s.i.endpoint.EventDrivenConsumer       : stopped _org.springframework.integration.errorLogger
2019-05-30 17:00:10.000  INFO 12020 --- [           main] o.s.i.endpoint.EventDrivenConsumer       : Removing {message-handler:consumingChannelConfig.countDownLatchHandler.serviceActivator} as a subscriber to the 'consumingChannel' channel
2019-05-30 17:00:10.000  INFO 12020 --- [           main] o.s.integration.channel.DirectChannel    : Channel 'application.consumingChannel' has 0 subscriber(s).
2019-05-30 17:00:10.000  INFO 12020 --- [           main] o.s.i.endpoint.EventDrivenConsumer       : stopped consumingChannelConfig.countDownLatchHandler.serviceActivator
2019-05-30 17:00:10.000  INFO 12020 --- [           main] o.s.i.endpoint.EventDrivenConsumer       : Removing {message-handler:producingChannelConfig.jmsMessageHandler.serviceActivator} as a subscriber to the 'producingChannel' channel
2019-05-30 17:00:10.000  INFO 12020 --- [           main] o.s.integration.channel.DirectChannel    : Channel 'application.producingChannel' has 0 subscriber(s).
2019-05-30 17:00:10.001  INFO 12020 --- [           main] o.s.i.endpoint.EventDrivenConsumer       : stopped producingChannelConfig.jmsMessageHandler.serviceActivator
2019-05-30 17:00:10.001  INFO 12020 --- [           main] o.s.s.c.ThreadPoolTaskScheduler          : Shutting down ExecutorService 'taskScheduler'
2019-05-30 17:00:10.009  INFO 12020 --- [           main] o.a.activemq.broker.TransportConnector   : Connector vm://localhost stopped
2019-05-30 17:00:10.010  INFO 12020 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.9 (localhost, ID:DESKTOP-2RB3C1U-59524-1559228408439-0:1) is shutting down
2019-05-30 17:00:10.019  INFO 12020 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.9 (localhost, ID:DESKTOP-2RB3C1U-59524-1559228408439-0:1) uptime 1.775 seconds
2019-05-30 17:00:10.019  INFO 12020 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.9 (localhost, ID:DESKTOP-2RB3C1U-59524-1559228408439-0:1) is shutdown
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 4.699 s - in com.codenotfound.jms.SpringJmsApplicationTest
[INFO]
[INFO] Results:
[INFO]
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  8.719 s
[INFO] Finished at: 2019-05-30T17:00:10+02:00
[INFO] ------------------------------------------------------------------------
{% endhighlight %}

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-jms/tree/master/spring-jms-integration){:target="_blank"}.
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

In this tutorial, we looked at the different Spring Integration JMS components and used them to connect to ActiveMQ.

Leave a message below if you enjoyed this post.

Thanks!
