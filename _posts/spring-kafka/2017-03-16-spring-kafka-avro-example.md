---
title: "Spring Kafka - Apache Avro Example"
permalink: /2017/03/spring-kafka-apache-avro-example.html
excerpt: "A detailed step-by-step tutorial on how to implement an Apache Avro Serializer &amp; Deserializer using Spring Kafka and Spring Boot."
date: 2017-03-16
modified: 2017-04-17
header:
  teaser: "assets/images/spring-kafka-teaser.jpg"
categories: [Spring Kafka]
tags: [Apache Kafka, Apache Avro, Avro, Deserializer, Example, Maven, Serializer, Spring, Spring Boot, Spring Kafka, Tutorial]
redirect_from:
  - /2017/03/spring-kafka-avro-serializer-deserializer.html
  - /2017/03/spring-kafka-avro-example.html
published: true
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/spring-logo.jpg" alt="spring logo" class="logo">
</figure>

[Apache Avro](https://avro.apache.org/docs/current/) is a data serialization system. It uses JSON for defining data types and protocols, and serializes data in a compact binary format. In the following tutorial we will configure, build and run an example in which we will send/receive an Avro message to/from Apache Kafka using Apache Avro, Spring Kafka, Spring Boot and Maven.

Tools used:
* Apache Avro 1.8
* Spring Kafka 1.2
* Spring Boot 1.5
* Maven 3.5

# General Project Setup

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
  <url>https://www.codenotfound.com/2017/03/spring-kafka-apache-avro-example.html</url>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.2.RELEASE</version>
  </parent>

  <properties>
    <java.version>1.8</java.version>

    <spring-kafka.version>1.2.0.RELEASE</spring-kafka.version>
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

Kafka stores and transports `Byte` arrays in its topics. But as we are working with Avro objects we need to transform to/from these `Byte` arrays. Before version 0.9.0.0, the Kafka Java API used implementations of `Encoder`/`Decoder` interfaces to handle transformations but these have been replaced by `Serializer`/`Deserializer` interface implementations in the new API.

> Kafka ships with a number of [built in (de)serializers](https://kafka.apache.org/0102/javadoc/org/apache/kafka/common/serialization/Serializer.html) but an Avro one is not included.

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
          "Can't serialize data='" + data + "' for topic='" + topic + "'", ex);
    }
  }
}
```

Now we need to change the `SenderConfig` to start using our custom `Serializer` implementation. This is done by setting the <var>'VALUE_SERIALIZER_CLASS_CONFIG'</var> property to the `AvroSerializer` class. In addition we change the `ProducerFactory` and `KafkaTemplate` generic type so that it specifies `User` instead of `String`.

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

  @Value("${kafka.bootstrap-servers}")
  private String bootstrapServers;

  @Bean
  public Map<String, Object> producerConfigs() {
    Map<String, Object> props = new HashMap<>();

    props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
    props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
    props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, AvroSerializer.class);

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

  private static final Logger LOGGER = LoggerFactory.getLogger(Sender.class);

  @Value("${topic.avro}")
  private String avroTopic;

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
        LOGGER.debug("data='{}'", DatatypeConverter.printHexBinary(data));

        DatumReader<GenericRecord> datumReader =
            new SpecificDatumReader<>(targetType.newInstance().getSchema());
        Decoder decoder = DecoderFactory.get().binaryDecoder(data, null);

        result = (T) datumReader.read(null, decoder);
        LOGGER.debug("deserialized data='{}'", result);
      }
      return result;
    } catch (Exception ex) {
      throw new SerializationException(
          "Can't deserialize data '" + Arrays.toString(data) + "' from topic '" + topic + "'", ex);
    }
  }
}
```

