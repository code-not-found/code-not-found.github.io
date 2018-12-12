---
title: "Spring JMS Message Selector Example"
permalink: /spring-jms-message-selector-example.html
excerpt: "A detailed step-by-step tutorial on how to implement a message selector using Spring JMS and Spring Boot."
date: 2018-12-12
last_modified_at: 2018-12-12
header:
  teaser: "assets/images/spring-jms/spring-jms-message-selector.png"
categories: [Spring JMS]
tags: [ActiveMQ, Apache ActiveMQ, Example, Maven, Message Selector, Spring, Spring Boot, Spring JMS, Tutorial]
published: true
---

<img src="{{ site.url }}/assets/images/spring-jms/spring-jms-message-selector.png" alt="spring jms message selector" class="align-right title-image">

In this post I'm going to show you how to configure a [JMS message selector](https://docs.oracle.com/cd/E19798-01/821-1841/bncer/index.html){:target="_blank"} using [Spring JMS](https://docs.spring.io/spring/docs/5.1.3.RELEASE/spring-framework-reference/integration.html#jms){:target="_blank"}.

You'll also see how to add information to a message so that you can select it.

So here we go.

## 1. What is a JMS Message Selector?

If your messaging application needs to **filter the messages it receives**, you can use a JMS message selector. It allows a message consumer to specify the messages it is interested in.

A selector is a String that contains an expression. The syntax of the expression is based on a subset of the [SQL92](https://en.wikipedia.org/wiki/SQL-92){:target="_blank"} conditional expression syntax.

Let's create an example to show how all of this works. We start from a previous [Spring JMS configuration]({{ site.url }}/spring-jms-boot-configuration-example.html) example.

We then modify the `Receiver` so that it receives high and low priority messages with different listeners. In the `Sender` we add a [JMS property]({{ site.url }}/jms-message-types-properties-overview.html) on which we can filter.

## 2. General Project Overview

We will use the following tools/frameworks:
* Spring JMS 5.1
* Spring Boot 2.1
* ActiveMQ 5.14
* Maven 3.5

Our project has the following directory structure:

<img src="{{ site.url }}/assets/images/spring-jms/spring-jms-message-selector-maven-project.png" alt="spring jms selector converter maven project">

## 3. Add a JMS Message Selector to a Listener

On the `@JmsListener` there is an optional message <var>selector</var> property you can define.

We create two listeners in the `Receiver`: one for high priority messages and one for low priority messages. Selection is done based on a <var>priority</var> JMS property that we will set in the `Sender`.

> Note that a message consumer receives only messages whose headers and properties match the selector. A message selector cannot select messages on the basis of the content of the message body.

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

  @JmsListener(destination = "${queue.boot}",
      selector = "priority = 'high'")
  public void receiveHigh(String message) {
    LOGGER.info("received high priority message='{}'", message);
    latch.countDown();
  }

  @JmsListener(destination = "${queue.boot}",
      selector = "priority = 'low'")
  public void receiveLow(String message) {
    LOGGER.info("received low priority message='{}'", message);
    latch.countDown();
  }
}
{% endhighlight %}

The `JmsTemplate` by default converts a String into a `TextMessage` using the `SimpleMessageConverter`. For more information on this you can check the [Spring JMS Message Converter example]({{ site.url }}/spring-jms-message-converter-example.html).

We need to use a [MessagePostProcessor](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/jms/core/MessagePostProcessor.html){:target="_blank"} to add a JMS property to a message after it has been processed by the converter.

Modify the `Sender` to check an `isHighPriority` parameter. If the value equals `true` a <var>priority</var> property with the value <var>high</var> is set. Otherwise the property is set to  <var>low</var>.

{% highlight java %}
package com.codenotfound.jms;

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
  private JmsTemplate jmsTemplate;

  public void send(String destination, String message,
      boolean isHighPriority) {
    LOGGER.info("sending message='{}' with highPriority='{}'",
        message, isHighPriority);

    if (isHighPriority) {
      jmsTemplate.convertAndSend(destination, message,
          messagePostProcessor -> {
            messagePostProcessor.setStringProperty("priority",
                "high");
            return messagePostProcessor;
          });
    } else {
      jmsTemplate.convertAndSend(destination, message,
          messagePostProcessor -> {
            messagePostProcessor.setStringProperty("priority",
                "low");
            return messagePostProcessor;
          });
    }
  }
}
{% endhighlight %}

## 4. Test the Message Filter

We modify the existing test case so that three messages are sent.

Two of them are high priority and will lower the `CountDownLatch` in the `Receiver`.

{% highlight java %}
package com.codenotfound.jms;

import static org.assertj.core.api.Assertions.assertThat;
import java.util.concurrent.TimeUnit;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.annotation.DirtiesContext;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
@DirtiesContext
public class SpringJmsApplicationTest {

  @Autowired
  private Sender sender;

