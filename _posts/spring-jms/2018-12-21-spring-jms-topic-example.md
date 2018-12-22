---
title: "Spring JMS Topic Example"
permalink: /spring-jms-topic-example.html
excerpt: "A detailed step-by-step tutorial on how to publish/subscribe to a JMS topic using Spring JMS and Spring Boot."
date: 2018-12-21
last_modified_at: 2018-12-22
header:
  teaser: "assets/images/spring-jms/spring-jms-topic.png"
categories: [Spring JMS]
tags: [ActiveMQ, Apache ActiveMQ, Example, Maven, Spring, Spring Boot, Spring JMS, Topic, Tutorial]
published: true
---

<img src="{{ site.url }}/assets/images/spring-jms/spring-jms-topic.png" alt="spring jms topic" class="align-right title-image">

This code sample will show you how to send a JMS message to a topic.

So if you want a to know how to **publish/subscribe** with [Spring JMS](https://docs.spring.io/spring/docs/5.1.3.RELEASE/spring-framework-reference/integration.html#jms){:target="_blank"}.

Keep reading…

If you want to learn more about Spring JMS - head on over to the [Spring JMS tutorials]({{ site.url }}/spring-jms-tutorials) page.
{: .notice--primary}

## 1. What is a Topic?

A [JMS topic](http://activemq.apache.org/how-does-a-queue-compare-to-a-topic.html){:target="_blank"} is a destination that delivers messages to multiple subscribers.

It implements publish and subscribe semantics. When you publish a message it goes to all the subscribers who are interested.

Let's build an example where we send a JMS message to a topic using Spring JMS. On the topic, we will register two subscribers who will each receive the message.

We start from a previous [Spring JMS sample application]({{ site.url }}/spring-jms-activemq-example.html).

## 2. General Project Overview

We will use the following tools/frameworks:
* Spring JMS 5.1
* Spring Boot 2.1
* ActiveMQ 5.14
* Maven 3.5

Our project has the following directory structure:

<img src="{{ site.url }}/assets/images/spring-jms/spring-jms-topic-maven-project.png" alt="spring jms topic maven project">

## 3. Publishing Messages to a JMS Topic

Spring JMS components distinguish between [two types of JMS domains](https://docs.spring.io/spring/docs/current/spring-framework-reference/integration.html#jms-destinations){:target="_blank"}:
1. A point-to-point domain that uses queues
2. A publish-subscribe domain that uses topics

This means that when you [use the JmsTemplate to send JMS messages]({{ site.url }}/spring-jms-jmstemplate-example.html) it can send to queues or topics but not both at the same time!

The boolean property `pubSubDomain` is used to configure the JmsTemplate with knowledge of what JMS domain is being used. By default, the value of this property is false. This indicates that <var>queues</var> are used.

As we want to send a message to a topic we need to update our `SenderConfig` configuration. Use the `setPubSubDomain()` method on the `JmsTemplate` to set `pubSubDomain` to <var>true</var>.

> If you are using the [autoconfigured JmsTemplate]({{ site.url }}/spring-jms-annotations-example.html) you can change the JMS domain by setting the <var>spring.jms.pub-sub-domain</var> [application property (# JMS section)](https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html).

{% highlight java %}
package com.codenotfound.jms;

import org.apache.activemq.ActiveMQConnectionFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.jms.connection.CachingConnectionFactory;
import org.springframework.jms.core.JmsTemplate;

@Configuration
public class SenderConfig {

  @Value("${activemq.broker-url}")
  private String brokerUrl;

  @Bean
  public ActiveMQConnectionFactory senderActiveMQConnectionFactory() {
    ActiveMQConnectionFactory activeMQConnectionFactory =
        new ActiveMQConnectionFactory();
    activeMQConnectionFactory.setBrokerURL(brokerUrl);

    return activeMQConnectionFactory;
  }

  @Bean
  public CachingConnectionFactory cachingConnectionFactory() {
    return new CachingConnectionFactory(
        senderActiveMQConnectionFactory());
  }

  @Bean
  public JmsTemplate jmsTemplate() {
    JmsTemplate jmsTemplate =
        new JmsTemplate(cachingConnectionFactory());
    jmsTemplate.setPubSubDomain(true);

    return jmsTemplate;
  }
}
{% endhighlight %}

## 4. Subscribing to messages from a JMS Topic

Also on the `ListenerContainer`, we need to indicate if we want to use queues or topics. You can configure this on the container or the factory used to create the container.

In this example, we use the `setPubSubDomain()` method on the `ListenerContainerFactory` to specify that we want to use topics.

{% highlight java %}
package com.codenotfound.jms;

import org.apache.activemq.ActiveMQConnectionFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.jms.annotation.EnableJms;
import org.springframework.jms.config.DefaultJmsListenerContainerFactory;

@Configuration
@EnableJms
public class ReceiverConfig {

  @Value("${activemq.broker-url}")
  private String brokerUrl;

  @Bean
  public ActiveMQConnectionFactory receiverActiveMQConnectionFactory() {
    ActiveMQConnectionFactory activeMQConnectionFactory =
        new ActiveMQConnectionFactory();
    activeMQConnectionFactory.setBrokerURL(brokerUrl);

    return activeMQConnectionFactory;
  }

  @Bean
  public DefaultJmsListenerContainerFactory jmsListenerContainerFactory() {
    DefaultJmsListenerContainerFactory factory =
        new DefaultJmsListenerContainerFactory();
    factory
        .setConnectionFactory(receiverActiveMQConnectionFactory());
    factory.setPubSubDomain(true);

    return factory;
  }
}
{% endhighlight %}

In the `Receiver` class we add an extra `receive()` method to demonstrate that we will receive the same message in each listener.

Annotate the method with @JmsListener and specify the same destination. Once this is done Spring will create a subscription for each listener.

{% highlight java %}
package com.codenotfound.jms;

import java.util.concurrent.CountDownLatch;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.jms.annotation.JmsListener;
import org.springframework.stereotype.Component;

@Component
public class Receiver {

  private static final Logger LOGGER =
      LoggerFactory.getLogger(Receiver.class);

  private CountDownLatch latch = new CountDownLatch(2);

  public CountDownLatch getLatch() {
    return latch;
  }

  @JmsListener(destination = "${destination.topic}")
  public void receive1(String message) {
    LOGGER.info("'subscriber1' received message='{}'", message);
    latch.countDown();
  }

  @JmsListener(destination = "${destination.topic}")
  public void receive2(String message) {
    LOGGER.info("'subscriber2' received message='{}'", message);
    latch.countDown();
  }
}
{% endhighlight %}

## 5. Testing the JMS Topic

We reuse the same test case.

In the `Receiver` we increased the value of the `CountDownLatch` to <var>2</var>. This means that if we send one message it should be received in both our subscribers.

{% highlight java %}
package com.codenotfound;

import static org.assertj.core.api.Assertions.assertThat;
import java.util.concurrent.TimeUnit;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;
import com.codenotfound.jms.Receiver;
import com.codenotfound.jms.Sender;

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringJmsApplicationTest {

  @Autowired
  private Sender sender;

  @Autowired
  private Receiver receiver;

  @Test
  public void testReceive() throws Exception {
    sender.send("order-001");

    receiver.getLatch().await(10000, TimeUnit.MILLISECONDS);
    assertThat(receiver.getLatch().getCount()).isEqualTo(0);
  }
}
{% endhighlight %}

Open a command prompt in the root directory and fire up the test case.

{% highlight plaintext %}
mvn test
{% endhighlight %}

In the logs, we see that the same message is indeed received by both subscribers.

{% highlight plaintext %}
.   ____          _            __ _ _
/\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
\\/  ___)| |_)| | | | | || (_| |  ) ) ) )
'  |____| .__|_| |_|_| |_\__, | / / / /
=========|_|==============|___/=/_/_/_/
:: Spring Boot ::        (v2.1.1.RELEASE)

