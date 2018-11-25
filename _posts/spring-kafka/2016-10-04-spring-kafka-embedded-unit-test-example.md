---
title: "Spring Kafka Embedded Unit Test Example"
permalink: /spring-kafka-embedded-unit-test-example.html
excerpt: "A detailed step-by-step tutorial on how to unit test your Spring Kafka application using an embedded broker."
date: 2016-10-04
modified: 2018-11-24
header:
  teaser: "assets/images/spring-kafka/spring-kafka-test-example.png"
categories: [Spring Kafka]
tags: [Apache Kafka, Embedded, Example, Maven, Server, Spring Boot, Spring Kafka, Test, Testing, Tutorial, Unit]
redirect_from:
  - /2016/09/spring-kafka-embedded-kafka-test-example.html
  - /2016/10/spring-kafka-embedded-server-unit-test.html
  - /spring-kafka-test-example.html
published: true
---

<img src="{{ site.url }}/assets/images/spring-kafka/spring-kafka-test-example.png" alt="spring kafka test example" class="align-right title-image">

This guide will teach you everything you need to know about Spring Kafka Test.

How to test a consumer.

And how to test a producer.

Using an embedded Kafka broker.

Let's get started…

If you want to learn more about Spring Kafka - head on over to the [Spring Kafka tutorials page]({{ site.url }}/spring-kafka-tutorials).
{: .notice--primary}

## 1. What is Spring Kafka Test?

