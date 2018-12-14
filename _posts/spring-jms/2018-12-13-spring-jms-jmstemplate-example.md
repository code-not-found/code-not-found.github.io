---
title: "Spring JMS JmsTemplate Example"
permalink: /spring-jms-jmstemplate-example.html
excerpt: "A detailed step-by-step tutorial on how to use JmsTemplate in combination with Spring JMS and Spring Boot."
date: 2018-12-13
last_modified_at: 2018-12-14
header:
  teaser: "assets/images/spring-jms/spring-jms-jmstemplate.png"
categories: [Spring JMS]
tags: [ActiveMQ, Apache ActiveMQ, Example, Maven, JmsTemplate, Spring, Spring Boot, Spring JMS, Tutorial]
published: true
---

<img src="{{ site.url }}/assets/images/spring-jms/spring-jms-jmstemplate.png" alt="spring jms jmstemplate" class="align-right title-image">

Today I'm going to show you **exactly** how to use the [Spring JmsTemplate](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/jms/core/JmsTemplate.html){:target="_blank"}.

The best part?

You're going to see a detailed example to get you up and running in record time.

So without further ado, let's get started…

If you want to learn more about Spring JMS - head on over to the [Spring JMS tutorials]({{ site.url }}/spring-jms-tutorials) page.
{: .notice--primary}

## 1. What is Spring JmsTemplate?

