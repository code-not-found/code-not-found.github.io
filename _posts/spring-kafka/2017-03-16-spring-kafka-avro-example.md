---
title: Spring Kafka - Apache Avro Example
permalink: /2017/03/spring-kafka-apache-avro-example.html
excerpt: A detailed step-by-step tutorial on how to implement an Apache Avro Serializer &amp; Deserializer using Spring Kafka and Spring Boot.
date: 2017-03-16 21:00
categories: [Spring Kafka]
tags: [Apache Kafka, Apache Avro, Avro, Deserializer, Example, Maven, Serializer, Spring, Spring Boot, Spring Kafka, Tutorial]
published: true
redirect_from:
  - /2017/03/spring-kafka-avro-serializer-deserializer.html
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/spring-logo.png" alt="spring logo">
</figure>

[Apache Avro](https://avro.apache.org/docs/current/) is a data serialization system. It uses JSON for defining data types and protocols, and serializes data in a compact binary format. In the following tutorial we will configure, build and run an example in which we will send/receive an Avro messages to/from Apache Kafka using Spring Kafka, Spring Boot and Maven.

Tools used:
* Spring Boot 1.5
* Spring Kafka 1.1
* Apache Avro 1.8
* Maven 3

Avro relies on schemas which are defined using JSON. Schemas are composed of primitive types. For this example we will use [the 'User' schema from the Apache Avro getting started guide](https://avro.apache.org/docs/current/gettingstartedjava.html#Defining+a+schema) as shown below. This schema is stored in the <var>user.avsc</var> file located under <var>src/main/resources/avro</var>.

``` json
{"namespace": "example.avro",
 "type": "record",
 "name": "User",
 "fields": [
     {"name": "name", "type": "string"},
     {"name": "favorite_number",  "type": ["int", "null"]},
     {"name": "favorite_color", "type": ["string", "null"]}
 ]
}
```

Avro ships with code generation which allows us to automatically create Java classes based on the above defined <var>'User'</var> schema. Once we have generated the relevant classes, there is no need to use the schema directly in our program. The classes can be generated using the <var>avro-tools.jar</var> or via the [Avro Maven plugin](https://mvnrepository.com/artifact/org.apache.avro/avro-maven-plugin), we will use the latter in this example.

We start from a previous [Spring Boot Kafka example]({{ site.url }}/2016/09/spring-kafka-consumer-producer-example.html) and add the `avro` dependency to the Maven POM file. In addition we configure the `avro-maven-plugin` to run the <var>'schema'</var> goal on all schema's that are found in the <var>/src/main/resources/avro/</var> location as shown below.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>spring-kafka-avro</artifactId>
  <version>0.0.1-SNAPSHOT</version>

  <name>spring-kafka-avro</name>
  <description>Spring Kafka - Apache Avro Example</description>
  <url>https://www.codenotfound.com/2017/03/spring-kafka-avro-example</url>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.2.RELEASE</version>
  </parent>

  <properties>
    <java.version>1.8</java.version>

    <spring-kafka.version>1.1.2.RELEASE</spring-kafka.version>
    <avro.version>1.8.1</avro.version>
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
    <!-- avro -->
    <dependency>
      <groupId>org.apache.avro</groupId>
      <artifactId>avro</artifactId>
      <version>${avro.version}</version>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <!-- spring-boot-maven-plugin -->
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
      <!-- avro-maven-plugin -->
      <plugin>
        <groupId>org.apache.avro</groupId>
        <artifactId>avro-maven-plugin</artifactId>
        <version>${avro.version}</version>
        <executions>
          <execution>
            <phase>generate-sources</phase>
            <goals>
              <goal>schema</goal>
            </goals>
            <configuration>
              <sourceDirectory>${project.basedir}/src/main/resources/avro/</sourceDirectory>
              <outputDirectory>${project.build.directory}/generated/avro</outputDirectory>
            </configuration>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
</project>
```

In order to trigger the code generation via Maven, executed following command:

``` plaintext
mvn generate-sources
```

This results in the generation of a `User` class which contains the schema and a number of `Builder` methods to construct a `User` object.

<figure>
    <img src="{{ site.url }}/assets/images/spring-kafka/avro-generated-java-classes.png" alt="avro generated java classes">
</figure>

# Producing Avro Messages to a Kafka Topic

Kafka stores and transports `Byte` arrays in its topics. But as we are working with Avro objects we need to transform to/from these `Byte` arrays. Before version 0.9.0.0, the Kafka Java API used implementations of `Encoder`/`Decoder` interfaces to handle transformations but these have been replaced by `Serializer`/`Deserializer` interface implementations in the new API. Kafka ships with a number of [built in (de)serializers](https://kafka.apache.org/0100/javadoc/org/apache/kafka/common/serialization/Serializer.html) but an Avro one is not included.

To tackle this we will create an `AvroSerializer` class that implements the `Serializer` interface specifically for Avro objects. We then implement the `serialize()` method which takes as input a topic name and a data object which in our case is an Avro object that extends `SpecificRecordBase`. The method [serializes the Avro object to a byte array](https://cwiki.apache.org/confluence/display/AVRO/FAQ#FAQ-Serializingtoabytearray) and returns the result.

``` java
package com.codenotfound.kafka.serializer;

import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.util.Map;

import javax.xml.bind.DatatypeConverter;

import org.apache.avro.generic.GenericDatumWriter;
import org.apache.avro.generic.GenericRecord;
import org.apache.avro.io.BinaryEncoder;
import org.apache.avro.io.DatumWriter;
import org.apache.avro.io.EncoderFactory;
import org.apache.avro.specific.SpecificRecordBase;
import org.apache.kafka.common.errors.SerializationException;
import org.apache.kafka.common.serialization.Serializer;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class AvroSerializer<T extends SpecificRecordBase> implements Serializer<T> {

  private static final Logger LOGGER = LoggerFactory.getLogger(AvroSerializer.class);

  @Override
  public void close() {
    // No-op
  }

  @Override
  public void configure(Map<String, ?> arg0, boolean arg1) {
    // No-op
  }

  @Override
  public byte[] serialize(String topic, T data) {
    try {
      byte[] result = null;

      if (data != null) {
        LOGGER.debug("data='{}'", data);
        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
        BinaryEncoder binaryEncoder =
            EncoderFactory.get().binaryEncoder(byteArrayOutputStream, null);

        DatumWriter<GenericRecord> datumWriter = new GenericDatumWriter<>(data.getSchema());
        datumWriter.write(data, binaryEncoder);

        binaryEncoder.flush();
        byteArrayOutputStream.close();

        result = byteArrayOutputStream.toByteArray();
        LOGGER.debug("serialized data='{}'", DatatypeConverter.printHexBinary(result));
      }

      return result;
    } catch (IOException ex) {
      throw new SerializationException(
          "Can't serialize data [" + data + "] for topic [" + topic + "]", ex);
    }
  }
}
```

Now we need to change the `SenderConfig` to start using our custom `Serializer` implementation. This is done by setting the <var>VALUE_SERIALIZER_CLASS_CONFIG</var> property to the `AvroSerializer` class. In addition we change the `ProducerFactory` and `KafkaTemplate` generic type so that it specifies `User` instead of `String`.

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

import com.codenotfound.kafka.serializer.AvroSerializer;

import example.avro.User;

@Configuration
public class SenderConfig {

  @Value("${kafka.bootstrap.servers}")
  private String bootstrapServers;

  @Bean
  public Map<String, Object> producerConfigs() {
    Map<String, Object> props = new HashMap<>();

    props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
    props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
    props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, AvroSerializer.class);
    props.put(ProducerConfig.MAX_BLOCK_MS_CONFIG, 5000);

    return props;
  }

  @Bean
  public ProducerFactory<String, User> producerFactory() {
    return new DefaultKafkaProducerFactory<>(producerConfigs());
  }

  @Bean
  public KafkaTemplate<String, User> kafkaTemplate() {
    return new KafkaTemplate<>(producerFactory());
  }

  @Bean
  public Sender sender() {
    return new Sender();
  }
}
```

The only thing left to do is to update the `Sender` class so that it's `send()` method accepts an Avro `User` object as input. Note that we also update the `KafkaTemplate` generic type.


``` java
package com.codenotfound.kafka.producer;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.kafka.core.KafkaTemplate;

import example.avro.User;

public class Sender {

  @Value("${kafka.avro.topic}")
  private String avroTopic;

  private static final Logger LOGGER = LoggerFactory.getLogger(Sender.class);

  @Autowired
  private KafkaTemplate<String, User> kafkaTemplate;

  public void send(User user) {
    LOGGER.info("sending user='{}'", user.toString());
    kafkaTemplate.send(avroTopic, user);
  }
}
```

# Consuming Avro Messages from a Kafka Topic

Received messages need to be deserialized back to the Avro format. To achieve this we create an `AvroDeserializer` class that implements the `Deserializer` interface. The `deserialize()` method takes as input a topic name and [a Byte array which is decoded back into an Avro object](https://cwiki.apache.org/confluence/display/AVRO/FAQ#FAQ-Deserializingfromabytearray). The schema that needs to be used for the decoding is retrieved from the `targetType` class parameter that needs to be passed as an argument to the `AvroDeserializer` constructor.

``` java
package com.codenotfound.kafka.serializer;

import java.util.Arrays;
import java.util.Map;

import javax.xml.bind.DatatypeConverter;

import org.apache.avro.generic.GenericRecord;
import org.apache.avro.io.DatumReader;
import org.apache.avro.io.Decoder;
import org.apache.avro.io.DecoderFactory;
import org.apache.avro.specific.SpecificDatumReader;
import org.apache.avro.specific.SpecificRecordBase;
import org.apache.kafka.common.errors.SerializationException;
import org.apache.kafka.common.serialization.Deserializer;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class AvroDeserializer<T extends SpecificRecordBase> implements Deserializer<T> {

  private static final Logger LOGGER = LoggerFactory.getLogger(AvroDeserializer.class);

  protected final Class<T> targetType;

  public AvroDeserializer(Class<T> targetType) {
    this.targetType = targetType;
  }

  @Override
  public void close() {
    // No-op
  }

  @Override
  public void configure(Map<String, ?> arg0, boolean arg1) {
    // No-op
  }

  @SuppressWarnings("unchecked")
  @Override
  public T deserialize(String topic, byte[] data) {
    try {
      T result = null;

      if (data != null) {
        LOGGER.debug("serialized data='{}'", DatatypeConverter.printHexBinary(data));

        DatumReader<GenericRecord> datumReader =
            new SpecificDatumReader<>(targetType.newInstance().getSchema());
        Decoder decoder = DecoderFactory.get().binaryDecoder(data, null);

        result = (T) datumReader.read(null, decoder);
        LOGGER.debug("data='{}'", result);
      }

      return result;
    } catch (Exception ex) {
      throw new SerializationException(
          "Can't deserialize data [" + Arrays.toString(data) + "] from topic [" + topic + "]", ex);
    }
  }
}
```

The `ReceiverConfig` needs to be updated so that the `AvroDeserializer` is used as value for the <var>VALUE_DESERIALIZER_CLASS_CONFIG</var> property. We also change the `ConsumerFactory` and `ConcurrentKafkaListenerContainerFactory` generic type so that it specifies `User` instead of `String`. The `DefaultKafkaConsumerFactory` is created by passing a new `AvroDeserializer` that takes <var>'User.class'</var> as constructor argument.

> The `Class<?>` targetType of the `AvroDeserializer` is need to allow the deserialization of a consumed `byte[]` to the proper target object (in this example the `User` class).

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

import com.codenotfound.kafka.serializer.AvroDeserializer;

import example.avro.User;

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
    props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, AvroDeserializer.class);
    props.put(ConsumerConfig.GROUP_ID_CONFIG, "avro");

    return props;
  }

  @Bean
  public ConsumerFactory<String, User> consumerFactory() {
    return new DefaultKafkaConsumerFactory<>(consumerConfigs(), new StringDeserializer(),
        new AvroDeserializer<>(User.class));
  }

  @Bean
  public ConcurrentKafkaListenerContainerFactory<String, User> kafkaListenerContainerFactory() {
    ConcurrentKafkaListenerContainerFactory<String, User> factory =
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

Just like with the `Sender` class, the argument of the `receive()` method of `Receiver` class needs to be changed to the Avro `User` class.

``` java
package com.codenotfound.kafka.consumer;

import java.util.concurrent.CountDownLatch;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.kafka.annotation.KafkaListener;

import example.avro.User;

public class Receiver {

  private static final Logger LOGGER = LoggerFactory.getLogger(Receiver.class);

  private CountDownLatch latch = new CountDownLatch(1);

  @KafkaListener(topics = "${kafka.avro.topic}")
  public void receive(User user) {
    LOGGER.info("received user='{}'", user.toString());
    latch.countDown();
  }

  public CountDownLatch getLatch() {
    return latch;
  }
}
```

# Test Sending and Receiving Avro Messages on Kafka

The `SpringKafkaApplicationTest` test case demonstrates the above sample code. A JUnit ClassRule [starts an embedded Kafka and ZooKeeper server]({{ site.url }}/2016/10/spring-kafka-embedded-server-unit-test.html). Before the test case starts we wait until all the partitions are assigned to our `Receiver` by looping over the available `ConcurrentMessageListenerContainer` (if we don't do this the message will already be sent before the listeners are assigned to the topic).

In the `testReceiver()` test case an Avro `User` object is created using the `Builder` methods. This user is then sent to <var>avro.t</var> topic. Finally the `CountDownLatch` from the `Receiver` is used to verify that a message was successfully received.

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

import example.avro.User;

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
  public static KafkaEmbedded embeddedKafka = new KafkaEmbedded(1, true, "avro.t");

  @BeforeClass
  public static void setUpBeforeClass() throws Exception {
    System.setProperty("kafka.bootstrap.servers", embeddedKafka.getBrokersAsString());
  }

  @SuppressWarnings("unchecked")
  @Before
  public void setUp() throws Exception {
    // wait until the partitions are assigned
    for (MessageListenerContainer messageListenerContainer : kafkaListenerEndpointRegistry
        .getListenerContainers()) {
      if (messageListenerContainer instanceof ConcurrentMessageListenerContainer) {
        ConcurrentMessageListenerContainer<String, User> concurrentMessageListenerContainer =
            (ConcurrentMessageListenerContainer<String, User>) messageListenerContainer;

        ContainerTestUtils.waitForAssignment(concurrentMessageListenerContainer,
            embeddedKafka.getPartitionsPerTopic());
      }
    }
  }

  @Test
  public void testReceiver() throws Exception {
    User user = User.newBuilder().setName("John Doe").setFavoriteColor("green")
        .setFavoriteNumber(null).build();

    sender.send(user);

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

2017-03-18 20:27:54.855  INFO 4500 --- [           main] c.c.kafka.SpringKafkaApplicationTest     : Starting SpringKafkaApplicationTest on cnf-pc with PID 4500 (started by CodeNotFound in c:\code\st\spring-kaf
ka\spring-kafka-avro)
2017-03-18 20:27:54.856 DEBUG 4500 --- [           main] c.c.kafka.SpringKafkaApplicationTest     : Running with Spring Boot v1.5.2.RELEASE, Spring v4.3.7.RELEASE
2017-03-18 20:27:54.856  INFO 4500 --- [           main] c.c.kafka.SpringKafkaApplicationTest     : No active profile set, falling back to default profiles: default
2017-03-18 20:27:54.877  INFO 4500 --- [           main] s.c.a.AnnotationConfigApplicationContext : Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@4604b900: start
up date [Sat Mar 18 20:27:54 CET 2017]; root of context hierarchy
2017-03-18 20:27:55.287  INFO 4500 --- [           main] trationDelegate$BeanPostProcessorChecker : Bean 'org.springframework.kafka.annotation.KafkaBootstrapConfiguration' of type [org.springframework
.kafka.annotation.KafkaBootstrapConfiguration$$EnhancerBySpringCGLIB$$e407da82] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
2017-03-18 20:27:55.485  INFO 4500 --- [           main] o.s.c.support.DefaultLifecycleProcessor  : Starting beans in phase 0
2017-03-18 20:27:55.495  INFO 4500 --- [           main] o.a.k.clients.consumer.ConsumerConfig    : ConsumerConfig values:
2017-03-18 20:27:55.496  INFO 4500 --- [           main] o.a.k.clients.consumer.ConsumerConfig    : ConsumerConfig values:
2017-03-18 20:27:55.517  INFO 4500 --- [           main] o.a.kafka.common.utils.AppInfoParser     : Kafka version : 0.10.1.1
2017-03-18 20:27:55.518  INFO 4500 --- [           main] o.a.kafka.common.utils.AppInfoParser     : Kafka commitId : f10ef2720b03b247
2017-03-18 20:27:55.530  INFO 4500 --- [           main] c.c.kafka.SpringKafkaApplicationTest     : Started SpringKafkaApplicationTest in 0.953 seconds (JVM running for 5.284)
2017-03-18 18:38:02.954  INFO 6740 --- [afka-consumer-1] o.a.k.c.c.internals.AbstractCoordinator  : (Re-)joining group avro
2017-03-18 18:38:02.964  INFO 6740 --- [quest-handler-4] kafka.coordinator.GroupCoordinator       : [GroupCoordinator 0]: Preparing to restabilize group avro with old generation 0
2017-03-18 18:38:02.970  INFO 6740 --- [quest-handler-4] kafka.coordinator.GroupCoordinator       : [GroupCoordinator 0]: Stabilized group avro generation 1
2017-03-18 18:38:02.975  INFO 6740 --- [quest-handler-6] kafka.coordinator.GroupCoordinator       : [GroupCoordinator 0]: Assignment received from leader for group avro for generation 1
2017-03-18 18:38:03.036  INFO 6740 --- [afka-consumer-1] o.a.k.c.c.internals.AbstractCoordinator  : Successfully joined group avro with generation 1
2017-03-18 18:38:03.037  INFO 6740 --- [afka-consumer-1] o.a.k.c.c.internals.ConsumerCoordinator  : Setting newly assigned partitions [avro.t-1, avro.t-0] for group avro
2017-03-18 18:38:03.056  INFO 6740 --- [afka-consumer-1] o.s.k.l.KafkaMessageListenerContainer    : partitions assigned:[avro.t-1, avro.t-0]
2017-03-18 18:38:03.119  INFO 6740 --- [           main] com.codenotfound.kafka.producer.Sender   : sending user='{"name": "John Doe", "favorite_number": null, "favorite_color": "green"}'
2017-03-18 18:38:03.122  INFO 6740 --- [           main] o.a.k.clients.producer.ProducerConfig    : ProducerConfig values:
2017-03-18 18:38:03.123  INFO 6740 --- [           main] o.a.k.clients.producer.ProducerConfig    : ProducerConfig values:
2017-03-18 18:38:03.134  INFO 6740 --- [           main] o.a.kafka.common.utils.AppInfoParser     : Kafka version : 0.10.1.1
2017-03-18 18:38:03.134  INFO 6740 --- [           main] o.a.kafka.common.utils.AppInfoParser     : Kafka commitId : f10ef2720b03b247
2017-03-18 18:38:03.230 DEBUG 6740 --- [           main] c.c.kafka.serializer.AvroSerializer      : data='{"name": "John Doe", "favorite_number": null, "favorite_color": "green"}'
2017-03-18 18:38:03.231 DEBUG 6740 --- [           main] c.c.kafka.serializer.AvroSerializer      : serialized data='104A6F686E20446F6502000A677265656E'
2017-03-18 18:38:03.263 DEBUG 6740 --- [afka-consumer-1] c.c.kafka.serializer.AvroDeserializer    : serialized data='104A6F686E20446F6502000A677265656E'
2017-03-18 18:38:03.263 DEBUG 6740 --- [afka-consumer-1] c.c.kafka.serializer.AvroDeserializer    : data='{"name": "John Doe", "favorite_number": null, "favorite_color": "green"}'
2017-03-18 18:38:03.269  INFO 6740 --- [afka-listener-1] c.codenotfound.kafka.consumer.Receiver   : received user='{"name": "John Doe", "favorite_number": null, "favorite_color": "green"}'
2017-03-18 18:38:03.273  INFO 6740 --- [           main] kafka.server.KafkaServer                 : [Kafka Server 0], shutting down
2017-03-18 18:38:03.274  INFO 6740 --- [           main] kafka.server.KafkaServer                 : [Kafka Server 0], Starting controlled shutdown
2017-03-18 18:38:03.282  INFO 6740 --- [quest-handler-2] kafka.controller.KafkaController         : [Controller 0]: Shutting down broker 0
2017-03-18 18:38:03.290  INFO 6740 --- [           main] kafka.server.KafkaServer                 : [Kafka Server 0], Controlled shutdown succeeded
2017-03-18 18:38:03.291  INFO 6740 --- [           main] kafka.network.SocketServer               : [Socket Server on Broker 0], Shutting down
2017-03-18 18:38:03.295  INFO 6740 --- [afka-consumer-1] o.a.k.c.c.internals.AbstractCoordinator  : Marking the coordinator localhost:55754 (id: 2147483647 rack: null) dead for group avro
2017-03-18 18:38:03.298  INFO 6740 --- [           main] kafka.network.SocketServer               : [Socket Server on Broker 0], Shutdown completed
2017-03-18 18:38:03.298  INFO 6740 --- [           main] kafka.server.KafkaRequestHandlerPool     : [Kafka Request Handler on Broker 0], shutting down
2017-03-18 18:38:03.300  INFO 6740 --- [           main] kafka.server.KafkaRequestHandlerPool     : [Kafka Request Handler on Broker 0], shut down completely
2017-03-18 18:38:03.302  INFO 6740 --- [           main] lientQuotaManager$ThrottledRequestReaper : [ThrottledRequestReaper-Fetch], Shutting down
2017-03-18 18:38:03.658  INFO 6740 --- [estReaper-Fetch] lientQuotaManager$ThrottledRequestReaper : [ThrottledRequestReaper-Fetch], Stopped
2017-03-18 18:38:03.658  INFO 6740 --- [           main] lientQuotaManager$ThrottledRequestReaper : [ThrottledRequestReaper-Fetch], Shutdown completed
2017-03-18 18:38:03.659  INFO 6740 --- [           main] lientQuotaManager$ThrottledRequestReaper : [ThrottledRequestReaper-Produce], Shutting down
2017-03-18 18:38:04.659  INFO 6740 --- [tReaper-Produce] lientQuotaManager$ThrottledRequestReaper : [ThrottledRequestReaper-Produce], Stopped
2017-03-18 18:38:04.659  INFO 6740 --- [           main] lientQuotaManager$ThrottledRequestReaper : [ThrottledRequestReaper-Produce], Shutdown completed
2017-03-18 18:38:04.660  INFO 6740 --- [           main] kafka.server.KafkaApis                   : [KafkaApi-0] Shutdown complete.
2017-03-18 18:38:04.662  INFO 6740 --- [           main] kafka.server.ReplicaManager              : [Replica Manager on Broker 0]: Shutting down
2017-03-18 18:38:04.663  INFO 6740 --- [           main] kafka.server.ReplicaFetcherManager       : [ReplicaFetcherManager on broker 0] shutting down
2017-03-18 18:38:04.664  INFO 6740 --- [           main] kafka.server.ReplicaFetcherManager       : [ReplicaFetcherManager on broker 0] shutdown completed
2017-03-18 18:38:04.665  INFO 6740 --- [           main] perationPurgatory$ExpiredOperationReaper : [ExpirationReaper-0], Shutting down
2017-03-18 18:38:04.771  INFO 6740 --- [irationReaper-0] perationPurgatory$ExpiredOperationReaper : [ExpirationReaper-0], Stopped
2017-03-18 18:38:04.771  INFO 6740 --- [           main] perationPurgatory$ExpiredOperationReaper : [ExpirationReaper-0], Shutdown completed
2017-03-18 18:38:04.771  INFO 6740 --- [           main] perationPurgatory$ExpiredOperationReaper : [ExpirationReaper-0], Shutting down
2017-03-18 18:38:04.827  INFO 6740 --- [irationReaper-0] perationPurgatory$ExpiredOperationReaper : [ExpirationReaper-0], Stopped
2017-03-18 18:38:04.827  INFO 6740 --- [           main] perationPurgatory$ExpiredOperationReaper : [ExpirationReaper-0], Shutdown completed
2017-03-18 18:38:04.837  INFO 6740 --- [           main] kafka.server.ReplicaManager              : [Replica Manager on Broker 0]: Shut down completely
2017-03-18 18:38:04.838  INFO 6740 --- [           main] perationPurgatory$ExpiredOperationReaper : [ExpirationReaper-0], Shutting down
2017-03-18 18:38:04.934  INFO 6740 --- [irationReaper-0] perationPurgatory$ExpiredOperationReaper : [ExpirationReaper-0], Stopped
2017-03-18 18:38:04.934  INFO 6740 --- [           main] perationPurgatory$ExpiredOperationReaper : [ExpirationReaper-0], Shutdown completed
2017-03-18 18:38:04.935  INFO 6740 --- [           main] kafka.log.LogManager                     : Shutting down.
2017-03-18 18:38:04.936  INFO 6740 --- [           main] kafka.log.LogCleaner                     : Shutting down the log cleaner.
2017-03-18 18:38:04.937  INFO 6740 --- [           main] kafka.log.LogCleaner                     : [kafka-log-cleaner-thread-0], Shutting down
2017-03-18 18:38:04.937  INFO 6740 --- [leaner-thread-0] kafka.log.LogCleaner                     : [kafka-log-cleaner-thread-0], Stopped
2017-03-18 18:38:04.937  INFO 6740 --- [           main] kafka.log.LogCleaner                     : [kafka-log-cleaner-thread-0], Shutdown completed
2017-03-18 18:38:05.251  INFO 6740 --- [           main] kafka.log.LogManager                     : Shutdown complete.
2017-03-18 18:38:05.252  INFO 6740 --- [           main] kafka.coordinator.GroupCoordinator       : [GroupCoordinator 0]: Shutting down.
2017-03-18 18:38:05.252  INFO 6740 --- [           main] perationPurgatory$ExpiredOperationReaper : [ExpirationReaper-0], Shutting down
2017-03-18 18:38:05.335  INFO 6740 --- [irationReaper-0] perationPurgatory$ExpiredOperationReaper : [ExpirationReaper-0], Stopped
2017-03-18 18:38:05.335  INFO 6740 --- [           main] perationPurgatory$ExpiredOperationReaper : [ExpirationReaper-0], Shutdown completed
2017-03-18 18:38:05.335  INFO 6740 --- [           main] perationPurgatory$ExpiredOperationReaper : [ExpirationReaper-0], Shutting down
2017-03-18 18:38:05.536  INFO 6740 --- [irationReaper-0] perationPurgatory$ExpiredOperationReaper : [ExpirationReaper-0], Stopped
2017-03-18 18:38:05.536  INFO 6740 --- [           main] perationPurgatory$ExpiredOperationReaper : [ExpirationReaper-0], Shutdown completed
2017-03-18 18:38:05.537  INFO 6740 --- [           main] kafka.coordinator.GroupCoordinator       : [GroupCoordinator 0]: Shutdown complete.
2017-03-18 18:38:05.539  INFO 6740 --- [           main] .TopicDeletionManager$DeleteTopicsThread : [delete-topics-thread-0], Shutting down
2017-03-18 18:38:05.540  INFO 6740 --- [topics-thread-0] .TopicDeletionManager$DeleteTopicsThread : [delete-topics-thread-0], Stopped
2017-03-18 18:38:05.540  INFO 6740 --- [           main] .TopicDeletionManager$DeleteTopicsThread : [delete-topics-thread-0], Shutdown completed
2017-03-18 18:38:05.544  INFO 6740 --- [           main] kafka.controller.PartitionStateMachine   : [Partition state machine on Controller 0]: Stopped partition state machine
2017-03-18 18:38:05.544  INFO 6740 --- [           main] kafka.controller.ReplicaStateMachine     : [Replica state machine on controller 0]: Stopped replica state machine
2017-03-18 18:38:05.547  INFO 6740 --- [           main] kafka.controller.RequestSendThread       : [Controller-0-to-broker-0-send-thread], Shutting down
2017-03-18 18:38:05.547  INFO 6740 --- [r-0-send-thread] kafka.controller.RequestSendThread       : [Controller-0-to-broker-0-send-thread], Stopped
2017-03-18 18:38:05.549  INFO 6740 --- [           main] kafka.controller.RequestSendThread       : [Controller-0-to-broker-0-send-thread], Shutdown completed
2017-03-18 18:38:05.550  INFO 6740 --- [           main] kafka.controller.KafkaController         : [Controller 0]: Broker 0 resigned as the controller
2017-03-18 18:38:05.551  INFO 6740 --- [127.0.0.1:55750] org.I0Itec.zkclient.ZkEventThread        : Terminate ZkClient event thread.
2017-03-18 18:38:05.552  INFO 6740 --- [0 cport:55750):] o.a.z.server.PrepRequestProcessor        : Processed session termination for sessionid: 0x15ae27f51df0001
2017-03-18 18:38:05.557  INFO 6740 --- [ry:/127.0.0.1:0] o.apache.zookeeper.server.NIOServerCnxn  : Closed socket connection for client /127.0.0.1:55757 which had sessionid 0x15ae27f51df0001
2017-03-18 18:38:05.558  INFO 6740 --- [           main] org.apache.zookeeper.ZooKeeper           : Session: 0x15ae27f51df0001 closed
2017-03-18 18:38:05.558  INFO 6740 --- [ain-EventThread] org.apache.zookeeper.ClientCnxn          : EventThread shut down for session: 0x15ae27f51df0001
2017-03-18 18:38:05.559  INFO 6740 --- [           main] kafka.server.KafkaServer                 : [Kafka Server 0], shut down completed
2017-03-18 18:38:05.646  INFO 6740 --- [127.0.0.1:55750] org.I0Itec.zkclient.ZkEventThread        : Terminate ZkClient event thread.
2017-03-18 18:38:05.647  INFO 6740 --- [0 cport:55750):] o.a.z.server.PrepRequestProcessor        : Processed session termination for sessionid: 0x15ae27f51df0000
2017-03-18 18:38:05.652  INFO 6740 --- [           main] org.apache.zookeeper.ZooKeeper           : Session: 0x15ae27f51df0000 closed
2017-03-18 18:38:05.652  INFO 6740 --- [ry:/127.0.0.1:0] o.apache.zookeeper.server.NIOServerCnxn  : Closed socket connection for client /127.0.0.1:55753 which had sessionid 0x15ae27f51df0000
2017-03-18 18:38:05.652  INFO 6740 --- [ain-EventThread] org.apache.zookeeper.ClientCnxn          : EventThread shut down for session: 0x15ae27f51df0000
2017-03-18 18:38:05.652  INFO 6740 --- [           main] o.a.zookeeper.server.ZooKeeperServer     : shutting down
2017-03-18 18:38:05.652  INFO 6740 --- [           main] o.a.zookeeper.server.SessionTrackerImpl  : Shutting down
2017-03-18 18:38:05.653  INFO 6740 --- [           main] o.a.z.server.PrepRequestProcessor        : Shutting down
2017-03-18 18:38:05.653  INFO 6740 --- [           main] o.a.z.server.SyncRequestProcessor        : Shutting down
2017-03-18 18:38:05.653  INFO 6740 --- [0 cport:55750):] o.a.z.server.PrepRequestProcessor        : PrepRequestProcessor exited loop!
2017-03-18 18:38:05.653  INFO 6740 --- [   SyncThread:0] o.a.z.server.SyncRequestProcessor        : SyncRequestProcessor exited!
2017-03-18 18:38:05.653  INFO 6740 --- [           main] o.a.z.server.FinalRequestProcessor       : shutdown of request processor complete
2017-03-18 18:38:05.654  INFO 6740 --- [ry:/127.0.0.1:0] o.a.z.server.NIOServerCnxnFactory        : NIOServerCnxn factory exited run method
2017-03-18 18:38:06.000  INFO 6740 --- [ SessionTracker] o.a.zookeeper.server.SessionTrackerImpl  : SessionTrackerImpl exited loop!
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 10.161 sec - in com.codenotfound.kafka.SpringKafkaApplicationTests
2017-03-18 18:38:06.680  INFO 6740 --- [       Thread-8] s.c.a.AnnotationConfigApplicationContext : Closing org.springframework.context.annotation.AnnotationConfigApplicationContext@4604b900: startup
date [Sat Mar 18 18:38:00 CET 2017]; root of context hierarchy
2017-03-18 18:38:06.682  INFO 6740 --- [       Thread-8] o.s.c.support.DefaultLifecycleProcessor  : Stopping beans in phase 0

Results :

Tests run: 3, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 43.294 s
[INFO] Finished at: 2017-03-18T18:38:36+01:00
[INFO] Final Memory: 18M/209M
[INFO] ------------------------------------------------------------------------
```

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-kafka/tree/master/spring-kafka-avro).
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This concludes the example on how to send/receive Avro messages using Spring Kafka. I created this blog post based on a user request so if you found this tutorial useful or would like to see another variation, let me know.