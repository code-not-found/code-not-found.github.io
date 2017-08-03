---
title: "Spring Kafka - Listener Example"
permalink: /spring-kafka-listener-example.html
excerpt: "A detailed step-by-step tutorial on how to consume different message types from multiple topics using Spring Kafka and Spring Boot."
date: 2017-08-06
modified: 2017-08-06
header:
  teaser: "assets/images/header/spring-kafka-teaser.png"
categories: [Spring Kafka]
tags: [Apache Kafka, Example, Listener, Maven, Spring, Spring Boot, Spring Kafka, Tutorial]
published: false
---
 
<figure>
    <img src="{{ site.url }}/assets/images/logo/spring-logo.png" alt="spring logo" class="logo">
</figure>

[Starting with version 1.1](http://docs.spring.io/spring-kafka/docs/1.1.0.RELEASE/reference/html/whats-new-part.html#_batch_listeners){:target="_blank"} of Spring Kafka, `@KafkaListener` methods can be configured to receive a batch of consumer records from the consumer poll operation.

The following example shows how to setup a batch listener using Spring Kafka, Spring Boot, and Maven.

If you want to learn more about Spring Kafka - head on over to the [Spring Kafka tutorials page]({{ site.url }}/spring-kafka/).
{: .notice--primary}

# General Project Setup

Tools used:
* Spring Kafka 1.2
* Spring Boot 1.5
* Maven 3.5

The general project and `Sender` configuration are identical to a previous [Spring Boot Kafka example]({{ site.url }}/spring-kafka-consumer-producer-example.html). As such we won't go into detail on how these are setup.

# Configuring a Batch Listener and Batch Size

Enabling batch receiving of messages can be achieved by setting the `batchListener` property. This is done by calling the `setBatchListener()` method on the listener container factory (`ConcurrentKafkaListenerContainerFactory` in this example) with a value `true` as shown below.

By default, the number of records received in each batch is dynamically calculated. By setting the <var>'MAX_POLL_RECORDS_CONFIG'</var> property on the `ConsumerConfig` we can set an upper limit for the batch size. For this example, we define a maximum of 10 messages to be returned per poll.

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
    props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
    props.put(ConsumerConfig.GROUP_ID_CONFIG, "batch");
    // maximum records per poll
    props.put(ConsumerConfig.MAX_POLL_RECORDS_CONFIG, "10");

    return props;
  }

  @Bean
  public ConsumerFactory<String, String> consumerFactory() {
    return new DefaultKafkaConsumerFactory<>(consumerConfigs());
  }

  @Bean
  public ConcurrentKafkaListenerContainerFactory<String, String> kafkaListenerContainerFactory() {
    ConcurrentKafkaListenerContainerFactory<String, String> factory =
        new ConcurrentKafkaListenerContainerFactory<>();
    factory.setConsumerFactory(consumerFactory());
    // enable batch listening
    factory.setBatchListener(true);

    return factory;
  }

  @Bean
  public Receiver receiver() {
    return new Receiver();
  }
}
```

The `receive()` method of the `Receiver` listener POJO needs to be updated to receive a `List` of payloads (in this example these are simple `String` objects). Alternatively a list of `Message<?>` or `ConsumerRecord<?, ?>` objects [can be configured](http://docs.spring.io/spring-kafka/docs/1.2.0.RELEASE/reference/html/_reference.html#__kafkalistener_annotation){:target="_blank"}.

For logging purposes, we also add the partition and offset headers of each message to the `receive()` method. These headers are available in a list and map to the received messages based on the index within the list.

The `CountDownLatch` value that is used in the unit test case is increased so that we can send out a batch of 20 messages.

``` java
package com.codenotfound.kafka.consumer;

import java.util.List;
import java.util.concurrent.CountDownLatch;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.kafka.support.KafkaHeaders;
import org.springframework.messaging.handler.annotation.Header;

public class Receiver {

  private static final Logger LOGGER = LoggerFactory.getLogger(Receiver.class);

  public static final int COUNT = 20;

  private CountDownLatch latch = new CountDownLatch(COUNT);

  public CountDownLatch getLatch() {
    return latch;
  }