  @Autowired
  private Receiver receiver;

  @Test
  public void testReceive() throws Exception {
    sender.send("selector.q", "High priority!", true);
    sender.send("selector.q", "Low priority!", false);

    receiver.getLatch().await(10000, TimeUnit.MILLISECONDS);
    assertThat(receiver.getLatch().getCount()).isEqualTo(0);
  }
}
{% endhighlight %}

Open a command prompt in the rott directory and start the test case.

{% highlight plaintext %}
mvn test
{% endhighlight %}

In the output logs, we can see that the messages are received in the respective JMS Listeners.

{% highlight plaintext %}
.   ____          _            __ _ _
/\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
\\/  ___)| |_)| | | | | || (_| |  ) ) ) )
 '  |____| .__|_| |_|_| |_\__, | / / / /
=========|_|==============|___/=/_/_/_/
:: Spring Boot ::        (v2.1.1.RELEASE)

2018-12-12 10:45:11.263  INFO 16664 --- [           main] c.c.jms.SpringJmsApplicationTest         : Starting SpringJmsApplicationTest on DESKTOP-2RB3C1U with PID 16664 (started by Codenotfound in C:\Users\Codenotfound\repos\spring-jms\spring-jms-message-selector)
2018-12-12 10:45:11.263  INFO 16664 --- [           main] c.c.jms.SpringJmsApplicationTest         : No active profile set, falling back to default profiles: default
2018-12-12 10:45:12.541  INFO 16664 --- [           main] o.apache.activemq.broker.BrokerService   : Using Persistence Adapter: MemoryPersistenceAdapter
2018-12-12 10:45:12.604  INFO 16664 --- [  JMX connector] o.a.a.broker.jmx.ManagementContext       : JMX consoles can connect to service:jmx:rmi:///jndi/rmi://localhost:1099/jmxrmi
2018-12-12 10:45:12.691  INFO 16664 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.8 (localhost, ID:DESKTOP-2RB3C1U-57881-1544607912557-0:1) is starting
2018-12-12 10:45:12.707  INFO 16664 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.8 (localhost, ID:DESKTOP-2RB3C1U-57881-1544607912557-0:1) started
2018-12-12 10:45:12.707  INFO 16664 --- [           main] o.apache.activemq.broker.BrokerService   : For help or more information please see: http://activemq.apache.org
2018-12-12 10:45:12.738  INFO 16664 --- [           main] o.a.activemq.broker.TransportConnector   : Connector vm://localhost started
2018-12-12 10:45:12.801  INFO 16664 --- [           main] c.c.jms.SpringJmsApplicationTest         : Started SpringJmsApplicationTest in 1.905 seconds (JVM running for 3.03)
2018-12-12 10:45:13.019  INFO 16664 --- [           main] com.codenotfound.jms.Sender              : sending message='High priority 1!' with highPriority='true'
2018-12-12 10:45:13.051  INFO 16664 --- [           main] com.codenotfound.jms.Sender              : sending message='Low priority 1!' with highPriority='false'
2018-12-12 10:45:13.051  INFO 16664 --- [           main] com.codenotfound.jms.Sender              : sending message='High priority 2!' with highPriority='true'
2018-12-12 10:45:13.051  INFO 16664 --- [enerContainer-1] com.codenotfound.jms.Receiver            : received high priority message='High priority 1!'
2018-12-12 10:45:13.051  INFO 16664 --- [enerContainer-1] com.codenotfound.jms.Receiver            : received low priority message='Low priority 1!'
2018-12-12 10:45:13.051  INFO 16664 --- [enerContainer-1] com.codenotfound.jms.Receiver            : received high priority message='High priority 2!'
2018-12-12 10:45:14.062  INFO 16664 --- [           main] o.a.activemq.broker.TransportConnector   : Connector vm://localhost stopped
2018-12-12 10:45:14.062  INFO 16664 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.8 (localhost, ID:DESKTOP-2RB3C1U-57881-1544607912557-0:1) is shutting down
2018-12-12 10:45:14.066  INFO 16664 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.8 (localhost, ID:DESKTOP-2RB3C1U-57881-1544607912557-0:1) uptime 1.742 seconds
2018-12-12 10:45:14.066  INFO 16664 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.8 (localhost, ID:DESKTOP-2RB3C1U-57881-1544607912557-0:1) is shutdown
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 3.915 s - in com.codenotfound.jms.SpringJmsApplicationTest
[INFO]
[INFO] Results:
[INFO]
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 11.260 s
[INFO] Finished at: 2018-12-12T10:45:14+01:00
[INFO] ------------------------------------------------------------------------
{% endhighlight %}

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-jms/tree/master/spring-jms-message-selector){:target="_blank"}.
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

Setting up a JMS message selector is straightforward when you use Spring JMS.

Make sure to leave a comment if the example was helpful.

Thanks!
