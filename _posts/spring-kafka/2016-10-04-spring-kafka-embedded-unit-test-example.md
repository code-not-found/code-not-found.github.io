---
title: "Spring Kafka - Embedded Unit Test Example"
permalink: /spring-kafka-embedded-unit-test-example.html
excerpt: "A detailed step-by-step tutorial on how to unit test your Spring Kafka application using an embedded server."
date: 2016-10-04
modified: 2016-10-04
header:
  teaser: "assets/images/teaser/spring-kafka-teaser.png"
categories: [Spring Kafka]
tags: [Apache Kafka, Embedded, Example, Maven, Server, Spring Boot, Spring Kafka, Test, Testing, Tutorial, Unit]
redirect_from:
  - /2016/09/spring-kafka-embedded-kafka-test-example.html
  - /2016/10/spring-kafka-embedded-server-unit-test.html
  - /spring-kafka-test-example.html
published: true
---

<figure>
    <img src="{{ site.url }}/assets/images/logo/spring-logo.png" alt="spring logo" class="logo">
</figure>

The [Spring Kafka project](https://projects.spring.io/spring-kafka/){:target="_blank"} comes with a `spring-kafka-test` JAR that contains a number of [useful utilities](http://docs.spring.io/spring-kafka/docs/1.2.2.RELEASE/reference/html/_reference.html#testing){:target="_blank"} to assist you with your application unit testing. These include an embedded Kafka server, some static methods to setup consumers/producers and utility methods to fetch results.

Let's demonstrate how these test utilities can be used with a code sample. We will start from a previous [Spring Kafka Maven project]({{ site.url }}/spring-kafka-consumer-producer-example.html) in which we created a consumer and producer using Spring Kafka, Spring Boot, and Maven.

If you want to learn more about Spring Kafka - head on over to the [Spring Kafka tutorials page]({{ site.url }}/spring-kafka/).
{: .notice--primary}

# General Project Setup

Tools used:
* Spring Kafka 1.2
* Spring Boot 1.5
* Maven 3.5

We start by adding the `spring-kafka-test` dependency to the Maven POM file in addition to the Spring Kafka and Spring Boot dependencies.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>spring-kafka-test</artifactId>
  <version>0.0.1-SNAPSHOT</version>

  <name>spring-kafka-test</name>
  <description>Spring Kafka - Embedded Unit Test Example</description>
  <url>https://www.codenotfound.com/spring-kafka-embedded-unit-test-example.html</url>

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

The message consumer and producer classes from the Hello World example are unchanged so we won't go into detail explaining them. You can check out the [Spring Kafka tutorial]({{ site.url }}/spring-kafka-consumer-producer-example.html) for more details.

# Unit Testing with an Embedded Kafka

`spring-kafka-test` includes an embedded Kafka server that can be created via a JUnit `@ClassRule` annotation. The rule will start a [ZooKeeper](https://zookeeper.apache.org/){:target="_blank"} and [Kafka](https://kafka.apache.org/){:target="_blank"} server instance on a random port before all the test cases are run, and stops the instances once the test cases are finished.

The `KafkaEmbedded` constructor takes as parameters: the number of Kafka servers to start, whether a controlled shutdown is needed and the topics that need to be created on the server.

> Always pass the topics as a parameter to the embedded Kafka server. This assures that the topics are not auto-created and present when the `MessageListener` connects.

As the embedded server is started on a random port, we need to change the property value that is used by the `SenderConfig` and `ReceiverConfig` classes. This is done by calling the `getBrokersAsString()` method and setting the value to the <var>'kafka.bootstrap-servers'</var> property before the tests are started.

> In order to have the correct broker address set on the `Sender` and `Receiver` beans during each test case we need to use the `@DirtiesContext` on all test classes. The reason for this is that each test case contains its own embedded Kafka broker that will each be created on a new random port. By [rebuilding the application context](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/integration-testing.html#__dirtiescontext){:target="_blank"}, the beans will always be set with the current broker address.

The below snippet shows how the embedded Kafa is defined in each test class.

``` java
  @ClassRule
  public static KafkaEmbedded embeddedKafka = new KafkaEmbedded(1, true, "topic");

  @BeforeClass
  public static void setUpBeforeClass() {
    System.setProperty("kafka.bootstrap-servers", embeddedKafka.getBrokersAsString());
  }
```

# Testing the Producer

In the `SpringKafkaSenderTest` test case we will be testing the `Sender` by sending a message to a <var>'sender.t'</var> topic. We will verify whether the sending works by setting up a _test-listener_ on the topic. All of the setup will be done before the test case runs using the `@Before` annotation.

For creating the needed consumer properties a static `consumerProps()` method provided by `KafkaUtils` is used. We then create a `DefaultKafkaConsumerFactory` and `ContainerProperties` which contains runtime properties (in this case the topic name) for the listener container. Both are then passed to the `KafkaMessageListenerContainer` constructor.

Received messages need to be stored somewhere. In this example, a thread safe `BlockingQueue` is used. We create a new `MessageListener` and in the `onMessage()` method we add the received message to the `BlockingQueue`.

> The listener is started by starting the container.

In order to avoid that we send a message before the container has required the number of assigned partitions, we use the `waitForAssignment()` method on the `ContainerTestUtils` helper class.

The actual unit test itself consists out of sending a greeting and asserting that the received value is the same as the one that was sent.

The Spring Kafka Test JAR ships with a number of [Hamcrest Matchers](http://docs.spring.io/spring-kafka/docs/1.2.2.RELEASE/reference/html/_reference.html#_hamcrest_matchers){:target="_blank"} that allow checking if the key, value or partition of a received message matches with an expected value. In the below unit test we use a `Matcher` to check the value of the received message.

The JAR also includes some [AssertJ conditions](http://docs.spring.io/spring-kafka/docs/1.2.2.RELEASE/reference/html/_reference.html#_assertj_conditions){:target="_blank"} that allow asserting if a received message contains a specific key, value or partition. We illustrate the usage of such a condition by asserting that the key of the received message is `null`.

> For both the Hamcrest matchers and AssertJ conditions, make sure the static imports have been specified.

> Note the `@DirtiesContext` annotation that ensures the correct Kafka broker address is set as explained above.

``` java
package com.codenotfound.kafka.producer;

import static org.assertj.core.api.Assertions.assertThat;
import static org.junit.Assert.assertThat;
import static org.springframework.kafka.test.assertj.KafkaConditions.key;
import static org.springframework.kafka.test.hamcrest.KafkaMatchers.hasValue;

import java.util.Map;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.TimeUnit;

import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.junit.After;
import org.junit.Before;
import org.junit.BeforeClass;
import org.junit.ClassRule;
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
import org.springframework.kafka.test.rule.KafkaEmbedded;
import org.springframework.kafka.test.utils.ContainerTestUtils;
import org.springframework.kafka.test.utils.KafkaTestUtils;
import org.springframework.test.annotation.DirtiesContext;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
@DirtiesContext
public class SpringKafkaSenderTest {

  private static final Logger LOGGER = LoggerFactory.getLogger(SpringKafkaSenderTest.class);

  private static String SENDER_TOPIC = "sender.t";

  @Autowired
  private Sender sender;

  private KafkaMessageListenerContainer<String, String> container;

  private BlockingQueue<ConsumerRecord<String, String>> records;

  @ClassRule
  public static KafkaEmbedded embeddedKafka = new KafkaEmbedded(1, true, SENDER_TOPIC);

  @BeforeClass
  public static void setUpBeforeClass() {
    System.setProperty("kafka.bootstrap-servers", embeddedKafka.getBrokersAsString());
  }

  @Before
  public void setUp() throws Exception {
    // set up the Kafka consumer properties
    Map<String, Object> consumerProperties =
        KafkaTestUtils.consumerProps("sender", "false", embeddedKafka);

    // create a Kafka consumer factory
    DefaultKafkaConsumerFactory<String, String> consumerFactory =
        new DefaultKafkaConsumerFactory<String, String>(consumerProperties);

    // set the topic that needs to be consumed
    ContainerProperties containerProperties = new ContainerProperties(SENDER_TOPIC);

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
    ContainerTestUtils.waitForAssignment(container, embeddedKafka.getPartitionsPerTopic());
  }

  @After
  public void tearDown() {
    // stop the container
    container.stop();
  }

  @Test
  public void testSend() throws InterruptedException {
    // send the message
    String greeting = "Hello Spring Kafka Sender!";
    sender.send(SENDER_TOPIC, greeting);

    // check that the message was received
    ConsumerRecord<String, String> received = records.poll(10, TimeUnit.SECONDS);
    // Hamcrest Matchers to check the value
    assertThat(received, hasValue(greeting));
    // AssertJ Condition to check the key
    assertThat(received).has(key(null));
  }
}
```

# Testing the Consumer

The second `SpringKafkaReceiverTest` test class focuses on the `Receiver` which listens to a <var>'receiver.t'</var> topic as defined in the <var>applications.yml</var> properties file. In order to check the correct working, we use a _test-template_ to send a message to this topic. All of the setup will be done before the test case runs using the `@Before` annotation.

The producer properties are created using the static `senderProps()` method provided by `KafkaUtils`. These properties are then used to create a `DefaultKafkaProducerFactory` which is in turn used to create a `KafkaTemplate`. Finally we set the default topic that the template uses to <var>'receiver.t'</var>.

We need to ensure that the `Receiver` is initialized before sending the test message. For this we use the `waitForAssignment()` method of `ContainerTestUtils`. The link to the message listener container is acquired by auto-wiring the `KafkaListenerEndpointRegistry` which manages the lifecycle of the listener containers that are not created manually.

> Note that if you do not create the topics using the `KafkaEmbedded` constructor you need to manually set the partitions per topic to 1 in the `waitForAssignment()` method instead of getting the partitions from the embedded Kafka server. The reason for this is that [it looks like 1 is used as a default for the number of partitions](http://stackoverflow.com/a/38660145/4201470){:target="_blank"} in case topics are created implicitly. 

In the test, we send a greeting and check that the message was received by asserting that the latch of the `Receiver` was lowered to zero.

``` java
package com.codenotfound.kafka.consumer;

import static org.assertj.core.api.Assertions.assertThat;

import java.util.Map;
import java.util.concurrent.TimeUnit;

import org.junit.Before;
import org.junit.BeforeClass;
import org.junit.ClassRule;
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
import org.springframework.kafka.test.rule.KafkaEmbedded;
import org.springframework.kafka.test.utils.ContainerTestUtils;
import org.springframework.kafka.test.utils.KafkaTestUtils;
import org.springframework.test.annotation.DirtiesContext;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
@DirtiesContext
public class SpringKafkaReceiverTest {

  private static final Logger LOGGER = LoggerFactory.getLogger(SpringKafkaReceiverTest.class);

  private static String RECEIVER_TOPIC = "receiver.t";

  @Autowired
  private Receiver receiver;

  private KafkaTemplate<String, String> template;

  @Autowired
  private KafkaListenerEndpointRegistry kafkaListenerEndpointRegistry;

  @ClassRule
  public static KafkaEmbedded embeddedKafka = new KafkaEmbedded(1, true, RECEIVER_TOPIC);

  @BeforeClass
  public static void setUpBeforeClass() {
    System.setProperty("kafka.bootstrap-servers", embeddedKafka.getBrokersAsString());
  }

  @Before
  public void setUp() throws Exception {
    // set up the Kafka producer properties
    Map<String, Object> senderProperties =
        KafkaTestUtils.senderProps(embeddedKafka.getBrokersAsString());

    // create a Kafka producer factory
    ProducerFactory<String, String> producerFactory =
        new DefaultKafkaProducerFactory<String, String>(senderProperties);

    // create a Kafka template
    template = new KafkaTemplate<>(producerFactory);
    // set the default topic to send to
    template.setDefaultTopic(RECEIVER_TOPIC);

    // wait until the partitions are assigned
    for (MessageListenerContainer messageListenerContainer : kafkaListenerEndpointRegistry
        .getListenerContainers()) {
      ContainerTestUtils.waitForAssignment(messageListenerContainer,
          embeddedKafka.getPartitionsPerTopic());
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

20:48:03.077 [main] INFO  c.c.k.c.SpringKafkaReceiverTest - Starting SpringKafkaReceiverTest on cnf-pc with PID 6036 (started by CodeNotFound in c:\codenotfound\code\spring-kafka\spring-kafka-test-embedded)
20:48:03.077 [main] DEBUG c.c.k.c.SpringKafkaReceiverTest - Running with Spring Boot v1.5.4.RELEASE, Spring v4.3.9.RELEASE
20:48:03.079 [main] INFO  c.c.k.c.SpringKafkaReceiverTest - No active profile set, falling back to default profiles: default
20:48:03.758 [main] INFO  c.c.k.c.SpringKafkaReceiverTest - Started SpringKafkaReceiverTest in 0.971 seconds (JVM running for 5.136)
20:48:05.205 [main] DEBUG c.c.k.c.SpringKafkaReceiverTest - test-sender sent message='Hello Spring Kafka Receiver!'
20:48:05.231 [org.springframework.kafka.KafkaListenerEndpointContainer#0-0-C-1] INFO  c.c.kafka.consumer.Receiver - received payload='Hello Spring Kafka Receiver!'
20:48:06.213 [main] ERROR o.a.zookeeper.server.ZooKeeperServer - ZKShutdownHandler is not registered, so ZooKeeper server won't take any action on ERROR or SHUTDOWN server state changes
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 7.979 sec - in com.codenotfound.kafka.consumer.SpringKafkaReceiverTest

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.5.4.RELEASE)

20:48:07.526 [main] INFO  c.c.k.producer.SpringKafkaSenderTest - Starting SpringKafkaSenderTest on cnf-pc with PID 6036 (started by CodeNotFound in c:\codenotfound\code\spring-kafka\spring-kafka-test-embedded)
20:48:07.526 [main] DEBUG c.c.k.producer.SpringKafkaSenderTest - Running with Spring Boot v1.5.4.RELEASE, Spring v4.3.9.RELEASE
20:48:07.526 [main] INFO  c.c.k.producer.SpringKafkaSenderTest - No active profile set, falling back to default profiles: default
20:48:07.721 [main] INFO  c.c.k.producer.SpringKafkaSenderTest - Started SpringKafkaSenderTest in 0.252 seconds (JVM running for 9.1)
20:48:07.743 [org.springframework.kafka.KafkaListenerEndpointContainer#0-0-C-1] WARN  o.apache.kafka.clients.NetworkClient - Error while fetching metadata with correlation id 2 : {receiver.t=LEADER_NOT_AVAILABLE}
20:48:08.933 [main] INFO  c.codenotfound.kafka.producer.Sender - sending payload='Hello Spring Kafka Sender!' to topic='sender.t'
20:48:08.948 [-L-1] DEBUG c.c.k.producer.SpringKafkaSenderTest - test-listener received message='ConsumerRecord(topic = sender.t, partition = 1, offset = 0, CreateTime = 1501440488941, checksum = 2655582397, serialized key size = -1, serialized value size = 26, key = null, value = Hello Spring Kafka Sender!)'
20:48:12.425 [main] ERROR o.a.zookeeper.server.ZooKeeperServer - ZKShutdownHandler is not registered, so ZooKeeper server won't take any action on ERROR or SHUTDOWN server state changes
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 6.241 sec - in com.codenotfound.kafka.producer.SpringKafkaSenderTest

Results :

Tests run: 2, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 17.895 s
[INFO] Finished at: 2017-07-30T20:48:14+02:00
[INFO] Final Memory: 27M/218M
[INFO] ------------------------------------------------------------------------
```

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-kafka/tree/master/spring-kafka-test-embedded).
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This concludes our example in which we unit test sending and receiving from Spring Kafka by starting an embedded Kafka server.

Feel free to drop a line in case of any questions or if you found this post helpful.
