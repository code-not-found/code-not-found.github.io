---
title: "JMS - ActiveMQ Message Priority Example"
permalink: /jms-activemq-message-priority-example.html
excerpt: "A detailed example using ActiveMQ that shows how to specify a priority level when sending a JMS message to a queue."
date: 2014-01-18
last_modified_at: 2014-01-18
header:
  teaser: "assets/images/teaser/jms-teaser.png"
categories: [JMS]
tags: [Apache ActiveMQ, ActiveMQ, Example, Java Message Service, JMS, Level, Message, Priority, Queue]
redirect_from:
  - /2014/01/jms-priority-activemq-maven.html
  - /2014/01/jms-priority-using-activemq.html
  - /2014/01/jms-activemq-message-priority-example.html
---

<figure>
    <img src="{{ site.url }}/assets/images/logo/jms-logo.png" alt="jms logo" class="logo">
</figure>

Priority levels are a powerful instrument on JMS messages which allow building robust applications where for example peak traffic will not block important messages (set with a higher priority) from getting through the queue.

The following post explains the basics of JMS priority and illustrates them with a code sample using ActiveMQ and Maven.

# Setting JMS Message Priority Levels

Message priority levels can be used to instruct the JMS provider to deliver urgent messages first. The message's priority is contained in the [JMSPriority header]({{ site.url }}/jms-message-types-properties-overview.html). There are ten levels of priority, ranging from 0 (lowest) to 9 (highest). If you do not specify a priority level, the default level is set to 4.

You can set the priority level in either of two ways:
* You can use the `setPriority()` method of the `MessageProducer` interface to set the priority level for all messages sent by that producer. For example, the following call sets a priority level of '7' for a producer:

``` java
producer.setPriority(7);
```

* You can use the long form of the `send()` or the `publish()` method to set the priority level for a specific message. The third argument sets the priority level. For example, the following send call sets the priority level for message to '3':

``` java
producer.send(message, DeliveryMode.NON_PERSISTENT, 3, 0);
```

> Setting the priority directly on the JMS Message using the `setJMSPriority()` method of the `Message` interface does not work as in that case the priority of the producer is taken!

