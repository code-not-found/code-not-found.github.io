---
title: "JMS - Point-to-Point Messaging Example using ActiveMQ and Maven"
permalink: /jms-point-to-point-messaging-example-activemq-maven.html
excerpt: "A JMS Point-to-Point messaging example using ActiveMQ and Maven."
date: 2014-10-16
last_modified_at: 2014-10-16
header:
  teaser: "assets/images/teaser/jms-teaser.png"
categories: [JMS]
tags: [ActiveMQ, Apache ActiveMQ, Example, Java Message Service, JMS, Maven, Point-To-Point, PTP]
redirect_from:
  - /2014/10/jms-point-to-point-messaging-using.html
  - /2014/10/jms-point-to-point-messaging-example-activemq-maven.html
---

<figure>
    <img src="{{ site.url }}/assets/images/logo/jms-logo.png" alt="jms logo" class="logo">
</figure>

A point-to-point (PTP) product or application is built on the concept of message queues, senders and receivers. Each message is addressed to a specific queue and receiving clients extract messages from the queues established to hold their messages. Queues retain all messages sent to them until the messages are consumed.

The following post introduces the basic concepts of JMS point-to-point messaging and illustrates them with a code sample using ActiveMQ and Maven.

# Point-to-Point Messaging

<figure>
    <img src="{{ site.url }}/assets/images/posts/jms/jms-point-to-point-messaging.png" alt="jms point-to-point messaging">
</figure>

PTP messaging has the following characteristics:
* Each message has **only one consumer**.
* A sender and a receiver of a message have **no timing dependencies**. The receiver can fetch the message whether or not it was running when the client sent the message.
* The receiver **acknowledges the successful processing** of a message.

# ActiveMQ Example

Let's illustrate the above characteristics by creating a message producer that sends a message containing a first and last name to a queue. In turn a message consumer will read the message and transform it into a greeting. The code is very similar to the [JMS Hello World example]({{ site.url }}/jms-hello-world-activemq-maven.html) but contains a few key differences explained below.

Tools used:
* ActiveMQ 5.15
* Maven 3.5

