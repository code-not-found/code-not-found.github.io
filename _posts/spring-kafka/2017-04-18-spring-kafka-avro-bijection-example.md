---
title: "Spring Kafka - Avro Bijection Example"
permalink: /2017/04/spring-kafka-avro-bijection-example.html
excerpt: "A detailed step-by-step tutorial on how to implement an Avro Serializer &amp; Deserializer using Twitter Bijection, Spring Kafka and Spring Boot."
date: 2017-04-18
modified: 2017-04-18
header:
  teaser: "assets/images/spring-kafka-teaser.jpg"
categories: [Spring Kafka]
tags: [Apache Kafka, Apache Avro, Avro, Bijection, Deserializer, Example, Maven, Serializer, Spring, Spring Boot, Spring Kafka, Tutorial, Twitter Bijection]
redirect_from:
  - /2017/03/spring-kafka-avro-bijection-example.html
published: true
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/spring-logo.jpg" alt="spring logo" class="logo">
</figure>

[Twitter Bijection](https://github.com/twitter/bijection) is an invertible function library that converts back and forth between two types. It supports a number of types including Apache Avro. In the following tutorial we will configure, build and run an example in which we will send/receive an Avro message to/from Apache Kafka using Bijection, Apache Avro, Spring Kafka, Spring Boot and Maven.

Tools used:
* Twitter Bijection 0.9
* Apache Avro 1.8
* Spring Kafka 1.2
* Spring Boot 1.5
* Maven 3.5

# General Project Setup

We base this example on a previous [Spring Kafka Avro serializer/deserializer example]({{ site.url }}//2017/03/spring-kafka-apache-avro-example.html) in which we used the Avro API's to serialize and deserialize objects. For this tutorial we will be using the Bijection APIs which are a bit easier to use as we will see further down below.

Starting point is again the <var>user.avsc</var> schema from the [Avro getting started guide](https://avro.apache.org/docs/current/gettingstartedjava.html#Defining+a+schema). It describes the fields and their types of a `User` type.

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

We setup our project using [Maven](https://maven.apache.org/). In the POM file we add the `bijection-avro_2.11` dependency. The <var>artifactId</var> suffix of the dependency (in this case __2.11_) highlights the [Scala](https://www.scala-lang.org/) version used to compile the JAR.

> Note that we choose the <var>2.11</var> version of `bijection-avro` since `spring-kafka-test` includes a dependency on the <var>2.11</var> version of the `scala-library`.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>spring-kafka-avro-bijection</artifactId>
  <version>0.0.1-SNAPSHOT</version>

  <name>spring-kafka-avro-bijection</name>
  <description>Spring Kafka - Avro Bijection Example</description>
  <url>https://www.codenotfound.com/2017/03/spring-kafka-avro-bijection-example.html</url>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.2.RELEASE</version>
  </parent>

  <properties>
    <java.version>1.8</java.version>

    <spring-kafka.version>1.2.0.RELEASE</spring-kafka.version>
    <avro.version>1.8.1</avro.version>
    <bijection.version>0.9.5</bijection.version>
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
    <!-- bijection-avro -->
    <dependency>
      <groupId>com.twitter</groupId>
      <artifactId>bijection-avro_2.11</artifactId>
      <version>${bijection.version}</version>
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

Generation of the Avro `User` class is done by executing below Maven command. The result is a `User` class that contains the schema and `Builder` methods.

``` plaintext
mvn generate-sources
```

<figure>
    <img src="{{ site.url }}/assets/images/spring-kafka/bijection-avro-generated-java-classes.png" alt="bijection avro generated java classes">
</figure>

# Producing Avro Messages to a Kafka Topic

Serializing an Avro message to a `byte[]` array using Bijection can be achieved in just two lines of code as shown below.

We first create an `Injection` which is an object that can make the conversion in one way or the other. This is done by calling the static `toBinary()` method on the `GenericAvroCodecs` class. This returns and `Injection` capable of serializing and deserializing a generic Avro record using `org.apache.avro.io.BinaryEncoder`. As input parameter we need to supply the Avro schema which we get from the passed object.

The 'apply()' method is then used to create the `Byte` array which is returned.

``` java
package com.codenotfound.kafka.serializer;

import java.util.Map;

import javax.xml.bind.DatatypeConverter;

import org.apache.avro.generic.GenericRecord;
import org.apache.avro.specific.SpecificRecordBase;
import org.apache.kafka.common.serialization.Serializer;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import com.twitter.bijection.Injection;
import com.twitter.bijection.avro.GenericAvroCodecs;

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
    LOGGER.debug("data to serialize='{}'", data);

    Injection<GenericRecord, byte[]> genericRecordInjection =
        GenericAvroCodecs.toBinary(data.getSchema());
    byte[] result = genericRecordInjection.apply(data);

    LOGGER.debug("serialized data='{}'", DatatypeConverter.printHexBinary(result));
    return result;
  }
}
```

# Consuming Avro Messages from a Kafka Topic

Deserializing an Avro message from a `byte[]` array using Bijection is also done using an `Injection`. Creation is identical as to what we did in the `AvroSerializer` class.

We then create a GenericRecord from the received data using the `invert()` method. Finally using `deepCopy()` we extract the received data object and return it.

``` java
package com.codenotfound.kafka.serializer;

import java.util.Arrays;
import java.util.Map;

import javax.xml.bind.DatatypeConverter;

import org.apache.avro.Schema;
import org.apache.avro.generic.GenericRecord;
import org.apache.avro.specific.SpecificData;
import org.apache.avro.specific.SpecificRecordBase;
import org.apache.kafka.common.errors.SerializationException;
import org.apache.kafka.common.serialization.Deserializer;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import com.twitter.bijection.Injection;
import com.twitter.bijection.avro.GenericAvroCodecs;

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
    LOGGER.debug("data to deserialize='{}'", DatatypeConverter.printHexBinary(data));
    try {
      // get the schema
      Schema schema = targetType.newInstance().getSchema();

      Injection<GenericRecord, byte[]> genericRecordInjection = GenericAvroCodecs.toBinary(schema);
      GenericRecord genericRecord = genericRecordInjection.invert((byte[]) data).get();
      T result = (T) SpecificData.get().deepCopy(schema, genericRecord);

      LOGGER.debug("data='{}'", result);
      return result;
    } catch (Exception e) {
      throw new SerializationException(
          "Can't deserialize data [" + Arrays.toString(data) + "] from topic [" + topic + "]", e);
    }
  }
}
```

# Test Sending and Receiving Avro Messages on Kafka

The `SpringKafkaApplicationTest` test case demonstrates the above sample code. [An embedded Kafka and ZooKeeper server are automatically started]({{ site.url }}/2016/10/spring-kafka-embedded-server-unit-test.html) using a JUnit ClassRule. Using `@Before` we wait until all the partitions are assigned to our `Receiver` by looping over the available `ConcurrentMessageListenerContainer` (if we don't do this the message will already be sent before the listeners are assigned to the topic).

In the `testReceiver()` test case an Avro `User` object is created using the `Builder` methods. This user is then sent to <var>'avro-bijection.t'</var> topic. Finally the `CountDownLatch` from the `Receiver` is used to verify that a message was successfully received.

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
  public static KafkaEmbedded embeddedKafka = new KafkaEmbedded(1, true, "avro-bijection.t");

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
    User user = User.newBuilder().setName("John Doe").setFavoriteColor("blue")
        .setFavoriteNumber(null).build();
    sender.send(user);

    receiver.getLatch().await(10000, TimeUnit.MILLISECONDS);
    assertThat(receiver.getLatch().getCount()).isEqualTo(0);
  }
}
```

> Note that the sample code also contains `AvroSerializerTest` and `AvroDeserializerTest` unit test cases to verify the serialization classes.

Trigger the above test case using a command prompt and following Maven command:

``` plaintext
mvn test
```

Maven will do the necessary and the outcome should be a successful build as shown below:

``` plaintext
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.5.2.RELEASE)

22:04:35.984 [main] INFO  c.c.kafka.SpringKafkaApplicationTest - Starting SpringKafkaApplicationTest on cnf-pc with PID 4424 (started by CodeNotFound in c:\code\st\spring-kafka\spring-kafka-avro-bijection)
22:04:35.984 [main] DEBUG c.c.kafka.SpringKafkaApplicationTest - Running with Spring Boot v1.5.2.RELEASE, Spring v4.3.7.RELEASE
22:04:35.984 [main] INFO  c.c.kafka.SpringKafkaApplicationTest - No active profile set, falling back to default profiles: default
22:04:36.002 [main] INFO  o.s.c.a.AnnotationConfigApplicationContext - Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@bd2f5a9: startup date [Tue Apr 18 22:04:36 CEST 2017]; root of context hierarchy
22:04:36.368 [main] INFO  o.s.c.s.PostProcessorRegistrationDelegate$BeanPostProcessorChecker - Bean 'org.springframework.kafka.annotation.KafkaBootstrapConfiguration' of type [org.springframework.kafka.annotation.KafkaBootstrapConfiguration$$EnhancerBySpringCGLIB$$be4aa608] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
22:04:36.611 [main] INFO  o.s.c.s.DefaultLifecycleProcessor - Starting beans in phase 0
22:04:36.659 [main] INFO  c.c.kafka.SpringKafkaApplicationTest - Started SpringKafkaApplicationTest in 0.995 seconds (JVM running for 5.352)
22:04:37.889 [org.springframework.kafka.KafkaListenerEndpointContainer#0-0-C-1] INFO  o.s.k.l.KafkaMessageListenerContainer - partitions revoked:[]
22:04:37.964 [org.springframework.kafka.KafkaListenerEndpointContainer#0-0-C-1] INFO  o.s.k.l.KafkaMessageListenerContainer - partitions assigned:[avro-bijection.t-0, avro-bijection.t-1]
22:04:37.993 [main] INFO  c.codenotfound.kafka.producer.Sender - sending user='{"name": "John Doe", "favorite_number": null, "favorite_color": "blue"}'
22:04:38.011 [main] DEBUG c.c.kafka.serializer.AvroSerializer - data to serialize='{"name": "John Doe", "favorite_number": null, "favorite_color": "blue"}'
22:04:38.011 [main] DEBUG c.c.kafka.serializer.AvroSerializer - serialized data='104A6F686E20446F65020008626C7565'
22:04:38.032 [org.springframework.kafka.KafkaListenerEndpointContainer#0-0-C-1] DEBUG c.c.k.serializer.AvroDeserializer - data to deserialize='104A6F686E20446F65020008626C7565'
22:04:38.032 [org.springframework.kafka.KafkaListenerEndpointContainer#0-0-C-1] DEBUG c.c.k.serializer.AvroDeserializer - data='{"name": "John Doe", "favorite_number": null, "favorite_color": "blue"}'
22:04:38.039 [org.springframework.kafka.KafkaListenerEndpointContainer#0-0-L-1] INFO  c.c.kafka.consumer.Receiver - received user='{"name": "John Doe", "favorite_number": null, "favorite_color": "blue"}'
22:04:40.300 [main] ERROR o.a.zookeeper.server.ZooKeeperServer - ZKShutdownHandler is not registered, so ZooKeeper server won't take any action on ERROR or SHUTDOWN server state changes
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 9.197 sec - in com.codenotfound.kafka.SpringKafkaApplicationTest
22:04:41.326 [Thread-8] INFO  o.s.c.a.AnnotationConfigApplicationContext - Closing org.springframework.context.annotation.AnnotationConfigApplicationContext@bd2f5a9: startup date [Tue Apr 18 22:04:36CEST 2017]; root of context hierarchy
22:04:41.330 [Thread-8] INFO  o.s.c.s.DefaultLifecycleProcessor - Stopping beans in phase 0

Results :

Tests run: 3, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 42.206 s
[INFO] Finished at: 2017-04-18T22:05:11+02:00
[INFO] Final Memory: 18M/212M
[INFO] ------------------------------------------------------------------------
```

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-kafka/tree/master/spring-kafka-avro-bijection).
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This wraps up the example on how to send/receive Avro messages using Twitter Bijection and Spring Kafka.

Let me know if something is missing or if you were able to successfully use the above code.