  @KafkaListener(id = "batch-listener", topics = "${kafka.topic.batch}")
  public void receive(List<String> data,
      @Header(KafkaHeaders.RECEIVED_PARTITION_ID) List<Integer> partitions,
      @Header(KafkaHeaders.OFFSET) List<Long> offsets) {

    LOGGER.info("start of batch receive");
    for (int i = 0; i < data.size(); i++) {
      LOGGER.info("received message='{}' with partition-offset='{}'", data.get(i),
          partitions.get(i) + "-" + offsets.get(i));
      // handle message

      latch.countDown();
    }
    LOGGER.info("end of batch receive");
  }
}
```

# Testing the Batch Listener

The `SpringKafkaApplicationTest` test case starts an [embedded Kafka and ZooKeeper server]({{ site.url }}/spring-kafka-embedded-unit-test-example.html) using a JUnit `ClassRule`.

Using `@Before` we wait until all the partitions are assigned to our `Receiver` by looping over the available `ConcurrentMessageListenerContainer` (if we don't do this the message will already be sent before the listeners are assigned to the topic).

The `testReceiver()` method uses a `for` loop to send out as many messages as were configured on the `CountDownLatch` in the `Receiver`. The result is that our listener starts receiving batches of messages from the Kafka broker partitions (2 partitions are created by default on the embedded broker).

``` java
package com.codenotfound.kafka;

import static org.assertj.core.api.Assertions.assertThat;

import java.util.concurrent.TimeUnit;

import org.junit.Before;
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

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringKafkaApplicationTest {

  private static String BATCH_TOPIC = "batch.t";

  @Autowired
  private Sender sender;

  @Autowired
  private Receiver receiver;

  @Autowired
  private KafkaListenerEndpointRegistry kafkaListenerEndpointRegistry;

  @ClassRule
  public static KafkaEmbedded embeddedKafka = new KafkaEmbedded(1, true, BATCH_TOPIC);

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
  public void testReceive() throws Exception {
    //
    int numberOfMessages = Receiver.COUNT;
    for (int i = 0; i < numberOfMessages; i++) {
      sender.send(BATCH_TOPIC, "message " + i);
    }

    receiver.getLatch().await(10000, TimeUnit.MILLISECONDS);
    assertThat(receiver.getLatch().getCount()).isEqualTo(0);
  }
}
```

Run the test case by entering following Maven command at the command prompt:

``` plaintext
mvn test
```

The result should be 20 message that get sent and received from a <var>'batch.t'</var> topic as shown below:

``` plaintext
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.5.4.RELEASE)

