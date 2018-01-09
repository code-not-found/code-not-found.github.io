---
title: "Spring Kafka - Consumer Producer Example"
permalink: /spring-kafka-consumer-producer-example.html
excerpt: "A detailed step-by-step tutorial on how to implement an Apache Kafka Consumer and Producer using Spring Kafka and Spring Boot."
date: 2016-09-20
last_modified_at: 2016-09-20
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
* Spring Kafka 1.3
* Spring Boot 1.5
* Maven 3.5

> Note that Spring Kafka 1.3 uses the Apache Kafka 0.11.0.x, 1.0.x clients. For more information consult [the complete Kafka client compatibility list](https://projects.spring.io/spring-kafka/#kafka-client-compatibility){:target="_blank"}.

We will be building and running our example using [Apache Maven](https://maven.apache.org/){:target="_blank"}. Shown below is the XML representation of our Maven project in a POM file. It contains the needed dependencies for compiling and running the example.

In order to run the Kafka consumer and producer, we will use the [Spring Boot](https://projects.spring.io/spring-boot/){:target="_blank"} project. To facilitate the management of the different Spring dependencies, [Spring Boot Starters](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters){:target="_blank"} are used which are a set of convenient dependency descriptors that you can include in your application.

The `spring-boot-starter` dependency is the core starter, it includes auto-configuration, logging, and YAML support. The `spring-boot-starter-test` includes the dependencies for testing Spring Boot applications with libraries that include [JUnit](http://junit.org/junit4/){:target="_blank"}, [Hamcrest](http://hamcrest.org/JavaHamcrest/){:target="_blank"} and [Mockito](http://site.mockito.org/){:target="_blank"}.

To avoid having to manage the version compatibility of the different Spring dependencies, we will inherit the defaults from the `spring-boot-starter-parent` parent POM.

A dependency to `spring-kafka` is added in addition to a property that specifies the version. At the time of writing the latest stable release was <var>'1.3.2.RELEASE'</var>.

We also include `spring-kafka-test` in order to have access to an embedded Kafka broker when running our unit test.

In the plugins section, we included the `spring-boot-maven-plugin` Maven plugin so that we can build a single, runnable "uber-jar", which is convenient to execute and transport our written code. In addition, the plugin allowq us to start the example via a Maven command.

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
    <version>1.5.9.RELEASE</version>
  </parent>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <java.version>1.8</java.version>

    <spring-kafka.version>1.3.2.RELEASE</spring-kafka.version>
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

Spring Boot is used in order to make a Spring Kafka example application that you can "just run". We start by creating a `SpringKafkaApplication` class which contains the `main()` method that uses Spring Boot's `SpringApplication.run()` to launch the application.

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

> This example will send/receive a simple `String`. If you would like to send more complex objects you could, for example, use an [Avro Kafka serializer]({{ site.url }}/spring-kafka-apache-avro-serializer-deserializer-example.html) or the [Kafka Jsonserializer]({{ site.url }}/spring-kafka-json-serializer-deserializer-example.html) that ships with Spring Kafka.

We also create an <var>application.yml</var> [YAML](http://yaml.org/){:target="_blank"} properties file under <var>src/main/resources</var>. Properties from this file will be [injected by Spring Boot](https://docs.spring.io/spring-boot/docs/1.5.9.RELEASE/reference/html/boot-features-external-config.html#boot-features-external-config){:target="_blank"} into our configuration beans using the `@Value` annotation.

``` yaml
kafka:
  bootstrap-servers: localhost:9092
  topic:
    helloworld: helloworld.t
```

# Create a Spring Kafka Message Producer

For sending messages we will be using the `KafkaTemplate` which wraps a `Producer` and provides [convenience methods](https://docs.spring.io/spring-kafka/docs/1.3.2.RELEASE/reference/html/_reference.html#kafka-template){:target="_blank"} to send data to Kafka topics. The template provides asynchronous send methods which return a `ListenableFuture`.

In the `Sender` class, the `KafkaTemplate` is auto-wired as the creation will be done further below in a separate `SenderConfig` class.

For this example we will use the `send()` method that takes as input a topic name and a `String` payload that needs to be sent.

> Note that the Kafka broker default settings cause it to auto-create a topic when a request for an unknown topic is received.

``` java
package com.codenotfound.kafka.producer;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.core.KafkaTemplate;

public class Sender {

  private static final Logger LOGGER =
      LoggerFactory.getLogger(Sender.class);

  @Autowired
  private KafkaTemplate<String, String> kafkaTemplate;

  public void send(String topic, String payload) {
    LOGGER.info("sending payload='{}' to topic='{}'", payload, topic);
    kafkaTemplate.send(topic, payload);
  }
}

The creation of the `KafkaTemplate` and `Sender` is handled in the `SenderConfig` class. The class is annotated with `@Configuration` which [indicates](http://docs.spring.io/spring/docs/4.3.13.RELEASE/spring-framework-reference/html/beans.html#beans-java-basic-concepts){:target="_blank"} that the class can be used by the Spring IoC container as a source of bean definitions.

In order to be able to use the Spring Kafka template, we need to configure a `ProducerFactory` and provide it in the template's constructor.

The producer factory needs to be set with a number of mandatory properties amongst which the <var>'BOOTSTRAP_SERVERS_CONFIG'</var> property that specifies a list of _host:port_ pairs used for establishing the initial connections to the Kafka cluster. Note that this value is configurable as it is fetched from the <var>application.yml</var> configuration file.

A message in Kafka is a key-value pair with a small amount of associated metadata. As Kafka stores and transports `Byte` arrays, we need to specify the format from which the key and value will be serialized. In this example we are sending a `String` as payload, as such we specify the `StringSerializer` class which will take care of the needed transformation.

For a complete list of the other configuration parameters, you can consult the [Kafka ProducerConfig API](https://kafka.apache.org/0110/javadoc/org/apache/kafka/clients/producer/ProducerConfig.html){:target="_blank"}.

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
    props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG,
        bootstrapServers);
    props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG,
        StringSerializer.class);
    props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,
        StringSerializer.class);

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

The `@KafkaListener` annotation creates a `ConcurrentMessageListenerContainer` message listener container behind the scenes for each annotated method. In order to do so, a factory bean with name `kafkaListenerContainerFactory` is expected that we will configure in the next section.

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

  private static final Logger LOGGER =
      LoggerFactory.getLogger(Receiver.class);

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

The creation and configuration of the different Spring Beans needed for the `Receiver` POJO are grouped in the `ReceiverConfig` class. Similar to the `SenderConfig` it is annotated with `@Configuration`.

> Note the `@EnableKafka` annotation which enables the detection of the `@KafkaListener` annotation that was used on the previous `Receiver` class.

The `kafkaListenerContainerFactory()` is used by the `@KafkaListener` annotation from the `Receiver` in order to configure a `MessageListenerContainer`. In order to create it, a `ConsumerFactory` and accompanying configuration `Map` is needed.

In this example, a number of mandatory properties are set amongst which the initial connection and deserializer parameters.

We also specify a <var>'GROUP_ID_CONFIG'</var> which allows to [identify the group](https://stackoverflow.com/a/41377616/4201470){:target="_blank"} this consumer belongs to. Messages will effectively be load balanced over consumer instances that have the same group id.

On top of that we also set <var>'AUTO_OFFSET_RESET_CONFIG'</var> to <kbd>"earliest"</kbd>. This ensures that our consumer reads from the beginning of the topic even if some messages were already been sent before it was able to start up.

For a complete list of the other configuration parameters, you can consult the [Kafka ConsumerConfig API](https://kafka.apache.org/0110/javadoc/index.html?org/apache/kafka/clients/consumer/ConsumerConfig.html){:target="_blank"}.

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
import org.springframework.kafka.config.KafkaListenerContainerFactory;
import org.springframework.kafka.core.ConsumerFactory;
import org.springframework.kafka.core.DefaultKafkaConsumerFactory;
import org.springframework.kafka.listener.ConcurrentMessageListenerContainer;

@Configuration
@EnableKafka
public class ReceiverConfig {

  @Value("${kafka.bootstrap-servers}")
  private String bootstrapServers;

  @Bean
  public Map<String, Object> consumerConfigs() {
    Map<String, Object> props = new HashMap<>();
    // list of host:port pairs used for establishing the initial connections to the Kafka cluster
    props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG,
        bootstrapServers);
    props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG,
        StringDeserializer.class);
    props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG,
        StringDeserializer.class);
    // allows a pool of processes to divide the work of consuming and processing records
    props.put(ConsumerConfig.GROUP_ID_CONFIG, "helloworld");
    // automatically reset the offset to the earliest offset
    props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");

    return props;
  }

  @Bean
  public ConsumerFactory<String, String> consumerFactory() {
    return new DefaultKafkaConsumerFactory<>(consumerConfigs());
  }

  @Bean
  public KafkaListenerContainerFactory<ConcurrentMessageListenerContainer<String, String>> kafkaListenerContainerFactory() {
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
 
 We then check if the `CountDownLatch` from the `Receiver` was lowered from 1 to 0 as this indicates a message was processed by the `receive()` method.

An embedded Kafka broker is automatically started by using a `@ClassRule`. Check out following [Spring Kafka test example]({{ site.url }}/spring-kafka-embedded-unit-test-example.html) for more detailed information on this topic.

As the embedded server is started on a random port, we provide a dedicated <var>src/test/resources/apppication.yml</var> properties file for testing which uses the `spring.embedded.kafka.brokers` system property that the `@ClassRule` sets to the address of the broker(s).

``` yaml
kafka:
  bootstrap-servers: ${spring.embedded.kafka.brokers}
  topic:
    helloworld: helloworld.t
```

> Below test case can also be executed after you [install Kafka and Zookeeper]({{ site.url }}/apache-kafka-download-installation.html) on your local system. Just comment out the `@ClassRule` and change the <var>'bootstrap-servers'</var> property of the application properties file located in <var>src/test/resources</var> to the address of the local broker.

``` java
package com.codenotfound.kafka;

import static org.assertj.core.api.Assertions.assertThat;

import java.util.concurrent.TimeUnit;

import org.junit.ClassRule;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.kafka.test.rule.KafkaEmbedded;
import org.springframework.test.annotation.DirtiesContext;
import org.springframework.test.context.junit4.SpringRunner;

import com.codenotfound.kafka.consumer.Receiver;
import com.codenotfound.kafka.producer.Sender;

@RunWith(SpringRunner.class)
@SpringBootTest
@DirtiesContext
public class SpringKafkaApplicationTest {

  private static final String HELLOWORLD_TOPIC = "helloworld.t";

  @ClassRule
  public static KafkaEmbedded embeddedKafka =
      new KafkaEmbedded(1, true, HELLOWORLD_TOPIC);

  @Autowired
  private Receiver receiver;

  @Autowired
  private Sender sender;

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
 :: Spring Boot ::        (v1.5.9.RELEASE)

20:36:16.436 [main] INFO  c.c.kafka.SpringKafkaApplicationTest - Starting SpringKafkaApplicationTest on cnf-pc with PID 2676 (started by CodeNotFound in c:\codenotfound\code\spring-kafka\spring-kafka-helloworld)
20:36:16.437 [main] INFO  c.c.kafka.SpringKafkaApplicationTest - No active profile set, falling back to default profiles: default
20:36:17.240 [main] INFO  c.c.kafka.SpringKafkaApplicationTest - Started SpringKafkaApplicationTest in 1.183 seconds (JVM running for 5.671)
20:36:17.347 [main] INFO  c.codenotfound.kafka.producer.Sender - sending payload='Hello Spring Kafka!' to topic='helloworld.t'
20:36:17.683 [org.springframework.kafka.KafkaListenerEndpointContainer#0-0-C-1] INFO  c.c.kafka.consumer.Receiver - received payload='Hello Spring Kafka!'
20:36:20.298 [main] ERROR o.a.zookeeper.server.ZooKeeperServer - ZKShutdownHandler is not registered, so ZooKeeper server won't take any action on ERROR or SHUTDOWN server state changes
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 9.086 sec - in com.codenotfound.kafka.SpringKafkaApplicationTest

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 11.521 s
[INFO] Finished at: 2018-01-09T20:36:21+01:00
[INFO] Final Memory: 17M/226M
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
