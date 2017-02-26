---
title: Spring Kafka - Consumer &amp; Producer Example
permalink: /2016/09/spring-kafka-consumer-producer-example.html
excerpt: A detailed step-by-step tutorial on how to implement an Apache Kafka Consumer and Producer using Spring Kafka and Spring Boot.
date: 2016-09-20 21:00
categories: [Spring Kafka]
tags: [Apache Kafka, Consumer, Example, Hello World, Maven, Producer, Spring, Spring Boot, Spring Kafka, Tutorial]
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/spring-logo.png" alt="spring logo">
</figure>

The [Spring for Apache Kafka (spring-kafka) project](https://projects.spring.io/spring-kafka/) applies core Spring concepts to the development of Kafka-based messaging solutions. It provides a 'template' as a high-level abstraction for sending messages. It also provides support for Message-driven POJOs with `@KafkaListener` annotations and a 'listener container'.

In the following tutorial we will configure, build and run a Hello World example in which we will send/receive messages to/from Apache Kafka using Spring Kafka, Spring Boot and Maven. Before running below code, make sure that [Apache Kafka is installed and started]({{ site.url }}/2016/09/apache-kafka-download-installation.html).

> Spring Kafka 1.1 uses the Apache Kafka 0.10.x.x client.

Tools used:
* Spring Kafka 1.1
* Spring Boot 1.4
* Maven 3

We start by defining a Maven POM file which contains the dependencies for the needed [Spring projects](https://spring.io/projects). The POM inherits from the `spring-boot-starter-parent` project and declares dependencies to `spring-boot-starter` and `spring-boot-starter-test` starters.

A dependency to `spring-kafka` is added in addition to a property that specifies the version. At the time of writing the latest stable release was <var>'1.1.1.RELEASE'</var>.

We also include the `spring-boot-maven-plugin` Maven plugin so that we can build a single, runnable "über-jar", which is convenient to execute and transport the written code.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.codenotfound</groupId>
    <artifactId>spring-kafka-helloworld-example</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <name>spring-kafka-helloworld-example</name>
    <description>Spring Kafka - Consumer & Producer Example</description>
    <url>http://www.codenotfound.com/2016/09/spring-kafka-consumer-producer-example.html</url>

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
        </plugins>
    </build>

</project>
```

We will use Spring Boot in order to make a Spring Kafka example application that you can "just run". We start by creating an `SpringKafkaApplication` that contains the `main()` method that uses Spring Boot’s `SpringApplication.run()` method to launch an application. The `@SpringBootApplication` annotation is a convenience annotation that adds: `@Configuration`, `@EnableAutoConfiguration` and `@ComponentScan`.

For more information on Spring Boot you can check out the [Spring Boot getting started guide](https://spring.io/guides/gs/spring-boot/).

``` java
package com.codenotfound;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringKafkaApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringKafkaApplication.class, args);
    }
}
```

# Create a Spring Kafka Message Producer

For sending messages we will be using the `KafkaTemplate` which wraps a producer and provides convenience methods to send data to Kafka topics. The template provides both asynchronous and synchronous methods, with the asynchronous methods returning a `Future`.

In the below `Sender` class, the `KafkaTemplate` is autowired as the actual creation of the `Bean` will be done in a separate `SenderConfig` class. In this example we will be using the async `send()` method and we will also configure the `KafkaTemplate` with a `ProducerListener` to get an async callback with the results of the send (success or failure) instead of waiting for the Future to complete. If the sending of a message is successful we simply create a log statement.

``` java
package com.codenotfound.producer;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.support.SendResult;
import org.springframework.util.concurrent.ListenableFuture;
import org.springframework.util.concurrent.ListenableFutureCallback;

public class Sender {

    private static final Logger LOGGER = LoggerFactory
            .getLogger(Sender.class);

    @Autowired
    private KafkaTemplate<Integer, String> kafkaTemplate;

    public void sendMessage(String topic, String message) {
        // the KafkaTemplate provides asynchronous send methods returning a
        // Future
        ListenableFuture<SendResult<Integer, String>> future = kafkaTemplate
                .send(topic, message);

        // you can register a callback with the listener to receive the result
        // of the send asynchronously
        future.addCallback(
                new ListenableFutureCallback<SendResult<Integer, String>>() {

                    @Override
                    public void onSuccess(
                            SendResult<Integer, String> result) {
                        LOGGER.info("sent message='{}' with offset={}",
                                message,
                                result.getRecordMetadata().offset());
                    }

                    @Override
                    public void onFailure(Throwable ex) {
                        LOGGER.error("unable to send message='{}'",
                                message, ex);
                    }
                });

        // alternatively, to block the sending thread, to await the result,
        // invoke the future’s get() method
    }
}
```

The creation of the `KafkaTemplate` and `Sender` is handled in the `SenderConfig` class. The class is annoted with `@Configuration` which indicates that the class can be used by the Spring IoC container as a source of bean definitions.

In order to be able to use the Spring Kafka template we need to configure a `ProducerFactory` and provide it in the template’s constructor. The producer factory needs to be set with a number of properties amongst which the `BOOTSTRAP_SERVERS_CONFIG` property that is fetched from the <var>application.properties</var> configuration file. For a complete list of the available configuration parameters you can consult the [Kafka ProducerConfig API](https://kafka.apache.org/0100/javadoc/org/apache/kafka/clients/producer/ProducerConfig.html).

``` java
package com.codenotfound.producer;

