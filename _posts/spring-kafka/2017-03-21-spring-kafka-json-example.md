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

[Apache Kafka](https://kafka.apache.org/) stores and transports `Byte` arrays in its topics. It ships with a number of [built in (de)serializers](https://kafka.apache.org/0100/javadoc/org/apache/kafka/common/serialization/Serializer.html) but a JSON one is not included. Luckily for us, [Spring Kafka](https://projects.spring.io/spring-kafka/) includes a support package that contains a [JSON (de)serializer](https://github.com/spring-projects/spring-kafka/tree/master/spring-kafka/src/main/java/org/springframework/kafka/support/serializer) that uses a [Jackson](https://github.com/FasterXML/jackson) `ObjectMapper` under the covers.

We start from a previous [Spring Kafka tutorial]({{ site.url }}/2016/09/spring-kafka-consumer-producer-example.html). The only thing that needs to be added to the Maven POM file is the `spring-boot-starter-web` dependency which will indirectly include the needed `jackson-*` JAR dependencies.

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

For this example we will be sending a `Car` object to a <var>json.t</var> topic. Let’s use following class representing a car with a basic structure.

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

In order to use the `JsonSerializer` for converting the `Car` object that is sent to the topic, we need to set the value of the <var>VALUE_SERIALIZER_CLASS_CONFIG</var> Producer configuration property to the `JsonSerializer` class. In addition we change the `ProducerFactory` and `KafkaTemplate` generic type so that it specifies `Car` instead of `String`.


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

  @Value("${kafka.bootstrap.servers}")
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

The `Sender` class is updated accordingly so that it's `send()` method accepts an `Car` object as input. We also update the `KafkaTemplate` generic type.

``` java
package com.codenotfound.kafka.producer;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.kafka.core.KafkaTemplate;

import com.codenotfound.model.Car;

public class Sender {

  @Value("${kafka.json.topic}")
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

In order to be able to receive the JSON serialized messages we need to update the value of the <var>VALUE_DESERIALIZER_CLASS_CONFIG</var> property so that it points to the `JsonDeserializer` class. The `ConsumerFactory` and `ConcurrentKafkaListenerContainerFactory` generic type needs to be changed so that it specifies `Car` instead of `String`.

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

  @Value("${kafka.bootstrap.servers}")
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

Identical to the `Sender` class, the argument of the `receive()` method of `Receiver` class needs to be changed to the `Car` class.

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

  @KafkaListener(topics = "${kafka.json.topic}")
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




---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-kafka/tree/master/spring-kafka-json).
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This concludes the example on how to send/receive JSON messages using Spring Kafka. If you have some questions or remarks, drop me a line below.