16:46:18.654 [main] INFO  c.c.kafka.SpringKafkaApplicationTest - Starting SpringKafkaApplicationTest on cnf-pc with PID 3636 (started by CodeNotFound in c:\codenotfound\spring-kafka\spring-kafka-batch-listener)
16:46:18.655 [main] INFO  c.c.kafka.SpringKafkaApplicationTest - No active profile set, falling back to default profiles: default
16:46:19.344 [main] INFO  c.c.kafka.SpringKafkaApplicationTest - Started SpringKafkaApplicationTest in 0.995 seconds (JVM running for 5.206)
16:46:20.774 [main] INFO  c.codenotfound.kafka.producer.Sender - sending data='message 0' to topic='batch.t'
16:46:20.821 [main] INFO  c.codenotfound.kafka.producer.Sender - sending data='message 1' to topic='batch.t'
16:46:20.821 [main] INFO  c.codenotfound.kafka.producer.Sender - sending data='message 2' to topic='batch.t'
16:46:20.822 [main] INFO  c.codenotfound.kafka.producer.Sender - sending data='message 3' to topic='batch.t'
16:46:20.822 [main] INFO  c.codenotfound.kafka.producer.Sender - sending data='message 4' to topic='batch.t'
16:46:20.822 [main] INFO  c.codenotfound.kafka.producer.Sender - sending data='message 5' to topic='batch.t'
16:46:20.823 [main] INFO  c.codenotfound.kafka.producer.Sender - sending data='message 6' to topic='batch.t'
16:46:20.824 [main] INFO  c.codenotfound.kafka.producer.Sender - sending data='message 7' to topic='batch.t'
16:46:20.824 [main] INFO  c.codenotfound.kafka.producer.Sender - sending data='message 8' to topic='batch.t'
16:46:20.824 [main] INFO  c.codenotfound.kafka.producer.Sender - sending data='message 9' to topic='batch.t'
16:46:20.824 [main] INFO  c.codenotfound.kafka.producer.Sender - sending data='message 10' to topic='batch.t'
16:46:20.825 [main] INFO  c.codenotfound.kafka.producer.Sender - sending data='message 11' to topic='batch.t'
16:46:20.828 [main] INFO  c.codenotfound.kafka.producer.Sender - sending data='message 12' to topic='batch.t'
16:46:20.829 [main] INFO  c.codenotfound.kafka.producer.Sender - sending data='message 13' to topic='batch.t'
16:46:20.829 [main] INFO  c.codenotfound.kafka.producer.Sender - sending data='message 14' to topic='batch.t'
16:46:20.829 [main] INFO  c.codenotfound.kafka.producer.Sender - sending data='message 15' to topic='batch.t'
16:46:20.830 [main] INFO  c.codenotfound.kafka.producer.Sender - sending data='message 16' to topic='batch.t'
16:46:20.830 [main] INFO  c.codenotfound.kafka.producer.Sender - sending data='message 17' to topic='batch.t'
16:46:20.830 [main] INFO  c.codenotfound.kafka.producer.Sender - sending data='message 18' to topic='batch.t'
16:46:20.831 [main] INFO  c.codenotfound.kafka.producer.Sender - sending data='message 19' to topic='batch.t'
16:46:20.855 [batch-listener-0-C-1] INFO  c.c.kafka.consumer.Receiver - start of batch receive
16:46:20.856 [batch-listener-0-C-1] INFO  c.c.kafka.consumer.Receiver - received message='message 1' with partition-offset='0-0'
16:46:20.856 [batch-listener-0-C-1] INFO  c.c.kafka.consumer.Receiver - received message='message 3' with partition-offset='0-1'
16:46:20.856 [batch-listener-0-C-1] INFO  c.c.kafka.consumer.Receiver - received message='message 5' with partition-offset='0-2'
16:46:20.856 [batch-listener-0-C-1] INFO  c.c.kafka.consumer.Receiver - received message='message 7' with partition-offset='0-3'
16:46:20.856 [batch-listener-0-C-1] INFO  c.c.kafka.consumer.Receiver - received message='message 9' with partition-offset='0-4'
16:46:20.856 [batch-listener-0-C-1] INFO  c.c.kafka.consumer.Receiver - received message='message 11' with partition-offset='0-5'
16:46:20.856 [batch-listener-0-C-1] INFO  c.c.kafka.consumer.Receiver - received message='message 13' with partition-offset='0-6'
16:46:20.856 [batch-listener-0-C-1] INFO  c.c.kafka.consumer.Receiver - received message='message 15' with partition-offset='0-7'
16:46:20.856 [batch-listener-0-C-1] INFO  c.c.kafka.consumer.Receiver - end of batch receive
16:46:20.861 [batch-listener-0-C-1] INFO  c.c.kafka.consumer.Receiver - start of batch receive
16:46:20.861 [batch-listener-0-C-1] INFO  c.c.kafka.consumer.Receiver - received message='message 0' with partition-offset='1-0'
16:46:20.861 [batch-listener-0-C-1] INFO  c.c.kafka.consumer.Receiver - received message='message 2' with partition-offset='1-1'
16:46:20.861 [batch-listener-0-C-1] INFO  c.c.kafka.consumer.Receiver - received message='message 4' with partition-offset='1-2'
16:46:20.861 [batch-listener-0-C-1] INFO  c.c.kafka.consumer.Receiver - received message='message 6' with partition-offset='1-3'
16:46:20.861 [batch-listener-0-C-1] INFO  c.c.kafka.consumer.Receiver - received message='message 8' with partition-offset='1-4'
16:46:20.861 [batch-listener-0-C-1] INFO  c.c.kafka.consumer.Receiver - received message='message 10' with partition-offset='1-5'
16:46:20.861 [batch-listener-0-C-1] INFO  c.c.kafka.consumer.Receiver - received message='message 12' with partition-offset='1-6'
16:46:20.861 [batch-listener-0-C-1] INFO  c.c.kafka.consumer.Receiver - received message='message 14' with partition-offset='1-7'
16:46:20.861 [batch-listener-0-C-1] INFO  c.c.kafka.consumer.Receiver - received message='message 16' with partition-offset='1-8'
16:46:20.861 [batch-listener-0-C-1] INFO  c.c.kafka.consumer.Receiver - received message='message 18' with partition-offset='1-9'
16:46:20.862 [batch-listener-0-C-1] INFO  c.c.kafka.consumer.Receiver - end of batch receive
16:46:20.862 [batch-listener-0-C-1] INFO  c.c.kafka.consumer.Receiver - start of batch receive
16:46:20.862 [batch-listener-0-C-1] INFO  c.c.kafka.consumer.Receiver - received message='message 17' with partition-offset='0-8'
16:46:20.862 [batch-listener-0-C-1] INFO  c.c.kafka.consumer.Receiver - received message='message 19' with partition-offset='0-9'
16:46:20.862 [batch-listener-0-C-1] INFO  c.c.kafka.consumer.Receiver - end of batch receive
16:46:22.857 [main] ERROR o.a.zookeeper.server.ZooKeeperServer - ZKShutdownHandler is not registered, so ZooKeeper server won't take any action on ERROR or SHUTDOWN server state changes
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 9.168 sec - in com.codenotfound.kafka.SpringKafkaApplicationTest

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 12.360 s
[INFO] Finished at: 2017-07-14T16:46:23+02:00
[INFO] Final Memory: 27M/218M
[INFO] ------------------------------------------------------------------------
```

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-kafka/tree/master/spring-kafka-batch-listener).
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This concludes setting up a Spring Kafka batch listener on a Kafka topic.

If you liked this tutorial or have any questions, leave a comment below.
