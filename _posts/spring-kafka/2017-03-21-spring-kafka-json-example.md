---
title: Spring Kafka - JSON Example
permalink: /2017/03/spring-kafka-json-example.html
excerpt: A detailed step-by-step tutorial on how to send/receive JSON messages using Spring Kafka and Spring Boot.
date: 2017-03-21 21:00
categories: [Spring Kafka]
tags: [Apache Kafka, Example, Maven, JSON, JsonDeserializer, JsonSerializer, Spring, Spring Boot, Spring Kafka, Tutorial]
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/spring-logo.png" alt="spring logo">
</figure>

[JSON](http://www.json.org/) (JavaScript Object Notation) is a lightweight data-interchange format that uses human-readable text to transmit data objects. It is built on two structures: a collection of name/value pairs and an ordered list of values. The following tutorial illustrates how to send/receive a Java object as a JSON `byte[]` array to/from Apache Kafka using Spring Kafka, Spring Boot and Maven.

Tools used:
* Spring Boot 1.5
* Spring Kafka 1.1
* Maven 3

[Apache Kafka](https://kafka.apache.org/) stores and transports `Byte` arrays in its topics. It ships with a number of [built in (de)serializers](https://kafka.apache.org/0100/javadoc/org/apache/kafka/common/serialization/Serializer.html) but a JSON one is not included. Luckily, the [Spring Kafka framework](https://projects.spring.io/spring-kafka/) includes a support package that contains a [JSON (de)serializer](https://github.com/spring-projects/spring-kafka/tree/master/spring-kafka/src/main/java/org/springframework/kafka/support/serializer) that uses a [Jackson](https://github.com/FasterXML/jackson) `ObjectMapper` under the covers.

We base the below example on a previous [Spring Kafka tutorial]({{ site.url }}/2016/09/spring-kafka-consumer-producer-example.html). The only thing that needs to be added to the Maven POM file for working with JSON is the `spring-boot-starter-web` dependency which will indirectly include the needed `jackson-*` JAR dependencies.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>spring-kafka-json</artifactId>
  <version>0.0.1-SNAPSHOT</version>

  <name>spring-kafka-json</name>
  <description>Spring Kafka - JSON Example</description>
  <url>https://www.codenotfound.com/2017/03/spring-kafka-json-example</url>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.2.RELEASE</version>
  </parent>

  <properties>
    <java.version>1.8</java.version>

    <spring-kafka.version>1.1.2.RELEASE</spring-kafka.version>
  </properties>

  <dependencies>
    <!-- spring-boot -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
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

For this example we will be sending a `Car` object. to a <var>'json.t'</var> topic. Let's use following class representing a car with a basic structure.

``` java
package com.codenotfound.model;

public class Car {

  private String make;
  private String manufacturer;
  private String id;

  public Car() {
    super();
  }

  public Car(String make, String manufacturer, String id) {
    super();
    this.make = make;
    this.manufacturer = manufacturer;
    this.id = id;
  }

  public String getMake() {
    return make;
  }

  public void setMake(String make) {
    this.make = make;
  }

  public String getManufacturer() {
    return manufacturer;
  }


  public void setManufacturer(String manufacturer) {
    this.manufacturer = manufacturer;
  }

  public String getId() {
    return id;
  }


  public void setId(String id) {
    this.id = id;
  }

  @Override
  public String toString() {
    return "Car [make=" + make + ", manufacturer=" + manufacturer + ", id=" + id + "]";
  }
}
```

# Producing JSON Messages to a Kafka Topic

In order to use the `JsonSerializer` for converting the `Car` object that is sent, we need to set the value of the <var>'VALUE_SERIALIZER_CLASS_CONFIG'</var> `Producer` configuration property to the `JsonSerializer` class. In addition we change the `ProducerFactory` and `KafkaTemplate` generic type so that it specifies `Car` instead of `String`.


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
import org.springframework.kafka.support.serializer.JsonSerializer;

import com.codenotfound.model.Car;

@Configuration
public class SenderConfig {

  @Value("${kafka.servers.bootstrap}")
  private String bootstrapServers;

  @Bean
  public Map<String, Object> producerConfigs() {
    Map<String, Object> props = new HashMap<>();

    props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
    props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
    props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, JsonSerializer.class);
    props.put(ProducerConfig.MAX_BLOCK_MS_CONFIG, 5000);

    return props;
  }

  @Bean
  public ProducerFactory<String, Car> producerFactory() {
    return new DefaultKafkaProducerFactory<>(producerConfigs());
  }

  @Bean
  public KafkaTemplate<String, Car> kafkaTemplate() {
    return new KafkaTemplate<>(producerFactory());
  }

  @Bean
  public Sender sender() {
    return new Sender();
  }
}
```

The `Sender` class is updated accordingly so that it's `send()` method accepts a `Car` object as input. We also update the `KafkaTemplate` generic type.

``` java
package com.codenotfound.kafka.producer;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.kafka.core.KafkaTemplate;

import com.codenotfound.model.Car;

public class Sender {

  @Value("${kafka.topic.json}")
  private String jsonTopic;

  private static final Logger LOGGER = LoggerFactory.getLogger(Sender.class);

  @Autowired
  private KafkaTemplate<String, Car> kafkaTemplate;

  public void send(Car car) {
    LOGGER.info("sending car='{}'", car.toString());
    kafkaTemplate.send(jsonTopic, car);
  }
}
```

# Consuming JSON Messages from a Kafka Topic

To receive the JSON serialized message we need to update the value of the <var>'VALUE_DESERIALIZER_CLASS_CONFIG'</var> property so that it points to the `JsonDeserializer` class. The `ConsumerFactory` and `ConcurrentKafkaListenerContainerFactory` generic type needs to be changed so that it specifies `Car` instead of `String`.

> Note that the `JsonDeserializer` requires an additional `Class<?>` targetType argument to allow the deserialization of a consumed `byte[]` to the proper target object (in this example the `Car` class).

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
import org.springframework.kafka.support.serializer.JsonDeserializer;

import com.codenotfound.model.Car;

@Configuration
@EnableKafka
public class ReceiverConfig {

  @Value("${kafka.servers.bootstrap}")
  private String bootstrapServers;

  @Bean
  public Map<String, Object> consumerConfigs() {
    Map<String, Object> props = new HashMap<>();
    props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
    props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
    props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, JsonDeserializer.class);
    props.put(ConsumerConfig.GROUP_ID_CONFIG, "json");

    return props;
  }

  @Bean
  public ConsumerFactory<String, Car> consumerFactory() {
    return new DefaultKafkaConsumerFactory<>(consumerConfigs(), new StringDeserializer(),
        new JsonDeserializer<>(Car.class));
  }

  @Bean
  public ConcurrentKafkaListenerContainerFactory<String, Car> kafkaListenerContainerFactory() {
    ConcurrentKafkaListenerContainerFactory<String, Car> factory =
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

Identical to the updated `Sender` class, the argument of the `receive()` method of `Receiver` class needs to be changed to the `Car` class.

``` java
package com.codenotfound.kafka.consumer;

import java.util.concurrent.CountDownLatch;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.kafka.annotation.KafkaListener;

import com.codenotfound.model.Car;

public class Receiver {

  private static final Logger LOGGER = LoggerFactory.getLogger(Receiver.class);

  private CountDownLatch latch = new CountDownLatch(1);

  @KafkaListener(topics = "${kafka.topic.json}")
  public void receive(Car car) {
    LOGGER.info("received car='{}'", car.toString());
    latch.countDown();
  }

  public CountDownLatch getLatch() {
    return latch;
  }
}
```

# Test Sending and Receiving JSON Messages on Kafka

The Maven project contains a `SpringKafkaApplicationTest` test case to demonstrate the above sample code. A JUnit ClassRule [starts an embedded Kafka and ZooKeeper server]({{ site.url }}/2016/10/spring-kafka-embedded-server-unit-test.html). Using `@Before` we wait until all the partitions are assigned to our `Receiver` by looping over the available `ConcurrentMessageListenerContainer` (if we don't do this the message will already be sent before the listeners are assigned to the topic).

In the `testReceiver()` test case we create a `Car` object and send  it to the <var>'json.t'</var> topic. Finally the `CountDownLatch` from the `Receiver` is used to verify that a message was successfully received.

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
import org.springframework.kafka.listener.ConcurrentMessageListenerContainer;
import org.springframework.kafka.listener.MessageListenerContainer;
import org.springframework.kafka.test.rule.KafkaEmbedded;
import org.springframework.kafka.test.utils.ContainerTestUtils;
import org.springframework.test.context.junit4.SpringRunner;

import com.codenotfound.kafka.consumer.Receiver;
import com.codenotfound.kafka.producer.Sender;
import com.codenotfound.model.Car;

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringKafkaApplicationTest {

  @Autowired
  private Sender sender;

  @Autowired
  private Receiver receiver;

  @Autowired
  private KafkaListenerEndpointRegistry kafkaListenerEndpointRegistry;

  @ClassRule
  public static KafkaEmbedded embeddedKafka = new KafkaEmbedded(1, true, "json.t");

  @BeforeClass
  public static void setUpBeforeClass() throws Exception {
    System.setProperty("kafka.servers.bootstrap", embeddedKafka.getBrokersAsString());
  }

  @SuppressWarnings("unchecked")
  @Before
  public void setUp() throws Exception {
    // wait until the partitions are assigned
    for (MessageListenerContainer messageListenerContainer : kafkaListenerEndpointRegistry
        .getListenerContainers()) {
      if (messageListenerContainer instanceof ConcurrentMessageListenerContainer) {
        ConcurrentMessageListenerContainer<String, Car> concurrentMessageListenerContainer =
            (ConcurrentMessageListenerContainer<String, Car>) messageListenerContainer;

        ContainerTestUtils.waitForAssignment(concurrentMessageListenerContainer,
            embeddedKafka.getPartitionsPerTopic());
      }
    }
  }

  @Test
  public void testReceive() throws Exception {
    Car car = new Car("Passat", "Volkswagen", "ABC-123");

    sender.send(car);

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
 :: Spring Boot ::        (v1.5.2.RELEASE)

2017-03-21 20:48:57.677  INFO 4776 --- [           main] c.c.kafka.SpringKafkaApplicationTest     : Starting SpringKafkaApplicationTest on cnf-pc with PID 4776 (started by CodeNotFound in c:\code\st\spring-kafka\spring-kafka-json)
2017-03-21 20:48:57.678 DEBUG 4776 --- [           main] c.c.kafka.SpringKafkaApplicationTest     : Running with Spring Boot v1.5.2.RELEASE, Spring v4.3.7.RELEASE
2017-03-21 20:48:57.678  INFO 4776 --- [           main] c.c.kafka.SpringKafkaApplicationTest     : No active profile set, falling back to default profiles: default
2017-03-21 20:48:57.727  INFO 4776 --- [           main] o.s.w.c.s.GenericWebApplicationContext   : Refreshing org.springframework.web.context.support.GenericWebApplicationContext@2f08c4b: startup date [Tue Mar 21 20:48:57 CET 2017]; root of context hierarchy
2017-03-21 20:48:58.572  INFO 4776 --- [           main] trationDelegate$BeanPostProcessorChecker : Bean 'org.springframework.kafka.annotation.KafkaBootstrapConfiguration' of type [org.springframework.kafka.annotation.KafkaBootstrapConfiguration$$EnhancerBySpringCGLIB$$b1b59555] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
2017-03-21 20:48:59.185  INFO 4776 --- [           main] s.w.s.m.m.a.RequestMappingHandlerAdapter : Looking for @ControllerAdvice: org.springframework.web.context.support.GenericWebApplicationContext@2f08c4b: startup date [Tue Mar 21 20:48:57 CET 2017]; root of context hierarchy
2017-03-21 20:48:59.253  INFO 4776 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/error]}" onto public org.springframework.http.ResponseEntity<java.util.Map<java.lang.String, java.lang.Object>> org.springframework.boot.autoconfigure.web.BasicErrorController.error(javax.servlet.http.HttpServletRequest)
2017-03-21 20:48:59.255  INFO 4776 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/error],produces=[text/html]}" onto public org.springframework.web.servlet.ModelAndView org.springframework.boot.autoconfigure.web.BasicErrorController.errorHtml(javax.servlet.http.HttpServletRequest,javax.servlet.http.HttpServletResponse)
2017-03-21 20:48:59.304  INFO 4776 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/webjars/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2017-03-21 20:48:59.305  INFO 4776 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2017-03-21 20:48:59.346  INFO 4776 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**/favicon.ico] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2017-03-21 20:48:59.520  INFO 4776 --- [           main] o.s.c.support.DefaultLifecycleProcessor  : Starting beans in phase 0
2017-03-21 20:48:59.531  INFO 4776 --- [           main] o.a.k.clients.consumer.ConsumerConfig    : ConsumerConfig values:
2017-03-21 20:48:59.532  INFO 4776 --- [           main] o.a.k.clients.consumer.ConsumerConfig    : ConsumerConfig values:
2017-03-21 20:48:59.558  INFO 4776 --- [           main] o.a.kafka.common.utils.AppInfoParser     : Kafka version : 0.10.1.1
2017-03-21 20:48:59.558  INFO 4776 --- [           main] o.a.kafka.common.utils.AppInfoParser     : Kafka commitId : f10ef2720b03b247
2017-03-21 20:48:59.571  INFO 4776 --- [           main] c.c.kafka.SpringKafkaApplicationTest     : Started SpringKafkaApplicationTest in 2.149 seconds (JVM running for 6.566)
2017-03-21 20:50:06.970  INFO 1236 --- [afka-consumer-1] o.a.k.c.c.internals.AbstractCoordinator  : Successfully joined group json with generation 1
2017-03-21 20:50:06.971  INFO 1236 --- [afka-consumer-1] o.a.k.c.c.internals.ConsumerCoordinator  : Setting newly assigned partitions [json.t-1, json.t-0] for group json
2017-03-21 20:50:06.988  INFO 1236 --- [afka-consumer-1] o.s.k.l.KafkaMessageListenerContainer    : partitions assigned:[json.t-1, json.t-0]
2017-03-21 20:50:07.009  INFO 1236 --- [           main] com.codenotfound.kafka.producer.Sender   : sending car='Car [make=Passat, manufacturer=Volkswagen, id=ABC-123]'
2017-03-21 20:50:07.012  INFO 1236 --- [           main] o.a.k.clients.producer.ProducerConfig    : ProducerConfig values:
2017-03-21 20:50:07.013  INFO 1236 --- [           main] o.a.k.clients.producer.ProducerConfig    : ProducerConfig values:
2017-03-21 20:50:07.023  INFO 1236 --- [           main] o.a.kafka.common.utils.AppInfoParser     : Kafka version : 0.10.1.1
2017-03-21 20:50:07.023  INFO 1236 --- [           main] o.a.kafka.common.utils.AppInfoParser     : Kafka commitId : f10ef2720b03b247
2017-03-21 20:50:07.241  INFO 1236 --- [afka-listener-1] c.codenotfound.kafka.consumer.Receiver   : received car='Car [make=Passat, manufacturer=Volkswagen, id=ABC-123]'
2017-03-21 20:50:07.245  INFO 1236 --- [           main] kafka.server.KafkaServer                 : [Kafka Server 0], shutting down
2017-03-21 20:50:07.246  INFO 1236 --- [           main] kafka.server.KafkaServer                 : [Kafka Server 0], Starting controlled shutdown
2017-03-21 20:50:07.256  INFO 1236 --- [quest-handler-4] kafka.controller.KafkaController         : [Controller 0]: Shutting down broker 0
2017-03-21 20:50:07.264  INFO 1236 --- [           main] kafka.server.KafkaServer                 : [Kafka Server 0], Controlled shutdown succeeded
2017-03-21 20:50:07.266  INFO 1236 --- [           main] kafka.network.SocketServer               : [Socket Server on Broker 0], Shutting down
2017-03-21 20:50:07.269  INFO 1236 --- [afka-consumer-1] o.a.k.c.c.internals.AbstractCoordinator  : Marking the coordinator localhost:56210 (id: 2147483647 rack: null) dead for group json
2017-03-21 20:50:07.274  INFO 1236 --- [           main] kafka.network.SocketServer               : [Socket Server on Broker 0], Shutdown completed
2017-03-21 20:50:07.275  INFO 1236 --- [           main] kafka.server.KafkaRequestHandlerPool     : [Kafka Request Handler on Broker 0], shutting down
2017-03-21 20:50:07.277  INFO 1236 --- [           main] kafka.server.KafkaRequestHandlerPool     : [Kafka Request Handler on Broker 0], shut down completely
2017-03-21 20:50:07.281  INFO 1236 --- [           main] lientQuotaManager$ThrottledRequestReaper : [ThrottledRequestReaper-Fetch], Shutting down
2017-03-21 20:50:07.390  INFO 1236 --- [estReaper-Fetch] lientQuotaManager$ThrottledRequestReaper : [ThrottledRequestReaper-Fetch], Stopped
2017-03-21 20:50:07.390  INFO 1236 --- [           main] lientQuotaManager$ThrottledRequestReaper : [ThrottledRequestReaper-Fetch], Shutdown completed
2017-03-21 20:50:07.390  INFO 1236 --- [           main] lientQuotaManager$ThrottledRequestReaper : [ThrottledRequestReaper-Produce], Shutting down
2017-03-21 20:50:07.416  INFO 1236 --- [tReaper-Produce] lientQuotaManager$ThrottledRequestReaper : [ThrottledRequestReaper-Produce], Stopped
2017-03-21 20:50:07.416  INFO 1236 --- [           main] lientQuotaManager$ThrottledRequestReaper : [ThrottledRequestReaper-Produce], Shutdown completed
2017-03-21 20:50:07.416  INFO 1236 --- [           main] kafka.server.KafkaApis                   : [KafkaApi-0] Shutdown complete.
2017-03-21 20:50:07.417  INFO 1236 --- [           main] kafka.server.ReplicaManager              : [Replica Manager on Broker 0]: Shutting down
2017-03-21 20:50:07.418  INFO 1236 --- [           main] kafka.server.ReplicaFetcherManager       : [ReplicaFetcherManager on broker 0] shutting down
2017-03-21 20:50:07.419  INFO 1236 --- [           main] kafka.server.ReplicaFetcherManager       : [ReplicaFetcherManager on broker 0] shutdown completed
2017-03-21 20:50:07.419  INFO 1236 --- [           main] perationPurgatory$ExpiredOperationReaper : [ExpirationReaper-0], Shutting down
2017-03-21 20:50:07.502  INFO 1236 --- [irationReaper-0] perationPurgatory$ExpiredOperationReaper : [ExpirationReaper-0], Stopped
2017-03-21 20:50:07.502  INFO 1236 --- [           main] perationPurgatory$ExpiredOperationReaper : [ExpirationReaper-0], Shutdown completed
2017-03-21 20:50:07.502  INFO 1236 --- [           main] perationPurgatory$ExpiredOperationReaper : [ExpirationReaper-0], Shutting down
2017-03-21 20:50:07.626  INFO 1236 --- [irationReaper-0] perationPurgatory$ExpiredOperationReaper : [ExpirationReaper-0], Stopped
2017-03-21 20:50:07.626  INFO 1236 --- [           main] perationPurgatory$ExpiredOperationReaper : [ExpirationReaper-0], Shutdown completed
2017-03-21 20:50:07.639  INFO 1236 --- [           main] kafka.server.ReplicaManager              : [Replica Manager on Broker 0]: Shut down completely
2017-03-21 20:50:07.639  INFO 1236 --- [           main] perationPurgatory$ExpiredOperationReaper : [ExpirationReaper-0], Shutting down
2017-03-21 20:50:07.659  INFO 1236 --- [           main] perationPurgatory$ExpiredOperationReaper : [ExpirationReaper-0], Shutdown completed
2017-03-21 20:50:07.659  INFO 1236 --- [irationReaper-0] perationPurgatory$ExpiredOperationReaper : [ExpirationReaper-0], Stopped
2017-03-21 20:50:07.660  INFO 1236 --- [           main] kafka.log.LogManager                     : Shutting down.
2017-03-21 20:50:07.660  INFO 1236 --- [           main] kafka.log.LogCleaner                     : Shutting down the log cleaner.
2017-03-21 20:50:07.661  INFO 1236 --- [           main] kafka.log.LogCleaner                     : [kafka-log-cleaner-thread-0], Shutting down
2017-03-21 20:50:07.661  INFO 1236 --- [leaner-thread-0] kafka.log.LogCleaner                     : [kafka-log-cleaner-thread-0], Stopped
2017-03-21 20:50:07.661  INFO 1236 --- [           main] kafka.log.LogCleaner                     : [kafka-log-cleaner-thread-0], Shutdown completed
2017-03-21 20:50:07.938  INFO 1236 --- [           main] kafka.log.LogManager                     : Shutdown complete.
2017-03-21 20:50:07.938  INFO 1236 --- [           main] kafka.coordinator.GroupCoordinator       : [GroupCoordinator 0]: Shutting down.
2017-03-21 20:50:07.939  INFO 1236 --- [           main] perationPurgatory$ExpiredOperationReaper : [ExpirationReaper-0], Shutting down
2017-03-21 20:50:08.029  INFO 1236 --- [irationReaper-0] perationPurgatory$ExpiredOperationReaper : [ExpirationReaper-0], Stopped
2017-03-21 20:50:08.029  INFO 1236 --- [           main] perationPurgatory$ExpiredOperationReaper : [ExpirationReaper-0], Shutdown completed
2017-03-21 20:50:08.029  INFO 1236 --- [           main] perationPurgatory$ExpiredOperationReaper : [ExpirationReaper-0], Shutting down
2017-03-21 20:50:08.061  INFO 1236 --- [irationReaper-0] perationPurgatory$ExpiredOperationReaper : [ExpirationReaper-0], Stopped
2017-03-21 20:50:08.061  INFO 1236 --- [           main] perationPurgatory$ExpiredOperationReaper : [ExpirationReaper-0], Shutdown completed
2017-03-21 20:50:08.061  INFO 1236 --- [           main] kafka.coordinator.GroupCoordinator       : [GroupCoordinator 0]: Shutdown complete.
2017-03-21 20:50:08.063  INFO 1236 --- [           main] .TopicDeletionManager$DeleteTopicsThread : [delete-topics-thread-0], Shutting down
2017-03-21 20:50:08.063  INFO 1236 --- [topics-thread-0] .TopicDeletionManager$DeleteTopicsThread : [delete-topics-thread-0], Stopped
2017-03-21 20:50:08.063  INFO 1236 --- [           main] .TopicDeletionManager$DeleteTopicsThread : [delete-topics-thread-0], Shutdown completed
2017-03-21 20:50:08.065  INFO 1236 --- [           main] kafka.controller.PartitionStateMachine   : [Partition state machine on Controller 0]: Stopped partition state machine
2017-03-21 20:50:08.065  INFO 1236 --- [           main] kafka.controller.ReplicaStateMachine     : [Replica state machine on controller 0]: Stopped replica state machine
2017-03-21 20:50:08.067  INFO 1236 --- [           main] kafka.controller.RequestSendThread       : [Controller-0-to-broker-0-send-thread], Shutting down
2017-03-21 20:50:08.067  INFO 1236 --- [r-0-send-thread] kafka.controller.RequestSendThread       : [Controller-0-to-broker-0-send-thread], Stopped
2017-03-21 20:50:08.067  INFO 1236 --- [           main] kafka.controller.RequestSendThread       : [Controller-0-to-broker-0-send-thread], Shutdown completed
2017-03-21 20:50:08.068  INFO 1236 --- [           main] kafka.controller.KafkaController         : [Controller 0]: Broker 0 resigned as the controller
2017-03-21 20:50:08.068  INFO 1236 --- [127.0.0.1:56206] org.I0Itec.zkclient.ZkEventThread        : Terminate ZkClient event thread.
2017-03-21 20:50:08.069  INFO 1236 --- [0 cport:56206):] o.a.z.server.PrepRequestProcessor        : Processed session termination for sessionid: 0x15af26b4a640001
2017-03-21 20:50:08.076  INFO 1236 --- [ry:/127.0.0.1:0] o.apache.zookeeper.server.NIOServerCnxn  : Closed socket connection for client /127.0.0.1:56213 which had sessionid 0x15af26b4a640001
2017-03-21 20:50:08.076  INFO 1236 --- [           main] org.apache.zookeeper.ZooKeeper           : Session: 0x15af26b4a640001 closed
2017-03-21 20:50:08.077  INFO 1236 --- [ain-EventThread] org.apache.zookeeper.ClientCnxn          : EventThread shut down for session: 0x15af26b4a640001
2017-03-21 20:50:08.078  INFO 1236 --- [           main] kafka.server.KafkaServer                 : [Kafka Server 0], shut down completed
2017-03-21 20:50:08.154  INFO 1236 --- [127.0.0.1:56206] org.I0Itec.zkclient.ZkEventThread        : Terminate ZkClient event thread.
2017-03-21 20:50:08.155  INFO 1236 --- [0 cport:56206):] o.a.z.server.PrepRequestProcessor        : Processed session termination for sessionid: 0x15af26b4a640000
2017-03-21 20:50:08.160  INFO 1236 --- [           main] org.apache.zookeeper.ZooKeeper           : Session: 0x15af26b4a640000 closed
2017-03-21 20:50:08.160  INFO 1236 --- [ain-EventThread] org.apache.zookeeper.ClientCnxn          : EventThread shut down for session: 0x15af26b4a640000
2017-03-21 20:50:08.160  INFO 1236 --- [ry:/127.0.0.1:0] o.apache.zookeeper.server.NIOServerCnxn  : Closed socket connection for client /127.0.0.1:56209 which had sessionid 0x15af26b4a640000
2017-03-21 20:50:08.161  INFO 1236 --- [           main] o.a.zookeeper.server.ZooKeeperServer     : shutting down
2017-03-21 20:50:08.161  INFO 1236 --- [           main] o.a.zookeeper.server.SessionTrackerImpl  : Shutting down
2017-03-21 20:50:08.161  INFO 1236 --- [           main] o.a.z.server.PrepRequestProcessor        : Shutting down
2017-03-21 20:50:08.161  INFO 1236 --- [           main] o.a.z.server.SyncRequestProcessor        : Shutting down
2017-03-21 20:50:08.161  INFO 1236 --- [0 cport:56206):] o.a.z.server.PrepRequestProcessor        : PrepRequestProcessor exited loop!
2017-03-21 20:50:08.161  INFO 1236 --- [   SyncThread:0] o.a.z.server.SyncRequestProcessor        : SyncRequestProcessor exited!
2017-03-21 20:50:08.161  INFO 1236 --- [           main] o.a.z.server.FinalRequestProcessor       : shutdown of request processor complete
2017-03-21 20:50:08.162  INFO 1236 --- [ry:/127.0.0.1:0] o.a.z.server.NIOServerCnxnFactory        : NIOServerCnxn factory exited run method
2017-03-21 20:50:08.500  INFO 1236 --- [ SessionTracker] o.a.zookeeper.server.SessionTrackerImpl  : SessionTrackerImpl exited loop!
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 9.717 sec - in com.codenotfound.kafka.SpringKafkaApplicationTest
2017-03-21 20:50:09.171  INFO 1236 --- [       Thread-8] o.s.w.c.s.GenericWebApplicationContext   : Closing org.springframework.web.context.support.GenericWebApplicationContext@2f08c4b: startup date [Tue Mar 21 20:50:03 CET 2017]; root of context hierarchy
2017-03-21 20:50:09.175  INFO 1236 --- [       Thread-8] o.s.c.support.DefaultLifecycleProcessor  : Stopping beans in phase 0

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 41.654 s
[INFO] Finished at: 2017-03-21T20:50:39+01:00
[INFO] Final Memory: 17M/220M
[INFO] ------------------------------------------------------------------------
```

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-kafka/tree/master/spring-kafka-json).
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This concludes the example on how to send/receive JSON messages using Spring Kafka. If you have some questions or remarks, drop me a line below.