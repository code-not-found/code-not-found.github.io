---
title: JMS - Hello World using ActiveMQ 
permalink: /2014/10/jms-hello-world-activemq-maven.html
excerpt: A JMS Hello World example using ActiveMQ and Maven.
date: 2014-10-16 21:00
categories: [JMS]
tags: [ActiveMQ, Apache ActiveMQ, Example, Hello World, Java Message Service, JMS, Maven]
redirect_from:
  - /2014/10/jms-hello-world-using-activemq.html
  - /2014/04/jms-spring-hello-world-using-activemq.html
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/jms-logo.png" alt="jms logo">
</figure>

The Java Message Service (JMS) API is a Java Message Oriented Middleware (MOM) API for sending messages between two or more clients. It is a Java API that allows applications to create, send, receive, and read messages. The JMS API enables communication that is loosely coupled, asynchronous and reliable. The current version of the JMS specification is version 1.1. The following post introduces the basic JMS concepts and illustrates them with a JMS Hello World example using ActiveMQ and Maven.

To use JMS, one must have a JMS provider that can manage the sessions, queues and topics. Some examples of known JMS providers are [Apache ActiveMQ](http://activemq.apache.org/), WebSphere MQ from IBM or SonicMQ from Aurea Software. Starting from Java EE version 1.4, a JMS provider has to be contained in all Java EE application servers.

# The JMS API Programming Model

<figure>
    <img src="{{ site.url }}/assets/images/jms/jms-api-programming-model.png" alt="jms api programming model">
</figure>

The basic building blocks of the JMS API programming model are shown above. At the top we have the `ConnectionFactory` object which is the object a client uses to create a connection to a JMS provider. A connection factory encapsulates a set of connection configuration parameters like for example the broker URL. A connection factory is a JMS administered object that is typically created by an administrator and later used by JMS clients.

When you have a `ConnectionFactory` object, you can use it to create a connection. A `Connection` object encapsulates a virtual connection with a JMS provider. For example, a connection could represent an open TCP/IP socket between a client and a provider service daemon. Before an application completes, it must close any connections that were created. Failure to close a connection can cause resources not to be released by the JMS provider.

> Closing a connection also closes its sessions and their message producers/message consumers.

A session is a single-threaded context for producing and consuming messages. A session provides a transactional context with which to group a set of sends and receives into an atomic unit of work. `Session` objects are created on top of connections.

> As mentioned above, it is important to note that everything from a session down is single-threaded!

A `MessageProducer` is an object that is created by a session and used for sending messages to a destination. You use a `Session` object to create a message producer for a destination.

> It is possible to create an unidentified producer by specifying a null `Destination` as argument to the `createProducer()` method. When sending a message, overload the send method with the needed destination as the first parameter.

A `MessageConsumer` is an object that is created by a session and used for receiving messages sent to a destination. After you have created a message consumer it becomes active, and you can use it to receive messages. Message delivery does not begin until you start the connection you created by calling its `start()` method.

> Remember to always to call the `start()` method on the `Connection` object in order to receive messages!

A `Destination` is the object a client uses to specify the target of messages it produces and the source of messages it consumes. In the point-to-point messaging domain, destinations are called queues. In the publish/subscribe messaging domain, destinations are called topics.

For more detailed information please check the [JMS API programming model chapter](http://docs.oracle.com/javaee/6/tutorial/doc/bnceh.html) of the [Java EE 6 tutorial](http://docs.oracle.com/javaee/6/tutorial/doc/index.html).

# ActiveMQ Example

Let's illustrate the above by creating a message producer that sends a message containing a first and last name to a Hello World queue. In turn a message consumer will read the message and transform it into a greeting. The example uses Maven and assumes a default ActiveMQ message broker is up and running.

Tools used:
* ActiveMQ 5.14
* Maven 3

First let's look at the below Maven POM file which contains the needed dependencies for Logback, JUnit and ActiveMQ.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.codenotfound</groupId>
    <artifactId>jms-activemq-helloworld</artifactId>
    <version>1.0</version>
    <packaging>jar</packaging>

    <name>JMS - Hello World using ActiveMQ</name>
    <url>https://codenotfound.com/2014/10/jms-hello-world-using-activemq.html</url>

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

The `Producer` class below illustrates how to use the JMS API programming model. The `create()` method will first create an instance of the `ConnectionFactory` which is in turn used to create a connection to ActiveMQ using the default broker URL. Using the `Connection` instance, a `Session` is created which is used to create a `Destination` and `MessageProducer` instance. The destination type in the below example is a queue.

The class also contains a `close()` method which allows to correctly release the resources at the JMS provider. The depending `Session` and `MessageProducer` objects are automatically closed when calling this method.

The `sendName()` method takes as input a first and last name and concatenates them into a single string. Using the session a JMS `TextMessage` is created on which the string is set. Using the message producer the message is sent to the JMS provider.

``` java
package com.codenotfound.jms;

import javax.jms.Connection;
import javax.jms.ConnectionFactory;
import javax.jms.Destination;
import javax.jms.JMSException;
import javax.jms.MessageProducer;
import javax.jms.Session;
import javax.jms.TextMessage;

import org.apache.activemq.ActiveMQConnection;
import org.apache.activemq.ActiveMQConnectionFactory;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class Producer {

    private static final Logger LOGGER = LoggerFactory
            .getLogger(Producer.class);

    private Connection connection;
    private Session session;
    private MessageProducer messageProducer;

    public void create(String destinationName) throws JMSException {

        // create a Connection Factory
        ConnectionFactory connectionFactory = new ActiveMQConnectionFactory(
                ActiveMQConnection.DEFAULT_BROKER_URL);

        // create a Connection
        connection = connectionFactory.createConnection();

        // create a Session
        session = connection.createSession(false,
                Session.AUTO_ACKNOWLEDGE);

        // create the Destination to which messages will be sent
        Destination destination = session.createQueue(destinationName);

        // create a Message Producer for sending messages
        messageProducer = session.createProducer(destination);
    }

    public void close() throws JMSException {
        connection.close();
    }

    public void sendName(String firstName, String lastName)
            throws JMSException {

        String text = firstName + " " + lastName;

        // create a JMS TextMessage
        TextMessage textMessage = session.createTextMessage(text);

        // send the message to the queue destination
        messageProducer.send(textMessage);

        LOGGER.debug("producer sent message with text='{}'", text);
    }
}
```

For receiving messages, a `Consumer` class is defined which has the same `create()` and `close()` methods as the above `Producer`. The main difference is that in the case of a message consumer the connection is started.

The `getGreeting()` method will receive the next message from the destination that was configured on the message consumer. A timeout parameter is passed to the `receive()` method in order to avoid waiting for an indefinite amount of time in case no message is present on the destination. As a result a check is needed to see if the `receive()` method returned a message or null.

If a message was received it is cast to a `TextMessage` and the received text is converted into a greeting that is returned. In case the message was `null` a default <var>"no greeting"</var> is returned.

``` java
package com.codenotfound.jms;

import javax.jms.Connection;
import javax.jms.ConnectionFactory;
import javax.jms.Destination;
import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.MessageConsumer;
import javax.jms.Session;
import javax.jms.TextMessage;

import org.apache.activemq.ActiveMQConnection;
import org.apache.activemq.ActiveMQConnectionFactory;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class Consumer {

    private static final Logger LOGGER = LoggerFactory
            .getLogger(Consumer.class);

    private static String NO_GREETING = "no greeting";

    private Connection connection;
    private Session session;
    private MessageConsumer messageConsumer;

    public void create(String destinationName) throws JMSException {

        // create a Connection Factory
        ConnectionFactory connectionFactory = new ActiveMQConnectionFactory(
                ActiveMQConnection.DEFAULT_BROKER_URL);

        // create a Connection
        connection = connectionFactory.createConnection();

        // create a Session
        session = connection.createSession(false,
                Session.AUTO_ACKNOWLEDGE);

        // create the Destination from which messages will be received
        Destination destination = session.createQueue(destinationName);

        // create a Message Consumer for receiving messages
        messageConsumer = session.createConsumer(destination);

        // start the connection in order to receive messages
        connection.start();
    }

    public void close() throws JMSException {
        connection.close();
    }

    public String getGreeting(int timeout) throws JMSException {

        String greeting = NO_GREETING;

        // read a message from the queue destination
        Message message = messageConsumer.receive(timeout);

        // check if a message was received
        if (message != null) {
            // cast the message to the correct type
            TextMessage textMessage = (TextMessage) message;

            // retrieve the message content
            String text = textMessage.getText();
            LOGGER.debug("consumer received message with text='{}'",
                    text);

            // create greeting
            greeting = "Hello " + text + "!";
        } else {
            LOGGER.debug("consumer received no message");
        }

        LOGGER.info("greeting={}", greeting);
        return greeting;
    }
}
```

In order to test above classes, below JUnit test class is created which contains two test cases. The first is a `testGetGreeting()` test case in which the producer is used to send a first and last name. Using the consumer the sent message is read and converted into a greeting. The second `testNoGreeting()` test case verifies the correct working of reading a destination when it contains no messages and as such <var>"no greeting"</var> is returned.

``` java
package com.codenotfound.jms;

import static org.junit.Assert.assertEquals;
import static org.junit.Assert.fail;

import javax.jms.JMSException;

import org.junit.AfterClass;
import org.junit.BeforeClass;
import org.junit.Test;

public class ProducerTest {

    private static Producer producer;
    private static Consumer consumer;

    @BeforeClass
    public static void setUpBeforeClass() throws JMSException {
        producer = new Producer();
        producer.create("helloworld.q");

        consumer = new Consumer();
        consumer.create("helloworld.q");
    }

    @AfterClass
    public static void tearDownAfterClass() throws JMSException {
        producer.close();
        consumer.close();
    }

    @Test
    public void testGetGreeting() {
        try {
            producer.sendName("John", "Doe");

            String greeting = consumer.getGreeting(1000);
            assertEquals("Hello John Doe!", greeting);

        } catch (JMSException e) {
            fail("a JMS Exception occurred");
        }
    }

    @Test
    public void testNoGreeting() {
        try {
            String greeting = consumer.getGreeting(1000);
            assertEquals("no greeting", greeting);

        } catch (JMSException e) {
            fail("a JMS Exception occurred");
        }
    }
}

```

Make sure a default [ActiveMQ message broker is up and running]({{ site.url }}/2014/01/jms-apache-activemq-installation.html), open a command prompt and execute following Maven command: 

``` plaintext
mvn test
```

This will trigger Maven to run the above test case and will result in the following log statements.

``` plaintext
13:38:19.844 DEBUG [main][Consumer] consumer received no message
13:38:19.858 INFO  [main][Consumer] greeting=no greeting
13:38:19.911 DEBUG [main][Producer] producer sent message with text='John Doe'
13:38:19.912 DEBUG [main][Consumer] consumer received message with text='John Doe'
13:38:19.912 INFO  [main][Consumer] greeting=Hello John Doe!
Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 1.579 sec
```

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/jms/tree/master/jms-activemq-helloworld).
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This concludes the JMS Hello World example using ActiveMQ. Note that the code also contains a `UnidentifiedProducer` class and corresponding JUnit test class which illustrates the overloading of the `send()` method with the needed destination. If you found this post helpful or have any questions or remarks, please leave a comment.