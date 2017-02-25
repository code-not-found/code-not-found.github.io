---
title: JMS - Publish/Subscribe messaging example using ActiveMQ & Maven
permalink: /2014/11/jms-publish-subscribe-messaging-example-activemq-maven.html
excerpt: A JMS Publish/Subscribe messaging example using ActiveMQ and Maven.
date: 2014-11-26 21:00
categories: [JMS]
tags: [ActiveMQ, Apache ActiveMQ, durable subscription, Example, Java Message Service, JMS, publish, subscribe]
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/jms-logo.png" alt="jms logo">
</figure>

In a publish/subscribe (pub/sub) product or application, clients address messages to a topic, which functions somewhat like a bulletin board. Subscribers can receive information, in the form of messages, from publishers. Topics retain messages only as long as it takes to distribute them to current subscribers. The following post introduces the basic concepts of JMS point-to-point messaging and illustrates them with a code sample using ActiveMQ and Maven.

# Publish/Subscribe Messaging

<figure>
    <img src="{{ site.url }}/assets/images/jms/jms-publish-subscribe-messaging.png" alt="jms publish subscribe messaging">
</figure>

Pub/sub messaging has the following characteristics:
* Each message can have **multiple consumers**.
* Publishers and subscribers have a **timing dependency**. A client that subscribes to a topic can consume only messages published after the client has created a subscription, and the subscriber must continue to be active in order for it to consume messages.

The JMS API relaxes this timing dependency mentioned in the second bullet to some extent by allowing subscribers to create durable subscriptions, which receive messages sent while the subscribers are not active. **Durable subscriptions** provide the flexibility and reliability of queues but still allow clients to send messages to many recipients.

# ActiveMQ Example

Let's illustrate the above characteristics by creating a message producer that sends a message containing a first and last name to a topic. In turn a message consumer will read the message and transform it into a greeting. The code is very similar to the [JMS Hello World example]({{ site.url }}/2014/10/jms-hello-world-activemq-maven.html) but contains a few key differences explained below.

Tools used:
* ActiveMQ 5.14
* Maven 3


The code is built and run using Maven. Specified below is the Maven POM file which contains the needed dependencies for Logback, JUnit and ActiveMQ.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.codenotfound</groupId>
    <artifactId>jms-activemq-publish-subscribe</artifactId>
    <version>1.0</version>
    <packaging>jar</packaging>

    <name>JMS - Publish/Subscribe messaging using ActiveMQ</name>
    <url>https://codenotfound.com/2014/11/jms-publish-subscribe-messaging-example-activemq-maven.html</url>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <java.version>1.8</java.version>

        <logback.version>1.2.1</logback.version>
        <slf4j.version>1.7.24</slf4j.version>
        <junit.version>4.12</junit.version>
        <activemq.version>5.14.3</activemq.version>

        <maven-compiler-plugin.version>3.1</maven-compiler-plugin.version>
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

    private static final Logger LOGGER = LoggerFactory
            .getLogger(Publisher.class);

    private String clientId;
    private Connection connection;
    private Session session;
    private MessageProducer messageProducer;

    public void create(String clientId, String topicName) throws JMSException {
        this.clientId = clientId;

        // create a Connection Factory
        ConnectionFactory connectionFactory = new ActiveMQConnectionFactory(
                ActiveMQConnection.DEFAULT_BROKER_URL);

        // create a Connection
        connection = connectionFactory.createConnection();
        connection.setClientID(clientId);

        // create a Session
        session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);

        // create the Topic to which messages will be sent
        Topic topic = session.createTopic(topicName);

        // create a MessageProducer for sending messages
        messageProducer = session.createProducer(topic);
    }

    public void closeConnection() throws JMSException {
        connection.close();
    }

    public void sendName(String firstName, String lastName) throws JMSException {
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

    private static final Logger LOGGER = LoggerFactory
            .getLogger(Subscriber.class);

    private static final String NO_GREETING = "no greeting";

    private String clientId;
    private Connection connection;
    private Session session;
    private MessageConsumer messageConsumer;

    public void create(String clientId, String topicName) throws JMSException {
        this.clientId = clientId;

        // create a Connection Factory
        ConnectionFactory connectionFactory = new ActiveMQConnectionFactory(
                ActiveMQConnection.DEFAULT_BROKER_URL);

        // create a Connection
        connection = connectionFactory.createConnection();
        connection.setClientID(clientId);

        // create a Session
        session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);

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
            LOGGER.debug(clientId + ": received message with text='{}'", text);

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
            subscriber1NonDurableSubscriber, subscriber2NonDurableSubscriber;

    @BeforeClass
    public static void setUpBeforeClass() throws Exception {
        publisherPublishSubscribe = new Publisher();
        publisherPublishSubscribe.create("publisher-publishsubscribe",
                "publishsubscribe.t");

        publisherMultipleConsumers = new Publisher();
        publisherMultipleConsumers.create("publisher-multipleconsumers",
                "multipleconsumers.t");

        publisherNonDurableSubscriber = new Publisher();
        publisherNonDurableSubscriber.create("publisher-nondurablesubscriber",
                "nondurablesubscriber.t");

        subscriberPublishSubscribe = new Subscriber();
        subscriberPublishSubscribe.create("subscriber-publishsubscribe",
                "publishsubscribe.t");

        subscriber1MultipleConsumers = new Subscriber();
        subscriber1MultipleConsumers.create("subscriber1-multipleconsumers",
                "multipleconsumers.t");

        subscriber2MultipleConsumers = new Subscriber();
        subscriber2MultipleConsumers.create("subscriber2-multipleconsumers",
                "multipleconsumers.t");

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

            String greeting1 = subscriber1MultipleConsumers.getGreeting(1000);
            assertEquals("Hello Gandalf the Grey!", greeting1);

            String greeting2 = subscriber2MultipleConsumers.getGreeting(1000);
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

            String greeting1 = subscriber1NonDurableSubscriber
                    .getGreeting(1000);
            assertEquals("Hello Bilbo Baggins!", greeting1);
            String greeting2 = subscriber1NonDurableSubscriber
                    .getGreeting(1000);
            assertEquals("Hello Frodo Baggins!", greeting2);

            String greeting3 = subscriber2NonDurableSubscriber
                    .getGreeting(1000);
            assertEquals("Hello Frodo Baggins!", greeting3);
            String greeting4 = subscriber2NonDurableSubscriber
                    .getGreeting(1000);
            assertEquals("no greeting", greeting4);

        } catch (JMSException e) {
            fail("a JMS Exception occurred");
        }
    }
}
```

Make sure a default [ActiveMQ message broker is up and running]({{ site.url }}/2014/01/jms-apache-activemq-installation.html), open a command prompt and execute following Maven command: 

``` plaintext
mvn -Dtest=SubscriberTest test
```





















