---
title: "Spring Kafka - JSON Serializer Example"
permalink: /2017/03/spring-kafka-json-serializer-example.html
excerpt: "A detailed step-by-step tutorial on how to send/receive JSON messages using Spring Kafka and Spring Boot."
date: 2017-03-21
modified: 2017-03-21
header:
  teaser: "assets/images/spring-kafka-teaser.jpg"
categories: [Spring Kafka]
tags: [Apache Kafka, Example, Maven, JSON, Deserializer, Serializer, Spring, Spring Boot, Spring Kafka, Tutorial]
redirect_from:
  - /2017/03/spring-kafka-json-example.html
published: true
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/spring-logo.jpg" alt="spring logo" class="logo">
</figure>

[JSON](http://www.json.org/) (JavaScript Object Notation) is a lightweight data-interchange format that uses human-readable text to transmit data objects. It is built on two structures: a collection of name/value pairs and an ordered list of values.

The following tutorial illustrates how to send/receive a Java object as a JSON `byte[]` array to/from Apache Kafka using Spring Kafka, Spring Boot and Maven.

# General Project Setup

If you want to learn more about Spring Kafka - head on over to the [Spring Kafka tutorials page]({{ site.url }}/spring-kafka/).
{: .notice--primary}

Tools used:
* Spring Kafka 1.2
* Spring Boot 1.5
* Maven 3.5

[Apache Kafka](https://kafka.apache.org/) stores and transports `Byte` arrays in its topics. It ships with a number of [built in (de)serializers](https://kafka.apache.org/0100/javadoc/org/apache/kafka/common/serialization/Serializer.html) but a JSON one is not included. Luckily, the [Spring Kafka framework](https://projects.spring.io/spring-kafka/) includes a support package that contains a [JSON (de)serializer](https://github.com/spring-projects/spring-kafka/tree/master/spring-kafka/src/main/java/org/springframework/kafka/support/serializer) that uses a [Jackson](https://github.com/FasterXML/jackson) `ObjectMapper` under the covers.

We base the below example on a previous [Spring Kafka example]({{ site.url }}/2016/09/spring-kafka-consumer-producer-example.html). The only thing that needs to be added to the Maven POM file for working with JSON is the `spring-boot-starter-web` dependency which will indirectly include the needed `jackson-*` JAR dependencies.

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

    <spring-kafka.version>1.2.0.RELEASE</spring-kafka.version>
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

# Object Model to Serialize/Deserialize

To illustrate the example we will send a `Car` object to a <var>'json.t'</var> topic. Let's use following class representing a car with a basic structure.

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

In order to use the `JsonSerializer`, shipped with Spring Kafka, we need to set the value of the producer's <var>'VALUE_SERIALIZER_CLASS_CONFIG'</var> configuration property to the `JsonSerializer` class. In addition we change the `ProducerFactory` and `KafkaTemplate` generic type so that it specifies `Car` instead of `String`. This will result in the `Car` object to be serialized in a JSON `byte[]` message.

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

  @Value("${kafka.bootstrap-servers}")
  private String bootstrapServers;

  @Bean
  public Map<String, Object> producerConfigs() {
    Map<String, Object> props = new HashMap<>();

    props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
    props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
    props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, JsonSerializer.class);

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

The `Sender` class is updated accordingly so that it's `send()` method accepts a `Car` object as input. We also update the `KafkaTemplate` generic type from `String` to `Car`.

``` java
package com.codenotfound.kafka.producer;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.kafka.core.KafkaTemplate;

import com.codenotfound.model.Car;

public class Sender {

  private static final Logger LOGGER = LoggerFactory.getLogger(Sender.class);

  @Value("${topic.json}")
  private String jsonTopic;

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

> Note that the `JsonDeserializer` requires a `Class<?>` argument to allow the deserialization of a consumed `byte[]` to the proper target object (in this example the `Car` class).

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

  @Value("${kafka.bootstrap-servers}")
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

Identical to the updated `Sender` class, the argument of the `receive()` method of the `Receiver` class needs to be changed to the `Car` type.

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

  public CountDownLatch getLatch() {
    return latch;
  }

  @KafkaListener(topics = "${topic.json}")
  public void receive(Car car) {
    LOGGER.info("received car='{}'", car.toString());
    latch.countDown();
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

22:11:39.942 [main] INFO  c.c.kafka.SpringKafkaApplicationTest - Starting SpringKafkaApplicationTest on cnf-pc with PID 2760 (started by CodeNotFound in c:\code\st\spring-kafka\spring-kafka-json)
22:11:39.943 [main] DEBUG c.c.kafka.SpringKafkaApplicationTest - Running with Spring Boot v1.5.2.RELEASE, Spring v4.3.7.RELEASE
22:11:39.943 [main] INFO  c.c.kafka.SpringKafkaApplicationTest - No active profile set, falling back to default profiles: default
22:11:39.978 [main] INFO  o.s.w.c.s.GenericWebApplicationContext - Refreshing org.springframework.web.context.support.GenericWebApplicationContext@4cc61eb1: startup date [Sun Apr 16 22:11:39 CEST 2017]; root of context hierarchy
22:11:40.854 [main] INFO  o.s.c.s.PostProcessorRegistrationDelegate$BeanPostProcessorChecker - Bean 'org.springframework.kafka.annotation.KafkaBootstrapConfiguration' of type [org.springframework.kafka.annotation.KafkaBootstrapConfiguration$$EnhancerBySpringCGLIB$$26b361b1] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
22:11:41.462 [main] INFO  o.s.w.s.m.m.a.RequestMappingHandlerAdapter - Looking for @ControllerAdvice: org.springframework.web.context.support.GenericWebApplicationContext@4cc61eb1: startup date [Sun Apr 16 22:11:39 CEST 2017]; root of context hierarchy
22:11:41.525 [main] INFO  o.s.w.s.m.m.a.RequestMappingHandlerMapping - Mapped "{[/error]}" onto public org.springframework.http.ResponseEntity<java.util.Map<java.lang.String, java.lang.Object>> org.springframework.boot.autoconfigure.web.BasicErrorController.error(javax.servlet.http.HttpServletRequest)
22:11:41.526 [main] INFO  o.s.w.s.m.m.a.RequestMappingHandlerMapping - Mapped "{[/error],produces=[text/html]}" onto public org.springframework.web.servlet.ModelAndView org.springframework.boot.autoconfigure.web.BasicErrorController.errorHtml(javax.servlet.http.HttpServletRequest,javax.servlet.http.HttpServletResponse)
22:11:41.559 [main] INFO  o.s.w.s.h.SimpleUrlHandlerMapping - Mapped URL path [/webjars/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
22:11:41.559 [main] INFO  o.s.w.s.h.SimpleUrlHandlerMapping - Mapped URL path [/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
22:11:41.596 [main] INFO  o.s.w.s.h.SimpleUrlHandlerMapping - Mapped URL path [/**/favicon.ico] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
22:11:41.773 [main] INFO  o.s.c.s.DefaultLifecycleProcessor - Starting beans in phase 0
22:11:41.819 [main] INFO  c.c.kafka.SpringKafkaApplicationTest - Started SpringKafkaApplicationTest in 2.149 seconds (JVM running for 6.241)
22:11:43.196 [org.springframework.kafka.KafkaListenerEndpointContainer#0-0-C-1] INFO  o.s.k.l.KafkaMessageListenerContainer - partitions revoked:[]
22:11:43.268 [org.springframework.kafka.KafkaListenerEndpointContainer#0-0-C-1] INFO  o.s.k.l.KafkaMessageListenerContainer - partitions assigned:[json.t-1, json.t-0]
22:11:43.320 [main] INFO  c.codenotfound.kafka.producer.Sender - sending car='Car [make=Passat, manufacturer=Volkswagen, id=ABC-123]'
22:11:43.428 [org.springframework.kafka.KafkaListenerEndpointContainer#0-0-L-1] INFO  c.c.kafka.consumer.Receiver - received car='Car [make=Passat, manufacturer=Volkswagen, id=ABC-123]'
22:11:46.209 [main] ERROR o.a.zookeeper.server.ZooKeeperServer - ZKShutdownHandler is not registered, so ZooKeeper server won't take any action on ERROR or SHUTDOWN server state changes
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 10.965 sec - in com.codenotfound.kafka.SpringKafkaApplicationTest
22:11:47.222 [Thread-8] INFO  o.s.w.c.s.GenericWebApplicationContext - Closing org.springframework.web.context.support.GenericWebApplicationContext@4cc61eb1: startup date [Sun Apr 16 22:11:39 CEST 201
7]; root of context hierarchy
22:11:47.227 [Thread-8] INFO  o.s.c.s.DefaultLifecycleProcessor - Stopping beans in phase 0

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 43.242 s
[INFO] Finished at: 2017-04-16T22:12:17+02:00
[INFO] Final Memory: 17M/226M
[INFO] ------------------------------------------------------------------------
```

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-kafka/tree/master/spring-kafka-json).
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This concludes the example on how to use the Spring Kafka JsonSerializer/JsonDeserializer in combination with Apache Kafka.

If you have some questions or remarks, drop me a line below.