import java.util.HashMap;
import java.util.Map;

import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.common.serialization.IntegerSerializer;
import org.apache.kafka.common.serialization.StringSerializer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.core.DefaultKafkaProducerFactory;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.core.ProducerFactory;

@Configuration
public class SenderConfig {

    @Value("${kafka.bootstrap.servers}")
    private String bootstrapServers;

    @Bean
    public Map producerConfigs() {
        Map props = new HashMap<>();
        // list of host:port pairs used for establishing the initial connections
        // to the Kakfa cluster
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG,
                bootstrapServers);
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG,
                IntegerSerializer.class);
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,
                StringSerializer.class);
        // value to block, after which it will throw a TimeoutException
        props.put(ProducerConfig.MAX_BLOCK_MS_CONFIG, 5000);

        return props;
    }

    @Bean
    public ProducerFactory producerFactory() {
        return new DefaultKafkaProducerFactory<>(producerConfigs());
    }

    @Bean
    public KafkaTemplate kafkaTemplate() {
        return new KafkaTemplate(producerFactory());
    }

    @Bean
    public Sender sender() {
        return new Sender();
    }
}
```

# Create a Spring Kafka Message Consumer

Like with any messaging-based application, you need to create a receiver that will handle the published messages. The Receiver is nothing more than a simple POJO that defines a method for receiving messages. In the below example we named the method `receiveMessage`, but you can name it anything you want.

The `@KafkaListener` annotation creates a message listener container behind the scenes for each annotated method, using a `ConcurrentMessageListenerContainer`. By default, a bean with name `kafkaListenerContainerFactory` is expected that we will setup in the next section. Using the `topics` element, we specify the topics for this listener. For more information on the other available elements, you can consult the [KafkaListener API documentation](http://docs.spring.io/spring-kafka/api/org/springframework/kafka/annotation/KafkaListener.html).

> For testing convenience, we added a `CountDownLatch`. This allows the POJO to signal that a message is received. This is something you are not likely to implement in a production application. 

``` java
package com.codenotfound.consumer;

import java.util.concurrent.CountDownLatch;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.kafka.annotation.KafkaListener;

public class Receiver {

    private static final Logger LOGGER = LoggerFactory
            .getLogger(Receiver.class);

    private CountDownLatch latch = new CountDownLatch(1);

    @KafkaListener(topics = "helloworld.t")
    public void receiveMessage(String message) {
        LOGGER.info("received message='{}'", message);
        latch.countDown();
    }

    public CountDownLatch getLatch() {
        return latch;
    }
}
```

The creation and configuration of the different Spring Beans needed for the `Receiver` POJO are grouped in the `ReceiverConfig` class. Note that we need to add the `@EnableKafka` annotation to enable support for the `@KafkaListener` annotation that was used on the `Receiver`.

The `kafkaListenerContainerFactory()` is used by the `@KafkaListener` annotation from the `Receiver`. In order to create it, a `ConsumerFactory` and accompanying configuration `Map` is needed. In this example only the mandatory configuration parameters are set, for a complete list consult the [Kafka ConsumerConfig API](https://kafka.apache.org/0100/javadoc/index.html?org/apache/kafka/clients/consumer/ConsumerConfig.html). 

``` java
package com.codenotfound.consumer;

import java.util.HashMap;
import java.util.Map;

import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.common.serialization.IntegerDeserializer;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.annotation.EnableKafka;
import org.springframework.kafka.config.ConcurrentKafkaListenerContainerFactory;
import org.springframework.kafka.core.ConsumerFactory;
import org.springframework.kafka.core.DefaultKafkaConsumerFactory;

@Configuration
@EnableKafka
public class ReceiverConfig {

    @Value("${kafka.bootstrap.servers}")
    private String bootstrapServers;

