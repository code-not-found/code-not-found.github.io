---
title: Spring Kafka - Avro Serializer &amp; Deserializer 
permalink: /2017/03/spring-kafka-avro-serializer-deserializer.html
excerpt: A detailed step-by-step tutorial on how to implement Apache Avro serialization &amp; deserialization using Spring Kafka and Spring Boot.
date: 2017-03-16 21:00
categories: [Spring Kafka]
tags: [Apache Kafka, Apache Avro, Avro, Deserializer, Example, Maven, Serializer, Spring, Spring Boot, Spring Kafka, Tutorial]
published: true
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/spring-logo.png" alt="spring logo">
</figure>

[Apache Avro](https://avro.apache.org/docs/current/) is a data serialization system. It uses JSON for defining data types and protocols, and serializes data in a compact binary format. In the following tutorial we will configure, build and run an example in which we will send/receive binary Avro messages to/from Apache Kafka using Spring Kafka, Spring Boot and Maven.

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

Avro ships with code generation which allows us to automatically create Java classes based on the above defined <var>'User'</var> schema. Once we have generated the relevant classes, there is no need to use the schema directly in our program. The classes can be generated using the <var>avro-tools.jar</var> or via the Avro Maven plugin, we will use the latter in this example.

We start from a previous [Spring Boot Kafka example]({{ site.url }}/2016/09/spring-kafka-consumer-producer-example.html) and add the `avro` Maven dependency to the dependencies section. In addition we configure the `avro-maven-plugin` to run the <var>'schema'</var> goal on all schema's that are found in the <var>/src/main/resources/avro/</var> location as shown below.

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

This results in the generation of a `User` class which contains the schema and a number of `Builder` methods to construct a User object.

# Producing Avro Messages to a Kafka Topic

Kafka stores and transports `Byte` arrays in its topics. But as we are working with Avro objects we need to transform to/from these `Byte` arrays. Before version 0.9.0.0, the Kafka Java API used implementations of `Encoder`/`Decoder` interfaces to handle transformations but these have been replaced by `Serializer`/`Deserializer` interface implementations in the new API. Kafka ships with a number of [built in (de)serializers](https://kafka.apache.org/0100/javadoc/org/apache/kafka/common/serialization/Serializer.html) but an Avro one is not included.

To tackle this we will create an `AvroSerializer` class that implements the `Serializer` interface specifically for Avro objects. We then implement the `serialize()` method which takes as input a topic name and a data object which in our case is an Avro object which extends `SpecificRecordBase`. The method [serializes the Avro object to a byte array](https://cwiki.apache.org/confluence/display/AVRO/FAQ#FAQ-Serializingtoabytearray) and returns the result.

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

Now we need to change the `SenderConfig` to start using our custom `Serializer` implementation. This is done by setting the `VALUE_SERIALIZER_CLASS_CONFIG` property to the `AvroSerializer` class. In addition we change the `ProducerFactory` and `KafkaTemplate` generic type so that it specifies `User` instead of `String`.

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

The `ReceiverConfig` needs to be updated so that the `AvroDeserializer` is used as value for the `VALUE_DESERIALIZER_CLASS_CONFIG` property. We also change the `ConsumerFactory` and `ConcurrentKafkaListenerContainerFactory` generic type so that it specifies `User` instead of `String`. The `DefaultKafkaConsumerFactory` is created by passing a new `AvroDeserializer` that takes `User.class` as constructor argument.

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

Just like with the Sender class, the argument of the `receive()` method of Receiver class needs to be changed to the Avro `User` class.

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

# Test Sending and Receiving of Avro Messages on Kafka

In order to verify that we are able to send and receive a message to and from Kafka, a basic `SpringKafkaApplicationTests` test case is used. It contains a `testReceiver()` unit test case that uses the `Sender` to send a message to the <var>helloworld.t</var> topic on the Kafka bus. We then use the `CountDownLatch` from the `Receiver` to verify that a message was received.

> Note that if the <var>helloworld.t</var> topic does not exist on the Kafka bus, the test case will not be successful. Reason for this is that when the consumer is starting up it tries to access the topic and fails. [By the time it retries, the topic has been created by the `sendMessage()` but also has data in it, so the consumer tries to connect at "latest"](https://issues.apache.org/jira/browse/KAFKA-3334), which is the offset of the most recent message, as such the consumer never receives the initial message. The next time you run the test case it will work since the topic exists.

``` java
package com.codenotfound.kafka;

import static org.assertj.core.api.Assertions.assertThat;

import java.util.concurrent.TimeUnit;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import com.codenotfound.kafka.consumer.Receiver;
import com.codenotfound.kafka.producer.Sender;

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringKafkaApplicationTests {

    @Autowired
    private Sender sender;

    @Autowired
    private Receiver receiver;

    @Test
    public void testReceiver() throws Exception {
        sender.sendMessage("helloworld.t", "Hello Spring Kafka!");

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

2017-03-11 19:39:36.634  INFO 3852 --- [           main] c.c.kafka.SpringKafkaApplicationTests    : Starting SpringKafkaApplicationTests on cnf-pc with PID 3852 (started by CodeNotFound in c:\code\st\spring-kafka\spring-kafka-helloworld-example)
2017-03-11 19:39:36.635  INFO 3852 --- [           main] c.c.kafka.SpringKafkaApplicationTests    : No active profile set, falling back to default profiles: default
2017-03-11 19:39:36.658  INFO 3852 --- [           main] s.c.a.AnnotationConfigApplicationContext : Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@5c90e579: startup date [Sat Mar 11 19:39:36 CET 2017]; root of context hierarchy
2017-03-11 19:39:37.037  INFO 3852 --- [           main] trationDelegate$BeanPostProcessorChecker : Bean 'org.springframework.kafka.annotation.KafkaBootstrapConfiguration' of type [org.springframework.kafka.annotation.KafkaBootstrapConfiguration$$EnhancerBySpringCGLIB$$3bb11429] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
2017-03-11 19:39:37.220  INFO 3852 --- [           main] o.s.c.support.DefaultLifecycleProcessor  : Starting beans in phase 0
2017-03-11 19:39:37.239  INFO 3852 --- [           main] o.a.k.clients.consumer.ConsumerConfig    : ConsumerConfig values:
2017-03-11 19:39:37.278  INFO 3852 --- [           main] o.a.k.clients.consumer.ConsumerConfig    : ConsumerConfig values:
2017-03-11 19:39:37.393  INFO 3852 --- [           main] o.a.kafka.common.utils.AppInfoParser     : Kafka version : 0.10.1.1
2017-03-11 19:39:37.393  INFO 3852 --- [           main] o.a.kafka.common.utils.AppInfoParser     : Kafka commitId : f10ef2720b03b247
2017-03-11 19:39:37.405  INFO 3852 --- [           main] c.c.kafka.SpringKafkaApplicationTests    : Started SpringKafkaApplicationTests in 1.109 seconds (JVM running for 1.726)
2017-03-11 19:39:37.446  INFO 3852 --- [           main] o.a.k.clients.producer.ProducerConfig    : ProducerConfig values:
2017-03-11 19:39:37.447  INFO 3852 --- [           main] o.a.k.clients.producer.ProducerConfig    : ProducerConfig values:
2017-03-11 19:39:37.460  INFO 3852 --- [           main] o.a.kafka.common.utils.AppInfoParser     : Kafka version : 0.10.1.1
2017-03-11 19:39:37.460  INFO 3852 --- [           main] o.a.kafka.common.utils.AppInfoParser     : Kafka commitId : f10ef2720b03b247
2017-03-11 19:39:37.532  INFO 3852 --- [afka-consumer-1] o.a.k.c.c.internals.AbstractCoordinator  : Discovered coordinator 192.168.1.5:9092 (id: 2147483647 rack: null) for group helloworld.
2017-03-11 19:39:37.536  INFO 3852 --- [afka-consumer-1] o.a.k.c.c.internals.ConsumerCoordinator  : Revoking previously assigned partitions [] for group helloworld
2017-03-11 19:39:37.536  INFO 3852 --- [afka-consumer-1] o.s.k.l.KafkaMessageListenerContainer    : partitions revoked:[]
2017-03-11 19:39:37.536  INFO 3852 --- [afka-consumer-1] o.a.k.c.c.internals.AbstractCoordinator  : (Re-)joining group helloworld
2017-03-11 19:39:37.546  INFO 3852 --- [afka-consumer-1] o.a.k.c.c.internals.AbstractCoordinator  : Successfully joined group helloworld with generation 7
2017-03-11 19:39:37.547  INFO 3852 --- [afka-consumer-1] o.a.k.c.c.internals.ConsumerCoordinator  : Setting newly assigned partitions [helloworld.t-0] for group helloworld
2017-03-11 19:39:37.556  INFO 3852 --- [afka-consumer-1] o.s.k.l.KafkaMessageListenerContainer    : partitions assigned:[helloworld.t-0]
2017-03-11 19:39:37.572  INFO 3852 --- [ad | producer-1] com.codenotfound.kafka.producer.Sender   : sent message='Hello Spring Kafka!' with offset=3
2017-03-11 19:39:37.581  INFO 3852 --- [afka-listener-1] c.codenotfound.kafka.consumer.Receiver   : received message='Hello Spring Kafka!'
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 1.417 sec - in com.codenotfound.kafka.SpringKafkaApplicationTests
2017-03-11 19:39:37.620  INFO 3852 --- [       Thread-2] s.c.a.AnnotationConfigApplicationContext : Closing org.springframework.context.annotation.AnnotationConfigApplicationContext@5c90e579: startupdate [Sat Mar 11 19:39:36 CET 2017]; root of context hierarchy
2017-03-11 19:39:37.622  INFO 3852 --- [       Thread-2] o.s.c.support.DefaultLifecycleProcessor  : Stopping beans in phase 0
2017-03-11 19:39:38.592  INFO 3852 --- [afka-consumer-1] essageListenerContainer$ListenerConsumer : Consumer stopped
2017-03-11 19:39:38.593  INFO 3852 --- [       Thread-2] o.a.k.clients.producer.KafkaProducer     : Closing the Kafka producer with timeoutMillis = 9223372036854775807 ms.

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 4.342 s
[INFO] Finished at: 2017-03-11T19:39:38+01:00
[INFO] Final Memory: 15M/227M
[INFO] ------------------------------------------------------------------------
```

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-kafka/tree/master/spring-kafka-avro).
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This wraps up our example in which we used a Spring Kafka template to create a producer and Spring Kafka listener to create a consumer. If you found this sample useful or have a question you would like to ask, drop a line below!