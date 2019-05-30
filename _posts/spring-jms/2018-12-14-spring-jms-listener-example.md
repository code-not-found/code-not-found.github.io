---
title: "Spring JMS Listener Example"
permalink: /spring-jms-listener-example.html
excerpt: "A detailed step-by-step tutorial on how a Spring JMS listener works in combination with Spring Boot."
date: 2018-12-14
last_modified_at: 2019-05-30
header:
  teaser: "assets/images/spring-jms/spring-jms-listener.png"
categories: [Spring JMS]
tags: [ActiveMQ, Apache ActiveMQ, Example, JmsListener, Maven, Listener, Spring, Spring Boot, Spring JMS, Tutorial]
published: true
---

<img src="{{ site.url }}/assets/images/spring-jms/spring-jms-listener.png" alt="spring jms listener" class="align-right title-image">

In this post, I will show you how to receive messages using a [Spring JMS Listener](https://docs.spring.io/spring/docs/5.1.7.RELEASE/spring-framework-reference/integration.html#jms-receiving){:target="_blank"}.

First, I'll explain the different options available.

Then, I'll build a detailed example.

Sound good? Let's dive right in…

If you want to learn more about Spring JMS - head on over to the [Spring JMS tutorials]({{ site.url }}/spring-jms-tutorials) page.
{: .notice--primary}

## 1. What is a Spring JMS Listener?

In order to asynchronously receive JMS messages, Spring offers a solution to create **message-driven POJOs** (MDP).

A [message listener container](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/jms/listener/AbstractJmsListeningContainer.html){:target="_blank"} is used to receive messages from a JMS broker. The container is a wrapper of sorts that calls a simple POJO listener class when a message arrives.

The one restriction on an MDP is that it must implement the [MessageListener](https://javaee.github.io/javaee-spec/javadocs/javax/jms/MessageListener.html){:target="_blank"} interface.

> Note that you can also [synchronously receive JMS messages using the JmsTemplate]({{ site.url }}/spring-jms-jmstemplate-example.html).

Let's show how the above concepts work.

We start from a previous [Spring JMS Example using ActiveMQ]({{ site.url }}/spring-jms-activemq-example.html).

We adapt it so that an order message is sent to an order queue. A JMS listener will pick up the message and send a status message to two different status queues. On each queue, a different message listener container will read the status.

## 2. General Project Overview

We will use the following tools/frameworks:
* Spring JMS 5.1
* Spring Boot 2.1
* ActiveMQ 5.15
* Maven 3.6

Our project has the following directory structure:

<img src="{{ site.url }}/assets/images/spring-jms/spring-jms-listener-maven-project.png" alt="spring jms listener maven project">

## 3. Configure a Spring JMS Listener Container

A message listener container handles all the **complexity of receiving JMS messages**.

It is responsible for all threading of message reception and dispatches into the listener for processing. The container is the intermediary between an MDP and a messaging provider. It pulls the messages off a queue/topic and feeds them to your message listener.

There are two standard JMS message listener containers packaged with Spring:
1. The [DefaultMessageListenerContainer](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/jms/listener/DefaultMessageListenerContainer.html){:target="_blank"} (DMLC): uses a pull approach. It sits in infinite loop to receive messages.
2. The [SimpleMessageListenerContainer](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/jms/listener/SimpleMessageListenerContainer.html){:target="_blank"} (SMLC): uses a push approach.

The `DefaultMessageListenerContainer` [is considered](http://forum.spring.io/forum/other-spring-related/remoting/24208-what-s-the-best-practice-for-using-jms-in-spring){:target="_blank"} a recommendable approach in many environments.

It is the only listener container that does not impose the thread management onto the JMS provider (as it does not use/block JMS provider threads). The DMLC is also able to gracefully recover from JMS provider failure, such as connection loss. And it is the only variant that supports external transaction managers, in particular for XA transactions.

Alternatively, consider using `SimpleMessageListenerContainer`, but only for native JMS usage without XA, and only if your JMS provider gracefully handles thread management and connection recovery.

Let's create both types of listener containers in the `ReceiverConfig` class.

Create a `DefaultJmsListenerContainerFactory` and a `SimpleJmsListenerContainerFactory`. Note that they both require a `ConnectionFactory`.

We use the `setConcurrency()` method on the DMLC to set the "lower-upper" limits. The listener container created will always hold on to the minimum number of consumers and will slowly scale up to the maximum number of consumers in case of an increasing load.

We then use the factories to create a `DefaultMessageListenerContainer` and a `SimpleMessageListenerContainer`.

The `SimpleJmsListenerEndpoint` defines `Destination` and the `MessageListener` to invoke to process an incoming message. For testing purposes, we reuse the `StatusMessageListener` for both containers but give them a different identifier.

{% highlight java %}
package com.codenotfound.jms;

import org.apache.activemq.ActiveMQConnectionFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.jms.annotation.EnableJms;
import org.springframework.jms.config.DefaultJmsListenerContainerFactory;
import org.springframework.jms.config.SimpleJmsListenerContainerFactory;
import org.springframework.jms.config.SimpleJmsListenerEndpoint;
import org.springframework.jms.listener.DefaultMessageListenerContainer;
import org.springframework.jms.listener.SimpleMessageListenerContainer;

@Configuration
@EnableJms
public class ReceiverConfig {

  @Value("${activemq.broker-url}")
  private String brokerUrl;

  @Value("${destination.status1}")
  private String status1Destination;

  @Value("${destination.status2}")
  private String status2Destination;

  @Bean
  public ActiveMQConnectionFactory receiverActiveMQConnectionFactory() {
    ActiveMQConnectionFactory activeMQConnectionFactory =
        new ActiveMQConnectionFactory();
    activeMQConnectionFactory.setBrokerURL(brokerUrl);

    return activeMQConnectionFactory;
  }

  @Bean
  public DefaultJmsListenerContainerFactory orderDefaultJmsListenerContainerFactory() {
    DefaultJmsListenerContainerFactory factory =
        new DefaultJmsListenerContainerFactory();
    factory
        .setConnectionFactory(receiverActiveMQConnectionFactory());
    factory.setConcurrency("3-10");

    return factory;
  }

  @Bean
  public SimpleJmsListenerContainerFactory orderSimpleJmsListenerContainerFactory() {
    SimpleJmsListenerContainerFactory factory =
        new SimpleJmsListenerContainerFactory();
    factory
        .setConnectionFactory(receiverActiveMQConnectionFactory());

    return factory;
  }

  @Bean
  public DefaultMessageListenerContainer orderMessageListenerContainer() {
    SimpleJmsListenerEndpoint endpoint =
        new SimpleJmsListenerEndpoint();
    endpoint.setMessageListener(new StatusMessageListener("DMLC"));
    endpoint.setDestination(status1Destination);

    return orderDefaultJmsListenerContainerFactory()
        .createListenerContainer(endpoint);
  }

  @Bean
  public SimpleMessageListenerContainer statusMessageListenerContainer() {
    SimpleJmsListenerEndpoint endpoint =
        new SimpleJmsListenerEndpoint();
    endpoint.setMessageListener(new StatusMessageListener("SMLC"));
    endpoint.setDestination(status2Destination);

    return orderSimpleJmsListenerContainerFactory()
        .createListenerContainer(endpoint);
  }
}
{% endhighlight %}

## 4. Create a Spring JMS Listener

To create a JMS listener you need to implement the `MessageListener` interface. It has an `onMessage()` method that is triggered for each message that is received.

The below `StatusMessageListener` tries to cast the received message to a `TextMessage`. If successful it logs the content and lowers a `CountDownLatch` that we will use for testing purposes.

As we set a `StatusMessageListener` instance on both the DMLC and SMLC containers we add an `id`. This allows us to log in which container a message is received.

{% highlight java %}
package com.codenotfound.jms;

import java.util.concurrent.CountDownLatch;
import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.MessageListener;
import javax.jms.TextMessage;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class StatusMessageListener implements MessageListener {

  private static final Logger LOGGER =
      LoggerFactory.getLogger(Sender.class);

  private String id;

  private CountDownLatch latch = new CountDownLatch(1);

  public StatusMessageListener(String id) {
    super();
    this.id = id;
  }

  @Override
  public void onMessage(Message message) {
    if (message instanceof TextMessage) {
      try {
        String text = ((TextMessage) message).getText();
        LOGGER.info("id='{}' received status='{}'", id, text);
        latch.countDown();
      } catch (JMSException e) {
        LOGGER.error("unable to read message payload", e);
      }
    } else {
      LOGGER.error("received unsupported message type");
    }
  }

  public CountDownLatch getLatch() {
    return latch;
  }
}
{% endhighlight %}

There is even an [easier way](https://docs.spring.io/spring/docs/5.1.7.RELEASE/spring-framework-reference/integration.html#jms-annotated){:target="_blank"} to create a Spring JMS listener.

Simply decorate a Bean method with the `@JmsListener` annotation. This causes a listener container to be created on the specified `destination` using a `ContainerFactory`.

If not set, a default container factory is assumed to be available with a bean name of <var>jmsListenerContainerFactory</var> unless an explicit default has been provided through configuration.

> Processing of `@JmsListener` annotations is performed by registering a `JmsListenerAnnotationBeanPostProcessor`. This can be done manually or, more conveniently, through the `@EnableJms` annotation.

So for this example we create an `OrderReceiver` class that contains a `receiveOrder()` method annotated with `@JmsListener`. The method receives a simple order and logs it. We then use an auto-wired `JmsTemplate` to send a status message to the <var>status1</var> and <var>status2</var> destinations.

Note that `@EnableJms` was specified on the earlier defined `ReceiverConfig` class.

{% highlight java %}
package com.codenotfound.jms;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.jms.annotation.JmsListener;
import org.springframework.jms.core.JmsTemplate;
import org.springframework.stereotype.Component;

@Component
public class OrderReceiver {

  private static final Logger LOGGER =
      LoggerFactory.getLogger(OrderReceiver.class);

  @Value("${destination.status1}")
  private String status1Destination;

  @Value("${destination.status2}")
  private String status2Destination;

  @Autowired
  JmsTemplate jmsTemplate;

  @JmsListener(destination = "${destination.order}",
      containerFactory = "orderDefaultJmsListenerContainerFactory")
  public void receiveOrder(String order) {
    LOGGER.info("received order='{}'", order);
    jmsTemplate.convertAndSend(status1Destination, "Accepted");
    jmsTemplate.convertAndSend(status2Destination, "Accepted");
  }
}
{% endhighlight %}

## 5. Testing the JMS listener

We change the existing test case to check if our different message listeners work.

Send an order message to the order destination. Then check if the `CountDownLatch` was lowered in both the DMLC and SMLC message listeners.

{% highlight java %}
package com.codenotfound;

import static org.assertj.core.api.Assertions.assertThat;
import java.util.concurrent.TimeUnit;
import org.apache.activemq.junit.EmbeddedActiveMQBroker;
import org.junit.ClassRule;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.jms.listener.DefaultMessageListenerContainer;
import org.springframework.jms.listener.SimpleMessageListenerContainer;
import org.springframework.test.annotation.DirtiesContext;
import org.springframework.test.context.junit4.SpringRunner;
import com.codenotfound.jms.Sender;
import com.codenotfound.jms.StatusMessageListener;

@RunWith(SpringRunner.class)
@SpringBootTest
@DirtiesContext
public class SpringJmsApplicationTest {

  @ClassRule
  public static EmbeddedActiveMQBroker broker =
      new EmbeddedActiveMQBroker();

  @Autowired
  private Sender sender;

  @Autowired
  private DefaultMessageListenerContainer dmlc;

  @Autowired
  private SimpleMessageListenerContainer smlc;

  @Test
  public void testReceive() throws Exception {
    sender.send("order-002");

    StatusMessageListener status1MessageListener =
        (StatusMessageListener) dmlc.getMessageListener();
    status1MessageListener.getLatch().await(10000,
        TimeUnit.MILLISECONDS);
    assertThat(status1MessageListener.getLatch().getCount())
        .isEqualTo(0);

    StatusMessageListener status2MessageListener =
        (StatusMessageListener) smlc.getMessageListener();
    status2MessageListener.getLatch().await(10000,
        TimeUnit.MILLISECONDS);
    assertThat(status2MessageListener.getLatch().getCount())
        .isEqualTo(0);
  }
}
{% endhighlight %}

Open a command prompt in the project root directory and launch the test case.

{% highlight plaintext %}
mvn test
{% endhighlight %}

In the output logs, we can see that the order message is received by the annotated listener. The status messages arrive in the DMLC and SMLC as shown below.

{% highlight plaintext %}
.   ____          _            __ _ _
/\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
\\/  ___)| |_)| | | | | || (_| |  ) ) ) )
 '  |____| .__|_| |_|_| |_\__, | / / / /