The [Spring Kafka project](https://spring.io/projects/spring-kafka){:target="_blank"} comes with a `spring-kafka-test` JAR that contains a number of [useful utilities](http://docs.spring.io/spring-kafka/docs/2.2.0.RELEASE/reference/html/_reference.html#testing){:target="_blank"} to assist you with your application unit testing.

These include an embedded Kafka broker, some static methods to setup consumers/producers and utility methods to fetch results. Let's demonstrate how these test utilities can be used with a code sample.

We will start from a previous [Spring Kafka example]({{ site.url }}/spring-kafka-consumer-producer-example.html) in which we created a consumer and producer using Spring Kafka, Spring Boot, and Maven.

A dedicated unit test case for the producer shows how to check that messages are being sent. A second unit test case verifies that messages are received.

## 2. General Project Overview

Tools used:
* Spring Kafka 2.2
* Spring Boot 2.1
* Maven 3.5

The project has the following directory structure:

<img src="{{ site.url }}/assets/images/spring-kafka/spring-kafka-test-example-maven-project.png" alt="spring kafka test example maven project">

## 3. Maven Setup

Make sure the `spring-kafka-test` dependency is included in the Maven POM file together with the Spring Kafka and Spring Boot dependencies.

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>spring-kafka-unit-test</artifactId>
  <version>0.0.1-SNAPSHOT</version>

  <name>spring-kafka-unit-test</name>
  <description>Spring Kafka Embedded Unit Test Example</description>
  <url>https://codenotfound.com/spring-kafka-embedded-unit-test-example.html</url>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.0.RELEASE</version>
    <relativePath /> <!-- lookup parent from repository -->
  </parent>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>1.8</java.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.kafka</groupId>
      <artifactId>spring-kafka</artifactId>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework.kafka</groupId>
      <artifactId>spring-kafka-test</artifactId>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>

</project>
{% endhighlight %}

The message consumer and producer classes from the Hello World example are unchanged so we won't go into detail explaining them. You can check out the [Spring Kafka Maven project](https://github.com/code-not-found/spring-kafka/tree/master/spring-kafka-hello-world) for more details.

## 4. Unit Testing with an Embedded Kafka

`spring-kafka-test` includes an embedded Kafka broker that can be created via a JUnit `@ClassRule` annotation. The rule will start a [ZooKeeper](https://zookeeper.apache.org/){:target="_blank"} and [Kafka](https://kafka.apache.org/){:target="_blank"} server instance on a random port before all the test cases are run, and stops the instances once the test cases are finished.

The `EmbeddedKafkaRule` constructor takes as parameters: the number of Kafka servers to start, whether a controlled shutdown is needed and the topics that need to be created on the server.

The below snippet shows how the embedded Kafa is defined in each test class.

{% highlight java %}
  @ClassRule
  public static EmbeddedKafkaRule embeddedKafka = new EmbeddedKafkaRule(1, true, "topic");
{% endhighlight %}

> Always pass the topics as a parameter to the embedded Kafka server. This assures that the topics are not auto-created and present when the `MessageListener` connects.

As the embedded broker is started on a random port, we can't use the fix value in the <var>src/main/resources/application.yml</var> properties file. Luckily the `@ClassRule` sets a `spring.embedded.kafka.brokers` system property to the address of the embedded broker(s). We will assign the value of this property to the `kafka.bootstrap-servers` property that is used by the `SenderConfig` and `ReceiverConfig` classes. Since this is only needed during unit testing, we create a dedicated _test_ <var>application.yml</var> properties file under <var>src/test/resources</var>.

In other words, the <var>application.yml</var> located in the <var>src/test/resources</var> directory contains following:

{% highlight yaml %}
kafka:
  bootstrap-servers: ${spring.embedded.kafka.brokers}
  topic:
    receiver: receiver.t
{% endhighlight %}

> To have the correct broker address set on the `Sender` and `Receiver` beans during each test case, we need to use the `@DirtiesContext` on all test classes. The reason for this is that each test case contains its own embedded Kafka broker that will each be created on a new random port. By [rebuilding the application context](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/integration-testing.html#__dirtiescontext){:target="_blank"}, the beans will always be set with the current broker address.

## 5. Testing the Producer

In the `SpringKafkaSenderTest` test case, we will be testing the `Sender` by sending a message to a <var>'sender.t'</var> topic. We will verify whether the sending works by setting up a _test-listener_ on the topic. All the setup will be done before the test case runs using the `@Before` annotation.

For creating the needed consumer properties a static `consumerProps()` method provided by `KafkaUtils` is used. We then create a `DefaultKafkaConsumerFactory` and `ContainerProperties` which contains runtime properties (in this case the topic name) for the listener container. Both are then passed to the `KafkaMessageListenerContainer` constructor.

Received messages need to be stored somewhere. In this example, a thread-safe `BlockingQueue` is used. We create a new `MessageListener` and in the `onMessage()` method we add the received message to the `BlockingQueue`.

> The listener is started by starting the container.

In order to avoid that we send a message before the container has required the number of assigned partitions, we use the `waitForAssignment()` method on the `ContainerTestUtils` helper class.

The actual unit test itself consists out of sending a greeting and asserting that the received value is the same as the one that was sent.

The Spring Kafka Test JAR ships with some [Hamcrest Matchers](http://docs.spring.io/spring-kafka/docs/2.2.0.RELEASE/reference/html/_reference.html#_hamcrest_matchers){:target="_blank"} that allow checking if the key, value or partition of a received message matches with an expected value. In the below unit test we use a `Matcher` to check the value of the received message.

The JAR also includes some [AssertJ conditions](http://docs.spring.io/spring-kafka/docs/2.2.0.RELEASE/reference/html/_reference.html#_assertj_conditions){:target="_blank"} that allow asserting if a received message contains a specific key, value or partition. We illustrate the usage of such a condition by asserting that the key of the received message is `null`.

> For both the Hamcrest matchers and AssertJ conditions, make sure the static imports have been specified.

> Note the `@DirtiesContext` annotation that ensures the correct Kafka broker address is set as explained above.

{% highlight java %}
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
import org.junit.ClassRule;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.kafka.core.DefaultKafkaConsumerFactory;
import org.springframework.kafka.listener.ContainerProperties;
import org.springframework.kafka.listener.KafkaMessageListenerContainer;
import org.springframework.kafka.listener.MessageListener;
import org.springframework.kafka.test.rule.EmbeddedKafkaRule;
import org.springframework.kafka.test.utils.ContainerTestUtils;
import org.springframework.kafka.test.utils.KafkaTestUtils;
import org.springframework.test.annotation.DirtiesContext;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
@DirtiesContext
public class SpringKafkaSenderTest {

  private static final Logger LOGGER =
      LoggerFactory.getLogger(SpringKafkaSenderTest.class);

  private static String SENDER_TOPIC = "sender.t";

  @Autowired
  private Sender sender;

  private KafkaMessageListenerContainer<String, String> container;

  private BlockingQueue<ConsumerRecord<String, String>> records;

  @ClassRule
  public static EmbeddedKafkaRule embeddedKafka =
      new EmbeddedKafkaRule(1, true, SENDER_TOPIC);

  @Before
  public void setUp() throws Exception {
    // set up the Kafka consumer properties
    Map<String, Object> consumerProperties =
        KafkaTestUtils.consumerProps("sender", "false",
            embeddedKafka.getEmbeddedKafka());

    // create a Kafka consumer factory
    DefaultKafkaConsumerFactory<String, String> consumerFactory =
        new DefaultKafkaConsumerFactory<String, String>(
            consumerProperties);

    // set the topic that needs to be consumed
    ContainerProperties containerProperties =
        new ContainerProperties(SENDER_TOPIC);

    // create a Kafka MessageListenerContainer
    container = new KafkaMessageListenerContainer<>(consumerFactory,
        containerProperties);

    // create a thread safe queue to store the received message
    records = new LinkedBlockingQueue<>();

    // setup a Kafka message listener
    container
        .setupMessageListener(new MessageListener<String, String>() {
          @Override
          public void onMessage(
              ConsumerRecord<String, String> record) {
            LOGGER.debug("test-listener received message='{}'",
                record.toString());
            records.add(record);
          }
        });

    // start the container and underlying message listener
    container.start();

    // wait until the container has the required number of assigned partitions
    ContainerTestUtils.waitForAssignment(container,
        embeddedKafka.getEmbeddedKafka().getPartitionsPerTopic());
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
    sender.send(greeting);

    // check that the message was received
    ConsumerRecord<String, String> received =
        records.poll(10, TimeUnit.SECONDS);
    // Hamcrest Matchers to check the value
    assertThat(received, hasValue(greeting));
    // AssertJ Condition to check the key
    assertThat(received).has(key(null));
  }
}
{% endhighlight %}

## 6. Testing the Consumer

The second `SpringKafkaReceiverTest` test class focuses on the `Receiver` which listens to a <var>'receiver.t'</var> topic as defined in the <var>applications.yml</var> properties file. In order to check the correct working, we use a _test-template_ to send a message to this topic. All of the setup will be done before the test case runs using the `@Before` annotation.

The producer properties are created using the static `senderProps()` method provided by `KafkaUtils`. These properties are then used to create a `DefaultKafkaProducerFactory` which is in turn used to create a `KafkaTemplate`. Finally, we set the default topic that the template uses to <var>'receiver.t'</var>.

We need to ensure that the `Receiver` is initialized before sending the test message. For this we use the `waitForAssignment()` method of `ContainerTestUtils`. The link to the message listener container is acquired by auto-wiring the `KafkaListenerEndpointRegistry` which manages the lifecycle of the listener containers that are not created manually.

> Note that if you do not create the topics using the `EmbeddedKafkaRule` constructor you need to manually set the partitions per topic to 1 in the `waitForAssignment()` method instead of getting the partitions from the embedded Kafka server. The reason for this is that [it looks like 1 is used as a default for the number of partitions](http://stackoverflow.com/a/38660145/4201470){:target="_blank"} in case topics are created implicitly.

In the test, we send a greeting and check that the message was received by asserting that the latch of the `Receiver` was lowered to zero.

{% highlight java %}
package com.codenotfound.kafka.consumer;

import static org.assertj.core.api.Assertions.assertThat;
import java.util.Map;
import java.util.concurrent.TimeUnit;
import org.junit.Before;
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
import org.springframework.kafka.test.rule.EmbeddedKafkaRule;
import org.springframework.kafka.test.utils.ContainerTestUtils;
import org.springframework.kafka.test.utils.KafkaTestUtils;
import org.springframework.test.annotation.DirtiesContext;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
@DirtiesContext
public class SpringKafkaReceiverTest {

  private static final Logger LOGGER =
      LoggerFactory.getLogger(SpringKafkaReceiverTest.class);

  private static String RECEIVER_TOPIC = "receiver.t";

  @Autowired
  private Receiver receiver;

  private KafkaTemplate<String, String> template;

  @Autowired
  private KafkaListenerEndpointRegistry kafkaListenerEndpointRegistry;

  @ClassRule
  public static EmbeddedKafkaRule embeddedKafka =
      new EmbeddedKafkaRule(1, true, RECEIVER_TOPIC);

  @Before
  public void setUp() throws Exception {
    // set up the Kafka producer properties
    Map<String, Object> senderProperties =
        KafkaTestUtils.senderProps(
            embeddedKafka.getEmbeddedKafka().getBrokersAsString());

    // create a Kafka producer factory
    ProducerFactory<String, String> producerFactory =
        new DefaultKafkaProducerFactory<String, String>(
            senderProperties);

    // create a Kafka template
    template = new KafkaTemplate<>(producerFactory);
    // set the default topic to send to
    template.setDefaultTopic(RECEIVER_TOPIC);

    // wait until the partitions are assigned
    for (MessageListenerContainer messageListenerContainer : kafkaListenerEndpointRegistry
        .getListenerContainers()) {
      ContainerTestUtils.waitForAssignment(messageListenerContainer,
          embeddedKafka.getEmbeddedKafka().getPartitionsPerTopic());
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
{% endhighlight %}

## 7. Running the Unit Test Cases

Let's run above test cases by opening a command prompt and executing following Maven command:

{% highlight plaintext %}
mvn test
{% endhighlight %}

Maven will download the needed dependencies, compile the code and run the unit test case. The result should be a successful build during which following logs are generated:

{% highlight plaintext %}
.   ____          _            __ _ _
/\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
\\/  ___)| |_)| | | | | || (_| |  ) ) ) )
'  |____| .__|_| |_|_| |_\__, | / / / /
=========|_|==============|___/=/_/_/_/
:: Spring Boot ::        (v2.1.0.RELEASE)

06:22:50.805 [main] INFO  c.c.k.c.SpringKafkaReceiverTest - Starting SpringKafkaReceiverTest on DESKTOP-2RB3C1U with PID 640 (started by Codenotfound in C:\Users\Codenotfound\repos\spring-kafka\spring-kafka-unit-test-classrule)
06:22:50.805 [main] DEBUG c.c.k.c.SpringKafkaReceiverTest - Running with Spring Boot v2.1.0.RELEASE, Spring v5.1.2.RELEASE
06:22:50.805 [main] INFO  c.c.k.c.SpringKafkaReceiverTest - No active profile set, falling back to default profiles: default
06:22:51.758 [main] INFO  c.c.k.c.SpringKafkaReceiverTest - Started SpringKafkaReceiverTest in 1.234 seconds (JVM running for 4.045)
06:22:52.133 [main] DEBUG c.c.k.c.SpringKafkaReceiverTest - test-sender sent message='Hello Spring Kafka Receiver!'
06:22:52.196 [org.springframework.kafka.KafkaListenerEndpointContainer#0-0-C-1] INFO  c.c.kafka.consumer.Receiver - received payload='Hello Spring Kafka Receiver!'
06:22:53.274 [NIOServerCxn.Factory:/127.0.0.1:0] WARN  o.a.zookeeper.server.NIOServerCnxn - Unable to read additional data from client sessionid 0x100357cf1f00001, likely client has closed socket
06:22:53.336 [kafka-producer-network-thread | producer-1] WARN  o.apache.kafka.clients.NetworkClient - [Producer clientId=producer-1] Connection to node 0 could not be established. Broker may not be available.
06:22:54.492 [kafka-producer-network-thread | producer-1] WARN  o.apache.kafka.clients.NetworkClient - [Producer clientId=producer-1] Connection to node 0 could not be established. Broker may not be available.
06:22:55.774 [kafka-producer-network-thread | producer-1] WARN  o.apache.kafka.clients.NetworkClient - [Producer clientId=producer-1] Connection to node 0 could not be established. Broker may not be available.
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 7.812 s - in com.codenotfound.kafka.consumer.SpringKafkaReceiverTest
[INFO] Running com.codenotfound.kafka.producer.SpringKafkaSenderTest
06:22:55.961 [main] WARN  k.server.BrokerMetadataCheckpoint - No meta.properties file under dir C:\Users\CODENO~1\AppData\Local\Temp\kafka-4730765835955741611\meta.properties
06:22:56.024 [main] WARN  k.server.BrokerMetadataCheckpoint - No meta.properties file under dir C:\Users\CODENO~1\AppData\Local\Temp\kafka-4730765835955741611\meta.properties

.   ____          _            __ _ _
/\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
\\/  ___)| |_)| | | | | || (_| |  ) ) ) )
'  |____| .__|_| |_|_| |_\__, | / / / /
=========|_|==============|___/=/_/_/_/
:: Spring Boot ::        (v2.1.0.RELEASE)

