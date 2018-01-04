---
title: "JMS - Publish/Subscribe messaging example using ActiveMQ and Maven"
permalink: /jms-publish-subscribe-messaging-example-activemq-maven.html
excerpt: "A JMS Publish/Subscribe messaging example using ActiveMQ and Maven."
date: 2014-11-26
last_modified_at: 2014-11-26
header:
  teaser: "assets/images/teaser/jms-teaser.png"
categories: [JMS]
tags: [ActiveMQ, Apache ActiveMQ, durable subscription, Example, Java Message Service, JMS, publish, subscribe]
redirect_from:
  - /2014/11/jms-publish-subscribe-messaging-example-activemq-maven.html
---

<figure>
    <img src="{{ site.url }}/assets/images/logo/jms-logo.png" alt="jms logo" class="logo">
</figure>

In a publish/subscribe (pub/sub) product or application, clients address messages to a topic, which functions somewhat like a bulletin board. Subscribers can receive information, in the form of messages, from publishers. Topics retain messages only as long as it takes to distribute them to current subscribers.

The following post introduces the basic concepts of JMS point-to-point messaging and illustrates them with a code sample using ActiveMQ and Maven.

# Publish/Subscribe Messaging

<figure>
    <img src="{{ site.url }}/assets/images/posts/jms/jms-publish-subscribe-messaging.png" alt="jms publish subscribe messaging">
</figure>

Pub/sub messaging has the following characteristics:
* Each message can have **multiple consumers**.
* Publishers and subscribers have a **timing dependency**. A client that subscribes to a topic can consume only messages published after the client has created a subscription, and the subscriber must continue to be active in order for it to consume messages.

The JMS API relaxes this timing dependency mentioned in the second bullet to some extent by allowing subscribers to create durable subscriptions, which receive messages sent while the subscribers are not active. **Durable subscriptions** provide the flexibility and reliability of queues but still allow clients to send messages to many recipients.

# ActiveMQ Example

Let's illustrate the above characteristics by creating a message producer that sends a message containing a first and last name to a topic. In turn a message consumer will read the message and transform it into a greeting. The code is very similar to the [JMS Hello World example]({{ site.url }}/jms-hello-world-activemq-maven.html) but contains a few key differences explained below.

Tools used:
* ActiveMQ 5.15
* Maven 3.5