The `ReceiverConfig` needs to be updated so that the `AvroDeserializer` is used as value for the <var>'VALUE_DESERIALIZER_CLASS_CONFIG'</var> property. We also change the `ConsumerFactory` and `ConcurrentKafkaListenerContainerFactory` generic type so that it specifies `User` instead of `String`. The `DefaultKafkaConsumerFactory` is created by passing a new `AvroDeserializer` that takes <var>'User.class'</var> as constructor argument.

> The `Class<?>` targetType of the `AvroDeserializer` is needed to allow the deserialization of a consumed `byte[]` to the proper target object (in this example the `User` class).

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

  @Value("${kafka.bootstrap-servers}")
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

Just like with the `Sender` class, the argument of the `receive()` method of the `Receiver` class needs to be changed to the Avro `User` class.

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

  public CountDownLatch getLatch() {
    return latch;
  }

  @KafkaListener(topics = "${topic.avro}")
  public void receive(User user) {
    LOGGER.info("received user='{}'", user.toString());
    latch.countDown();
  }
}
```

# Test Sending and Receiving Avro Messages on Kafka

The `SpringKafkaApplicationTest` test case demonstrates the above sample code. [An embedded Kafka and ZooKeeper server are automatically started]({{ site.url }}/2016/10/spring-kafka-embedded-server-unit-test.html) using a JUnit ClassRule. Using `@Before` we wait until all the partitions are assigned to our `Receiver` by looping over the available `ConcurrentMessageListenerContainer` (if we don't do this the message will already be sent before the listeners are assigned to the topic).

In the `testReceiver()` test case an Avro `User` object is created using the `Builder` methods. This user is then sent to <var>'avro.t'</var> topic. Finally the `CountDownLatch` from the `Receiver` is used to verify that a message was successfully received.

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
  public void testReceiver() throws Exception {
    User user = User.newBuilder().setName("John Doe").setFavoriteColor("green")
        .setFavoriteNumber(null).build();
    sender.send(user);

    receiver.getLatch().await(10000, TimeUnit.MILLISECONDS);
    assertThat(receiver.getLatch().getCount()).isEqualTo(0);
  }
}
```

> Note that the sample code also contains `AvroSerializerTest` and `AvroDeserializerTest` unit test cases to verify the serialization classes.

In order to run the above tests open a command prompt and execute following Maven command:

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

08:36:56.175 [main] INFO  c.c.kafka.SpringKafkaApplicationTest - Starting SpringKafkaApplicationTest on cnf-pc with PID 700 (started by CodeNotFound in c:\code\st\spring-kafka\spring-kafka-avro)
08:36:56.175 [main] INFO  c.c.kafka.SpringKafkaApplicationTest - No active profile set, falling back to default profiles: default
08:36:56.889 [main] INFO  c.c.kafka.SpringKafkaApplicationTest - Started SpringKafkaApplicationTest in 1.068 seconds (JVM running for 5.293)
08:36:58.223 [main] INFO  c.codenotfound.kafka.producer.Sender - sending user='{"name": "John Doe", "favorite_number": null, "favorite_color": "green"}'
08:36:58.271 [org.springframework.kafka.KafkaListenerEndpointContainer#0-0-L-1] INFO  c.c.kafka.consumer.Receiver - received user='{"name": "John Doe", "favorite_number": null, "favorite_color": "green"}'
08:37:00.240 [main] ERROR o.a.zookeeper.server.ZooKeeperServer - ZKShutdownHandler is not registered, so ZooKeeper server won't take any action on ERROR or SHUTDOWN server state changes
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 8.871 sec - in com.codenotfound.kafka.SpringKafkaApplicationTest

Results :

Tests run: 3, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 41.632 s
[INFO] Finished at: 2017-04-17T08:37:31+02:00
[INFO] Final Memory: 18M/212M
[INFO] ------------------------------------------------------------------------
```

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-kafka/tree/master/spring-kafka-avro).
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This concludes the example on how to send/receive Avro messages using Spring Kafka.

I created this blog post based on a user request so if you found this tutorial useful or would like to see another variation, let me know.