06:22:56.227 [main] INFO  c.c.k.producer.SpringKafkaSenderTest - Starting SpringKafkaSenderTest on DESKTOP-2RB3C1U with PID 640 (started by Codenotfound in C:\Users\Codenotfound\repos\spring-kafka\spring-kafka-unit-test-classrule)
06:22:56.227 [main] DEBUG c.c.k.producer.SpringKafkaSenderTest - Running with Spring Boot v2.1.0.RELEASE, Spring v5.1.2.RELEASE
06:22:56.227 [main] INFO  c.c.k.producer.SpringKafkaSenderTest - No active profile set, falling back to default profiles: default
06:22:56.602 [main] INFO  c.c.k.producer.SpringKafkaSenderTest - Started SpringKafkaSenderTest in 0.407 seconds (JVM running for 8.896)
06:22:56.852 [main] INFO  c.codenotfound.kafka.producer.Sender - sending payload='Hello Spring Kafka Sender!'
06:22:56.867 [-C-1] DEBUG c.c.k.producer.SpringKafkaSenderTest - test-listener received message='ConsumerRecord(topic = sender.t, partition = 1, offset = 0, CreateTime = 1543123376852, serialized key size = -1, serialized value size = 26, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = Hello Spring Kafka Sender!)'
06:22:57.302 [kafka-producer-network-thread | producer-1] WARN  o.apache.kafka.clients.NetworkClient - [Producer clientId=producer-1] Connection to node 0 could not be established. Broker may not be available.
06:22:59.193 [kafka-producer-network-thread | producer-1] WARN  o.apache.kafka.clients.NetworkClient - [Producer clientId=producer-1] Connection to node 0 could not be established. Broker may not be available.
06:23:00.052 [NIOServerCxn.Factory:/127.0.0.1:0] WARN  o.a.zookeeper.server.NIOServerCnxn - Unable to read additional data from client sessionid 0x100357d0d1b0000, likely client has closed socket
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 5.173 s - in com.codenotfound.kafka.producer.SpringKafkaSenderTest
[INFO]
[INFO] Results:
[INFO]
[INFO] Tests run: 2, Failures: 0, Errors: 0, Skipped: 0
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 16.145 s
[INFO] Finished at: 2018-11-25T06:23:01+01:00
[INFO] ------------------------------------------------------------------------
{% endhighlight %}

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-kafka/tree/master/spring-kafka-unit-test-classrule).
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This concludes our example in which we unit test sending and receiving from Spring Kafka by starting an embedded Kafka broker.

Feel free to drop a line in case of any questions or if you found this post helpful.