The code is built and run using [Maven](https://maven.apache.org/){:target="_blank"}. Specified below is the Maven POM file which contains the needed dependencies for Logback, JUnit and [Apache ActiveMQ](http://activemq.apache.org/){:target="_blank"}.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>jms-activemq-publish-subscribe</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>JMS - Publish/Subscribe messaging using ActiveMQ</name>
  <description>JMS - Publish/Subscribe messaging example using ActiveMQ and Maven</description>
  <url>https://codenotfound.com/jms-publish-subscribe-messaging-example-activemq-maven.html</url>

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

# Nondurable Subscription

The `Publisher` class contains a constructor which creates a message producer and needed connection and session objects. The `sendName()` operation takes as input a first and last name which are set on a `TextMessage` which in turn is sent to the topic set on the message producer.

``` java
package com.codenotfound.jms.pubsub;

import javax.jms.Connection;
import javax.jms.ConnectionFactory;
import javax.jms.JMSException;
import javax.jms.MessageProducer;
import javax.jms.Session;
import javax.jms.TextMessage;
import javax.jms.Topic;

import org.apache.activemq.ActiveMQConnection;
import org.apache.activemq.ActiveMQConnectionFactory;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class Publisher {

  private static final Logger LOGGER =
      LoggerFactory.getLogger(Publisher.class);

  private String clientId;
  private Connection connection;
  private Session session;
  private MessageProducer messageProducer;

  public void create(String clientId, String topicName)
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

    // create the Topic to which messages will be sent
    Topic topic = session.createTopic(topicName);

    // create a MessageProducer for sending messages
    messageProducer = session.createProducer(topic);
  }

  public void closeConnection() throws JMSException {
    connection.close();
  }

  public void sendName(String firstName, String lastName)
      throws JMSException {
    String text = firstName + " " + lastName;

    // create a JMS TextMessage
    TextMessage textMessage = session.createTextMessage(text);

    // send the message to the topic destination
    messageProducer.send(textMessage);

    LOGGER.debug(clientId + ": sent message with text='{}'", text);
  }
}
```

The `Subscriber` class contains a constructor which creates a message consumer and needed connection and session objects. The `getGreeting()` operation reads a message from the topic and creates a greeting which is returned. A `timeout` parameter is passed to assure that the method does not wait indefinitely for a message to arrive.

``` java
package com.codenotfound.jms.pubsub;

import javax.jms.Connection;
import javax.jms.ConnectionFactory;
import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.MessageConsumer;
import javax.jms.Session;
import javax.jms.TextMessage;
import javax.jms.Topic;

import org.apache.activemq.ActiveMQConnection;
import org.apache.activemq.ActiveMQConnectionFactory;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class Subscriber {

  private static final Logger LOGGER =
      LoggerFactory.getLogger(Subscriber.class);

  private static final String NO_GREETING = "no greeting";

  private String clientId;
  private Connection connection;
  private MessageConsumer messageConsumer;

  public void create(String clientId, String topicName)
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
        connection.createSession(false, Session.AUTO_ACKNOWLEDGE);

    // create the Topic from which messages will be received
    Topic topic = session.createTopic(topicName);

    // create a MessageConsumer for receiving messages
    messageConsumer = session.createConsumer(topic);

    // start the connection in order to receive messages
    connection.start();
  }

  public void closeConnection() throws JMSException {
    connection.close();
  }

  public String getGreeting(int timeout) throws JMSException {

    String greeting = NO_GREETING;

    // read a message from the topic destination
    Message message = messageConsumer.receive(timeout);

    // check if a message was received
    if (message != null) {
      // cast the message to the correct type
      TextMessage textMessage = (TextMessage) message;

      // retrieve the message content
      String text = textMessage.getText();
      LOGGER.debug(clientId + ": received message with text='{}'",
          text);

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

The below JUnit test class will be used to illustrate the Pub/Sub messaging characteristics mentioned at the beginning of this post. The `testGreeting()` test case verifies the correct working of the `getGreeting()` method of the `Subscriber` class.

The `testMultipleConsumers()` test case will verify that the same message can be read by multiple consumers. In order to test this, two `Subscriber` instances are created on the same <var>'multipleconsumers.t'</var> topic.

Finally the `testNonDurableSubscriber()` test case will illustrate the timing dependency between publisher and subscriber. First a message is sent to a topic on which only one subscriber listens. Then a second subscriber is added to the same topic and a second message is sent. The result is that the second subscriber only receives the second message and not the first one whereas the first subscriber has received both messages. 

``` java
package com.codenotfound.jms.pubsub;

import static org.junit.Assert.assertEquals;
import static org.junit.Assert.fail;

import javax.jms.JMSException;

import org.junit.AfterClass;
import org.junit.BeforeClass;
import org.junit.Test;

public class SubscriberTest {

  private static Publisher publisherPublishSubscribe,
      publisherMultipleConsumers, publisherNonDurableSubscriber;
  private static Subscriber subscriberPublishSubscribe,
      subscriber1MultipleConsumers, subscriber2MultipleConsumers,
      subscriber1NonDurableSubscriber,
      subscriber2NonDurableSubscriber;

  @BeforeClass
  public static void setUpBeforeClass() throws Exception {
    publisherPublishSubscribe = new Publisher();
    publisherPublishSubscribe.create("publisher-publishsubscribe",
        "publishsubscribe.t");

    publisherMultipleConsumers = new Publisher();
    publisherMultipleConsumers.create("publisher-multipleconsumers",
        "multipleconsumers.t");

    publisherNonDurableSubscriber = new Publisher();
    publisherNonDurableSubscriber.create(
        "publisher-nondurablesubscriber", "nondurablesubscriber.t");

    subscriberPublishSubscribe = new Subscriber();
    subscriberPublishSubscribe.create("subscriber-publishsubscribe",
        "publishsubscribe.t");

    subscriber1MultipleConsumers = new Subscriber();
    subscriber1MultipleConsumers.create(
        "subscriber1-multipleconsumers", "multipleconsumers.t");

    subscriber2MultipleConsumers = new Subscriber();
    subscriber2MultipleConsumers.create(
        "subscriber2-multipleconsumers", "multipleconsumers.t");

    subscriber1NonDurableSubscriber = new Subscriber();
    subscriber1NonDurableSubscriber.create(
        "subscriber1-nondurablesubscriber", "nondurablesubscriber.t");

    subscriber2NonDurableSubscriber = new Subscriber();
    subscriber2NonDurableSubscriber.create(
        "subscriber2-nondurablesubscriber", "nondurablesubscriber.t");
  }

  @AfterClass
  public static void tearDownAfterClass() throws Exception {
    publisherPublishSubscribe.closeConnection();
    publisherMultipleConsumers.closeConnection();
    publisherNonDurableSubscriber.closeConnection();

    subscriberPublishSubscribe.closeConnection();
    subscriber1MultipleConsumers.closeConnection();
    subscriber2MultipleConsumers.closeConnection();
    subscriber1NonDurableSubscriber.closeConnection();
    subscriber2NonDurableSubscriber.closeConnection();
  }

  @Test
  public void testGetGreeting() {
    try {
      publisherPublishSubscribe.sendName("Peregrin", "Took");

      String greeting1 = subscriberPublishSubscribe.getGreeting(1000);
      assertEquals("Hello Peregrin Took!", greeting1);

      String greeting2 = subscriberPublishSubscribe.getGreeting(1000);
      assertEquals("no greeting", greeting2);

    } catch (JMSException e) {
      fail("a JMS Exception occurred");
    }
  }

  @Test
  public void testMultipleConsumers() {
    try {
      publisherMultipleConsumers.sendName("Gandalf", "the Grey");

      String greeting1 =
          subscriber1MultipleConsumers.getGreeting(1000);
      assertEquals("Hello Gandalf the Grey!", greeting1);

      String greeting2 =
          subscriber2MultipleConsumers.getGreeting(1000);
      assertEquals("Hello Gandalf the Grey!", greeting2);

    } catch (JMSException e) {
      fail("a JMS Exception occurred");
    }
  }

  @Test
  public void testNonDurableSubscriber() {
    try {
      // nondurable subscriptions, will not receive messages sent while
      // the subscribers are not active
      subscriber2NonDurableSubscriber.closeConnection();

      publisherNonDurableSubscriber.sendName("Bilbo", "Baggins");

      // recreate a connection for the nondurable subscription
      subscriber2NonDurableSubscriber.create(
          "subscriber2-nondurablesubscriber",
          "nondurablesubscriber.t");

      publisherNonDurableSubscriber.sendName("Frodo", "Baggins");

      String greeting1 =
          subscriber1NonDurableSubscriber.getGreeting(1000);
      assertEquals("Hello Bilbo Baggins!", greeting1);
      String greeting2 =
          subscriber1NonDurableSubscriber.getGreeting(1000);
      assertEquals("Hello Frodo Baggins!", greeting2);

      String greeting3 =
          subscriber2NonDurableSubscriber.getGreeting(1000);
      assertEquals("Hello Frodo Baggins!", greeting3);
      String greeting4 =
          subscriber2NonDurableSubscriber.getGreeting(1000);
      assertEquals("no greeting", greeting4);

    } catch (JMSException e) {
      fail("a JMS Exception occurred");
    }
  }
}
```

Make sure a default [ActiveMQ message broker is up and running]({{ site.url }}/jms-apache-activemq-installation.html), open a command prompt and execute following Maven command:

``` plaintext
mvn -Dtest=SubscriberTest test
```

This will trigger Maven to run the above test cases which should result in the following log statements.

``` plaintext
07:24:00.299 DEBUG [main][Publisher] publisher-multipleconsumers: sent message with text='Gandalf the Grey'
07:24:00.303 DEBUG [main][Subscriber] subscriber1-multipleconsumers: received message with text='Gandalf the Grey'
07:24:00.303 INFO  [main][Subscriber] greeting=Hello Gandalf the Grey!
07:24:00.304 DEBUG [main][Subscriber] subscriber2-multipleconsumers: received message with text='Gandalf the Grey'
07:24:00.304 INFO  [main][Subscriber] greeting=Hello Gandalf the Grey!
07:24:00.306 DEBUG [main][Publisher] publisher-publishsubscribe: sent message with text='Peregrin Took'
07:24:00.306 DEBUG [main][Subscriber] subscriber-publishsubscribe: received message with text='Peregrin Took'
07:24:00.307 INFO  [main][Subscriber] greeting=Hello Peregrin Took!
07:24:01.307 DEBUG [main][Subscriber] subscriber-publishsubscribe: no message received
07:24:01.307 INFO  [main][Subscriber] greeting=no greeting
07:24:01.320 DEBUG [main][Publisher] publisher-nondurablesubscriber: sent message with text='Bilbo Baggins'
07:24:01.337 DEBUG [main][Publisher] publisher-nondurablesubscriber: sent message with text='Frodo Baggins'
07:24:01.338 DEBUG [main][Subscriber] subscriber1-nondurablesubscriber: received message with text='Bilbo Baggins'
07:24:01.338 INFO  [main][Subscriber] greeting=Hello Bilbo Baggins!
07:24:01.338 DEBUG [main][Subscriber] subscriber1-nondurablesubscriber: received message with text='Frodo Baggins'
07:24:01.338 INFO  [main][Subscriber] greeting=Hello Frodo Baggins!
07:24:01.339 DEBUG [main][Subscriber] subscriber2-nondurablesubscriber: received message with text='Frodo Baggins'
07:24:01.339 INFO  [main][Subscriber] greeting=Hello Frodo Baggins!
07:24:02.339 DEBUG [main][Subscriber] subscriber2-nondurablesubscriber: no message received
07:24:02.339 INFO  [main][Subscriber] greeting=no greeting
```

# Durable Subscription

As mentioned in the beginning of this post it is also possible to create a durable subscription which allows to receive messages sent while the subscribers are not active.

The JMS specification dictates that the identification of a specific durable subscription is done by a combination of the <var>'client ID'</var>, the <var>'durable subscription name'</var> and the <var>'topic name'</var>.

As a result the below `DurableSubscriber` has three main differences with the previous `Subscriber` class:
1. A `clientId` **is mandatory** on the connection in order to allow a JMS provider to uniquely identify a durable subscriber.
2. A durable subscriber is created using `Session.CreateDurableSubscriber`.
3. A `subscriptionName` is needed when creating the durable subscriber.

 > Note that creating a `MessageConsumer` provides the same features as creating a `TopicSubscriber`. The `TopicSubscriber` is provided to [support existing code](http://docs.oracle.com/javaee/6/api/javax/jms/TopicSubscriber.html){:target="_blank"}.

``` java
package com.codenotfound.jms.pubsub;

import javax.jms.Connection;
import javax.jms.ConnectionFactory;
import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.MessageConsumer;
import javax.jms.Session;
import javax.jms.TextMessage;
import javax.jms.Topic;

import org.apache.activemq.ActiveMQConnection;
import org.apache.activemq.ActiveMQConnectionFactory;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class DurableSubscriber {

  private static final Logger LOGGER =
      LoggerFactory.getLogger(DurableSubscriber.class);

  private static final String NO_GREETING = "no greeting";

  private String clientId;
  private Connection connection;
  private Session session;
  private MessageConsumer messageConsumer;

  private String subscriptionName;

  public void create(String clientId, String topicName,
      String subscriptionName) throws JMSException {
    this.clientId = clientId;
    this.subscriptionName = subscriptionName;

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

    // create the Topic from which messages will be received
    Topic topic = session.createTopic(topicName);

    // create a MessageConsumer for receiving messages
    messageConsumer =
        session.createDurableSubscriber(topic, subscriptionName);

    // start the connection in order to receive messages
    connection.start();
  }

  public void removeDurableSubscriber() throws JMSException {
    messageConsumer.close();
    session.unsubscribe(subscriptionName);
  }

  public void closeConnection() throws JMSException {
    connection.close();
  }

  public String getGreeting(int timeout) throws JMSException {

    String greeting = NO_GREETING;

    // read a message from the topic destination
    Message message = messageConsumer.receive(timeout);

    // check if a message was received
    if (message != null) {
      // cast the message to the correct type
      TextMessage textMessage = (TextMessage) message;

      // retrieve the message content
      String text = textMessage.getText();
      LOGGER.debug(clientId + ":  received message with text='{}'",
          text);

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

The below JUnit test class will be used to illustrate the durable subscriber messaging characteristics.

It contains a `testDurableSubscriber()` test case that will first remove one of the two durable subscribers that are listening on the <var>'durablesubscriber.t'</var> topic by closing it's connection to the broker. Then a first message is sent to this topic on which only one subscribers is still actively listening. The second subscriber is recreated using the same client ID and subscription name and a second message is sent. The expected result is that both subscribers receive the two messages.

> Note that in the `tearDownAfterClass()` method the durable subscriptions are removed in order to avoid an error when rerunning the test case. 

``` java
package com.codenotfound.jms.pubsub;

import static org.junit.Assert.assertEquals;
import static org.junit.Assert.fail;

import javax.jms.JMSException;

import org.junit.AfterClass;
import org.junit.BeforeClass;
import org.junit.Test;

public class DurableSubscriberTest {

  private static Publisher publisherPublishSubscribe,
      publisherDurableSubscriber;
  private static DurableSubscriber subscriberPublishSubscribe,

      subscriber1DurableSubscriber, subscriber2DurableSubscriber;

  @BeforeClass
  public static void setUpBeforeClass() throws Exception {
    publisherPublishSubscribe = new Publisher();
    publisherPublishSubscribe.create("publisher-publishsubscribe",
        "publishsubscribe.t");

    publisherDurableSubscriber = new Publisher();
    publisherDurableSubscriber.create("publisher-durablesubscriber",
        "durablesubscriber.t");

    subscriberPublishSubscribe = new DurableSubscriber();
    subscriberPublishSubscribe.create("subscriber-publishsubscribe",
        "publishsubscribe.t", "publishsubscribe");

    subscriber1DurableSubscriber = new DurableSubscriber();
    subscriber1DurableSubscriber.create(
        "subscriber1-durablesubscriber", "durablesubscriber.t",
        "durablesubscriber1");

    subscriber2DurableSubscriber = new DurableSubscriber();
    subscriber2DurableSubscriber.create(
        "subscriber2-durablesubscriber", "durablesubscriber.t",
        "durablesubscriber2");
  }

  @AfterClass
  public static void tearDownAfterClass() throws Exception {
    publisherPublishSubscribe.closeConnection();
    publisherDurableSubscriber.closeConnection();

    // remove the durable subscriptions
    subscriberPublishSubscribe.removeDurableSubscriber();
    subscriber1DurableSubscriber.removeDurableSubscriber();
    subscriber2DurableSubscriber.removeDurableSubscriber();

    subscriberPublishSubscribe.closeConnection();
    subscriber1DurableSubscriber.closeConnection();
    subscriber2DurableSubscriber.closeConnection();
  }

  @Test
  public void testGetGreeting() {
    try {
      publisherPublishSubscribe.sendName("Peregrin", "Took");

      String greeting1 = subscriberPublishSubscribe.getGreeting(1000);
      assertEquals("Hello Peregrin Took!", greeting1);

      String greeting2 = subscriberPublishSubscribe.getGreeting(1000);
      assertEquals("no greeting", greeting2);

    } catch (JMSException e) {
      fail("a JMS Exception occurred");
    }
  }

  @Test
  public void testDurableSubscriber() {
    try {
      // durable subscriptions, receive messages sent while the
      // subscribers are not active
      subscriber2DurableSubscriber.closeConnection();

      publisherDurableSubscriber.sendName("Bilbo", "Baggins");

      // recreate a connection for the durable subscription
      subscriber2DurableSubscriber.create(
          "subscriber2-durablesubscriber", "durablesubscriber.t",
          "durablesubscriber2");

      publisherDurableSubscriber.sendName("Frodo", "Baggins");

      String greeting1 =
          subscriber1DurableSubscriber.getGreeting(1000);
      assertEquals("Hello Bilbo Baggins!", greeting1);
      String greeting2 =
          subscriber2DurableSubscriber.getGreeting(1000);
      assertEquals("Hello Bilbo Baggins!", greeting2);

      String greeting3 =
          subscriber1DurableSubscriber.getGreeting(1000);
      assertEquals("Hello Frodo Baggins!", greeting3);
      String greeting4 =
          subscriber2DurableSubscriber.getGreeting(1000);
      assertEquals("Hello Frodo Baggins!", greeting4);

    } catch (JMSException e) {
      fail("a JMS Exception occurred");
    }
  }
}
```

Make sure a default [ActiveMQ message broker is up and running]({{ site.url }}/jms-apache-activemq-installation.html), open a command prompt and execute following Maven command:

``` plaintext
mvn -Dtest=DurableSubscriberTest test
```

This will trigger Maven to run the above test cases which should result in the following log statements.

``` plaintext
18:58:54.591 DEBUG [main][Publisher] publisher-durablesubscriber: sent message with text='Bilbo Baggins'
18:58:54.632 DEBUG [main][Publisher] publisher-durablesubscriber: sent message with text='Frodo Baggins'
18:58:54.633 DEBUG [main][DurableSubscriber] subscriber1-durablesubscriber:  received message with text='Bilbo Baggins'
18:58:54.634 INFO  [main][DurableSubscriber] greeting=Hello Bilbo Baggins!
18:58:54.635 DEBUG [main][DurableSubscriber] subscriber2-durablesubscriber:  received message with text='Bilbo Baggins'
18:58:54.635 INFO  [main][DurableSubscriber] greeting=Hello Bilbo Baggins!
18:58:54.636 DEBUG [main][DurableSubscriber] subscriber1-durablesubscriber:  received message with text='Frodo Baggins'
18:58:54.636 INFO  [main][DurableSubscriber] greeting=Hello Frodo Baggins!
18:58:54.636 DEBUG [main][DurableSubscriber] subscriber2-durablesubscriber:  received message with text='Frodo Baggins'
18:58:54.637 INFO  [main][DurableSubscriber] greeting=Hello Frodo Baggins!
18:58:54.669 DEBUG [main][Publisher] publisher-publishsubscribe: sent message with text='Peregrin Took'
18:58:54.670 DEBUG [main][DurableSubscriber] subscriber-publishsubscribe:  received message with text='Peregrin Took'
18:58:54.670 INFO  [main][DurableSubscriber] greeting=Hello Peregrin Took!
18:58:55.670 DEBUG [main][DurableSubscriber] subscriber-publishsubscribe: no message received
18:58:55.670 INFO  [main][DurableSubscriber] greeting=no greeting
```

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/jms/tree/master/jms-activemq-publish-subscribe){:target="_blank"}.
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This concludes the JMS publish/subscribe example using ActiveMQ. If you found this post helpful or have any questions or remarks, please leave a comment.
