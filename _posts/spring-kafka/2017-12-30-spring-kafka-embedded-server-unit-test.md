---
title: "Spring Kafka - Embedded Server Unit Test"
permalink: /spring-kafka-embedded-server-unit-test.html
excerpt: "A detailed step-by-step tutorial on how to test your application using an embedded Apache Kafka server together with Spring Kafka and Spring Boot."
date: 2016-10-04
modified: 2016-10-04
header:
  teaser: "assets/images/teaser/spring-kafka-teaser.png"
categories: [Spring Kafka]
tags: [Apache Kafka, Embedded, Example, Maven, Server, Spring Boot, Spring Kafka, Test, Testing, Tutorial, Unit]
redirect_from:
  - /2016/09/spring-kafka-embedded-kafka-test-example.html
  - /2016/10/spring-kafka-embedded-server-unit-test.html
published: false
---

<figure>
    <img src="{{ site.url }}/assets/images/logo/spring-logo.png" alt="spring logo" class="logo">
</figure>

The [Spring Kafka project](https://projects.spring.io/spring-kafka/){:target="_blank"} comes with a `spring-kafka-test` JAR that contains a number of [useful utilities](http://docs.spring.io/spring-kafka/docs/1.2.2.RELEASE/reference/html/_reference.html#testing){:target="_blank"}. to assist you with your application testing. These include: an embedded Kafka server, some static methods to setup consumers/producers and utility methods to fetch results

Let's demonstrate how above can be used with a simple code sample. We will reuse the Spring Kafka Hello World project from a previous post in which we created a consumer and producer using Spring Kafka, Spring Boot and Maven.

If you want to learn more about Spring Kafka - head on over to the [Spring Kafka tutorials page]({{ site.url }}/spring-kafka/).
{: .notice--primary}

# General Project Setup

Tools used:
* Spring Kafka 1.2
* Spring Boot 1.5
* Maven 3.5

Add the `spring-kafka-test` dependency to the Maven POM file in addition to the Spring Kafka and Spring Boot dependencies.

In the plugins section, we included the `maven-surefire-plugin` to trigger an `AllSpringKafkaTests` test suite class that will be used to start the embedded server for the different unit test cases in our project. This allows us to start a single embedded Kafka server and reuse it for the different unit test cases.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>spring-kafka-embedded-test</artifactId>
  <version>0.0.1-SNAPSHOT</version>

  <name>spring-kafka-embedded-test</name>
  <description>Spring-Kafka - Embedded Kafka Test Example</description>
  <url>https://www.codenotfound.com/2016/10/spring-kafka-embedded-server-unit-test.html</url>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.4.RELEASE</version>
  </parent>

  <properties>
    <java.version>1.8</java.version>

    <spring-kafka.version>1.2.2.RELEASE</spring-kafka.version>
    <maven-surefire-plugin.version>2.20</maven-surefire-plugin.version>
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
      <!-- maven-surefire-plugin -->
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <version>${maven-surefire-plugin.version}</version>
        <configuration>
          <includes>
            <include>AllSpringKafkaTests.java</include>
          </includes>
        </configuration>
      </plugin>
    </plugins>
  </build>
</project>
```

The message consumer and producer classes from the Hello World example are unchanged so we won't go into detail explaining them. You can check out the [Spring Boot Kafka example]({{ site.url }}/2016/09/spring-kafka-consumer-producer-example.html) from a previous post for more details.

# Unit Testing with an Embedded Kafka Server

`spring-kafka-test` includes an embedded Kafka server that can be created via a JUnit `@ClassRule` annotation. The rule will start a [ZooKeeper](https://zookeeper.apache.org/){:target="_blank"} and [Kafka](https://kafka.apache.org/){:target="_blank"} server instance on a random port before all the test cases are run, and stops the instances once the test cases are finished.

In order to support multiple unit test classes (in this example: `SpringKafkaApplicationTest`, `SpringKafkaSenderTest` and `SpringKafkaReceiverTest`), we will trigger the `@ClassRule` from a `Suite` class that bundles these test cases together. This allows us to only start the embedded broker once for all test cases. If you have only one test class then you can [trigger the @ClassRule directly from the test case](https://github.com/code-not-found/spring-kafka/blob/master/spring-kafka-helloworld/src/test/java/com/codenotfound/kafka/SpringKafkaApplicationTest.java){:target="_blank"}.

The `KafkaEmbedded` constructor takes as parameters: the number of Kafka brokers to start, whether a controlled shutdown is needed and the topics that need to be created on the broker.

> Note that we pass the <var>'sender.t'</var> and <var>'receiver.t'</var> topics as a parameter to the embedded kafka server. This assures that the topic is not auto-created and present when the `MessageListener` connects.

As the embedded server is started on a random port, we need to change the property value that is used by the `SenderConfig` and `ReceiverConfig` classes. This is done by calling the `getBrokersAsString()` method and setting the value to the <var>'kafka.bootstrap-servers'</var> property.

``` java
package com.codenotfound.kafka;

import org.junit.BeforeClass;
import org.junit.ClassRule;
import org.junit.runner.RunWith;
import org.junit.runners.Suite;
import org.junit.runners.Suite.SuiteClasses;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.kafka.test.rule.KafkaEmbedded;

import com.codenotfound.kafka.consumer.SpringKafkaReceiverTest;
import com.codenotfound.kafka.producer.SpringKafkaSenderTest;

@RunWith(Suite.class)
@SuiteClasses({SpringKafkaApplicationTest.class, SpringKafkaSenderTest.class,
    SpringKafkaReceiverTest.class})
public class AllSpringKafkaTests {

  private static final Logger LOGGER = LoggerFactory.getLogger(AllSpringKafkaTests.class);

  public static final String SENDER_TOPIC = "sender.t";
  public static final String RECEIVER_TOPIC = "receiver.t";

  @ClassRule
  public static KafkaEmbedded embeddedKafka =
      new KafkaEmbedded(1, true, SENDER_TOPIC, RECEIVER_TOPIC);

  @BeforeClass
  public static void setUpBeforeClass() throws Exception {
    String kafkaBootstrapServers = embeddedKafka.getBrokersAsString();

    LOGGER.debug("kafkaServers='{}'", kafkaBootstrapServers);
    // override the property in application.properties
    System.setProperty("kafka.bootstrap-servers", kafkaBootstrapServers);
  }
}
```

# Testing the Producer

In the `SpringKafkaSenderTest` unit test case we will be testing the `Sender` by sending a message to a <var>'sender.t'</var> topic. We will verify the correct sending by setting up a _test-listener_ on the topic. All of the setup will be done before the test case runs using the `@Before` annotation.

For creating the needed connection consumer properties a static `consumerProps()` method provided by `KafkaUtils` is used. We then create a `DefaultKafkaConsumerFactory` and `ContainerProperties` which contains runtime properties (in this case the topic name) for the listener container. Both are then used to set up the `KafkaMessageListenerContainer`.

Received messages need to be stored somewhere. In this example, a thread safe `BlockingQueue` is used. We create a new `MessageListener` and in the `onMessage()` method we add the received message to the `BlockingQueue`. The listener is started by starting the container.

In order to avoid that we send a message before the container has required the number of assigned partitions, we use the `waitForAssignment()` method on the `ContainerTestUtils` helper class.

The actual test itself consists out of sending a greeting and asserting that the received value is the same as the one that was sent using an AssertJ condition that is provided by `KafkaConditions` (also included in the `spring-kafka-test` dependency). 

``` java
package com.codenotfound.kafka.producer;

import static org.assertj.core.api.Assertions.assertThat;
import static org.springframework.kafka.test.assertj.KafkaConditions.value;

import java.util.Map;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.TimeUnit;

import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.kafka.core.DefaultKafkaConsumerFactory;
import org.springframework.kafka.listener.KafkaMessageListenerContainer;
import org.springframework.kafka.listener.MessageListener;
import org.springframework.kafka.listener.config.ContainerProperties;
import org.springframework.kafka.test.utils.ContainerTestUtils;
import org.springframework.kafka.test.utils.KafkaTestUtils;
import org.springframework.test.context.junit4.SpringRunner;

import com.codenotfound.kafka.AllSpringKafkaTests;

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringKafkaSenderTest {

  private static final Logger LOGGER = LoggerFactory.getLogger(SpringKafkaSenderTest.class);


  private KafkaMessageListenerContainer<String, String> container;

  private BlockingQueue<ConsumerRecord<String, String>> records;

  @Autowired
  private Sender sender;

  @Before
  public void setUp() throws Exception {
    // set up the Kafka consumer properties
    Map<String, Object> consumerProperties =
        KafkaTestUtils.consumerProps("sender_group", "false", AllSpringKafkaTests.embeddedKafka);

    // create a Kafka consumer factory
    DefaultKafkaConsumerFactory<String, String> consumerFactory =
        new DefaultKafkaConsumerFactory<String, String>(consumerProperties);

    // set the topic that needs to be consumed
    ContainerProperties containerProperties =
        new ContainerProperties(AllSpringKafkaTests.SENDER_TOPIC);

    // create a Kafka MessageListenerContainer
    container = new KafkaMessageListenerContainer<>(consumerFactory, containerProperties);

    // create a thread safe queue to store the received message
    records = new LinkedBlockingQueue<>();

    // setup a Kafka message listener
    container.setupMessageListener(new MessageListener<String, String>() {
      @Override
      public void onMessage(ConsumerRecord<String, String> record) {
        LOGGER.debug("test-listener received message='{}'", record.toString());
        records.add(record);
      }
    });

    // start the container and underlying message listener
    container.start();
    // wait until the container has the required number of assigned partitions
    ContainerTestUtils.waitForAssignment(container,
        AllSpringKafkaTests.embeddedKafka.getPartitionsPerTopic());
  }

  @After
  public void tearDown() {
    // stop the container
    container.stop();
  }

  @Test
  public void testSend() throws Exception {
    // send the message
    String greeting = "Hello Spring Kafka Sender!";
    sender.send(AllSpringKafkaTests.SENDER_TOPIC, greeting);

    // check that the message was received
    assertThat(records.poll(10, TimeUnit.SECONDS)).has(value(greeting));
  }
}
```

# Testing the Consumer

The second `SpringKafkaReceiverTest` test class focuses on the `Receiver` which listens to a <var>'receiver.t'</var> topic as defined in the <var>applications.yml</var> properties file. In order to check the correct working, we will use a _test-template_ to send a message to this topic. All of the setup will be done before the test case runs using the `@Before` annotation.

The producer connection properties are created using the static `senderProps()` method provided by `KafkaUtils`. These properties are then used to create a `DefaultKafkaProducerFactory` which is in turn used to create a `KafkaTemplate`. Finally we set the default topic that the template uses to <var>'receiver.t'</var>.

We need to ensure that the `Receiver` is initialized before sending the test message. For this we use the `waitForAssignment()` of `ContainerTestUtils`. The link to the message listener container is acquired by auto-wiring the `KafkaListenerEndpointRegistry` which manages the lifecycle of the listener containers that are not created manually.

> Note that if you do not manually create the topics using the `KafkaEmbedded` constructor (see `AllSpringKafkaTests`) you need to manually set the partitions per topic to 1 in the `waitForAssignment()` method instead of getting the partitions from the embedded Kafka server. The reason for this is that [it looks like 1 is used as a default for the number of partitions in case topics are created implicitly](http://stackoverflow.com/a/38660145/4201470){:target="_blank"}. 

In the test, we send a greeting and check that the message was received by asserting that the latch of the `Receiver` was lowered to zero.

``` java
package com.codenotfound.kafka.consumer;

import static org.assertj.core.api.Assertions.assertThat;

import java.util.Map;
import java.util.concurrent.TimeUnit;

import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.kafka.config.KafkaListenerEndpointRegistry;
import org.springframework.kafka.core.DefaultKafkaProducerFactory;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.core.ProducerFactory;
import org.springframework.kafka.listener.MessageListenerContainer;
import org.springframework.kafka.test.utils.ContainerTestUtils;
import org.springframework.kafka.test.utils.KafkaTestUtils;
import org.springframework.test.context.junit4.SpringRunner;

import com.codenotfound.kafka.AllSpringKafkaTests;

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringKafkaReceiverTest {

  private static final Logger LOGGER = LoggerFactory.getLogger(SpringKafkaReceiverTest.class);

  @Autowired
  private Receiver receiver;

  @Autowired
  private KafkaListenerEndpointRegistry kafkaListenerEndpointRegistry;

  private KafkaTemplate<String, String> template;

  @Before
  public void setUp() throws Exception {
    // set up the Kafka producer properties
    Map<String, Object> senderProperties =
        KafkaTestUtils.senderProps(AllSpringKafkaTests.embeddedKafka.getBrokersAsString());

    // create a Kafka producer factory
    ProducerFactory<String, String> producerFactory =
        new DefaultKafkaProducerFactory<String, String>(senderProperties);

    // create a Kafka template
    template = new KafkaTemplate<>(producerFactory);
    // set the default topic to send to
    template.setDefaultTopic(AllSpringKafkaTests.RECEIVER_TOPIC);

    // wait until the partitions are assigned
    for (MessageListenerContainer messageListenerContainer : kafkaListenerEndpointRegistry
        .getListenerContainers()) {
      ContainerTestUtils.waitForAssignment(messageListenerContainer,
          AllSpringKafkaTests.embeddedKafka.getPartitionsPerTopic());
    }
  }

  @Test
  public void testReceive() throws Exception {
    // send the message
    String greeting = "Hello Spring Kafka Receiver!";
    template.sendDefault(greeting);
    LOGGER.debug("test-sender sent message='{}'", greeting);

    receiver.getLatch().await(10000, TimeUnit.MILLISECONDS);
    // check that the message was received
    assertThat(receiver.getLatch().getCount()).isEqualTo(0);
  }
}
```

# Running the Unit Test Cases

Let's run above test cases by opening a command prompt and executing following Maven command:

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

11:52:33.853 [main] INFO  c.c.kafka.SpringKafkaApplicationTest - Starting SpringKafkaApplicationTest on cnf-pc with PID 4008 (started by CodeNotFound in c:\codenotfound\spring-kafka\spring-kafka-embedded-test)
11:52:33.854 [main] DEBUG c.c.kafka.SpringKafkaApplicationTest - Running with Spring Boot v1.5.2.RELEASE, Spring v4.3.7.RELEASE
11:52:33.855 [main] INFO  c.c.kafka.SpringKafkaApplicationTest - No active profile set, falling back to default profiles: default
11:52:34.546 [main] INFO  c.c.kafka.SpringKafkaApplicationTest - Started SpringKafkaApplicationTest in 0.982 seconds (JVM running for 5.125)
11:52:35.956 [main] INFO  c.codenotfound.kafka.producer.Sender - sending data='Hello Spring Kafka!' to topic='receiver.t'
11:52:36.017 [org.springframework.kafka.KafkaListenerEndpointContainer#0-0-L-1] INFO  c.c.kafka.consumer.Receiver - received message='Hello Spring Kafka!'
11:52:36.129 [main] INFO  c.codenotfound.kafka.producer.Sender - sending data='Hello Spring Kafka Sender!' to topic='sender.t'
11:52:36.139 [-L-1] DEBUG c.c.k.producer.SpringKafkaSenderTest - test-listener received message='ConsumerRecord(topic = sender.t, partition = 0, offset = 0, CreateTime = 1492509156132, checksum = 3163075024, serialized key size = -1, serialized value size = 26, key = null, value = Hello Spring Kafka Sender!)'
11:52:37.170 [main] DEBUG c.c.k.c.SpringKafkaReceiverTest - test-sender sent message='Hello Spring Kafka Receiver!'
11:52:37.179 [org.springframework.kafka.KafkaListenerEndpointContainer#0-0-L-1] INFO  c.c.kafka.consumer.Receiver - received message='Hello Spring Kafka Receiver!'
11:52:40.190 [main] ERROR o.a.zookeeper.server.ZooKeeperServer - ZKShutdownHandler is not registered, so ZooKeeper server won't take any action on ERROR or SHUTDOWN server state changes
Tests run: 3, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 11.595 sec - in com.codenotfound.kafka.AllSpringKafkaTests

Results :

Tests run: 3, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 43.630 s
[INFO] Finished at: 2017-04-18T11:53:11+02:00
[INFO] Final Memory: 16M/226M
[INFO] ------------------------------------------------------------------------
```

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-kafka/tree/master/spring-kafka-embedded-test).
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This concludes our example in which we test Spring Kafka by starting an embedded Kafka server.

Feel free to drop a line in case of any questions or if you found this post helpful.
