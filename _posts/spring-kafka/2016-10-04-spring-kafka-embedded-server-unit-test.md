---
title: Spring Kafka - Embedded Server Unit Test
permalink: /2016/10/spring-kafka-embedded-server-unit-test.html
excerpt: A detailed step-by-step tutorial on how to test your application using an embedded Apache Kafka server together with Spring Kafka and Spring Boot.
date: 2016-10-04 21:00
categories: [Spring Kafka]
tags: [Apache Kafka, Embedded, Example, Maven, Server, Spring Boot, Spring Kafka, Test, Testing, Tutorial, Unit  ]
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/spring-logo.png" alt="spring logo">
</figure>

The [Spring Kafka project](https://projects.spring.io/spring-kafka/) comes with a `spring-kafka-test` JAR that contains a number of useful utilities to assist you with your application testing. These include: an embedded Kafka server, some static methods to setup consumers/producers and some utility methods to fetch results.

Let's demonstrate how you can use these utilities with some runnable code. We will reuse the [Spring Kafka Hello World project](http://www.source4code.info/2016/09/spring-kafka-consumer-producer-example.html) from a previous post in which we created a consumer and producer using Spring Kafka, Spring Boot and Maven.


Tools used:
* Spring Kafka 1.1
* Spring Boot 1.4
* Maven 3

We need to add the `spring-kafka-test` dependency to the Maven POM file in addition to the Spring Kafka and Spring Boot dependencies. In the plugins section we included the `maven-surefire-plugin` to trigger a `AllSpringKafkaTests` test suite class that will be used to start the embedded server only once for the different unit test cases in our project.

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
    <url>http://www.codenotfound.com/2016/09/spring-kafka-embedded-kafka-test-example.html</url>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.4.1.RELEASE</version>
        <relativePath /> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <java.version>1.8</java.version>
        <spring-kafka.version>1.1.1.RELEASE</spring-kafka.version>
    </properties>

    <dependencies>
        <!-- Spring Kafka -->
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
            <version>${spring-kafka.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka-test</artifactId>
            <version>${spring-kafka.version}</version>
        </dependency>
        <!-- Spring Boot -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.19.1</version>
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

`spring-kafka-test` includes an embedded Kafka server that can be created via a JUnit `@ClassRule` annotation. The rule will start a ZooKeeper and Kafka server instance on a random port before all the test cases run, and stops the instances after they are finished. In order to support multiple unit test classes (`SpringKafkaSenderTests` and `SpringKafkaReceiverTests`), we will trigger the `@ClassRule` from a `Suite` class that bundles them together. 

 As the embedded server is started on a random port, we need to change the property value that is used by the `SenderConfig` and `ReceiverConfig` classes. This is done by calling the `getBrokersAsString()` method and setting the value to the '<var>kafka.bootstrap.servers</var>' property. 

``` java
package com.codenotfound;

import org.junit.BeforeClass;
import org.junit.ClassRule;
import org.junit.runner.RunWith;
import org.junit.runners.Suite;
import org.junit.runners.Suite.SuiteClasses;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.kafka.test.rule.KafkaEmbedded;

@RunWith(Suite.class)
@SuiteClasses({ SpringKafkaSenderTests.class,
        SpringKafkaReceiverTests.class })
public class AllSpringKafkaTests {

    private static final Logger LOGGER = LoggerFactory
            .getLogger(AllSpringKafkaTests.class);

    protected static final String HELLOWORLD_SENDER_TOPIC = "helloworld-sender.t";
    protected static final String HELLOWORLD_RECEIVER_TOPIC = "helloworld-receiver.t";

    @ClassRule
    public static KafkaEmbedded embeddedKafka = new KafkaEmbedded(1,
            true, HELLOWORLD_SENDER_TOPIC);

    @BeforeClass
    public static void setUpBeforeClass() throws Exception {
        String kafkaBootstrapServers = embeddedKafka
                .getBrokersAsString();
        LOGGER.debug("kafkaServers='{}'", kafkaBootstrapServers);
        // override the property in application.properties
        System.setProperty("kafka.bootstrap.servers",
                kafkaBootstrapServers);
    }
}
```

In the first test class we will be testing the `Sender` by sending a message to a '<var>helloworld-sender.t</var>' topic. We will verify the correct sending by setting up a message listener on the topic. For creating the consumer properties we use a static method provided by `KafkaUtils`. After setting up the `KafkaMessageListenerContainer` we setup a `MessageListener` and start the container. 

In order to avoid that we send the message before the container has required the number of assigned partitions, we use the `waitForAssignment()` method on the `ContainerTestUtils` helper class. We then send a greeting and assert that the received value is the same as the one that was sent using an AssertJ condition that is provided by `KafkaConditions` which is also provided via `spring-kafka-test`. 

``` java
package com.codenotfound;

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

import com.codenotfound.producer.Sender;

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringKafkaSenderTests {

    private static final Logger LOGGER = LoggerFactory
            .getLogger(SpringKafkaSenderTests.class);

    @Autowired
    private Sender sender;

    @Test
    public void testSender() throws Exception {
        // set up the Kafka consumer properties
        Map<String, Object> consumerProperties = KafkaTestUtils
                .consumerProps("helloworld_sender_group", "false",
                        AllSpringKafkaTests.embeddedKafka);

        // create a Kafka consumer factory
        DefaultKafkaConsumerFactory<Integer, String> consumerFactory = new DefaultKafkaConsumerFactory<Integer, String>(
                consumerProperties);
        // set the topic that needs to be consumed
        ContainerProperties containerProperties = new ContainerProperties(
                AllSpringKafkaTests.HELLOWORLD_SENDER_TOPIC);

        // create a Kafka MessageListenerContainer
        KafkaMessageListenerContainer<Integer, String> container = new KafkaMessageListenerContainer<>(
                consumerFactory, containerProperties);

        // create a thread safe queue to store the received message
        BlockingQueue<ConsumerRecord<Integer, String>> records = new LinkedBlockingQueue<>();
        // setup a Kafka message listener
        container.setupMessageListener(
                new MessageListener<Integer, String>() {
                    @Override
                    public void onMessage(
                            ConsumerRecord<Integer, String> record) {
                        LOGGER.debug(record.toString());
                        records.add(record);
                    }
                });

        // start the container and underlying message listener
        container.start();
        // wait until the container has the required number of assigned
        // partitions
        ContainerTestUtils.waitForAssignment(container,
                AllSpringKafkaTests.embeddedKafka
                        .getPartitionsPerTopic());

        // send the message
        String greeting = "Hello Spring Kafka Sender!";
        sender.sendMessage(AllSpringKafkaTests.HELLOWORLD_SENDER_TOPIC,
                greeting);
        // check that the message was received
        assertThat(records.poll(10, TimeUnit.SECONDS))
                .has(value(greeting));

        // stop the container
        container.stop();
    }
}
```

The second test class focuses on the `Receiver` which listens to a '<var>helloworld-receiver.t</var>' topic as defined in the applications.properties file. In order to check the correct working we will use a producer to send a message to this topic. The producer properties are created using the static method provided by `KafkaUtils` and used to create a `KafkaTemplate`. 

We need to ensure that the `Receiver` is initialized before sending the test message. We again use the `waitForAssignment()` of `ContainerTestUtils`. The link to the message listener container is acquired by autowiring the `KafkaListenerEndpointRegistry` which manages the lifecycle of the listener containers that are not created manually. We check that the message was received by asserting that the latch of the `Receiver` was lowered to zero. 

> Note that we manually set the partitions per topic to 1 in the `waitForAssignment()` method instead getting the partitions from the embedded Kafka server. The reason for this is that [it looks like 1 is used as a default for the number of partitions in case topics are created implicitly](http://stackoverflow.com/a/38660145/4201470). 

``` java
package com.codenotfound;

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

import com.codenotfound.consumer.Receiver;

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringKafkaReceiverTests {

    @Autowired
    private Receiver receiver;

    @Autowired
    KafkaListenerEndpointRegistry kafkaListenerEndpointRegistry;

    @Test
    public void testReceiver() throws Exception {
        // set up the Kafka producer properties
        Map<String, Object> senderProperties = KafkaTestUtils
                .senderProps(AllSpringKafkaTests.embeddedKafka
                        .getBrokersAsString());

        // create a Kafka producer factory
        ProducerFactory<Integer, String> producerFactory = new DefaultKafkaProducerFactory<Integer, String>(
                senderProperties);

        // create a Kafka template
        KafkaTemplate<Integer, String> template = new KafkaTemplate<>(
                producerFactory);
        // set the default topic to send to
        template.setDefaultTopic(
                AllSpringKafkaTests.HELLOWORLD_RECEIVER_TOPIC);

        // get the ConcurrentMessageListenerContainers
        for (MessageListenerContainer messageListenerContainer : kafkaListenerEndpointRegistry
                .getListenerContainers()) {
            if (messageListenerContainer instanceof ConcurrentMessageListenerContainer) {
                ConcurrentMessageListenerContainer<Integer, String> concurrentMessageListenerContainer = (ConcurrentMessageListenerContainer<Integer, String>) messageListenerContainer;

                // as the topic is created implicitly, the default number of
                // partitions is 1
                int partitionsPerTopic = 1;
                // wait until the container has the required number of assigned
                // partitions
                ContainerTestUtils.waitForAssignment(
                        concurrentMessageListenerContainer,
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
 :: Spring Boot ::        (v1.4.1.RELEASE)

14:14:39.835 INFO  [main][SpringKafkaSenderTests] Starting SpringKafkaSenderTests on cnf-pc with PID 2764 (started by CodeNotFound in c:\code\spring-kafka\spring-kafka-embedded-test)
14:14:39.835 DEBUG [main][SpringKafkaSenderTests] Running with Spring Boot v1.4.1.RELEASE, Spring v4.3.3.RELEASE
14:14:39.836 INFO  [main][SpringKafkaSenderTests] No active profile set, falling back to default profiles: default
14:14:39.856 INFO  [main][AnnotationConfigApplicationContext] Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@138fe6ec: startup date [Tue Oct 04 14:14:39 CEST 2016]; root of context hierarchy
14:14:40.372 INFO  [main][PostProcessorRegistrationDelegate$BeanPostProcessorChecker] Bean 'org.springframework.kafka.annotation.KafkaBootstrapConfiguration' of type [class org.springframework.kafka.annotation.KafkaBootstrapConfiguration$$EnhancerBySpringCGLIB$$319d2a18] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
14:14:40.561 INFO  [main][DefaultLifecycleProcessor] Starting beans in phase 0
14:14:40.607 INFO  [main][SpringKafkaSenderTests] Started SpringKafkaSenderTests in 1.025 seconds (JVM running for 5.346)
14:14:40.732 WARN  [org.springframework.kafka.KafkaListenerEndpointContainer#0-0-kafka-consumer-1][NetworkClient] Error while fetching metadata with correlation id 1 : {helloworld-receiver.t=LEADER_NOT_AVAILABLE}
14:14:40.910 WARN  [org.springframework.kafka.KafkaListenerEndpointContainer#0-0-kafka-consumer-1][NetworkClient] Error while fetching metadata with correlation id 2 : {helloworld-receiver.t=LEADER_NOT_AVAILABLE}
14:14:42.033 INFO  [org.springframework.kafka.KafkaListenerEndpointContainer#0-0-kafka-consumer-1][KafkaMessageListenerContainer] partitions revoked:[]
14:14:42.033 INFO  [-kafka-consumer-1][KafkaMessageListenerContainer] partitions revoked:[]
14:14:42.123 INFO  [-kafka-consumer-1][KafkaMessageListenerContainer] partitions assigned:[helloworld-sender.t-0, helloworld-sender.t-1]
14:14:42.123 INFO  [org.springframework.kafka.KafkaListenerEndpointContainer#0-0-kafka-consumer-1][KafkaMessageListenerContainer] partitions assigned:[helloworld-receiver.t-0]
14:14:42.350 INFO  [kafka-producer-network-thread | producer-1][Sender] sent message='Hello Spring Kafka Sender!' with offset=0
14:14:42.356 DEBUG [-kafka-listener-1][SpringKafkaSenderTests] ConsumerRecord(topic = helloworld-sender.t, partition = 1, offset = 0, CreateTime = 1475583282309, checksum = 4290501015, serialized keysize = -1, serialized value size = 26, key = null, value = Hello Spring Kafka Sender!)
14:14:43.362 INFO  [-kafka-consumer-1][KafkaMessageListenerContainer$ListenerConsumer] Consumer stopped
14:14:43.490 INFO  [org.springframework.kafka.KafkaListenerEndpointContainer#0-0-kafka-listener-1][Receiver] received message='Hello Spring Kafka Receiver!'
Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 11.94 sec - in com.codenotfound.AllSpringKafkaTests
14:14:47.449 INFO  [Thread-7][AnnotationConfigApplicationContext] Closing org.springframework.context.annotation.AnnotationConfigApplicationContext@138fe6ec: startup date [Tue Oct 04 14:14:39 CEST 2016]; root of context hierarchy
14:14:47.458 INFO  [Thread-7][DefaultLifecycleProcessor] Stopping beans in phase 0
14:14:47.496 INFO  [org.springframework.kafka.KafkaListenerEndpointContainer#0-0-kafka-consumer-1][KafkaMessageListenerContainer$ListenerConsumer] Consumer stopped

Results :

Tests run: 2, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 14.156 s
[INFO] Finished at: 2016-10-04T14:14:47+02:00
[INFO] Final Memory: 17M/226M
[INFO] ------------------------------------------------------------------------
```

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-kafka/tree/master/spring-kafka-embedded-test).
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This concludes our example in which we test Spring Kafka by starting an embedded Kafka server. Feel free to drop a line in case of any questions or if you found this post helpful.