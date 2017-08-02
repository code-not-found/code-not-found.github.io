---
title: "Spring Kafka - Consumer Producer Example"
permalink: /spring-kafka-consumer-producer-example.html
excerpt: "A detailed step-by-step tutorial on how to implement an Apache Kafka Consumer and Producer using Spring Kafka and Spring Boot."
date: 2016-09-20
modified: 2016-09-20
header:
  teaser: "assets/images/teaser/spring-kafka-teaser.png"
categories: [Spring Kafka]
tags: [Apache Kafka, Consumer, Example, Hello World, Maven, Producer, Spring, Spring Boot, Spring Kafka, Tutorial]
redirect_from:
  - /2016/09/spring-kafka-hello-world-consumer-producer-example.html
  - /2016/09/spring-kafka-hello-world-consumer.html
  - /2016/09/spring-kafka-consumer-producer-example.html
published: true
---

<figure>
    <img src="{{ site.url }}/assets/images/logo/spring-logo.png" alt="spring logo" class="logo">
</figure>

The [Spring for Apache Kafka (spring-kafka) project](https://projects.spring.io/spring-kafka/){:target="_blank"} applies core Spring concepts to the development of Kafka-based messaging solutions. It provides a 'template' as a high-level abstraction for sending messages. It also contains support for Message-driven POJOs with `@KafkaListener` annotations and a listener container.

In the following tutorial, we will configure, build and run a Hello World example in which we will send/receive messages to/from Apache Kafka using Spring Kafka, Spring Boot, and Maven.

If you want to learn more about Spring Kafka - head on over to the [Spring Kafka tutorials page]({{ site.url }}/spring-kafka/).
{: .notice--primary}

# General Project Setup

Tools used:
* Spring Kafka 1.2
* Spring Boot 1.5
* Maven 3.5

> Note that Spring Kafka 1.2 uses the Apache Kafka 0.10.2.x client. For more information consult [the complete Kafka client compatibility list](https://projects.spring.io/spring-kafka/#kafka-client-compatibility){:target="_blank"}. 

We start by defining a Maven POM file which contains the dependencies for the needed [Spring projects](https://spring.io/projects){:target="_blank"}. The POM inherits from the `spring-boot-starter-parent` project and declares dependencies to `spring-boot-starter` and `spring-boot-starter-test` [Spring Boot starters](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters){:target="_blank"}.

A dependency to `spring-kafka` is added in addition to a property that specifies the version. At the time of writing the latest stable release was <var>'1.2.2.RELEASE'</var>.

We also include `spring-kafka-test` in order to have access to an embedded Kafka broker when running our unit test.

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
  <url>https://www.codenotfound.com/spring-kafka-consumer-producer-example.html</url>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.4.RELEASE</version>
  </parent>

  <properties>
    <java.version>1.8</java.version>

    <spring-kafka.version>1.2.2.RELEASE</spring-kafka.version>
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

Spring Boot is used in order to make a Spring Kafka example application that you can "just run". We start by creating a `SpringKafkaApplication` which contains the `main()` method that uses Spring Boot's `SpringApplication.run()` to launch the application.

> Note that the `@SpringBootApplication` annotation is a convenience annotation that adds: `@Configuration`, `@EnableAutoConfiguration` and `@ComponentScan`.

For more information on Spring Boot, you can check out the [Spring Boot getting started guide](https://spring.io/guides/gs/spring-boot/){:target="_blank"}.

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

> The below sections will detail how to create a sender and receiver together with their respective configurations. It is also possible to have [Spring Boot autoconfigure Spring Kafka]({{ site.url }}/spring-kafka-boot-example.html) using default values so that actual code that needs to be written is reduced to a bare minimum.

> This example will send/receive a simple `String`. If you would like to send more complex objects you could for example use an [Avro Kafka serializer]({{ site.url }}/spring-kafka-apache-avro-serializer-deserializer-example.html) or the [Kafka Jsonserializer]({{ site.url }}/spring-kafka-json-serializer-deserializer-example.html) that ships with Spring Kafka.

We also create an <var>application.yml</var> [YAML](http://yaml.org/){:target="_blank"} properties file under <var>src/main/resources</var>. Properties from this file will be [injected by Spring Boot](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-external-config.html#boot-features-external-config){:target="_blank"} into our configuration beans using the `@Value` annotation.

``` yaml
kafka:
  bootstrap-servers: localhost:9092
  topic:
    helloworld: helloworld.t
```

# Create a Spring Kafka Message Producer

For sending messages we will be using the `KafkaTemplate` which wraps a `Producer` and provides [convenience methods](http://docs.spring.io/spring-kafka/docs/1.2.2.RELEASE/reference/html/_reference.html#_kafkatemplate){:target="_blank"} to send data to Kafka topics. The template provides asynchronous send methods which return a `ListenableFuture`.

In the `Sender` class, the `KafkaTemplate` is auto-wired as the creation will be done further below in a separate `SenderConfig` class.

For this example we will use the `send()` method that takes as input a topic name and the `String` payload that needs to be sent.

> Note that the Kafka broker default settings cause it to auto-create a topic when a request for an unknown topic is received.

``` java
package com.codenotfound.kafka.producer;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.core.KafkaTemplate;

public class Sender {

  private static final Logger LOGGER = LoggerFactory.getLogger(Sender.class);

  @Autowired
  private KafkaTemplate<String, String> kafkaTemplate;

  public void send(String topic, String payload) {
    LOGGER.info("sending payload='{}' to topic='{}'", payload, topic);
    kafkaTemplate.send(topic, payload);
  }
}
```

The creation of the `KafkaTemplate` and `Sender` is handled in the `SenderConfig` class. The class is annotated with `@Configuration` which [indicates](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/beans.html#beans-java-basic-concepts){:target="_blank"} that the class can be used by the Spring IoC container as a source of bean definitions.

In order to be able to use the Spring Kafka template, we need to configure a `ProducerFactory` and provide it in the template's constructor.

The producer factory needs to be set with a number of mandatory properties amongst which the <var>'BOOTSTRAP_SERVERS_CONFIG'</var> property that specifies a list of _host:port_ pairs used for establishing the initial connections to the Kafka cluster. Note that this value is configurable as it is fetched from the <var>application.yml</var> configuration file.

A message in Kafka is a key-value pair with a small amount of associated metadata. As Kafka stores and transports `Byte` arrays, we need to specify the format from which the key and value will be serialized. In this example we are sending a `String` as payload, as such we specify the `StringSerializer` class which will take care of the needed transformation.

For a complete list of the other configuration parameters, you can consult the [Kafka ProducerConfig API](https://kafka.apache.org/0102/javadoc/org/apache/kafka/clients/producer/ProducerConfig.html){:target="_blank"}.

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

The `@KafkaListener` annotation creates a message listener container behind the scenes for each annotated method, using a `ConcurrentMessageListenerContainer`. By default, a bean with name `kafkaListenerContainerFactory` is expected that we will setup in the next section.

Using the `topics` element, we specify the topics for this listener. The name of the topic is injected from the <var>application.yml</var> properties file.

For more information on the other available elements on the `KafkaListener`, you can consult the [API documentation](http://docs.spring.io/spring-kafka/api/org/springframework/kafka/annotation/KafkaListener.html){:target="_blank"}.

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
  public void receive(String payload) {
    LOGGER.info("received payload='{}'", payload);
    latch.countDown();
  }
}
```

The creation and configuration of the different Spring Beans needed for the `Receiver` POJO are grouped in the `ReceiverConfig` class.

> Note that we need to add the `@EnableKafka` annotation to enable support for the `@KafkaListener` annotation that was used on the `Receiver`.

The `kafkaListenerContainerFactory()` is used by the `@KafkaListener` annotation from the `Receiver`. In order to create it, a `ConsumerFactory` and accompanying configuration `Map` is needed.

In this example, a number of mandatory properties are set amongst which the initial connection and deserializer parameters. We also specify a <var>'GROUP_ID_CONFIG'</var> which allows to [identify the group](https://stackoverflow.com/a/41377616/4201470){:target="_blank"} this consumer belongs to. Messages will effectively be load balanced over consumer instances that have the same group id.

For a complete list of the other configuration parameters, you can consult the [Kafka ConsumerConfig API](https://kafka.apache.org/0102/javadoc/index.html?org/apache/kafka/clients/consumer/ConsumerConfig.html){:target="_blank"}.

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

 A basic `SpringKafkaApplicationTest` is provided in order to verify that we are able to send and receive a message to and from Apache Kafka. It contains a `testReceiver()` unit test case that uses the `Sender` bean to send a message to the <var>'helloworld.t'</var> topic on the Kafka bus.
 
 We then check the `CountDownLatch` from the `Receiver` to verify that a message was received.

An embedded Kafka broker is automatically started by using a `@ClassRule`. Check out following [Spring Kafka test example]({{ site.url }}/spring-kafka-embedded-unit-test-example.html) for more detailed information on this topic.

As the embedded server is started on a random port, we provide a dedicated <var>apppication.yml</var> properties file for testing in <var>src/test/resources</var> which uses the `spring.embedded.kafka.brokers` system property that the `@ClassRule` sets to the address of the broker(s).

Using `@Before` we wait until all the partitions are assigned to our `Receiver` by looping over the available `ConcurrentMessageListenerContainer` (if we don't do this the message might already be sent before the listeners are assigned to the topic).

> Below test case can also be executed after you [install Kafka and Zookeeper]({{ site.url }}/2016/09/apache-kafka-download-installation.html) on your local system. Just comment out the `@ClassRule` and change the <var>'bootstrap-servers'</var> property of the application properties file located in <var>src/test/resources</var> to the address of the local broker.

``` java
package com.codenotfound.kafka;

import static org.assertj.core.api.Assertions.assertThat;

import java.util.concurrent.TimeUnit;

import org.junit.Before;
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
 :: Spring Boot ::        (v1.5.4.RELEASE)

22:17:35.356 [main] INFO  c.c.kafka.SpringKafkaApplicationTest - Starting SpringKafkaApplicationTest on cnf-pc with PID 5772 (started by CodeNotFound in c:\codenotfound\code\spring-kafka\spring-kafka-helloworld)
22:17:35.357 [main] INFO  c.c.kafka.SpringKafkaApplicationTest - No active profile set, falling back to default profiles: default
22:17:36.033 [main] INFO  c.c.kafka.SpringKafkaApplicationTest - Started SpringKafkaApplicationTest in 0.971 seconds (JVM running for 5.054)
22:17:37.436 [main] INFO  c.codenotfound.kafka.producer.Sender - sending payload='Hello Spring Kafka!' to topic='helloworld.t'
22:17:37.500 [org.springframework.kafka.KafkaListenerEndpointContainer#0-0-C-1] INFO  c.c.kafka.consumer.Receiver - received payload='Hello Spring Kafka!'
22:17:38.528 [main] ERROR o.a.zookeeper.server.ZooKeeperServer - ZKShutdownHandler is not registered, so ZooKeeper server won't take any action on ERROR or SHUTDOWN server state changes
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 7.935 sec - in com.codenotfound.kafka.SpringKafkaApplicationTest

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 10.010 s
[INFO] Finished at: 2017-07-29T22:17:39+02:00
[INFO] Final Memory: 16M/227M
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
