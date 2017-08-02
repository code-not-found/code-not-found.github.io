---
title: "Spring Kafka - Spring Boot Example"
permalink: /spring-kafka-boot-example.html
excerpt: "A detailed step-by-step tutorial on how to setup Spring Kafka using Spring Boot autoconfiguration."
date: 2017-04-13
modified: 2017-04-13
header:
  teaser: "assets/images/teaser/spring-kafka-teaser.png"
categories: [Spring Kafka]
tags: [Autoconfig, Autoconfiguration, Apache Kafka, Example, Maven, Spring, Spring Boot, Spring Kafka, Tutorial]
redirect_from:
  - /2017/04/spring-kafka-boot-example.html
published: true
---

<figure>
    <img src="{{ site.url }}/assets/images/logo/spring-logo.png" alt="spring logo" class="logo">
</figure>

[Spring Boot auto-configuration](http://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-auto-configuration.html){:target="_blank"} attempts to automatically configure your Spring application based on the JAR dependencies that have been added. In other words, if the <var>spring-kafka-1.2.2.RELEASE.jar</var> is on the classpath and you have not manually configured any `Consumer` or `Provider` beans, then Spring Boot will auto-configure them using default values.

In order to demonstrate this behavior we will start from a previous [Spring Kafka tutorial]({{ site.url }}/spring-kafka-consumer-producer-example.html) in which we send/receive messages to/from an Apache Kafka topic using Spring Kafka. The original code will be reduced to a bare minimum in order to demonstrate Spring Boot's autoconfiguration.

If you want to learn more about Spring Kafka - head on over to the [Spring Kafka tutorials page]({{ site.url }}/spring-kafka/).
{: .notice--primary}

# General Project Setup

Tools used:
* Spring Kafka 1.2
* Spring Boot 1.5
* Maven 3.5

The project is built using [Maven](https://maven.apache.org/){:target="_blank"}. The Maven POM file contains the needed dependencies for [Spring Boot](https://projects.spring.io/spring-boot/){:target="_blank"} and [Spring Kafka](https://projects.spring.io/spring-kafka/){:target="_blank"} as shown below.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>spring-kafka-boot</artifactId>
  <version>0.0.1-SNAPSHOT</version>

  <name>spring-kafka-boot</name>
  <description>Spring Kafka - Spring Boot Example</description>
  <url>https://www.codenotfound.com/spring-kafka-boot-example.html</url>

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

The `SpringKafkaApplication` remains unchanged. What is important to note is that in order for the auto-configuration to work we need to opt-in by adding the `@EnableAutoConfiguration` or `@SpringBootApplication` (which is same as adding `@Configuration` `@EnableAutoConfiguration` `@ComponentScan`) annotation to one of our `@Configuration` classes.

> You should only ever add one `@EnableAutoConfiguration` annotation. It is recommended to add it to your primary `@Configuration` class.

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

# Autoconfigure the Spring Kafka Message Producer

The setup and creation of the `KafkaTemplate` and `Producer` beans is automatically done by Spring Boot. The only things left to do are auto-wiring the `KafkaTemplate` and using it in the `send()` method.

> By annotating the `Sender` class with `@Component`, Spring will instantiate this class as a bean that we will use in our test case. In order for this to work, we also need the `@EnableAutoConfiguration` which was indirectly specified on `SpringKafkaApplication` by using the `@SpringBootApplication` annotation.

``` java
package com.codenotfound.kafka.producer;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.stereotype.Component;

@Component
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

# Autoconfigure the Spring Kafka Message Consumer

Similar to the `Sender`, the setup and creation of the `ConcurrentKafkaListenerContainerFactory` and `KafkaMessageListenerContainer` beans is automatically done by Spring Boot.

The `@KafkaListener` annotation creates a message listener container for the annotated `receive()` method. The topic name is specified using the `${kafka.topic.boot}` placeholder for which the value will be automatically fetched from the <var>application.yml</var> properties file.

``` java
package com.codenotfound.kafka.consumer;

import java.util.concurrent.CountDownLatch;

import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Component;

@Component
public class Receiver {

  private static final Logger LOGGER = LoggerFactory.getLogger(Receiver.class);

  private CountDownLatch latch = new CountDownLatch(1);

  public CountDownLatch getLatch() {
    return latch;
  }

  @KafkaListener(topics = "${kafka.topic.boot}")
  public void receive(ConsumerRecord<?, ?> consumerRecord) {
    LOGGER.info("received payload='{}'", consumerRecord.toString());
    latch.countDown();
  }
}
```

For the `Receiver`, Spring Boot takes care of most of the configuration. There are however two properties that need to be explicitly set in the <var>application.yml</var> properties file:

1. The `kafka.consumer.auto-offset-reset` property needs to be set to <var>'earliest'</var> which ensures the new consumer group will get the message sent in case the container started after the send was completed.
2. The `kafka.consumer.group-id` property needs to be specified as we are [using group management to assign topic partitions to consumers](http://docs.confluent.io/current/clients/consumer.html#concepts){:target="_blank"}. In this example we will assign it the value <var>'boot'</var>.

``` yml
spring:
  kafka:
    consumer:
      auto-offset-reset: earliest
      group-id: boot

kafka:
  topic:
    boot: boot.t
```

> Scroll down to <var># APACHE KAFKA</var> in the following link in order to get a complete [overview of all the Spring Kafka properties](https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html){:target="_blank"} that can be set for auto configuration using the Spring Boot application properties file.

# Testing the Sender and Receiver

In order to verify that our code works, a basic `SpringKafkaApplicationTest` test case is used. It contains a `testReceiver()` unit test case that uses the `Sender` to send a message to the <var>'boot.t'</var> topic on the Kafka bus. We then use the `CountDownLatch` from the `Receiver` to verify that a message was successfully received.

The test case runs using [the embedded Kafka broker which is started via a JUnit @ClassRule]({{ site.url }}/spring-kafka-embedded-unit-test-example.html).

> Note that we have added a dedicated <var>application.yml</var> properties file under <var>src/test/resources</var> in order to override the default broker address with the address of the embedded broker using the `spring.kafka.bootstrap-servers` property.

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
import org.springframework.test.context.junit4.SpringRunner;

import com.codenotfound.kafka.consumer.Receiver;
import com.codenotfound.kafka.producer.Sender;

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringKafkaApplicationTest {

  private static String BOOT_TOPIC = "boot.t";

  @Autowired
  private Sender sender;

  @Autowired
  private Receiver receiver;

  @ClassRule
  public static KafkaEmbedded embeddedKafka = new KafkaEmbedded(1, true, BOOT_TOPIC);

  @Test
  public void testReceive() throws Exception {
    sender.send(BOOT_TOPIC, "Hello Boot!");

    receiver.getLatch().await(10000, TimeUnit.MILLISECONDS);
    assertThat(receiver.getLatch().getCount()).isEqualTo(0);
  }
}
```

Fire up the above test case by opening a command prompt and execute following Maven command:

``` plaintext
mvn test
```

Maven will then download the dependencies, compile the code and run the unit test case during which following logs should be generated:

``` plaintext
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.5.4.RELEASE)

20:58:18.284 [main] INFO  c.c.kafka.SpringKafkaApplicationTest - Starting SpringKafkaApplicationTest on cnf-pc with PID 6032 (started by CodeNotFound in c:\codenotfound\code\spring-kafka\spring-kafka-avr
o)
20:58:18.284 [main] DEBUG c.c.kafka.SpringKafkaApplicationTest - Running with Spring Boot v1.5.4.RELEASE, Spring v4.3.9.RELEASE
20:58:18.285 [main] INFO  c.c.kafka.SpringKafkaApplicationTest - No active profile set, falling back to default profiles: default
20:58:19.011 [main] INFO  c.c.kafka.SpringKafkaApplicationTest - Started SpringKafkaApplicationTest in 1.07 seconds (JVM running for 5.484)
20:58:20.348 [main] INFO  c.codenotfound.kafka.producer.Sender - sending user='{"name": "John Doe", "favorite_number": null, "favorite_color": "green"}'
20:58:20.364 [main] DEBUG c.c.kafka.serializer.AvroSerializer - data='{"name": "John Doe", "favorite_number": null, "favorite_color": "green"}'
20:58:20.364 [main] DEBUG c.c.kafka.serializer.AvroSerializer - serialized data='104A6F686E20446F6502000A677265656E'
20:58:20.383 [org.springframework.kafka.KafkaListenerEndpointContainer#0-0-C-1] DEBUG c.c.k.serializer.AvroDeserializer - data='104A6F686E20446F6502000A677265656E'
20:58:20.384 [org.springframework.kafka.KafkaListenerEndpointContainer#0-0-C-1] DEBUG c.c.k.serializer.AvroDeserializer - deserialized data='{"name": "John Doe", "favorite_number": null, "favorite_color": "green"}'
20:58:20.390 [org.springframework.kafka.KafkaListenerEndpointContainer#0-0-C-1] INFO  c.c.kafka.consumer.Receiver - received user='{"name": "John Doe", "favorite_number": null, "favorite_color": "green"}'
20:58:21.599 [main] ERROR o.a.zookeeper.server.ZooKeeperServer - ZKShutdownHandler is not registered, so ZooKeeper server won't take any action on ERROR or SHUTDOWN server state changes
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 8.248 sec - in com.codenotfound.kafka.SpringKafkaApplicationTest

Results :

Tests run: 3, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 11.217 s
[INFO] Finished at: 2017-08-02T20:58:22+02:00
[INFO] Final Memory: 19M/211M
[INFO] ------------------------------------------------------------------------
```

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-kafka/tree/master/spring-kafka-boot).
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

Using Spring Boot's autoconfiguration we were able to setup a `Sender` and `Receiver` using only a couple of lines of code. Hopefully, this example will kick-start your Spring Kafka development.

Feel free to leave a comment in case something was not clear or just to let me know if everything worked.
