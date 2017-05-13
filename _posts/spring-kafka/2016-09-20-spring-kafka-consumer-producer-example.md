---
title: "Spring Kafka - Consumer Producer Example"
permalink: /2016/09/spring-kafka-consumer-producer-example.html
excerpt: "A detailed step-by-step tutorial on how to implement an Apache Kafka Consumer and Producer using Spring Kafka and Spring Boot."
date: 2016-09-20
modified: 2016-09-20
header:
  teaser: "assets/images/spring-kafka-teaser.jpg"
categories: [Spring Kafka]
tags: [Apache Kafka, Consumer, Example, Hello World, Maven, Producer, Spring, Spring Boot, Spring Kafka, Tutorial]
redirect_from:
  - /2016/09/spring-kafka-hello-world-consumer-producer-example.html
  - /2016/09/spring-kafka-hello-world-consumer.html
published: true
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/spring-logo.jpg" alt="spring logo" class="logo">
</figure>

The [Spring for Apache Kafka (spring-kafka) project](https://projects.spring.io/spring-kafka/) applies core Spring concepts to the development of Kafka-based messaging solutions. It provides a 'template' as a high-level abstraction for sending messages. It also provides support for Message-driven POJOs with `@KafkaListener` annotations and a 'listener container'.

In the following tutorial we will configure, build and run a Hello World example in which we will send/receive messages to/from Apache Kafka using Spring Kafka, Spring Boot and Maven.

Checkout following link for more [Spring Kafka tutorials]({{ site.url }}/spring-kafka/).
{: .notice--primary}

Tools used:
* Spring Kafka 1.2
* Spring Boot 1.5
* Maven 3.5

> Note that Spring Kafka 1.2 uses the Apache Kafka 0.10.2.x client.

# General Project Setup

We start by defining a Maven POM file which contains the dependencies for the needed [Spring projects](https://spring.io/projects). The POM inherits from the `spring-boot-starter-parent` project and declares dependencies to `spring-boot-starter` and `spring-boot-starter-test` [Spring Boot starters](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters).

A dependency to `spring-kafka` is added in addition to a property that specifies the version. At the time of writing the latest stable release was <var>'1.2.0.RELEASE'</var>. We also include `spring-kafka-test` in order to have access to an embedded Kafka broker when running our unit test.

The `spring-boot-maven-plugin` Maven plugin is added so that we can build a single, runnable "uber-jar", which is convenient to execute and transport our written code.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>spring-kafka-helloworld</artifactId>
  <version>0.0.1-SNAPSHOT</version>

  <name>spring-kafka-helloworld</name>
  <description>Spring Kafka - Consumer Producer Example</description>
  <url>https://www.codenotfound.com/2016/09/spring-kafka-consumer-producer-example.html</url>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.3.RELEASE</version>
  </parent>

  <properties>
    <java.version>1.8</java.version>

    <spring-kafka.version>1.2.1.RELEASE</spring-kafka.version>
  </properties>

  <dependencies>
    <!-- spring-boot -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
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

We will use Spring Boot in order to make a Spring Kafka example application that you can "just run". We start by creating an `SpringKafkaApplication` which contains the `main()` method that uses Spring Boot's `SpringApplication.run()` method to launch the application. The `@SpringBootApplication` annotation is a convenience annotation that adds: `@Configuration`, `@EnableAutoConfiguration` and `@ComponentScan`.

For more information on Spring Boot you can check out the [Spring Boot getting started guide](https://spring.io/guides/gs/spring-boot/).

``` java
package com.codenotfound.kafka;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringKafkaApplication {

  public static void main(String[] args) {
    SpringApplication.run(SpringKafkaApplication.class, args);
  }
}
```

> The below sections will detail how to create a sender and receiver together with their respective configurations. Note that it is also possible to have [Spring Boot autoconfigure Spring Kafka]({{ site.url }}/2017/04/spring-kafka-boot-example.html) using default values so that actual code that needs to be written is reduced to a bare minimum.

> This example will send/receive a simple `String`. If you would like to send more complex objects you could for example use an [Avro Kafka serializer]({{ site.url }}/2017/03/spring-kafka-apache-avro-example.html) or the [Kafka Jsonserializer]({{ site.url }}/2017/03/spring-kafka-json-serializer-example.html) that ships with Spring Kafka.

# Create a Spring Kafka Message Producer

For sending messages we will be using the `KafkaTemplate` which wraps a `Producer` and provides convenience methods to send data to Kafka topics. The template provides both asynchronous and synchronous send methods, with the asynchronous methods returning a `Future`.

In the below `Sender` class, the `KafkaTemplate` is autowired as the actual creation of the `Bean` will be done in a separate `SenderConfig` class.

In this example we will be using the async `send()` method and we will also configure the returned `ListenableFuture` with a `ListenableFutureCallback` to get an async callback with the results of the send (success or failure) instead of waiting for the `Future` to complete. If the sending of a message is successful we simply create a log statement.

> Note that a Kafka broker by default auto-creates a topic when it receives a request for an unknown topic.

``` java
package com.codenotfound.kafka.producer;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.support.SendResult;
import org.springframework.util.concurrent.ListenableFuture;
import org.springframework.util.concurrent.ListenableFutureCallback;

public class Sender {

  private static final Logger LOGGER = LoggerFactory.getLogger(Sender.class);

  @Autowired
  private KafkaTemplate<String, String> kafkaTemplate;

  public void send(String topic, String message) {
    // the KafkaTemplate provides asynchronous send methods returning a Future
    ListenableFuture<SendResult<String, String>> future = kafkaTemplate.send(topic, message);

    // register a callback with the listener to receive the result of the send asynchronously
    future.addCallback(new ListenableFutureCallback<SendResult<String, String>>() {

      @Override
      public void onSuccess(SendResult<String, String> result) {
        LOGGER.info("sent message='{}' with offset={}", message,
            result.getRecordMetadata().offset());
      }

      @Override
      public void onFailure(Throwable ex) {
        LOGGER.error("unable to send message='{}'", message, ex);
      }
    });

    // or, to block the sending thread to await the result, invoke the future's get() method
  }
}
```

The creation of the `KafkaTemplate` and `Sender` is handled in the `SenderConfig` class. The class is annoted with `@Configuration` which indicates that the class can be used by the Spring IoC container as a source of bean definitions.

In order to be able to use the Spring Kafka template we need to configure a `ProducerFactory` and provide it in the template's constructor. The producer factory needs to be set with a number of properties amongst which the <var>'BOOTSTRAP_SERVERS_CONFIG'</var> property that is fetched from the <var>application.yml</var> configuration file.

For a complete list of the available configuration parameters you can consult the [Kafka ProducerConfig API](https://kafka.apache.org/0100/javadoc/org/apache/kafka/clients/producer/ProducerConfig.html).

``` java
package com.codenotfound.kafka.producer;

import java.util.HashMap;
import java.util.Map;

import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.common.serialization.StringSerializer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.core.DefaultKafkaProducerFactory;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.core.ProducerFactory;

@Configuration
public class SenderConfig {

  @Value("${kafka.bootstrap-servers}")
  private String bootstrapServers;

  @Bean
  public Map<String, Object> producerConfigs() {
    Map<String, Object> props = new HashMap<>();
    // list of host:port pairs used for establishing the initial connections to the Kakfa cluster
    props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
    props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
    props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);

    return props;
  }

  @Bean
  public ProducerFactory<String, String> producerFactory() {
    return new DefaultKafkaProducerFactory<>(producerConfigs());
  }

  @Bean
  public KafkaTemplate<String, String> kafkaTemplate() {
    return new KafkaTemplate<>(producerFactory());
  }

  @Bean
  public Sender sender() {
    return new Sender();
  }
}
```

# Create a Spring Kafka Message Consumer

Like with any messaging-based application, you need to create a receiver that will handle the published messages. The `Receiver` is nothing more than a simple POJO that defines a method for receiving messages. In the below example we named the method `receive()`, but you can name it anything you like.

The `@KafkaListener` annotation creates a message listener container behind the scenes for each annotated method, using a `ConcurrentMessageListenerContainer`. By default, a bean with name `kafkaListenerContainerFactory` is expected that we will setup in the next section. Using the `topics` element, we specify the topics for this listener.

For more information on the other available elements, you can consult the [KafkaListener API documentation](http://docs.spring.io/spring-kafka/api/org/springframework/kafka/annotation/KafkaListener.html).

> For testing convenience, we added a `CountDownLatch`. This allows the POJO to signal that a message is received. This is something you are not likely to implement in a production application. 

``` java
package com.codenotfound.kafka.consumer;

import java.util.concurrent.CountDownLatch;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.kafka.annotation.KafkaListener;

public class Receiver {

  private static final Logger LOGGER = LoggerFactory.getLogger(Receiver.class);

  private CountDownLatch latch = new CountDownLatch(1);

  public CountDownLatch getLatch() {
    return latch;
  }

  @KafkaListener(topics = "${kafka.topic.helloworld}")
  public void receive(String message) {
    LOGGER.info("received message='{}'", message);
    latch.countDown();
  }
}
```

The creation and configuration of the different Spring Beans needed for the `Receiver` POJO are grouped in the `ReceiverConfig` class. Note that we need to add the `@EnableKafka` annotation to enable support for the `@KafkaListener` annotation that was used on the `Receiver`.

The `kafkaListenerContainerFactory()` is used by the `@KafkaListener` annotation from the `Receiver`. In order to create it, a `ConsumerFactory` and accompanying configuration `Map` is needed. Apart from the <var>'AUTO_OFFSET_RESET_CONFIG'</var> property all configuration parameters are mandatory, for a complete list consult the [Kafka ConsumerConfig API](https://kafka.apache.org/0100/javadoc/index.html?org/apache/kafka/clients/consumer/ConsumerConfig.html). 

``` java
package com.codenotfound.kafka.consumer;

import java.util.HashMap;
import java.util.Map;

import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.annotation.EnableKafka;
import org.springframework.kafka.config.ConcurrentKafkaListenerContainerFactory;
import org.springframework.kafka.core.ConsumerFactory;
import org.springframework.kafka.core.DefaultKafkaConsumerFactory;

@Configuration
@EnableKafka
public class ReceiverConfig {

  @Value("${kafka.bootstrap-servers}")
  private String bootstrapServers;

  @Bean
  public Map<String, Object> consumerConfigs() {
    Map<String, Object> props = new HashMap<>();
    // list of host:port pairs used for establishing the initial connections to the Kakfa cluster
    props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
    props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
    props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
    // allows a pool of processes to divide the work of consuming and processing records
    props.put(ConsumerConfig.GROUP_ID_CONFIG, "helloworld");

    return props;
  }

  @Bean
  public ConsumerFactory<String, String> consumerFactory() {
    return new DefaultKafkaConsumerFactory<>(consumerConfigs());
  }

  @Bean
  public ConcurrentKafkaListenerContainerFactory<String, String> kafkaListenerContainerFactory() {
    ConcurrentKafkaListenerContainerFactory<String, String> factory =
        new ConcurrentKafkaListenerContainerFactory<>();
    factory.setConsumerFactory(consumerFactory());

    return factory;
  }

  @Bean
  public Receiver receiver() {
    return new Receiver();
  }
}
```

# Testing the Spring Kafka Template & Listener

In order to verify that we are able to send and receive a message to and from Kafka, a basic `SpringKafkaApplicationTest` test case is used. It contains a `testReceiver()` unit test case that uses the `Sender` to send a message to the <var>'helloworld.t'</var> topic on the Kafka bus. We then use the `CountDownLatch` from the `Receiver` to verify that a message was received.

An embedded Kafka broker is automatically started by using a `@ClassRule`. Check out following [Spring Kafka test example]({{ site.url }}/2016/10/spring-kafka-embedded-server-unit-test.html) for more detailed information on this topic.

Below test case can also be executed after you [install kafka and zookeeper]({{ site.url }}/2016/09/apache-kafka-download-installation.html) on your local system. You just need to comment out the lines annotated with `@ClassRule` and `@BeforeClass`.

``` java
package com.codenotfound.kafka;

import static org.assertj.core.api.Assertions.assertThat;

import java.util.concurrent.TimeUnit;

import org.junit.Before;
import org.junit.BeforeClass;
import org.junit.ClassRule;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.kafka.config.KafkaListenerEndpointRegistry;
import org.springframework.kafka.listener.MessageListenerContainer;
import org.springframework.kafka.test.rule.KafkaEmbedded;
import org.springframework.kafka.test.utils.ContainerTestUtils;
import org.springframework.test.context.junit4.SpringRunner;

import com.codenotfound.kafka.consumer.Receiver;
import com.codenotfound.kafka.producer.Sender;

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringKafkaApplicationTest {

  private static String HELLOWORLD_TOPIC = "helloworld.t";

  @Autowired
  private Sender sender;

  @Autowired
  private Receiver receiver;

  @Autowired
  private KafkaListenerEndpointRegistry kafkaListenerEndpointRegistry;

  @ClassRule
  public static KafkaEmbedded embeddedKafka = new KafkaEmbedded(1, true, HELLOWORLD_TOPIC);

  @BeforeClass
  public static void setUpBeforeClass() throws Exception {
    System.setProperty("kafka.bootstrap-servers", embeddedKafka.getBrokersAsString());
  }

  @Before
  public void setUp() throws Exception {
    // wait until the partitions are assigned
    for (MessageListenerContainer messageListenerContainer : kafkaListenerEndpointRegistry
        .getListenerContainers()) {
      ContainerTestUtils.waitForAssignment(messageListenerContainer,
          embeddedKafka.getPartitionsPerTopic());
    }
  }

  @Test
  public void testReceive() throws Exception {
    sender.send(HELLOWORLD_TOPIC, "Hello Spring Kafka!");

    receiver.getLatch().await(10000, TimeUnit.MILLISECONDS);
    assertThat(receiver.getLatch().getCount()).isEqualTo(0);
  }
}
```

In order to run the above example open a command prompt and execute following Maven command: 

``` plaintext
mvn test
```

Maven will download the needed dependencies, compile the code and run the unit test case. The result should be a successful build during which following logs are generated:

``` plaintext
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.5.3.RELEASE)

00:29:56.628 [main] INFO  c.c.k.SpringKafkaApplicationTests - Starting SpringKafkaApplicationTests on cnf-pc with PID 3280 (started by CodeNotFound in c:\code\st\spring-kafka\spring-kafka-helloworld)
00:29:56.628 [main] INFO  c.c.k.SpringKafkaApplicationTests - No active profile set, falling back to default profiles: default
00:29:57.303 [main] INFO  c.c.k.SpringKafkaApplicationTests - Started SpringKafkaApplicationTests in 0.991 seconds (JVM running for 5.285)
00:29:57.443 [kafka-producer-network-thread | producer-1] INFO  c.codenotfound.kafka.producer.Sender - sent message='Hello Spring Kafka!' with offset=0
00:29:58.649 [org.springframework.kafka.KafkaListenerEndpointContainer#0-0-L-1] INFO  c.c.kafka.consumer.Receiver - received message='Hello Spring Kafka!'
00:29:59.681 [main] ERROR o.a.zookeeper.server.ZooKeeperServer - ZKShutdownHandler is not registered, so ZooKeeper server won't take any action on ERROR or SHUTDOWN server state changes
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 8.135 sec - in com.codenotfound.kafka.SpringKafkaApplicationTests

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 40.252 s
[INFO] Finished at: 2017-04-16T00:30:30+02:00
[INFO] Final Memory: 17M/220M
[INFO] ------------------------------------------------------------------------
```

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-kafka/tree/master/spring-kafka-helloworld).
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This wraps up our example in which we used a Spring Kafka template to create a producer and Spring Kafka listener to create a consumer.

If you found this sample useful or have a question you would like to ask, drop a line below!
