---
title: "Spring Kafka - Embedded Server Unit Test"
permalink: /2016/10/spring-kafka-embedded-server-unit-test.html
excerpt: "A detailed step-by-step tutorial on how to test your application using an embedded Apache Kafka server together with Spring Kafka and Spring Boot."
date: 2016-10-04
modified: 2017-04-17
categories: [Spring Kafka]
tags: [Apache Kafka, Embedded, Example, Maven, Server, Spring Boot, Spring Kafka, Test, Testing, Tutorial, Unit]
published: true
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/spring-logo.jpg" alt="spring logo" class="logo">
</figure>

The [Spring Kafka project](https://projects.spring.io/spring-kafka/) comes with a `spring-kafka-test` JAR that contains a number of [useful utilities](http://docs.spring.io/spring-kafka/docs/1.2.0.RELEASE/reference/html/_reference.html#testing). to assist you with your application testing. These include:
* an embedded Kafka server
* some static methods to setup consumers/producers
* utility methods to fetch results

Let's demonstrate how above can be used with a simple code sample. We will reuse the Spring Kafka Hello World project from a previous post in which we created a consumer and producer using Spring Kafka, Spring Boot and Maven.


Tools used:
* Spring Kafka 1.2
* Spring Boot 1.5
* Maven 3.5

# General Project Setup

Add the `spring-kafka-test` dependency to the Maven POM file in addition to the Spring Kafka and Spring Boot dependencies.

In the plugins section we included the `maven-surefire-plugin` to trigger an `AllSpringKafkaTests` test suite class that will be used to start the embedded server for the different unit test cases in our project. This allows us to start a single embedded Kafka server and reuse it for the different unit test cases.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
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
    <version>1.5.2.RELEASE</version>
  </parent>

  <properties>
    <java.version>1.8</java.version>

    <spring-kafka.version>1.2.0.RELEASE</spring-kafka.version>
    <maven-surefire-plugin.version>2.19.1</maven-surefire-plugin.version>
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

The message consumer and producer classes from the Hello World example are unchanged so we won't go into detail explaining them. You can checkout the [Spring Boot Kafka example]({{ site.url }}/2016/09/spring-kafka-consumer-producer-example.html) from a previous post for more details.

# Unit Testing with an Embedded Kafka Server

`spring-kafka-test` includes an embedded Kafka server that can be created via a JUnit `@ClassRule` annotation. The rule will start a [ZooKeeper](https://zookeeper.apache.org/) and [Kafka](https://kafka.apache.org/) server instance on a random port before all the test cases are run, and stops the instances one the test cases are finished.

In order to support multiple unit test classes (in this example: `SpringKafkaApplicationTest`, `SpringKafkaSenderTest` and `SpringKafkaReceiverTest`), we will trigger the `@ClassRule` from a `Suite` class that bundles these test cases together.

As the embedded server is started on a random port, we need to change the property value that is used by the `SenderConfig` and `ReceiverConfig` classes. This is done by calling the `getBrokersAsString()` method and setting the value to the <var>'kafka.servers.bootstrap'</var> property.

> Note that we pass the <var>'sender.t'</var> topic as a parameter to the embedded kafka server. This assures that the topic is present when the `MessageListener` in the `SpringKafkaSenderTest` connects.

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

In the first test class we will be testing the `Sender` by sending a message to a <var>'sender.t'</var> topic. We will verify the correct sending by setting up a message listener on the topic. For creating the consumer properties we use a static method provided by `KafkaUtils`. After setting up the `KafkaMessageListenerContainer` we setup a `MessageListener` and start the container. 

In order to avoid that we send the message before the container has required the number of assigned partitions, we use the `waitForAssignment()` method on the `ContainerTestUtils` helper class. We then send a greeting and assert that the received value is the same as the one that was sent using an AssertJ condition that is provided by `KafkaConditions` which is also included in the `spring-kafka-test` dependency. 

``` java
package com.codenotfound.kafka;

import static org.assertj.core.api.Assertions.assertThat;
import static org.springframework.kafka.test.assertj.KafkaConditions.value;

import java.util.Map;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.TimeUnit;

import org.apache.kafka.clients.consumer.ConsumerRecord;
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

import com.codenotfound.kafka.producer.Sender;

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringKafkaSenderTest {

  private static final Logger LOGGER = LoggerFactory.getLogger(SpringKafkaSenderTest.class);

  @Autowired
  private Sender sender;

  @Test
  public void testSend() throws Exception {
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
    KafkaMessageListenerContainer<String, String> container =
        new KafkaMessageListenerContainer<>(consumerFactory, containerProperties);

    // create a thread safe queue to store the received message
    BlockingQueue<ConsumerRecord<String, String>> records = new LinkedBlockingQueue<>();
    // setup a Kafka message listener
    container.setupMessageListener(new MessageListener<String, String>() {
      @Override
      public void onMessage(ConsumerRecord<String, String> record) {
        LOGGER.debug(record.toString());
        records.add(record);
      }
    });

    // start the container and underlying message listener
    container.start();
    // wait until the container has the required number of assigned
    // partitions
    ContainerTestUtils.waitForAssignment(container,
        AllSpringKafkaTests.embeddedKafka.getPartitionsPerTopic());

    // send the message
    String greeting = "Hello Spring Kafka Sender!";
    sender.send(AllSpringKafkaTests.SENDER_TOPIC, greeting);
    // check that the message was received
    assertThat(records.poll(10, TimeUnit.SECONDS)).has(value(greeting));

    // stop the container
    container.stop();
  }
}
```

The second test class focuses on the `Receiver` which listens to a <var>'receiver.t'</var> topic as defined in the <var>applications.properties</var> file. In order to check the correct working we will use a producer to send a message to this topic. The producer properties are created using the static method provided by `KafkaUtils` and used to create a `KafkaTemplate`. 

We need to ensure that the `Receiver` is initialized before sending the test message. For this we use the `waitForAssignment()` of `ContainerTestUtils`. The link to the message listener container is acquired by autowiring the `KafkaListenerEndpointRegistry` which manages the lifecycle of the listener containers that are not created manually. We check that the message was received by asserting that the latch of the `Receiver` was lowered to zero. 

> Note that we manually set the partitions per topic to 1 in the `waitForAssignment()` method instead getting the partitions from the embedded Kafka server. The reason for this is that [it looks like 1 is used as a default for the number of partitions in case topics are created implicitly](http://stackoverflow.com/a/38660145/4201470). 

``` java
package com.codenotfound.kafka;

import static org.assertj.core.api.Assertions.assertThat;

import java.util.Map;
import java.util.concurrent.TimeUnit;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.kafka.config.KafkaListenerEndpointRegistry;
import org.springframework.kafka.core.DefaultKafkaProducerFactory;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.core.ProducerFactory;
import org.springframework.kafka.listener.ConcurrentMessageListenerContainer;
import org.springframework.kafka.listener.MessageListenerContainer;
import org.springframework.kafka.test.utils.ContainerTestUtils;
import org.springframework.kafka.test.utils.KafkaTestUtils;
import org.springframework.test.context.junit4.SpringRunner;

import com.codenotfound.kafka.consumer.Receiver;

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringKafkaReceiverTest {

  @Autowired
  private Receiver receiver;

  @Autowired
  KafkaListenerEndpointRegistry kafkaListenerEndpointRegistry;

  @SuppressWarnings("unchecked")
  @Test
  public void testReceive() throws Exception {
    // set up the Kafka producer properties
    Map<String, Object> senderProperties =
        KafkaTestUtils.senderProps(AllSpringKafkaTests.embeddedKafka.getBrokersAsString());

    // create a Kafka producer factory
    ProducerFactory<String, String> producerFactory =
        new DefaultKafkaProducerFactory<String, String>(senderProperties);

    // create a Kafka template
    KafkaTemplate<String, String> template = new KafkaTemplate<>(producerFactory);
    // set the default topic to send to
    template.setDefaultTopic(AllSpringKafkaTests.RECEIVER_TOPIC);

    // get the ConcurrentMessageListenerContainers
    for (MessageListenerContainer messageListenerContainer : kafkaListenerEndpointRegistry
        .getListenerContainers()) {
      if (messageListenerContainer instanceof ConcurrentMessageListenerContainer) {
        ConcurrentMessageListenerContainer<String, String> concurrentMessageListenerContainer =
            (ConcurrentMessageListenerContainer<String, String>) messageListenerContainer;

        // as the topic is created implicitly, the default number of
        // partitions is 1
        int partitionsPerTopic = 1;
        // wait until the container has the required number of assigned
        // partitions
        ContainerTestUtils.waitForAssignment(concurrentMessageListenerContainer,
            partitionsPerTopic);
      }
    }

    // send the message
    template.sendDefault("Hello Spring Kafka Receiver!");

    receiver.getLatch().await(10000, TimeUnit.MILLISECONDS);
    assertThat(receiver.getLatch().getCount()).isEqualTo(0);
  }
}
```

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
 :: Spring Boot ::        (v1.5.2.RELEASE)

2017-03-12 07:15:23.991  INFO 4176 --- [           main] c.c.kafka.SpringKafkaSenderTest          : Starting SpringKafkaSenderTest on cnf-pc with PID 4176 (started by CodeNotFound in c:\code\st\spring-kafka\spring-kafka-embedded-test)
2017-03-12 07:15:23.992  INFO 4176 --- [           main] c.c.kafka.SpringKafkaSenderTest          : No active profile set, falling back to default profiles: default
2017-03-12 07:15:24.011  INFO 4176 --- [           main] s.c.a.AnnotationConfigApplicationContext : Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@4985cbcb: startup date [Sun Mar 12 07:15:24 CET 2017]; root of context hierarchy
2017-03-12 07:15:24.423  INFO 4176 --- [           main] trationDelegate$BeanPostProcessorChecker : Bean 'org.springframework.kafka.annotation.KafkaBootstrapConfiguration' of type [org.springframework.kafka.annotation.KafkaBootstrapConfiguration$$EnhancerBySpringCGLIB$$9bfffabe] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
2017-03-12 07:15:24.642  INFO 4176 --- [           main] o.s.c.support.DefaultLifecycleProcessor  : Starting beans in phase 0
2017-03-12 07:15:24.652  INFO 4176 --- [           main] o.a.k.clients.consumer.ConsumerConfig    : ConsumerConfig values:
2017-03-12 07:15:24.653  INFO 4176 --- [           main] o.a.k.clients.consumer.ConsumerConfig    : ConsumerConfig values:
2017-03-12 07:15:24.677  INFO 4176 --- [           main] o.a.kafka.common.utils.AppInfoParser     : Kafka version : 0.10.1.1
2017-03-12 07:15:24.677  INFO 4176 --- [           main] o.a.kafka.common.utils.AppInfoParser     : Kafka commitId : f10ef2720b03b247
2017-03-12 07:15:24.690  INFO 4176 --- [           main] c.c.kafka.SpringKafkaSenderTest          : Started SpringKafkaSenderTest in 0.937 seconds (JVM running for 5.031)
2017-03-12 07:15:24.719  INFO 4176 --- [           main] o.a.k.clients.consumer.ConsumerConfig    : ConsumerConfig values:
2017-03-12 07:15:24.721  INFO 4176 --- [           main] o.a.k.clients.consumer.ConsumerConfig    : ConsumerConfig values:
2017-03-12 07:15:24.725  INFO 4176 --- [           main] o.a.kafka.common.utils.AppInfoParser     : Kafka version : 0.10.1.1
2017-03-12 07:15:24.725  INFO 4176 --- [           main] o.a.kafka.common.utils.AppInfoParser     : Kafka commitId : f10ef2720b03b247
2017-03-12 07:15:24.812  INFO 4176 --- [quest-handler-4] kafka.admin.AdminUtils$                  : Topic creation {"version":1,"partitions":{"0":[0]}}
2017-03-12 07:15:24.818  INFO 4176 --- [quest-handler-4] kafka.server.KafkaApis                   : [KafkaApi-0] Auto creation of topic helloworld-receiver.t with 1 partitions and replication factor 1 is successful
2017-03-12 07:15:24.825  INFO 4176 --- [127.0.0.1:51286] artitionStateMachine$TopicChangeListener : [TopicChangeListener on Controller 0]: New topics: [Set(helloworld-receiver.t)], deleted topics: [Set()], new partition replica assignment [Map([helloworld-receiver.t,0] -> List(0))]
2017-03-12 07:15:24.825  INFO 4176 --- [127.0.0.1:51286] kafka.controller.KafkaController         : [Controller 0]: New topic creation callback for [helloworld-receiver.t,0]
2017-03-12 07:15:24.828  INFO 4176 --- [127.0.0.1:51286] kafka.controller.KafkaController         : [Controller 0]: New partition creation callback for [helloworld-receiver.t,0]
2017-03-12 07:15:24.829  INFO 4176 --- [127.0.0.1:51286] kafka.controller.PartitionStateMachine   : [Partition state machine on Controller 0]: Invoking state change to NewPartition for partitions [helloworld-receiver.t,0]
2017-03-12 07:15:24.829  INFO 4176 --- [127.0.0.1:51286] kafka.controller.ReplicaStateMachine     : [Replica state machine on controller 0]: Invoking state change to NewReplica for replicas [Topic=helloworld-receiver.t,Partition=0,Replica=0]
2017-03-12 07:15:24.833  INFO 4176 --- [127.0.0.1:51286] kafka.controller.PartitionStateMachine   : [Partition state machine on Controller 0]: Invoking state change to OnlinePartition for partitions [helloworld-receiver.t,0]
2017-03-12 07:17:19.548  INFO 3880 --- [afka-consumer-1] o.a.k.c.c.internals.AbstractCoordinator  : Discovered coordinator localhost:51386 (id: 2147483647 rack: null) for group helloworld.
2017-03-12 07:17:19.548  INFO 3880 --- [afka-consumer-1] o.a.k.c.c.internals.AbstractCoordinator  : (Re-)joining group helloworld_sender_group
2017-03-12 07:17:19.548  INFO 3880 --- [afka-consumer-1] o.a.k.c.c.internals.AbstractCoordinator  : (Re-)joining group helloworld
2017-03-12 07:17:19.557  INFO 3880 --- [quest-handler-3] kafka.coordinator.GroupCoordinator       : [GroupCoordinator 0]: Preparing to restabilize group helloworld with old generation 0
2017-03-12 07:17:19.557  INFO 3880 --- [quest-handler-7] kafka.coordinator.GroupCoordinator       : [GroupCoordinator 0]: Preparing to restabilize group helloworld_sender_group with old generation 0
2017-03-12 07:17:19.567  INFO 3880 --- [quest-handler-7] kafka.coordinator.GroupCoordinator       : [GroupCoordinator 0]: Stabilized group helloworld_sender_group generation 1
2017-03-12 07:17:19.570  INFO 3880 --- [quest-handler-3] kafka.coordinator.GroupCoordinator       : [GroupCoordinator 0]: Stabilized group helloworld generation 1
2017-03-12 07:17:19.576  INFO 3880 --- [quest-handler-6] kafka.coordinator.GroupCoordinator       : [GroupCoordinator 0]: Assignment received from leader for group helloworld_sender_group for generation 1
2017-03-12 07:17:19.576  INFO 3880 --- [quest-handler-2] kafka.coordinator.GroupCoordinator       : [GroupCoordinator 0]: Assignment received from leader for group helloworld for generation 1
2017-03-12 07:17:19.635  INFO 3880 --- [afka-consumer-1] o.a.k.c.c.internals.AbstractCoordinator  : Successfully joined group helloworld with generation 1
2017-03-12 07:17:19.635  INFO 3880 --- [afka-consumer-1] o.a.k.c.c.internals.AbstractCoordinator  : Successfully joined group helloworld_sender_group with generation 1
2017-03-12 07:17:19.636  INFO 3880 --- [afka-consumer-1] o.a.k.c.c.internals.ConsumerCoordinator  : Setting newly assigned partitions [helloworld-receiver.t-0] for group helloworld
2017-03-12 07:17:19.636  INFO 3880 --- [afka-consumer-1] o.a.k.c.c.internals.ConsumerCoordinator  : Setting newly assigned partitions [helloworld-sender.t-0, helloworld-sender.t-1] for group helloworld_sender_group
2017-03-12 07:17:19.658  INFO 3880 --- [afka-consumer-1] o.s.k.l.KafkaMessageListenerContainer    : partitions assigned:[helloworld-receiver.t-0]
2017-03-12 07:17:19.658  INFO 3880 --- [afka-consumer-1] o.s.k.l.KafkaMessageListenerContainer    : partitions assigned:[helloworld-sender.t-0, helloworld-sender.t-1]
2017-03-12 07:17:19.733  INFO 3880 --- [           main] o.a.k.clients.producer.ProducerConfig    : ProducerConfig values:
2017-03-12 07:17:19.734  INFO 3880 --- [           main] o.a.k.clients.producer.ProducerConfig    : ProducerConfig values:
2017-03-12 07:17:19.744  INFO 3880 --- [           main] o.a.kafka.common.utils.AppInfoParser     : Kafka version : 0.10.1.1
2017-03-12 07:17:19.744  INFO 3880 --- [           main] o.a.kafka.common.utils.AppInfoParser     : Kafka commitId : f10ef2720b03b247
2017-03-12 07:17:19.929  INFO 3880 --- [ad | producer-1] com.codenotfound.kafka.producer.Sender   : sent message='Hello Spring Kafka Sender!' with offset=0
2017-03-12 07:17:20.942  INFO 3880 --- [afka-consumer-1] essageListenerContainer$ListenerConsumer : Consumer stopped
2017-03-12 07:17:20.954  INFO 3880 --- [           main] o.a.k.clients.producer.ProducerConfig    : ProducerConfig values:
2017-03-12 07:17:20.955  INFO 3880 --- [           main] o.a.k.clients.producer.ProducerConfig    : ProducerConfig values:
2017-03-12 07:17:20.958  INFO 3880 --- [           main] o.a.kafka.common.utils.AppInfoParser     : Kafka version : 0.10.1.1
2017-03-12 07:17:20.958  INFO 3880 --- [           main] o.a.kafka.common.utils.AppInfoParser     : Kafka commitId : f10ef2720b03b247
2017-03-12 07:17:21.073  INFO 3880 --- [afka-listener-1] c.codenotfound.kafka.consumer.Receiver   : received message='Hello Spring Kafka Receiver!'
2017-03-12 07:17:21.078  INFO 3880 --- [           main] kafka.server.KafkaServer                 : [Kafka Server 0], shutting down
2017-03-12 07:17:21.079  INFO 3880 --- [           main] kafka.server.KafkaServer                 : [Kafka Server 0], Starting controlled shutdown
2017-03-12 07:17:21.090  INFO 3880 --- [quest-handler-6] kafka.controller.KafkaController         : [Controller 0]: Shutting down broker 0
2017-03-12 07:17:21.103  INFO 3880 --- [           main] kafka.server.KafkaServer                 : [Kafka Server 0], Controlled shutdown succeeded
2017-03-12 07:17:21.106  INFO 3880 --- [           main] kafka.network.SocketServer               : [Socket Server on Broker 0], Shutting down
2017-03-12 07:17:21.113  INFO 3880 --- [           main] kafka.network.SocketServer               : [Socket Server on Broker 0], Shutdown completed
2017-03-12 07:17:21.114  INFO 3880 --- [           main] kafka.server.KafkaRequestHandlerPool     : [Kafka Request Handler on Broker 0], shutting down
2017-03-12 07:17:21.115  INFO 3880 --- [           main] kafka.server.KafkaRequestHandlerPool     : [Kafka Request Handler on Broker 0], shut down completely
2017-03-12 07:17:21.118  INFO 3880 --- [           main] lientQuotaManager$ThrottledRequestReaper : [ThrottledRequestReaper-Fetch], Shutting down
2017-03-12 07:17:21.160  INFO 3880 --- [afka-consumer-1] o.a.k.c.c.internals.AbstractCoordinator  : Marking the coordinator localhost:51386 (id: 2147483647 rack: null) dead for group helloworld
2017-03-12 07:17:21.456  INFO 3880 --- [           main] lientQuotaManager$ThrottledRequestReaper : [ThrottledRequestReaper-Fetch], Shutdown completed
2017-03-12 07:17:21.456  INFO 3880 --- [estReaper-Fetch] lientQuotaManager$ThrottledRequestReaper : [ThrottledRequestReaper-Fetch], Stopped
2017-03-12 07:17:21.457  INFO 3880 --- [           main] lientQuotaManager$ThrottledRequestReaper : [ThrottledRequestReaper-Produce], Shutting down
2017-03-12 07:17:22.456  INFO 3880 --- [tReaper-Produce] lientQuotaManager$ThrottledRequestReaper : [ThrottledRequestReaper-Produce], Stopped
2017-03-12 07:17:22.456  INFO 3880 --- [           main] lientQuotaManager$ThrottledRequestReaper : [ThrottledRequestReaper-Produce], Shutdown completed
2017-03-12 07:17:22.457  INFO 3880 --- [           main] kafka.server.KafkaApis                   : [KafkaApi-0] Shutdown complete.
2017-03-12 07:17:22.459  INFO 3880 --- [           main] kafka.server.ReplicaManager              : [Replica Manager on Broker 0]: Shutting down
2017-03-12 07:17:22.460  INFO 3880 --- [           main] kafka.server.ReplicaFetcherManager       : [ReplicaFetcherManager on broker 0] shutting down
2017-03-12 07:17:22.461  INFO 3880 --- [           main] kafka.server.ReplicaFetcherManager       : [ReplicaFetcherManager on broker 0] shutdown completed
2017-03-12 07:17:22.461  INFO 3880 --- [           main] perationPurgatory$ExpiredOperationReaper : [ExpirationReaper-0], Shutting down
2017-03-12 07:17:22.573  INFO 3880 --- [irationReaper-0] perationPurgatory$ExpiredOperationReaper : [ExpirationReaper-0], Stopped
2017-03-12 07:17:22.573  INFO 3880 --- [           main] perationPurgatory$ExpiredOperationReaper : [ExpirationReaper-0], Shutdown completed
2017-03-12 07:17:22.573  INFO 3880 --- [           main] perationPurgatory$ExpiredOperationReaper : [ExpirationReaper-0], Shutting down
2017-03-12 07:17:22.754  INFO 3880 --- [irationReaper-0] perationPurgatory$ExpiredOperationReaper : [ExpirationReaper-0], Stopped
2017-03-12 07:17:22.754  INFO 3880 --- [           main] perationPurgatory$ExpiredOperationReaper : [ExpirationReaper-0], Shutdown completed
2017-03-12 07:17:22.762  INFO 3880 --- [           main] kafka.server.ReplicaManager              : [Replica Manager on Broker 0]: Shut down completely
2017-03-12 07:17:22.762  INFO 3880 --- [           main] perationPurgatory$ExpiredOperationReaper : [ExpirationReaper-0], Shutting down
2017-03-12 07:17:22.933  INFO 3880 --- [irationReaper-0] perationPurgatory$ExpiredOperationReaper : [ExpirationReaper-0], Stopped
2017-03-12 07:17:22.933  INFO 3880 --- [           main] perationPurgatory$ExpiredOperationReaper : [ExpirationReaper-0], Shutdown completed
2017-03-12 07:17:22.934  INFO 3880 --- [           main] kafka.log.LogManager                     : Shutting down.
2017-03-12 07:17:22.935  INFO 3880 --- [           main] kafka.log.LogCleaner                     : Shutting down the log cleaner.
2017-03-12 07:17:22.936  INFO 3880 --- [           main] kafka.log.LogCleaner                     : [kafka-log-cleaner-thread-0], Shutting down
2017-03-12 07:17:22.936  INFO 3880 --- [leaner-thread-0] kafka.log.LogCleaner                     : [kafka-log-cleaner-thread-0], Stopped
2017-03-12 07:17:22.936  INFO 3880 --- [           main] kafka.log.LogCleaner                     : [kafka-log-cleaner-thread-0], Shutdown completed
2017-03-12 07:17:23.260  INFO 3880 --- [           main] kafka.log.LogManager                     : Shutdown complete.
2017-03-12 07:17:23.262  INFO 3880 --- [           main] kafka.coordinator.GroupCoordinator       : [GroupCoordinator 0]: Shutting down.
2017-03-12 07:17:23.262  INFO 3880 --- [           main] perationPurgatory$ExpiredOperationReaper : [ExpirationReaper-0], Shutting down
2017-03-12 07:17:23.325  INFO 3880 --- [irationReaper-0] perationPurgatory$ExpiredOperationReaper : [ExpirationReaper-0], Stopped
2017-03-12 07:17:23.325  INFO 3880 --- [           main] perationPurgatory$ExpiredOperationReaper : [ExpirationReaper-0], Shutdown completed
2017-03-12 07:17:23.325  INFO 3880 --- [           main] perationPurgatory$ExpiredOperationReaper : [ExpirationReaper-0], Shutting down
2017-03-12 07:17:23.470  INFO 3880 --- [           main] perationPurgatory$ExpiredOperationReaper : [ExpirationReaper-0], Shutdown completed
2017-03-12 07:17:23.470  INFO 3880 --- [irationReaper-0] perationPurgatory$ExpiredOperationReaper : [ExpirationReaper-0], Stopped
2017-03-12 07:17:23.471  INFO 3880 --- [           main] kafka.coordinator.GroupCoordinator       : [GroupCoordinator 0]: Shutdown complete.
2017-03-12 07:17:23.473  INFO 3880 --- [           main] .TopicDeletionManager$DeleteTopicsThread : [delete-topics-thread-0], Shutting down
2017-03-12 07:17:23.474  INFO 3880 --- [topics-thread-0] .TopicDeletionManager$DeleteTopicsThread : [delete-topics-thread-0], Stopped
2017-03-12 07:17:23.474  INFO 3880 --- [           main] .TopicDeletionManager$DeleteTopicsThread : [delete-topics-thread-0], Shutdown completed
2017-03-12 07:17:23.477  INFO 3880 --- [           main] kafka.controller.PartitionStateMachine   : [Partition state machine on Controller 0]: Stopped partition state machine
2017-03-12 07:17:23.478  INFO 3880 --- [           main] kafka.controller.ReplicaStateMachine     : [Replica state machine on controller 0]: Stopped replica state machine
2017-03-12 07:17:23.480  INFO 3880 --- [           main] kafka.controller.RequestSendThread       : [Controller-0-to-broker-0-send-thread], Shutting down
2017-03-12 07:17:23.480  INFO 3880 --- [r-0-send-thread] kafka.controller.RequestSendThread       : [Controller-0-to-broker-0-send-thread], Stopped
2017-03-12 07:17:23.480  INFO 3880 --- [           main] kafka.controller.RequestSendThread       : [Controller-0-to-broker-0-send-thread], Shutdown completed
2017-03-12 07:17:23.481  INFO 3880 --- [           main] kafka.controller.KafkaController         : [Controller 0]: Broker 0 resigned as the controller
2017-03-12 07:17:23.482  INFO 3880 --- [127.0.0.1:51382] org.I0Itec.zkclient.ZkEventThread        : Terminate ZkClient event thread.
2017-03-12 07:17:23.483  INFO 3880 --- [0 cport:51382):] o.a.z.server.PrepRequestProcessor        : Processed session termination for sessionid: 0x15ac129f2840001
2017-03-12 07:17:23.489  INFO 3880 --- [           main] org.apache.zookeeper.ZooKeeper           : Session: 0x15ac129f2840001 closed
2017-03-12 07:17:23.490  INFO 3880 --- [ain-EventThread] org.apache.zookeeper.ClientCnxn          : EventThread shut down for session: 0x15ac129f2840001
2017-03-12 07:17:23.490  INFO 3880 --- [ry:/127.0.0.1:0] o.apache.zookeeper.server.NIOServerCnxn  : Closed socket connection for client /127.0.0.1:51389 which had sessionid 0x15ac129f2840001
2017-03-12 07:17:23.491  INFO 3880 --- [           main] kafka.server.KafkaServer                 : [Kafka Server 0], shut down completed
2017-03-12 07:17:23.585  INFO 3880 --- [127.0.0.1:51382] org.I0Itec.zkclient.ZkEventThread        : Terminate ZkClient event thread.
2017-03-12 07:17:23.585  INFO 3880 --- [0 cport:51382):] o.a.z.server.PrepRequestProcessor        : Processed session termination for sessionid: 0x15ac129f2840000
2017-03-12 07:17:23.590  INFO 3880 --- [           main] org.apache.zookeeper.ZooKeeper           : Session: 0x15ac129f2840000 closed
2017-03-12 07:17:23.590  INFO 3880 --- [ry:/127.0.0.1:0] o.apache.zookeeper.server.NIOServerCnxn  : Closed socket connection for client /127.0.0.1:51385 which had sessionid 0x15ac129f2840000
2017-03-12 07:17:23.590  INFO 3880 --- [ain-EventThread] org.apache.zookeeper.ClientCnxn          : EventThread shut down for session: 0x15ac129f2840000
2017-03-12 07:17:23.590  INFO 3880 --- [           main] o.a.zookeeper.server.ZooKeeperServer     : shutting down
2017-03-12 07:17:23.590  INFO 3880 --- [           main] o.a.zookeeper.server.SessionTrackerImpl  : Shutting down
2017-03-12 07:17:23.590  INFO 3880 --- [           main] o.a.z.server.PrepRequestProcessor        : Shutting down
2017-03-12 07:17:23.591  INFO 3880 --- [           main] o.a.z.server.SyncRequestProcessor        : Shutting down
2017-03-12 07:17:23.591  INFO 3880 --- [   SyncThread:0] o.a.z.server.SyncRequestProcessor        : SyncRequestProcessor exited!
2017-03-12 07:17:23.591  INFO 3880 --- [           main] o.a.z.server.FinalRequestProcessor       : shutdown of request processor complete
2017-03-12 07:17:23.591  INFO 3880 --- [0 cport:51382):] o.a.z.server.PrepRequestProcessor        : PrepRequestProcessor exited loop!
2017-03-12 07:17:23.592  INFO 3880 --- [ry:/127.0.0.1:0] o.a.z.server.NIOServerCnxnFactory        : NIOServerCnxn factory exited run method
2017-03-12 07:17:24.000  INFO 3880 --- [ SessionTracker] o.a.zookeeper.server.SessionTrackerImpl  : SessionTrackerImpl exited loop!
Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 11.366 sec - in com.codenotfound.kafka.AllSpringKafkaTests
2017-03-12 07:17:24.622  INFO 3880 --- [       Thread-8] s.c.a.AnnotationConfigApplicationContext : Closing org.springframework.context.annotation.AnnotationConfigApplicationContext@4985cbcb: startup
date [Sun Mar 12 07:17:17 CET 2017]; root of context hierarchy
2017-03-12 07:17:24.624  INFO 3880 --- [       Thread-8] o.s.c.support.DefaultLifecycleProcessor  : Stopping beans in phase 0

Results :

Tests run: 2, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 43.265 s
[INFO] Finished at: 2017-03-12T07:17:54+01:00
[INFO] Final Memory: 16M/226M
[INFO] ------------------------------------------------------------------------
```

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-kafka/tree/master/spring-kafka-embedded-test).
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This concludes our example in which we test Spring Kafka by starting an embedded Kafka server. Feel free to drop a line in case of any questions or if you found this post helpful.