The code is built and run using [Maven](https://maven.apache.org/){:target="_blank"}. Specified below is the Maven POM file which contains the needed dependencies for Logback, JUnit and ActiveMQ.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>jms-activemq-point-to-point</artifactId>
  <version>1.0</version>
  <packaging>jar</packaging>

  <name>jms-activemq-point-to-point</name>
  <description>JMS - Point-to-Point Messaging Example using ActiveMQ and Maven</description>
  <url>https://codenotfound.com/jms-point-to-point-messaging-example-activemq-maven.html</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <java.version>1.8</java.version>

    <logback.version>1.2.3</logback.version>
    <slf4j.version>1.7.25</slf4j.version>
    <junit.version>4.12</junit.version>
    <activemq.version>5.15.2</activemq.version>

    <maven-compiler-plugin.version>3.7.0</maven-compiler-plugin.version>
  </properties>

  <dependencies>
    <!-- Logging -->
    <dependency>
      <groupId>ch.qos.logback</groupId>
      <artifactId>logback-classic</artifactId>
      <version>${logback.version}</version>
    </dependency>
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-api</artifactId>
      <version>${slf4j.version}</version>
    </dependency>
    <!-- JUnit -->
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>${junit.version}</version>
      <scope>test</scope>
    </dependency>
    <!-- ActiveMQ -->
    <dependency>
      <groupId>org.apache.activemq</groupId>
      <artifactId>activemq-all</artifactId>
      <version>${activemq.version}</version>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>${maven-compiler-plugin.version}</version>
        <configuration>
          <source>${java.version}</source>
          <target>${java.version}</target>
        </configuration>
      </plugin>
    </plugins>
  </build>
</project>
```

The `Producer` class contains a constructor which creates a message producer and needed connection and session objects. The `sendName()` operation takes as input a first and last name which are set on a `TextMessage` which in turn is sent to the queue set on the message producer.

``` java
package com.codenotfound.jms.ptp;

import javax.jms.Connection;
import javax.jms.ConnectionFactory;
import javax.jms.JMSException;
import javax.jms.MessageProducer;
import javax.jms.Queue;
import javax.jms.Session;
import javax.jms.TextMessage;

import org.apache.activemq.ActiveMQConnection;
import org.apache.activemq.ActiveMQConnectionFactory;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class Producer {

  private static final Logger LOGGER =
      LoggerFactory.getLogger(Producer.class);

  private String clientId;
  private Connection connection;
  private Session session;
  private MessageProducer messageProducer;

  public void create(String clientId, String queueName)
      throws JMSException {
    this.clientId = clientId;

    // create a Connection Factory
    ConnectionFactory connectionFactory =
        new ActiveMQConnectionFactory(
            ActiveMQConnection.DEFAULT_BROKER_URL);

    // create a Connection
    connection = connectionFactory.createConnection();
    connection.setClientID(clientId);

    // create a Session
    session =
        connection.createSession(false, Session.AUTO_ACKNOWLEDGE);

    // create the Queue to which messages will be sent
    Queue queue = session.createQueue(queueName);

    // create a MessageProducer for sending messages
    messageProducer = session.createProducer(queue);
  }

  public void closeConnection() throws JMSException {
    connection.close();
  }

  public void sendName(String firstName, String lastName)
      throws JMSException {
    String text = firstName + " " + lastName;

    // create a JMS TextMessage
    TextMessage textMessage = session.createTextMessage(text);

    // send the message to the queue destination
    messageProducer.send(textMessage);

    LOGGER.debug(clientId + ": sent message with text='{}'", text);
  }
}
```

The `Consumer` class contains a constructor which creates a message consumer and needed connection and session objects. A key difference with the [JMS Hello World example]({{ site.url }}/jms-hello-world-activemq-maven.html) is that the `Session` object is created with the `Session.CLIENT_ACKNOWLEDGE` parameter which requires a client to explicitly acknowledge a consumed message by calling the message's `acknowledge()` method.

The `getGreeting()` operation reads a message from the queue and creates a greeting which is returned. Aside from the `timeout` parameter an additional `acknowledge` parameter is passed which is used to determine whether the received message should be acknowledged or not.

``` java
package com.codenotfound.jms.ptp;

import javax.jms.Connection;
import javax.jms.ConnectionFactory;
import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.MessageConsumer;
import javax.jms.Queue;
import javax.jms.Session;
import javax.jms.TextMessage;

import org.apache.activemq.ActiveMQConnection;
import org.apache.activemq.ActiveMQConnectionFactory;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class Consumer {

  private static final Logger LOGGER =
      LoggerFactory.getLogger(Consumer.class);

  private static String NO_GREETING = "no greeting";

  private String clientId;
  private Connection connection;
  private MessageConsumer messageConsumer;

  public void create(String clientId, String queueName)
      throws JMSException {
    this.clientId = clientId;

    // create a Connection Factory
    ConnectionFactory connectionFactory =
        new ActiveMQConnectionFactory(
            ActiveMQConnection.DEFAULT_BROKER_URL);

    // create a Connection
    connection = connectionFactory.createConnection();
    connection.setClientID(clientId);

    // create a Session
    Session session =
        connection.createSession(false, Session.CLIENT_ACKNOWLEDGE);

    // create the Queue from which messages will be received
    Queue queue = session.createQueue(queueName);

    // create a MessageConsumer for receiving messages
    messageConsumer = session.createConsumer(queue);

    // start the connection in order to receive messages
    connection.start();
  }

  public void closeConnection() throws JMSException {
    connection.close();
  }

  public String getGreeting(int timeout, boolean acknowledge)
      throws JMSException {

    String greeting = NO_GREETING;

    // read a message from the queue destination
    Message message = messageConsumer.receive(timeout);

    // check if a message was received
    if (message != null) {
      // cast the message to the correct type
      TextMessage textMessage = (TextMessage) message;

      // retrieve the message content
      String text = textMessage.getText();
      LOGGER.debug(clientId + ": received message with text='{}'",
          text);

      if (acknowledge) {
        // acknowledge the successful processing of the message
        message.acknowledge();
        LOGGER.debug(clientId + ": message acknowledged");
      } else {
        LOGGER.debug(clientId + ": message not acknowledged");
      }

      // create greeting
      greeting = "Hello " + text + "!";
    } else {
      LOGGER.debug(clientId + ": no message received");
    }

    LOGGER.info("greeting={}", greeting);
    return greeting;
  }
}
```

The below JUnit test class will be used to illustrate the PTP messaging characteristics mentioned at the beginning of this post. The `testGreeting()` test case verifies the correct working of the `getGreeting()` method of the `Consumer` class.

The `testOnlyOneConsumer()` test case will verify that a message is read by only one consumer. The first consumer will receive the greeting and the second consumer will receive nothing.

The `testNoTimingDependencies()` test case illustrates that the consumer can successfully receive a message even if that consumer is created after the message was sent.

Finally the `testAcknowledgeProcessing()` test case will verify that a message is not removed by the JMS provider in case it was not acknowledged by the consumer. In order to simulate this we first call the `getGreeting()` method with the acknowledge parameter set to `false`. Then the `getGreeting()` method is called a second time and as the first call did not acknowledge the message it is still available on the queue.

``` java
package com.codenotfound.jms.ptp;

import static org.junit.Assert.assertEquals;
import static org.junit.Assert.fail;

import javax.jms.JMSException;
import javax.naming.NamingException;

import org.junit.AfterClass;
import org.junit.BeforeClass;
import org.junit.Test;

public class ConsumerTest {

  private static Producer producerPointToPoint,
      producerOnlyOneConsumer, producerNoTimingDependencies,
      producerAcknowledgeProcessing;
  private static Consumer consumerPointToPoint,
      consumer1OnlyOneConsumer, consumer2OnlyOneConsumer,
      consumerNoTimingDependencies, consumer1AcknowledgeProcessing,
      consumer2AcknowledgeProcessing;

  @BeforeClass
  public static void setUpBeforeClass()
      throws JMSException, NamingException {
    producerPointToPoint = new Producer();
    producerPointToPoint.create("producer-pointtopoint",
        "pointtopoint.q");

    producerOnlyOneConsumer = new Producer();
    producerOnlyOneConsumer.create("producer-onlyoneconsumer",
        "onlyoneconsumer.q");

    producerNoTimingDependencies = new Producer();
    producerNoTimingDependencies.create(
        "producer-notimingdependencies", "notimingdependencies.q");

    producerAcknowledgeProcessing = new Producer();
    producerAcknowledgeProcessing.create(
        "producer-acknowledgeprocessing", "acknowledgeprocessing.q");

    consumerPointToPoint = new Consumer();
    consumerPointToPoint.create("consumer-pointtopoint",
        "pointtopoint.q");

    consumer1OnlyOneConsumer = new Consumer();
    consumer1OnlyOneConsumer.create("consumer1-onlyoneconsumer",
        "onlyoneconsumer.q");

    consumer2OnlyOneConsumer = new Consumer();
    consumer2OnlyOneConsumer.create("consumer2-onlyoneconsumer",
        "onlyoneconsumer.q");

    // consumerNoTimingDependencies

    consumer1AcknowledgeProcessing = new Consumer();
    consumer1AcknowledgeProcessing.create(
        "consumer1-acknowledgeprocessing", "acknowledgeprocessing.q");

    consumer2AcknowledgeProcessing = new Consumer();
    consumer2AcknowledgeProcessing.create(
        "consumer2-acknowledgeprocessing", "acknowledgeprocessing.q");
  }

  @AfterClass
  public static void tearDownAfterClass() throws JMSException {
    producerPointToPoint.closeConnection();
    producerOnlyOneConsumer.closeConnection();
    producerNoTimingDependencies.closeConnection();
    producerAcknowledgeProcessing.closeConnection();

    consumerPointToPoint.closeConnection();
    consumer1OnlyOneConsumer.closeConnection();
    consumer2OnlyOneConsumer.closeConnection();
    consumerNoTimingDependencies.closeConnection();
    // consumer1AcknowledgeProcessing
    consumer2AcknowledgeProcessing.closeConnection();
  }

  @Test
  public void testGetGreeting() {
    try {
      producerPointToPoint.sendName("Frodo", "Baggins");

      String greeting = consumerPointToPoint.getGreeting(1000, true);
      assertEquals("Hello Frodo Baggins!", greeting);

    } catch (JMSException e) {
      fail("a JMS Exception occurred");
    }
  }

  @Test
  public void testOnlyOneConsumer() throws InterruptedException {
    try {
      producerOnlyOneConsumer.sendName("Legolas", "Greenleaf");

      String greeting1 =
          consumer1OnlyOneConsumer.getGreeting(1000, true);
      assertEquals("Hello Legolas Greenleaf!", greeting1);

      Thread.sleep(1000);

      String greeting2 =
          consumer2OnlyOneConsumer.getGreeting(1000, true);
      // each message has only one consumer
      assertEquals("no greeting", greeting2);

    } catch (JMSException e) {
      fail("a JMS Exception occurred");
    }
  }

  @Test
  public void testNoTimingDependencies() {
    try {
      producerNoTimingDependencies.sendName("Samwise", "Gamgee");
      // a receiver can fetch the message whether or not it was running
      // when the client sent the message
      consumerNoTimingDependencies = new Consumer();
      consumerNoTimingDependencies.create(
          "consumer-notimingdependencies", "notimingdependencies.q");

      String greeting =
          consumerNoTimingDependencies.getGreeting(1000, true);
      assertEquals("Hello Samwise Gamgee!", greeting);

    } catch (JMSException e) {
      fail("a JMS Exception occurred");
    }
  }

  @Test
  public void testAcknowledgeProcessing()
      throws InterruptedException {
    try {
      producerAcknowledgeProcessing.sendName("Gandalf", "the Grey");

      // consume the message without an acknowledgment
      String greeting1 =
          consumer1AcknowledgeProcessing.getGreeting(1000, false);
      assertEquals("Hello Gandalf the Grey!", greeting1);

      // close the MessageConsumer so the broker knows there is no
      // acknowledgment
      consumer1AcknowledgeProcessing.closeConnection();

      String greeting2 =
          consumer2AcknowledgeProcessing.getGreeting(1000, true);
      assertEquals("Hello Gandalf the Grey!", greeting2);

    } catch (JMSException e) {
      fail("a JMS Exception occurred");
    }
  }
}
```

Make sure a default [ActiveMQ message broker is up and running]({{ site.url }}/jms-apache-activemq-installation.html), open a command prompt and execute following Maven command:

``` plaintext
mvn test
```

This will trigger Maven to run the above test cases which should result in the following log statements.

``` plaintext
-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running com.codenotfound.jms.ptp.ConsumerTest
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/C:/code/local-repo/ch/qos/logback/logback-classic/1.2.3/logback-classic-1.2.3.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/C:/code/local-repo/org/apache/activemq/activemq-all/5.15.2/activemq-all-5.15.2.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [ch.qos.logback.classic.util.ContextSelectorStaticBinder]
21:14:49.716 DEBUG [main][Producer]producer-onlyoneconsumer: sent message with text='Legolas Greenleaf'
21:14:49.720 DEBUG [main][Consumer]consumer1-onlyoneconsumer: received message with text='Legolas Greenleaf'
21:14:49.721 DEBUG [main][Consumer]consumer1-onlyoneconsumer: message acknowledged
21:14:49.721 INFO  [main][Consumer]greeting=Hello Legolas Greenleaf!
21:14:51.722 DEBUG [main][Consumer]consumer2-onlyoneconsumer: no message received
21:14:51.722 INFO  [main][Consumer]greeting=no greeting
21:14:51.732 DEBUG [main][Producer]producer-notimingdependencies: sent message with text='Samwise Gamgee'
21:14:51.746 DEBUG [main][Consumer]consumer-notimingdependencies: received message with text='Samwise Gamgee'
21:14:51.747 DEBUG [main][Consumer]consumer-notimingdependencies: message acknowledged
21:14:51.747 INFO  [main][Consumer]greeting=Hello Samwise Gamgee!
21:14:51.757 DEBUG [main][Producer]producer-acknowledgeprocessing: sent message with text='Gandalf the Grey'
21:14:51.757 DEBUG [main][Consumer]consumer1-acknowledgeprocessing: received message with text='Gandalf the Grey'
21:14:51.757 DEBUG [main][Consumer]consumer1-acknowledgeprocessing: message not acknowledged
21:14:51.757 INFO  [main][Consumer]greeting=Hello Gandalf the Grey!
21:14:51.769 DEBUG [main][Consumer]consumer2-acknowledgeprocessing: received message with text='Gandalf the Grey'
21:14:51.769 DEBUG [main][Consumer]consumer2-acknowledgeprocessing: message acknowledged
21:14:51.770 INFO  [main][Consumer]greeting=Hello Gandalf the Grey!
21:14:51.777 DEBUG [main][Producer]producer-pointtopoint: sent message with text='Frodo Baggins'
21:14:51.777 DEBUG [main][Consumer]consumer-pointtopoint: received message with text='Frodo Baggins'
21:14:51.779 DEBUG [main][Consumer]consumer-pointtopoint: message acknowledged
21:14:51.779 INFO  [main][Consumer]greeting=Hello Frodo Baggins!
Tests run: 4, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 2.814 sec

Results :

Tests run: 4, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 6.363 s
[INFO] Finished at: 2018-01-04T21:14:51+01:00
[INFO] Final Memory: 22M/177M
[INFO] ------------------------------------------------------------------------
```

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/jms/tree/master/jms-activemq-point-to-point){:target="_blank"}.
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This concludes the JMS point-to-point example using ActiveMQ. If you found this post helpful or have any questions or remarks, please leave a comment.