It is very interesting to have a look at what the [Java Message Service 1.1 Specification](http://www.oracle.com/technetwork/java/docs-136352.html){:target="_blank"} has to say about priority and the different levels as it gives a better understanding of what can be achieved when used:

``` plaintext
JMS defines a ten-level priority value, with 0 as the lowest priority and 9 as 
the highest. In addition, clients should consider priorities 0-4 as gradations 
of normal priority and priorities 5-9 as gradations of expedited priority.

JMS does not require that a provider strictly implement priority ordering of 
messages; however, it should do its best to deliver expedited messages ahead of 
normal messages.
```

> When implementing JMS priority it is important to realize that correct configuration of the JMS provider and consumers/producers is key in getting higher-priority messages delivered before lower-priority ones. It is also important to note that **the JMS specification does not mandate a provider to implement a strict priority ordering of messages**.

For example [on ActiveMQ, there are a number of settings that need to be made in order to support message priority](http://activemq.apache.org/how-can-i-support-priority-queues.html). A typical example is lowering the consumer prefetch to 1 to ensure getting the high priority messages from the store ahead of lower priority messages. However this sort of tradeoff can have significant performance implications, so always test your scenarios thoroughly.

Checkout following link if you would like to [know more on how ActiveMQ message priorities work](http://blog.christianposta.com/activemq/activemq-message-priorities-how-it-works/){:target="_blank"}.

# ActiveMQ Message Priority Example

The below example uses Maven and assumes an ActiveMQ message broker is installed and up and running.

Tools used:
* ActiveMQ 5.15
* Maven 3.5

Let's illustrate JMS priority by creating a simple producer with two different `send()` methods. The first method will send a message to a queue with the default priority level and the second method will accept an additional parameter specifying the priority to be set on the message. We will then create a consumer to read the messages from the queue and observe in which order they are read.

First let's look at the below [Maven](https://maven.apache.org/){:target="_blank"} POM file which contains the needed dependencies for Logback, JUnit and [Apache ActiveMQ](http://activemq.apache.org/){:target="_blank"}.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>jms-activemq-priority</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>JMS - ActiveMQ Message Priority Example</name>
  <description>JMS - ActiveMQ Message Priority Example</description>
  <url>https://www.codenotfound.com/jms-activemq-message-priority-example.html</url>

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
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-api</artifactId>
      <version>${slf4j.version}</version>
    </dependency>
    <dependency>
      <groupId>ch.qos.logback</groupId>
      <artifactId>logback-classic</artifactId>
      <version>${ch.qos.logback.version}</version>
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

Next is the `Producer` class which contains the two `send()` methods: one which applies default priority and one which applies a custom priority. The class also contains two methods for opening/closing a connection to the message broker as well as a method for creating the message producer.

``` java
package com.codenotfound.jms.priority;

import javax.jms.Connection;
import javax.jms.ConnectionFactory;
import javax.jms.DeliveryMode;
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

  private static final Logger LOGGER =
      LoggerFactory.getLogger(Producer.class);

  private Connection connection;
  private Session session;
  private MessageProducer producer;

  public void openConnection() throws JMSException {
    // Create a new connection factory
    ConnectionFactory connectionFactory =
        new ActiveMQConnectionFactory(
            ActiveMQConnection.DEFAULT_BROKER_URL);
    connection = connectionFactory.createConnection();
  }

  public void closeConnection() throws JMSException {
    connection.close();
  }

  public void createProducer(String queue) throws JMSException {
    // Create a session for sending messages
    session =
        connection.createSession(false, Session.AUTO_ACKNOWLEDGE);

    // Create the destination to which messages will be sent
    Destination destination = session.createQueue(queue);

    // Create a MessageProducer for sending the messages
    producer = session.createProducer(destination);
  }

  public void send(String text) throws JMSException {
    TextMessage message = session.createTextMessage(text);
    producer.send(message);

    LOGGER.info("{} sent with default priority(=4)", text);
  }

  public void send(String text, int priority) throws JMSException {
    TextMessage message = session.createTextMessage(text);
    // Note: setting the priority directly on the JMS Message does not work
    // as in that case the priority of the producer is taken
    producer.send(message, DeliveryMode.PERSISTENT, priority, 0);

    LOGGER.info("{} sent with priority={}", text, priority);
  }
}
```

For receiving the messages, a `Consumer` class is created with two methods for opening/closing a connection to the message broker. In addition a method to create a message consumer for a specific queue and a method to receive a single message are available on the class.

> Note that instead of using the default connection to the ActiveMQ broker, we have specified the <var>'messagePrioritySupported'</var> property in order to assure the message are prioritized on consumer side.

``` java
package com.codenotfound.jms.priority;

import javax.jms.Connection;
import javax.jms.ConnectionFactory;
import javax.jms.Destination;
import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.MessageConsumer;
import javax.jms.Session;
import javax.jms.TextMessage;

import org.apache.activemq.ActiveMQConnectionFactory;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class Consumer {

  private static final Logger LOGGER =
      LoggerFactory.getLogger(Consumer.class);

  private Connection connection;
  private Session session;
  private MessageConsumer consumer;

  public void openConnection() throws JMSException {
    // Create a new connection factory
    ConnectionFactory connectionFactory =
        new ActiveMQConnectionFactory(
            "tcp://0.0.0.0:61616?jms.messagePrioritySupported=true");
    connection = connectionFactory.createConnection();
  }

  public void closeConnection() throws JMSException {
    connection.close();
  }

  public void createConsumer(String queue) throws JMSException {
    // Create a session for receiving messages
    session =
        connection.createSession(false, Session.AUTO_ACKNOWLEDGE);

    // Create the destination from which messages will be received
    Destination destination = session.createQueue(queue);

    // Create a MessageConsumer for receiving messages
    consumer = session.createConsumer(destination);

    // Start the connection in order to receive messages
    connection.start();
  }

  public String receive(int timeout) throws JMSException {
    // Read a message from the destination
    Message message = consumer.receive(timeout);

    // Cast the message to the correct type
    TextMessage input = (TextMessage) message;

    // Retrieve the message content
    String text = input.getText();
    LOGGER.info("{} received", text);

    return text;
  }
}
```

Last, a JUnit test class in which a first `testSend()` test case will send three messages with default priority to a <var>'priority.q'</var> queue. It then verifies if the messages are read in first in, first out (FIFO) order.

A second `testSendWithPriority()` test case which will send three messages with custom priorities where the last message gets the highest priority. In turn it verifies if the message are read in last in, first out (LIFO) order.

``` java
package com.codenotfound.jms.priority;

import static org.junit.Assert.assertEquals;

import javax.jms.JMSException;

import org.junit.AfterClass;
import org.junit.BeforeClass;
import org.junit.Test;

public class ProducerTest {

  private static Consumer consumer;
  private static Producer producer;

  private static String QUEUE = "priority.q";

  @BeforeClass
  public static void setUpBeforeClass() throws JMSException {
    producer = new Producer();
    producer.openConnection();
    producer.createProducer(QUEUE);

    consumer = new Consumer();
    consumer.openConnection();
    consumer.createConsumer(QUEUE);
  }

  @AfterClass
  public static void tearDownAfterClass() throws JMSException {
    producer.closeConnection();
    consumer.closeConnection();
  }

  @Test
  public void testSend() throws JMSException, InterruptedException {
    producer.send("message1");
    producer.send("message2");
    producer.send("message3");

    Thread.sleep(10000);

    // Messages should be received FIFO as priority is the same for all
    assertEquals("message1", consumer.receive(5000));
    assertEquals("message2", consumer.receive(5000));
    assertEquals("message3", consumer.receive(5000));
  }

  @Test
  public void testSendWithPriority()
      throws JMSException, InterruptedException {
    producer.send("message1", 1);
    producer.send("message2", 2);
    producer.send("message3", 3);

    Thread.sleep(10000);

    // Messages should be received LIFO as priority=1 is lower than
    // priority=3
    assertEquals("message3", consumer.receive(5000));
    assertEquals("message2", consumer.receive(5000));
    assertEquals("message1", consumer.receive(5000));
  }
}
```

Make sure an [ActiveMQ message broker is up and running]({{ site.url }}/jms-apache-activemq-installation.html), open a command prompt and execute following Maven command:

``` plaintext
mvn test
```

This will trigger Maven to run the above test case and results in the following log statements. Even though <var>'message1'</var> was sent first in both test cases, in the first test it is received first whereas in the second test it is received last because of the different assigned priority.

``` plaintext
-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running com.codenotfound.jms.priority.ProducerTest
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/C:/code/local-repo/ch/qos/logback/logback-classic/1.2.3/logback-classic-1.2.3.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/C:/code/local-repo/org/apache/activemq/activemq-all/5.15.2/activemq-all-5.15.2.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [ch.qos.logback.classic.util.ContextSelectorStaticBinder]
21:50:34.705 INFO  [main][Producer] message1 sent with default priority(=4)
21:50:34.715 INFO  [main][Producer] message2 sent with default priority(=4)
21:50:34.719 INFO  [main][Producer] message3 sent with default priority(=4)
21:50:44.721 INFO  [main][Consumer] message1 received
21:50:44.722 INFO  [main][Consumer] message2 received
21:50:44.723 INFO  [main][Consumer] message3 received
21:50:44.736 INFO  [main][Producer] message1 sent with priority=1
21:50:44.742 INFO  [main][Producer] message2 sent with priority=2
21:50:44.748 INFO  [main][Producer] message3 sent with priority=3
21:50:54.749 INFO  [main][Consumer] message3 received
21:50:54.750 INFO  [main][Consumer] message2 received
21:50:54.750 INFO  [main][Consumer] message1 received
Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 20.622 sec

Results :

Tests run: 2, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 22.719 s
[INFO] Finished at: 2018-01-04T21:50:54+01:00
[INFO] Final Memory: 19M/217M
[INFO] ------------------------------------------------------------------------
```

By browsing the <var>'priority.q'</var> using the ActiveMQ console we can verify the `JMSPriority` header that was set on the messages sent by the above test cases. The `testSend()` test case will create three messages a shown below, each with priority set to 4.

<figure>
    <img src="{{ site.url }}/assets/images/posts/jms/apache-activemq-message-browser-no-priority.png" alt="apache activemq message browser no priority">
</figure>

The second `testSendWithPriority()` test case results in three messages, each with a different priority ranging from 1 till 3.

<figure>
    <img src="{{ site.url }}/assets/images/posts/jms/apache-activemq-message-browser-with-priority.png" alt="apache activemq message browser with priority">
</figure>

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/jms/tree/master/jms-activemq-priority){:target="_blank"}.
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This concludes the example on how to use JMS message priority using ActiveMQ. If you found this post helpful or have any questions or remarks, please leave a comment.