2018-12-21 15:54:25.123  INFO 8420 --- [           main] c.codenotfound.SpringJmsApplicationTest  : Starting SpringJmsApplicationTest on DESKTOP-2RB3C1U with PID 8420 (started by Codenotfound in C:\Users\Codenotfound\repos\spring-jms\spring-jms-topic)
2018-12-21 15:54:25.123  INFO 8420 --- [           main] c.codenotfound.SpringJmsApplicationTest  : No active profile set, falling back to default profiles: default
2018-12-21 15:54:26.076  INFO 8420 --- [           main] c.codenotfound.SpringJmsApplicationTest  : Started SpringJmsApplicationTest in 1.234 seconds (JVM running for 2.817)
2018-12-21 15:54:26.264  INFO 8420 --- [           main] com.codenotfound.jms.Sender              : sending message='order-001' to destination='topic.t'
2018-12-21 15:54:26.295  INFO 8420 --- [enerContainer-1] com.codenotfound.jms.Receiver            : 'subscriber2' received message='order-001'
2018-12-21 15:54:26.295  INFO 8420 --- [enerContainer-1] com.codenotfound.jms.Receiver            : 'subscriber1' received message='order-001'
2018-12-21 15:54:27.310  INFO 8420 --- [           main] o.a.a.junit.EmbeddedActiveMQBroker       : Stopping Embedded ActiveMQ Broker: embedded-broker
2018-12-21 15:54:27.310  INFO 8420 --- [           main] o.a.activemq.broker.TransportConnector   : Connector vm://embedded-broker stopped
2018-12-21 15:54:27.310  INFO 8420 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.8 (embedded-broker, ID:DESKTOP-2RB3C1U-61759-1545404064482-0:1) is shutting down
2018-12-21 15:54:27.326  INFO 8420 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.8 (embedded-broker, ID:DESKTOP-2RB3C1U-61759-1545404064482-0:1) uptime 2.969 seconds
2018-12-21 15:54:27.326  INFO 8420 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.8 (embedded-broker, ID:DESKTOP-2RB3C1U-61759-1545404064482-0:1) is shutdown
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 3.625 s - in com.codenotfound.SpringJmsApplicationTest
[INFO]
[INFO] Results:
[INFO]
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 6.905 s
[INFO] Finished at: 2018-12-21T15:54:27+01:00
[INFO] ------------------------------------------------------------------------
{% endhighlight %}

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-jms/tree/master/spring-jms-topic){:target="_blank"}.
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

In this topic subscriber example, you learned how to configure Spring JMS so it sends/receives a message to/from a topic.

Drop a line if you have any questions.

Or just to let me know you found this post useful.

Thanks.