    @Bean
    public Map consumerConfigs() {
        Map props = new HashMap<>();
        // list of host:port pairs used for establishing the initial connections
        // to the Kakfa cluster
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG,
                bootstrapServers);
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG,
                IntegerDeserializer.class);
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG,
                StringDeserializer.class);
        // consumer groups allow a pool of processes to divide the work of
        // consuming and processing records
        props.put(ConsumerConfig.GROUP_ID_CONFIG, "helloworld");

        return props;
    }

    @Bean
    public ConsumerFactory consumerFactory() {
        return new DefaultKafkaConsumerFactory<>(consumerConfigs());
    }

    @Bean
    public ConcurrentKafkaListenerContainerFactory kafkaListenerContainerFactory() {
        ConcurrentKafkaListenerContainerFactory factory = new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());

        return factory;
    }

    @Bean
    public Receiver receiver() {
        return new Receiver();
    }
}
```

# Testing the Spring Kafka Template & Listener

> When executing below test case, make sure you have a running instance of Apache Kafka on port '9092' of your local machine. Note that it is also possible to [use Spring Kafka to automatically start an embedded Kafka broker as part of a unit test case]({{ site.url }}/2016/10/spring-kafka-embedded-server-unit-test.html).

In order to verify that we are able to send and receive a message to and from Kafka, a basic `SpringKafkaApplicationTests` test case is used. It contains a `testReceiver()` unit test case that uses the `Sender` to send a message to the <var>helloworld.t</var> topic on the Kafka bus. We then use the `CountDownLatch` from the `Receiver` to verify that a message was received. 

``` java
package com.codenotfound;

import static org.assertj.core.api.Assertions.assertThat;

import java.util.concurrent.TimeUnit;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import com.codenotfound.consumer.Receiver;
import com.codenotfound.producer.Sender;

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
 :: Spring Boot ::        (v1.4.1.RELEASE)

20:11:52.957 INFO  [main][SpringKafkaApplicationTests] Starting SpringKafkaApplicationTests on cnf-pc with PID 5668 (started by CodeNotFound in c:\code\spring-kafka\spring-kafka-helloworld-example)
20:11:52.957 DEBUG [main][SpringKafkaApplicationTests] Running with Spring Boot v1.4.1.RELEASE, Spring v4.3.3.RELEASE
20:11:52.957 INFO  [main][SpringKafkaApplicationTests] No active profile set, falling back to default profiles: default
20:11:53.001 INFO  [main][AnnotationConfigApplicationContext] Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@5fa07e12: startup date [Tue Sep 27 20:11:53 CEST 2016]; root of context hierarchy
20:11:53.472 INFO  [main][PostProcessorRegistrationDelegate$BeanPostProcessorChecker] Bean 'org.springframework.kafka.annotation.KafkaBootstrapConfiguration' of type [class org.springframework.kafka.annotation.KafkaBootstrapConfiguration$$EnhancerBySpringCGLIB$$cbff67af] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
20:11:53.643 INFO  [main][DefaultLifecycleProcessor] Starting beans in phase 0
20:11:53.817 INFO  [main][SpringKafkaApplicationTests] Started SpringKafkaApplicationTests in 1.224 seconds (JVM running for 1.933)
20:11:53.935 INFO  [org.springframework.kafka.KafkaListenerEndpointContainer#0-0-kafka-consumer-1][KafkaMessageListenerContainer] partitions revoked:[]
20:11:53.949 INFO  [org.springframework.kafka.KafkaListenerEndpointContainer#0-0-kafka-consumer-1][KafkaMessageListenerContainer] partitions assigned:[helloworld.t-0]
20:11:53.964 INFO  [kafka-producer-network-thread | producer-1][Sender] sent message='Hello Spring Kafka!' with offset=41
20:11:53.973 INFO  [org.springframework.kafka.KafkaListenerEndpointContainer#0-0-kafka-listener-1][Receiver] received message='Hello Spring Kafka!'
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 1.487 sec - in com.codenotfound.SpringKafkaApplicationTests
20:11:54.068 INFO  [Thread-1][AnnotationConfigApplicationContext] Closing org.springframework.context.annotation.AnnotationConfigApplicationContext@5fa07e12: startup date [Tue Sep 27 20:11:53 CEST 2016]; root of context hierarchy
20:11:54.070 INFO  [Thread-1][DefaultLifecycleProcessor] Stopping beans in phase 0
20:11:54.981 INFO  [org.springframework.kafka.KafkaListenerEndpointContainer#0-0-kafka-consumer-1][KafkaMessageListenerContainer$ListenerConsumer] Consumer stopped

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 4.468 s
[INFO] Finished at: 2016-09-27T20:11:55+02:00
[INFO] Final Memory: 16M/226M
[INFO] ------------------------------------------------------------------------
```

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-kafka/tree/master/spring-kafka-helloworld-example).
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This wraps up our example in which we used a Spring Kafka template to create a producer and Spring Kafka listener to create a consumer. If you found this sample useful or have a question you would like to ask, drop a line below!