=========|_|==============|___/=/_/_/_/
:: Spring Boot ::        (v2.1.5.RELEASE)

2019-05-30 14:01:04.096  INFO 11360 --- [           main] c.codenotfound.SpringJmsApplicationTest  : Starting SpringJmsApplicationTest on DESKTOP-2RB3C1U with PID 11360 (started by Codenotfound in C:\Users\Codenotfound\repos\spring-jms\spring-jms-listener)
2019-05-30 14:01:04.098  INFO 11360 --- [           main] c.codenotfound.SpringJmsApplicationTest  : No active profile set, falling back to default profiles: default
2019-05-30 14:01:05.298  INFO 11360 --- [           main] c.codenotfound.SpringJmsApplicationTest  : Started SpringJmsApplicationTest in 1.501 seconds (JVM running for 3.267)
2019-05-30 14:01:05.715  INFO 11360 --- [           main] com.codenotfound.jms.Sender              : sending message='order-002'
2019-05-30 14:01:05.747  INFO 11360 --- [enerContainer-3] com.codenotfound.jms.OrderReceiver       : received order='order-002'
2019-05-30 14:01:05.754  INFO 11360 --- [enerContainer-1] com.codenotfound.jms.Sender              : id='DMLC' received status='Accepted'
2019-05-30 14:01:05.774  INFO 11360 --- [ Session Task-1] com.codenotfound.jms.Sender              : id='SMLC' received status='Accepted'
2019-05-30 14:01:06.791  INFO 11360 --- [           main] o.a.a.junit.EmbeddedActiveMQBroker       : Stopping Embedded ActiveMQ Broker: embedded-broker
2019-05-30 14:01:06.795  INFO 11360 --- [           main] o.a.activemq.broker.TransportConnector   : Connector vm://embedded-broker stopped
2019-05-30 14:01:06.795  INFO 11360 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.9 (embedded-broker, ID:DESKTOP-2RB3C1U-59326-1559217663431-0:1) is shutting down
2019-05-30 14:01:06.812  INFO 11360 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.9 (embedded-broker, ID:DESKTOP-2RB3C1U-59326-1559217663431-0:1) uptime 3.666 seconds
2019-05-30 14:01:06.812  INFO 11360 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.9 (embedded-broker, ID:DESKTOP-2RB3C1U-59326-1559217663431-0:1) is shutdown
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 4.394 s - in com.codenotfound.SpringJmsApplicationTest
[INFO]
[INFO] Results:
[INFO]
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  9.908 s
[INFO] Finished at: 2019-05-30T14:01:07+02:00
[INFO] ------------------------------------------------------------------------
{% endhighlight %}

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-jms/tree/master/spring-jms-listener){:target="_blank"}.
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

In this guide, we took a look at the different JMS listeners that the Spring framework offers.

I hope you found the example useful.

If so, leave a comment below.

Thanks!
