---
title: "Spring Kafka - Spring Integration Example"
permalink: /spring-kafka-spring-integration-example.html
excerpt: "A detailed step-by-step tutorial on how to connect Apache Kafka to a Spring Integration Channel using Spring Kafka and Spring Boot."
date: 2017-05-13
modified: 2017-05-13
header:
  teaser: "assets/images/header/spring-kafka-teaser.png"
categories: [Spring Kafka]
tags: [Apache Kafka, Consumer, Example, Hello World, Maven, Producer, Spring, Spring Boot, Spring Integration, Spring Kafka, Tutorial]
redirect_from:
  - /2017/05/spring-kafka-integration-example.html
  - /spring-kafka-integration-example
published: true
---

<figure>
    <img src="{{ site.url }}/assets/images/logo/spring-logo.png" alt="spring logo" class="logo">
</figure>

[Spring Integration](https://projects.spring.io/spring-integration/){:target="_blank"} extends the Spring programming model to support the well-known [Enterprise Integration Patterns](http://www.enterpriseintegrationpatterns.com/){:target="_blank"}. It enables lightweight messaging within Spring-based applications and supports integration with external systems via declarative adapters.

The [Spring Integration Kafka](https://github.com/spring-projects/spring-integration-kafka){:target="_blank"} extension project provides inbound and outbound channel adapters specifically for [Apache Kafka](https://kafka.apache.org/){:target="_blank"}.

In this tutorial, we will configure, build and run a Hello World example in which we will send/receive messages to/from Apache Kafka using Spring Integration Kafka, Spring Boot, and Maven.

If you want to learn more about Spring Kafka - head on over to the [Spring Kafka tutorials page]({{ site.url }}/spring-kafka/).
{: .notice--primary}

# General Project Setup

Tools used:
* Spring Kafka 1.2
* Spring Integration 2.1
* Spring Boot 1.5
* Maven 3.5

The building of this project will be automated using [Maven](https://maven.apache.org/){:target="_blank"}. We include the needed Spring Integration dependencies using the `spring-boot-starter-integration` [Spring Boot starter](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters){:target="_blank"}. For testing support, we also include the `spring-boot-starter-test` starter.

As we will be using the Spring Integration Kafka extension, we add the corresponding `spring-integration-kafka` dependency. Starting from version 2.0 this project is a complete rewrite based on the [Spring for Apache Kafka](https://projects.spring.io/spring-kafka/){:target="_blank"} project which uses the pure java `Producer` and `Consumer` clients provided by Kafka. As such we also add the `spring-kafka` dependency for core functionality as well as `spring-kafka-test` in order to have access to an embedded Kafka broker when running our unit test.

The `spring-boot-maven-plugin` Maven plugin is added so that we can build a single, runnable JAR, which is convenient to execute and transport our written code.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>spring-kafka-integration-helloworld</artifactId>
  <version>0.0.1-SNAPSHOT</version>

  <name>spring-kafka-integration-helloworld</name>
  <description>Spring Kafka - Integration Example</description>
  <url>https://www.codenotfound.com/spring-kafka-spring-integration-example.html</url>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.4.RELEASE</version>
  </parent>

  <properties>
    <java.version>1.8</java.version>

    <spring-integration-kafka.version>2.1.0.RELEASE</spring-integration-kafka.version>
    <spring-kafka.version>1.2.2.RELEASE</spring-kafka.version>
  </properties>

  <dependencies>
    <!-- spring-boot -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-integration</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
    <!-- spring-integration -->
    <dependency>
      <groupId>org.springframework.integration</groupId>
      <artifactId>spring-integration-kafka</artifactId>
      <version>${spring-integration-kafka.version}</version>
    </dependency>
    <!-- spring-kafka -->
    <dependency>
      <groupId>org.springframework.kafka</groupId>
      <artifactId>spring-kafka</artifactId>
      <version>${spring-kafka.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework.kafka</groupId>
      <artifactId>spring-kafka-test</artifactId>
      <version>${spring-kafka.version}</version>
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

We create a `SpringKafkaIntegrationApplication` class that takes care of some [basic Spring Boot setup](http://docs.spring.io/autorepo/docs/spring-boot/current/reference/html/using-boot-using-springbootapplication-annotation.html){:target="_blank"} and which also allows launching the application.

``` java
package com.codenotfound.kafka.integration;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringKafkaIntegrationApplication {

  public static void main(String[] args) {
    SpringApplication.run(SpringKafkaIntegrationApplication.class, args);
  }
}
```

# Example Setup Overview

Spring Integration uses the concept of a **Message Channel** to pass along information from one component to another. It represents the "pipe" of a [pipes-and-filters architecture](http://www.enterpriseintegrationpatterns.com/patterns/messaging/PipesAndFilters.html){:target="_blank"}. A Message Channel may follow either [Point-to-Point](http://www.enterpriseintegrationpatterns.com/patterns/messaging/PointToPointChannel.html){:target="_blank"} or [Publish/Subscribe](http://www.enterpriseintegrationpatterns.com/patterns/messaging/PublishSubscribeChannel.html){:target="_blank"} semantics.

A **Message Endpoint** represents the "filter" of a [pipes-and-filters architecture](http://www.enterpriseintegrationpatterns.com/patterns/messaging/PipesAndFilters.html){:target="_blank"}. Spring Integration has a number of endpoint types that are supported. In this example, we will look at the endpoint types that allow us to connect to Kafka.

The first one is a [Service Activator](http://www.enterpriseintegrationpatterns.com/patterns/messaging/MessagingAdapter.html){:target="_blank"} which simply connects any existing Spring-managed bean to a channel. Spring Integration Kafka provides a `KafkaProducerMessageHandler` which handles a given message by using a `KafkaTemplate` to send data to Kafka topics. By connecting a channel as input to this Message Handler we can send messages to the Kafka bus. 

The second one is a [Channel Adapter](http://www.enterpriseintegrationpatterns.com/patterns/messaging/ChannelAdapter.html){:target="_blank"} endpoint that connects a Message Channel to some other system or transport. Channel Adapters may be either inbound (towards a channel) or outbound (from a channel). Spring Integration Kafka ships with an inbound `KafkaMessageDrivenChannelAdapter` which uses a spring-kafka `KafkaMessageListenerContainer` or `ConcurrentListenerContainer` to receive messages from Kafka topics.

<figure>
    <img src="{{ site.url }}/assets/images/spring-kafka/spring-kafka-spring-integration-example.png" alt="spring kafka spring integration example">
</figure>

Our example will consist out of two channels as shown in above diagram. The first _ProducingChannel_ will have a `kafkaMessageHandler` that subscribes to the channel and writes all received messages to a <var>'spring-integration-kafka.t'</var> topic.

A second _ConsumingChannel_ will connect to the same topic using a `KafkaMessageDrivenChannelAdapter`. A custom `CountDownLatchHandler` subscribes to this second channel and lowers a `CountDownLatch` in addition to logging the received message.

# Spring Integration Kafka Producer Channel

We start by defining the _ProducingChannel_ as a `DirectChannel` bean. This is the default channel provided by the framework, but you can use any of the [message channels Spring Integration provides](http://docs.spring.io/spring-integration/docs/current/reference/html/messaging-channels-section.html){:target="_blank"}.

Next, we create the `KafkaProducerMessageHandler` that will send messages received from the _ProducingChannel_ towards a topic. The name of this topic is defined on the handler using the `setTopicExpression()` setter or it is obtained from the `TOPIC` message header. We will use the latter in this example as you will see in the unit test case further below.

To illustrate that static values can also be set directly on the adapter, we assign a fix <var>'kafka-integration'</var> `kafka_messageKey` header by using `setMessageKeyExpression()`.

The `KafkaProducerMessageHandler` constructor requires a `KafkaTemplate` to be passed as a parameter. We create the template using a `ProducerFactory` and corresponding configuration. For more detailed information you can check the [Spring Kafka Producer tutorial section]({{ site.url }}/spring-kafka-consumer-producer-example.html#create-a-spring-kafka-message-producer).

The `KafkaProducerMessageHandler` is attached to the _ProducingChannel_ using the `@ServiceActivator` annotation. As `inputChannel` we need to specify the _ProducingChannel_ as a key/value pair in order to make the link.

``` java
package com.codenotfound.kafka.integration.channel;

import java.util.HashMap;
import java.util.Map;

import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.common.serialization.StringSerializer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.expression.common.LiteralExpression;
import org.springframework.integration.annotation.ServiceActivator;
import org.springframework.integration.channel.DirectChannel;
import org.springframework.integration.kafka.outbound.KafkaProducerMessageHandler;
import org.springframework.kafka.core.DefaultKafkaProducerFactory;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.core.ProducerFactory;
import org.springframework.messaging.MessageHandler;

@Configuration
public class ProducingChannelConfig {

  @Value("${kafka.bootstrap-servers}")
  private String bootstrapServers;

  @Bean
  public DirectChannel producingChannel() {
    return new DirectChannel();
  }

  @Bean
  @ServiceActivator(inputChannel = "producingChannel")
  public MessageHandler kafkaMessageHandler() {
    KafkaProducerMessageHandler<String, String> handler =
        new KafkaProducerMessageHandler<>(kafkaTemplate());
    handler.setMessageKeyExpression(new LiteralExpression("kafka-integration"));

    return handler;
  }

  @Bean
  public KafkaTemplate<String, String> kafkaTemplate() {
    return new KafkaTemplate<>(producerFactory());
  }

  @Bean
  public ProducerFactory<String, String> producerFactory() {
    return new DefaultKafkaProducerFactory<>(producerConfigs());
  }

  @Bean
  public Map<String, Object> producerConfigs() {
    Map<String, Object> properties = new HashMap<>();
    properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
    properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
    properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
    // introduce a delay on the send to allow more messages to accumulate
    properties.put(ProducerConfig.LINGER_MS_CONFIG, 1);

    return properties;
  }
}
```

# Spring Integration Kafka Consumer Channel

Similar to the _ProducingChannel_, a _ConsumingChannel_ bean is specified, again using the `DirectChannel` channel type.

We create a `KafkaMessageDrivenChannelAdapter` that can receive messages from one or more Kafka topics. The constructor takes a `MessageListenerContainer` as an input parameter. We then connect this Channel Adapter to the _ConsumingChannel_ by using the `setOutputChannel()` method.

In order to test our setup, a `CountDownLatchHandler` bean is defined that is linked to the _ConsumingChannel_ using the `@ServiceActivator` annotation.

In this example we setup the `MessageListenerContainer` using the `KafkaMessageListenerContainer` implementation. This is very similar to what we did in the [Spring Kakfa Consumer tutorial section]({{ site.url }}/spring-kafka-consumer-producer-example.html#create-a-spring-kafka-message-consumer). As such we won't go into further details.

> One small difference to note is the fact that we set the `AUTO_OFFSET_RESET_CONFIG` to <var>'earliest'</var>. This is done to avoid that the listener "misses" messages that have been sent before it was fully initialized.

``` java
package com.codenotfound.kafka.integration.channel;

import java.util.HashMap;
import java.util.Map;

import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.integration.annotation.ServiceActivator;
import org.springframework.integration.channel.DirectChannel;
import org.springframework.integration.kafka.inbound.KafkaMessageDrivenChannelAdapter;
import org.springframework.kafka.core.ConsumerFactory;
import org.springframework.kafka.core.DefaultKafkaConsumerFactory;
import org.springframework.kafka.listener.ConcurrentMessageListenerContainer;
import org.springframework.kafka.listener.config.ContainerProperties;

@Configuration
public class ConsumingChannelConfig {

  @Value("${kafka.bootstrap-servers}")
  private String bootstrapServers;

  @Value("${kafka.topic.spring-integration-kafka}")
  private String springIntegrationKafkaTopic;

  @Bean
  public DirectChannel consumingChannel() {
    return new DirectChannel();
  }

  @Bean
  public KafkaMessageDrivenChannelAdapter<String, String> kafkaMessageDrivenChannelAdapter() {
    KafkaMessageDrivenChannelAdapter<String, String> kafkaMessageDrivenChannelAdapter =
        new KafkaMessageDrivenChannelAdapter<>(kafkaListenerContainer());
    kafkaMessageDrivenChannelAdapter.setOutputChannel(consumingChannel());

    return kafkaMessageDrivenChannelAdapter;
  }

  @Bean
  @ServiceActivator(inputChannel = "consumingChannel")
  public CountDownLatchHandler countDownLatchHandler() {
    return new CountDownLatchHandler();
  }

  @SuppressWarnings("unchecked")
  @Bean
  public ConcurrentMessageListenerContainer<String, String> kafkaListenerContainer() {
    ContainerProperties containerProps = new ContainerProperties(springIntegrationKafkaTopic);

    return (ConcurrentMessageListenerContainer<String, String>) new ConcurrentMessageListenerContainer<>(
        consumerFactory(), containerProps);
  }

  @Bean
  public ConsumerFactory<?, ?> consumerFactory() {
    return new DefaultKafkaConsumerFactory<>(consumerConfigs());
  }

  @Bean
  public Map<String, Object> consumerConfigs() {
    Map<String, Object> properties = new HashMap<>();
    properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
    properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
    properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
    properties.put(ConsumerConfig.GROUP_ID_CONFIG, "spring-integration");
    // automatically reset the offset to the earliest offset
    properties.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");

    return properties;
  }
}
```

In order to be able to verify the correct working of our two connected channels, we will create a basic `CountDownLatchHandler` class that implements the `MessageHandler` interface. Messages from the attached _ConsumingChannel_ are logged and a `CountDownLatch` is lowered per message.

``` java
package com.codenotfound.kafka.integration.channel;

import java.util.concurrent.CountDownLatch;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.messaging.Message;
import org.springframework.messaging.MessageHandler;
import org.springframework.messaging.MessagingException;

public class CountDownLatchHandler implements MessageHandler {

  private static final Logger LOGGER = LoggerFactory.getLogger(CountDownLatchHandler.class);

  private CountDownLatch latch = new CountDownLatch(10);

  public CountDownLatch getLatch() {
    return latch;
  }

  @Override
  public void handleMessage(Message<?> message) throws MessagingException {
    LOGGER.info("received message='{}'", message);
    latch.countDown();
  }
}
```

# Spring Integration Kafka Test

Let's test the example using below `SpringKafkaIntegrationApplicationTest` unit test case. We setup an embedded Kafka broker using the JUnit `@ClassRule` annotation as we have seen in a previous [Spring Kafka test example]({{ site.url }}/spring-kafka-embedded-unit-test-example.html).

In order to get hold of our _ProducingChannel_, we auto-wire the `ApplicationContext` and use the `getBean()` method. We then create a for loop in which we sent 10 messages to the <var>'spring-integration-kafka.t'</var> topic using the channel's `send()` method. Note that we set the topic by adding a message header `Map` which contains the `KafkaHeaders.TOPIC` value which corresponds to the destination topic name.

The sent messages should be picked up by the _ConsumingChannel_ bean and when passed to the `CountDownLatchHandler` the `CountDownLatch` will be lowered from its initial value <var>'10'</var>. We then check if the 10 messages have been receive by asserting that the `CountDownLatch` value is equals to <var>'0'</var>.

``` java
package com.codenotfound.kafka.integration;

import static org.assertj.core.api.Assertions.assertThat;

import java.util.Collections;
import java.util.Map;
import java.util.concurrent.TimeUnit;

import org.junit.ClassRule;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.context.ApplicationContext;
import org.springframework.kafka.support.KafkaHeaders;
import org.springframework.kafka.test.rule.KafkaEmbedded;
import org.springframework.messaging.MessageChannel;
import org.springframework.messaging.support.GenericMessage;
import org.springframework.test.context.junit4.SpringRunner;

import com.codenotfound.kafka.integration.channel.CountDownLatchHandler;

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringKafkaIntegrationApplicationTest {

  private static final Logger LOGGER =
      LoggerFactory.getLogger(SpringKafkaIntegrationApplicationTest.class);

  @Autowired
  private ApplicationContext applicationContext;

  @Autowired
  private CountDownLatchHandler countDownLatchHandler;

  private static String SPRING_INTEGRATION_KAFKA_TOPIC = "spring-integration-kafka.t";

  @ClassRule
  public static KafkaEmbedded embeddedKafka =
      new KafkaEmbedded(1, true, SPRING_INTEGRATION_KAFKA_TOPIC);

  @Test
  public void testIntegration() throws Exception {
    MessageChannel producingChannel =
        applicationContext.getBean("producingChannel", MessageChannel.class);

    Map<String, Object> headers =
        Collections.singletonMap(KafkaHeaders.TOPIC, SPRING_INTEGRATION_KAFKA_TOPIC);

    LOGGER.info("sending 10 messages");
    for (int i = 0; i < 10; i++) {
      GenericMessage<String> message =
          new GenericMessage<>("Hello Spring Integration Kafka " + i + "!", headers);
      producingChannel.send(message);
      LOGGER.info("sent message='{}'", message);
    }

    countDownLatchHandler.getLatch().await(10000, TimeUnit.MILLISECONDS);
    assertThat(countDownLatchHandler.getLatch().getCount()).isEqualTo(0);
  }
}
```

Run the test case by opening a command prompt and issue following Maven command:

``` plaintext
mvn test
```

Maven will do the needed and the outcome of the test should show 10 messages being sent and received with a successful build as a result.

``` plaintext
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.5.4.RELEASE)

08:15:06.232 [main] INFO  c.c.k.i.SpringKafkaIntegrationApplicationTest - Starting SpringKafkaIntegrationApplicationTest on cnf-pc with PID 4872 (started by CodeNotFound in c:\codenotfound\spring-kafka\spring-kafka-integration-helloworld)
08:15:06.233 [main] INFO  c.c.k.i.SpringKafkaIntegrationApplicationTest - No active profile set, falling back to default profiles: default
08:15:07.454 [main] INFO  c.c.k.i.SpringKafkaIntegrationApplicationTest - Started SpringKafkaIntegrationApplicationTest in 1.505 seconds (JVM running for 5.936)
08:15:07.639 [main] INFO  c.c.k.i.SpringKafkaIntegrationApplicationTest - sending 10 messages
08:15:07.683 [main] INFO  c.c.k.i.SpringKafkaIntegrationApplicationTest - sent message='GenericMessage [payload=Hello Spring Integration Kafka 0!, headers={kafka_topic=spring-integration-kafka.t, id=16be84e9-cf8d-dcab-c1b4-0c48d65b53ff, timestamp=1494656107640}]'
08:15:07.691 [main] INFO  c.c.k.i.SpringKafkaIntegrationApplicationTest - sent message='GenericMessage [payload=Hello Spring Integration Kafka 1!, headers={kafka_topic=spring-integration-kafka.t, id=f7412b5f-20b9-3668-d5fa-d395a633ba31, timestamp=1494656107685}]'
08:15:07.691 [main] INFO  c.c.k.i.SpringKafkaIntegrationApplicationTest - sent message='GenericMessage [payload=Hello Spring Integration Kafka 2!, headers={kafka_topic=spring-integration-kafka.t, id=9e0f6210-f9ec-47db-2257-189f240f8c2f, timestamp=1494656107691}]'
08:15:07.691 [main] INFO  c.c.k.i.SpringKafkaIntegrationApplicationTest - sent message='GenericMessage [payload=Hello Spring Integration Kafka 3!, headers={kafka_topic=spring-integration-kafka.t, id=820874b4-6f73-e4e0-6f71-18c10fb2bb7f, timestamp=1494656107691}]'
08:15:07.692 [main] INFO  c.c.k.i.SpringKafkaIntegrationApplicationTest - sent message='GenericMessage [payload=Hello Spring Integration Kafka 4!, headers={kafka_topic=spring-integration-kafka.t, id=5447b799-0d7d-6b81-159c-01bfeae56ccf, timestamp=1494656107691}]'
08:15:07.692 [main] INFO  c.c.k.i.SpringKafkaIntegrationApplicationTest - sent message='GenericMessage [payload=Hello Spring Integration Kafka 5!, headers={kafka_topic=spring-integration-kafka.t, id=4093e7fb-c44c-8934-e5c2-09bf1009d0f4, timestamp=1494656107692}]'
08:15:07.692 [main] INFO  c.c.k.i.SpringKafkaIntegrationApplicationTest - sent message='GenericMessage [payload=Hello Spring Integration Kafka 6!, headers={kafka_topic=spring-integration-kafka.t, id=73439ebc-20af-5b58-49e4-fec30cfb3e7d, timestamp=1494656107692}]'
08:15:07.692 [main] INFO  c.c.k.i.SpringKafkaIntegrationApplicationTest - sent message='GenericMessage [payload=Hello Spring Integration Kafka 7!, headers={kafka_topic=spring-integration-kafka.t, id=dd566697-bd30-0a4c-a878-f28b27fa4b83, timestamp=1494656107692}]'
08:15:07.692 [main] INFO  c.c.k.i.SpringKafkaIntegrationApplicationTest - sent message='GenericMessage [payload=Hello Spring Integration Kafka 8!, headers={kafka_topic=spring-integration-kafka.t, id=37c4933d-f8c7-06f8-0f6c-29364146684b, timestamp=1494656107692}]'
08:15:07.692 [main] INFO  c.c.k.i.SpringKafkaIntegrationApplicationTest - sent message='GenericMessage [payload=Hello Spring Integration Kafka 9!, headers={kafka_topic=spring-integration-kafka.t, id=4746f1b2-e8e5-42b0-0d8e-df79e49cd109, timestamp=1494656107692}]'
08:15:08.796 [kafkaListenerContainer-0-C-1] INFO  c.c.k.i.c.CountDownLatchHandler - received message='GenericMessage [payload=Hello Spring Integration Kafka 0!, headers={kafka_offset=0, kafka_receivedMessageKey=kafka-integration, kafka_receivedPartitionId=0, kafka_receivedTopic=spring-integration-kafka.t}]'
08:15:08.797 [kafkaListenerContainer-0-C-1] INFO  c.c.k.i.c.CountDownLatchHandler - received message='GenericMessage [payload=Hello Spring Integration Kafka 1!, headers={kafka_offset=1, kafka_receivedMessageKey=kafka-integration, kafka_receivedPartitionId=0, kafka_receivedTopic=spring-integration-kafka.t}]'
08:15:08.797 [kafkaListenerContainer-0-C-1] INFO  c.c.k.i.c.CountDownLatchHandler - received message='GenericMessage [payload=Hello Spring Integration Kafka 2!, headers={kafka_offset=2, kafka_receivedMessageKey=kafka-integration, kafka_receivedPartitionId=0, kafka_receivedTopic=spring-integration-kafka.t}]'
08:15:08.797 [kafkaListenerContainer-0-C-1] INFO  c.c.k.i.c.CountDownLatchHandler - received message='GenericMessage [payload=Hello Spring Integration Kafka 3!, headers={kafka_offset=3, kafka_receivedMessageKey=kafka-integration, kafka_receivedPartitionId=0, kafka_receivedTopic=spring-integration-kafka.t}]'
08:15:08.797 [kafkaListenerContainer-0-C-1] INFO  c.c.k.i.c.CountDownLatchHandler - received message='GenericMessage [payload=Hello Spring Integration Kafka 4!, headers={kafka_offset=4, kafka_receivedMessageKey=kafka-integration, kafka_receivedPartitionId=0, kafka_receivedTopic=spring-integration-kafka.t}]'
08:15:08.797 [kafkaListenerContainer-0-C-1] INFO  c.c.k.i.c.CountDownLatchHandler - received message='GenericMessage [payload=Hello Spring Integration Kafka 5!, headers={kafka_offset=5, kafka_receivedMessageKey=kafka-integration, kafka_receivedPartitionId=0, kafka_receivedTopic=spring-integration-kafka.t}]'
08:15:08.797 [kafkaListenerContainer-0-C-1] INFO  c.c.k.i.c.CountDownLatchHandler - received message='GenericMessage [payload=Hello Spring Integration Kafka 6!, headers={kafka_offset=6, kafka_receivedMessageKey=kafka-integration, kafka_receivedPartitionId=0, kafka_receivedTopic=spring-integration-kafka.t}]'
08:15:08.797 [kafkaListenerContainer-0-C-1] INFO  c.c.k.i.c.CountDownLatchHandler - received message='GenericMessage [payload=Hello Spring Integration Kafka 7!, headers={kafka_offset=7, kafka_receivedMessageKey=kafka-integration, kafka_receivedPartitionId=0, kafka_receivedTopic=spring-integration-kafka.t}]'
08:15:08.798 [kafkaListenerContainer-0-C-1] INFO  c.c.k.i.c.CountDownLatchHandler - received message='GenericMessage [payload=Hello Spring Integration Kafka 8!, headers={kafka_offset=8, kafka_receivedMessageKey=kafka-integration, kafka_receivedPartitionId=0, kafka_receivedTopic=spring-integration-kafka.t}]'
08:15:08.798 [kafkaListenerContainer-0-C-1] INFO  c.c.k.i.c.CountDownLatchHandler - received message='GenericMessage [payload=Hello Spring Integration Kafka 9!, headers={kafka_offset=9, kafka_receivedMessageKey=kafka-integration, kafka_receivedPartitionId=0, kafka_receivedTopic=spring-integration-kafka.t}]'
08:15:10.345 [main] ERROR o.a.zookeeper.server.ZooKeeperServer - ZKShutdownHandler is not registered, so ZooKeeper server won't take any action on ERROR or SHUTDOWN server state changes
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 9.164 sec - in com.codenotfound.kafka.integration.SpringKafkaIntegrationApplicationTest

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 11.747 s
[INFO] Finished at: 2017-05-13T08:15:11+02:00
[INFO] Final Memory: 17M/227M
[INFO] ------------------------------------------------------------------------
```

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-kafka/tree/master/spring-kafka-spring-integration-helloworld).
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This concludes the Spring Kafka Integration example in which we demonstrated how you can consume "from" and produce "to" a Kafka topic.

Leave a comment if you have some questions or if you just enjoyed this post. Thanks!