The [JmsTemplate](https://docs.spring.io/spring/docs/5.1.3.RELEASE/spring-framework-reference/integration.html#jms-jmstemplate){:target="_blank"} is a central class from the Spring core package.

It simplifies the use of [JMS](https://en.wikipedia.org/wiki/Java_Message_Service){:target="_blank"} and gets rid of boilerplate code. It handles the creation and release of JMS resources when sending or receiving messages.

Let's create a code sample that shows how to configure the Spring `JmsTemplate`. We will send an order message to an <var>order</var> queue and then synchronously receive a status message from a <var>status</var> queue.

We start from a previous [Spring Jms example with ActiveMQ]({{ site.url }}/spring-jms-activemq-example.html).

## 2. General Project Overview

We will use the following tools/frameworks:
* Spring JMS 5.1
* Spring Boot 2.1
* ActiveMQ 5.14
* Maven 3.5

Our project has the following directory structure:

<img src="{{ site.url }}/assets/images/spring-jms/spring-jms-jmstemplate-maven-project.png" alt="spring jms jmstemplate maven project">

## 3. Configuring the JmsTemplate

The `JmsTemplate` was [originally designed](https://bsnyderblog.blogspot.com/2010/02/using-spring-jmstemplate-to-send-jms.html){:target="_blank"} to be used with a [J2EE container](https://docs.oracle.com/cd/E17802_01/j2ee/j2ee/1.4/docs/tutorial-update6/doc/Overview3.html){:target="_blank"}. It was the responsibility of the container to provide the necessary pooling of the JMS resources (connections, sessions, consumers and producers).

With the development of frameworks like [Spring Boot](https://spring.io/projects/spring-boot){:target="_blank"} a different solution was needed. As caching of JMS resources was no longer handled by the container.

This is where the [CachingConnectionFactory](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/jms/connection/CachingConnectionFactory.html){:target="_blank"} comes into play. It wraps a JMS provider's connection to provide caching of sessions, connections, and producers. It also handles automatic connection recovery.

By default, it uses a single session to create many connections. If you need to scale further, you can also specify the number of sessions to cache using the `sessionCacheSize` property.

Configuring a `CachingConnectionFactory` is quite simple. Pass a `ConnectionFactory` and don't forget to set the `sessionCacheSize` if you need to scale.

The `JmsTemplate` requires a reference to a ConnectionFactory. In this example, we pass the `CachingConnectionFactory`.

> If you run on a [J2EE runtime](https://en.wikipedia.org/wiki/Java_Platform,_Enterprise_Edition#Certified_referencing_runtimes){:target="_blank"} obtain the `ConnectionFactory` from the application's environment naming context via JNDI.

There are a number of items that are set by default:
* Dynamic destination creation is set to <var>queues</var>.
* JMS Sessions are "not transacted" and "auto-acknowledged".
* The template uses a `DynamicDestinationResolver` and a [SimpleMessageConverter](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/jms/support/converter/SimpleMessageConverter.html){:target="_blank"}.

In addition, we set the `defaultDestination` to which messages should be sent. And as we will also receive messages using the `JmsTemplate` we also set the `receiveTimeout`.

For more information check the [JmsTemplate documentation](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/jms/core/JmsTemplate.html){:target="_blank"}.

> If you use [Spring JMS autoconfiguration]({{ site.url }}/spring-jms-boot-configuration-example.html) you can use the [Spring Boot JMS application properties](https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html){:target="_blank"} (# JMS section) to set the above options.

{% highlight java %}
package com.codenotfound.jms;

import javax.jms.Destination;
import org.apache.activemq.ActiveMQConnectionFactory;
import org.apache.activemq.command.ActiveMQQueue;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.jms.connection.CachingConnectionFactory;
import org.springframework.jms.core.JmsTemplate;

@Configuration
public class SenderConfig {

  @Value("${activemq.broker-url}")
  private String brokerUrl;

  @Value("${destination.order}")
  private String orderDestination;

  @Value("${destination.status}")
  private String statusDestination;

  @Bean
  public ActiveMQConnectionFactory senderConnectionFactory() {
    ActiveMQConnectionFactory activeMQConnectionFactory =
        new ActiveMQConnectionFactory();
    activeMQConnectionFactory.setBrokerURL(brokerUrl);

    return activeMQConnectionFactory;
  }

  @Bean
  public CachingConnectionFactory cachingConnectionFactory() {
    CachingConnectionFactory cachingConnectionFactory =
        new CachingConnectionFactory(senderConnectionFactory());
    cachingConnectionFactory.setSessionCacheSize(10);

    return cachingConnectionFactory;
  }

  @Bean
  public Destination orderDestination() {
    return new ActiveMQQueue(orderDestination);
  }

  @Bean
  public Destination statusDestination() {
    return new ActiveMQQueue(statusDestination);
  }

  @Bean
  public JmsTemplate jmsTemplate() {
    JmsTemplate jmsTemplate =
        new JmsTemplate(cachingConnectionFactory());
    jmsTemplate.setDefaultDestination(orderDestination());
    jmsTemplate.setReceiveTimeout(5000);

    return jmsTemplate;
  }
}
{% endhighlight %}

## 4. Using JmsTemplate to Send and Receive Messages

Sending messages using the `JmsTemplate` can be done in two ways:
1. _Using send(messageCreator)_: The `MessageCreator` callback interface creates the JMS message.
2. _Using convertAndSend(message, messagePostProcessor)_: The `MessageConverter` assigned to the `JmsTemplate` creates the JMS message. The `MessagePostProcessor` allows for further modification of the message after it has been processed by the converter.

We will use the second approach to send a simple order message.

Create a `Sender` class and auto-wire the `JmsTemplate`.

Define a `sendOrder()` method that takes an `orderNumber` String as parameter. The `convertAndSend()` method on the template will take the String and convert it into a JMS `TextMessage`.

We use a `MessagePostProcessor` to retrieve the <var>JMSMessageID</var>. This is a JMS header field that contains the unique identifier of the message. We will use this value to select the corresponding status message that the `Receiver` will return.

In the `receiveOrderStatus()` method we use the `receiveSelectedAndConvert()` method to synchronously receive a status message. This means that this method will block until it receives the message.

We use a [Spring JMS message selector]({{ site.url }}/spring-jms-message-selector-example.html) to receive the status for the order previously sent.

{% highlight java %}
package com.codenotfound.jms;

import java.util.concurrent.atomic.AtomicReference;
import javax.jms.Destination;
import javax.jms.JMSException;
import javax.jms.Message;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jms.core.JmsTemplate;
import org.springframework.stereotype.Component;

@Component
public class Sender {

  private static final Logger LOGGER =
      LoggerFactory.getLogger(Sender.class);

  @Autowired
  private Destination statusDestination;

  @Autowired
  private JmsTemplate jmsTemplate;

  public String sendOrder(String orderNumber) throws JMSException {
    final AtomicReference<Message> message = new AtomicReference<>();

    jmsTemplate.convertAndSend(orderNumber, messagePostProcessor -> {
      message.set(messagePostProcessor);
      return messagePostProcessor;
    });

    String messageId = message.get().getJMSMessageID();
    LOGGER.info("sending OrderNumber='{}' with MessageId='{}'",
        orderNumber, messageId);

    return messageId;
  }

  public String receiveOrderStatus(String correlationId) {
    String status = (String) jmsTemplate.receiveSelectedAndConvert(
        statusDestination,
        "JMSCorrelationID = '" + correlationId + "'");
    LOGGER.info("receive Status='{}' for CorrelationId='{}'", status,
        correlationId);

    return status;
  }
}
{% endhighlight %}

In the `Receiver` class we configure a `JmsListener` to receive the order message.

We then create a status message on which we apply the [JMS Message ID Pattern](https://docs.oracle.com/cd/E13171_01/alsb/docs25/interopjms/MsgIDPatternforJMS.html#wp1047503){:target="_blank"}. We copy the message ID from the request to the correlation ID of the response.

This time we use the `send()` method on the `JmsTemplate` to send back the status message. In the `MessageCreator` we create a `TextMessage` and set a simple <var>Accepted</var> String as response.

Note that we reuse the same `JmsTemplate` in both `Sender` and `Receiver` class.

> You can configure a single `JmsTemplate` instance and then safely inject this shared reference into multiple collaborators since the template is thread-safe.

{% highlight java %}
package com.codenotfound.jms;

import javax.jms.Destination;
import javax.jms.TextMessage;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jms.annotation.JmsListener;
import org.springframework.jms.core.JmsTemplate;
import org.springframework.jms.support.JmsHeaders;
import org.springframework.messaging.handler.annotation.Header;
import org.springframework.stereotype.Component;

@Component
public class Receiver {

  @Autowired
  private Destination statusDestination;

  @Autowired
  private JmsTemplate jmsTemplate;

  private static final Logger LOGGER =
      LoggerFactory.getLogger(Receiver.class);

  @JmsListener(destination = "${destination.order}")
  public void receiveOrder(String orderNumber,
      @Header(JmsHeaders.MESSAGE_ID) String messageId) {
    LOGGER.info("received OrderNumber='{}' with MessageId='{}'",
        orderNumber, messageId);

    LOGGER.info("sending Status='Accepted' with CorrelationId='{}'",
        messageId);

    jmsTemplate.send(statusDestination, messageCreator -> {
      TextMessage message =
          messageCreator.createTextMessage("Accepted");
      message.setJMSCorrelationID(messageId);
      return message;
    });
  }
}
{% endhighlight %}

## 4. Testing the JmsTemplate

To test the setup we adapt the existing test case.

We use the `Sender` to send out an order message. We then receive the status message and check that it contains the <var>Accepted</var> state.

{% highlight java %}
package com.codenotfound;

import static org.assertj.core.api.Assertions.assertThat;
import org.apache.activemq.junit.EmbeddedActiveMQBroker;
import org.junit.ClassRule;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.annotation.DirtiesContext;
import org.springframework.test.context.junit4.SpringRunner;
import com.codenotfound.jms.Sender;

@RunWith(SpringRunner.class)
@SpringBootTest
@DirtiesContext
public class SpringJmsApplicationTest {

  @ClassRule
  public static EmbeddedActiveMQBroker broker =
      new EmbeddedActiveMQBroker();

  @Autowired
  private Sender sender;

  @Test
  public void testReceive() throws Exception {
    String correlationId = sender.sendOrder("order-001");
    String status = sender.receiveOrderStatus(correlationId);

    assertThat(status).isEqualTo("Accepted");
  }
}
{% endhighlight %}

Open a command prompt in the root directory and run the test case.

{% highlight plaintext %}
mvn test
{% endhighlight %}

In the output logs, we can see that the order and status messages are received.

{% highlight plaintext %}
.   ____          _            __ _ _
/\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
\\/  ___)| |_)| | | | | || (_| |  ) ) ) )
'  |____| .__|_| |_|_| |_\__, | / / / /
=========|_|==============|___/=/_/_/_/
:: Spring Boot ::        (v2.1.1.RELEASE)

2018-12-14 07:21:37.497  INFO 5668 --- [           main] c.codenotfound.SpringJmsApplicationTest  : Starting SpringJmsApplicationTest on DESKTOP-2RB3C1U with PID 5668 (started by Codenotfound in C:\Users\Codenotfound\repos\spring-jms\spring-jms-jmstemplate)
2018-12-14 07:21:37.498  INFO 5668 --- [           main] c.codenotfound.SpringJmsApplicationTest  : No active profile set, falling back to default profiles: default
2018-12-14 07:21:38.836  INFO 5668 --- [           main] c.codenotfound.SpringJmsApplicationTest  : Started SpringJmsApplicationTest in 1.641 seconds (JVM running for 3.167)
2018-12-14 07:21:39.103  INFO 5668 --- [           main] com.codenotfound.jms.Sender              : sending OrderNumber='order-001' with MessageId='ID:DESKTOP-2RB3C1U-54230-1544768496820-8:1:1:1:1'
2018-12-14 07:21:39.184  INFO 5668 --- [enerContainer-1] com.codenotfound.jms.Receiver            : received OrderNumber='order-001' with MessageId='ID:DESKTOP-2RB3C1U-54230-1544768496820-8:1:1:1:1'
2018-12-14 07:21:39.184  INFO 5668 --- [enerContainer-1] com.codenotfound.jms.Receiver            : sending Status='Accepted' with CorrelationId='ID:DESKTOP-2RB3C1U-54230-1544768496820-8:1:1:1:1'
2018-12-14 07:21:39.188  INFO 5668 --- [           main] com.codenotfound.jms.Sender              : receive Status='Accepted' for CorrelationId='ID:DESKTOP-2RB3C1U-54230-1544768496820-8:1:1:1:1'
2018-12-14 07:21:40.197  INFO 5668 --- [           main] o.a.a.junit.EmbeddedActiveMQBroker       : Stopping Embedded ActiveMQ Broker: embedded-broker
2018-12-14 07:21:40.199  INFO 5668 --- [           main] o.a.activemq.broker.TransportConnector   : Connector vm://embedded-broker stopped
2018-12-14 07:21:40.199  INFO 5668 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.8 (embedded-broker, ID:DESKTOP-2RB3C1U-54230-1544768496820-0:1) is shutting down
2018-12-14 07:21:40.202  INFO 5668 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.8 (embedded-broker, ID:DESKTOP-2RB3C1U-54230-1544768496820-0:1) uptime 3.484 seconds
2018-12-14 07:21:40.202  INFO 5668 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.8 (embedded-broker, ID:DESKTOP-2RB3C1U-54230-1544768496820-0:1) is shutdown
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 4.197 s - in com.codenotfound.SpringJmsApplicationTest
[INFO]
[INFO] Results:
[INFO]
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 7.560 s
[INFO] Finished at: 2018-12-14T07:21:40+01:00
[INFO] ------------------------------------------------------------------------
{% endhighlight %}

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-jms/tree/master/spring-jms-message-selector){:target="_blank"}.
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

In this tutorial, you learned how to configure the `JmsTemplate` to send and receive JMS messages.

If you have any questions.

Or if you enjoyed reading.

Drop a line below!
