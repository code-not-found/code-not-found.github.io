---
title: "Spring JMS Message Converter Example"
permalink: /spring-jms-message-converter-example.html
excerpt: "A detailed step-by-step tutorial on how to implement a message converter using Spring JMS and Spring Boot."
date: 2018-12-11
last_modified_at: 2019-05-30
header:
  teaser: "assets/images/spring-jms/spring-jms-message-converter.png"
categories: [Spring JMS]
tags: [ActiveMQ, Apache ActiveMQ, Example, Maven, Message Converter, Spring, Spring Boot, Spring JMS, Tutorial]
published: true
---

<img src="{{ site.url }}/assets/images/spring-jms/spring-jms-message-converter.png" alt="spring jms message converter" class="align-right title-image">

Today you're going to see how to implement a [MessageConverter](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/jms/support/converter/MessageConverter.html){:target="_blank"} using [Spring JMS](https://docs.spring.io/spring/docs/5.1.7.RELEASE/spring-framework-reference/integration.html#jms){:target="_blank"}.

The best part?

It's really easy to do.

So let's get down to business

If you want to learn more about Spring JMS - head on over to the [Spring JMS tutorials]({{ site.url }}/spring-jms-tutorials) page.
{: .notice--primary}

## 1. What is a Message Converter?

A `MessageConverter` specifies how to convert between Java objects and JMS messages.

Spring JMS comes with a number of implementations that are ready to use. By default, the `SimpleMessageConverter` is used by the framework. It is able to handle TextMessages, BytesMessages, MapMessages, and ObjectMessages.   

Let's build an example to show how you can use a message converter with Spring JMS. We start from a previous [Spring with JMS example]({{ site.url }}/spring-jms-annotations-example.html). We will adapt it so that we can send a `Person` object that gets converted to/from JSON.

> Note that Spring JMS ships with a `MappingJackson2MessageConverter` that converts messages to and from JSON. We will not use it and create our own custom implementation instead.

## 2. General Project Overview

We will use the following tools/frameworks:
* Spring JMS 5.1
* Spring Boot 2.1
* ActiveMQ 5.15
* Maven 3.6

Our project has the following directory structure:

<img src="{{ site.url }}/assets/images/spring-jms/spring-jms-message-converter-maven-project.png" alt="spring jms message converter maven project">

## 3. Create a Custom JSON Message Converter

First, we define a simple `Person` class that contains a name and age. It is a simple POJO with the needed constructors and getters/setters.

{% highlight java %}
package com.codenotfound.jms;

public class Person {

  private String name;

  private int age;

  public Person() {
    super();
  }

  public Person(String name, int age) {
    super();
    this.name = name;
    this.age = age;
  }

  public String getName() {
    return name;
  }

  public void setName(String name) {
    this.name = name;
  }

  public int getAge() {
    return age;
  }

  public void setAge(int age) {
    this.age = age;
  }

  @Override
  public String toString() {
    return "person[name=" + name + ", age=" + age + "]";
  }
}
{% endhighlight %}

Next, create a `PersonMessageConverter` class that implements the `MessageConverter` interface.

You need to implement two methods: `toMessage()` and `fromMessage()`. These specify how the conversion between the `Person` object and JMS message is done.

In the `toMessage()` method we create a `TextMessage`. As payload, we set the JSON String representation of a `Person`.

The `fromMessage()` method converts the JSON String from a JMS message into a `Person`.

The conversion between a Java object and JSON is done using a [Jackson](https://github.com/FasterXML/jackson){:target="_blank"} `ObjectMapper`.

Annotate the class with `@Component` so that Spring registers the class as a Bean. When Spring Boot detects this class it is [auto-configured on both the JmsTemplate and DefaultJmsListenerContainerFactory](https://github.com/spring-projects/spring-boot/pull/4284){:target="_blank"}.

> You can also set the `MessageConverter` on the `JmsTemplate` and `MessageListenerContainer` using `setMessageConverter()`. But this means you need to create these Beans as we saw in a previous [Spring JMS Example]({{ site.url }}/spring-jms-activemq-example.html).

{% highlight java %}
package com.codenotfound.jms;

import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.Session;
import javax.jms.TextMessage;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.jms.support.converter.MessageConverter;
import org.springframework.stereotype.Component;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;

@Component
public class PersonMessageConverter implements MessageConverter {

  private static final Logger LOGGER =
      LoggerFactory.getLogger(PersonMessageConverter.class);

  ObjectMapper mapper;

  public PersonMessageConverter() {
    mapper = new ObjectMapper();
  }

  @Override
  public Message toMessage(Object object, Session session)
      throws JMSException {
    Person person = (Person) object;
    String payload = null;
    try {
      payload = mapper.writeValueAsString(person);
      LOGGER.info("outbound json='{}'", payload);
    } catch (JsonProcessingException e) {
      LOGGER.error("error converting form person", e);
    }

    TextMessage message = session.createTextMessage();
    message.setText(payload);

    return message;
  }

  @Override
  public Object fromMessage(Message message) throws JMSException {
    TextMessage textMessage = (TextMessage) message;
    String payload = textMessage.getText();
    LOGGER.info("inbound json='{}'", payload);

    Person person = null;
    try {
      person = mapper.readValue(payload, Person.class);
    } catch (Exception e) {
      LOGGER.error("error converting to person", e);
    }

    return person;
  }
}
{% endhighlight %}

To wrap up, make sure to also change the `Sender` and `Receiver` so that a `Person` object is sent/received.

## 4. Test the JMS Message Converter

To test the message converter, create a `Person` and send it to the <var>converter.q</var> queue.

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
    Person person = new Person("John Doe", 20);
    sender.send("converter.q", person);

    receiver.getLatch().await(10000, TimeUnit.MILLISECONDS);
    assertThat(receiver.getLatch().getCount()).isEqualTo(0);
  }
}
{% endhighlight %}

Open a command prompt in the root directory of the project. Execute the following Maven command:

{% highlight plaintext %}
mvn test
{% endhighlight %}

The log output shows that the message is converted to/from a JSON representation.

{% highlight plaintext %}
 .   ____          _            __ _ _
/\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
\\/  ___)| |_)| | | | | || (_| |  ) ) ) )
 '  |____| .__|_| |_|_| |_\__, | / / / /
=========|_|==============|___/=/_/_/_/
:: Spring Boot ::        (v2.1.5.RELEASE)

2019-05-30 17:13:58.069  INFO 596 --- [           main] c.c.jms.SpringJmsApplicationTest         : Starting SpringJmsApplicationTest on DESKTOP-2RB3C1U with PID 596 (started by Codenotfound in C:\Users\Codenotfound\repos\spring-jms\spring-jms-message-converter)
2019-05-30 17:13:58.071  INFO 596 --- [           main] c.c.jms.SpringJmsApplicationTest         : No active profile set, falling back to default profiles: default
2019-05-30 17:13:59.844  INFO 596 --- [           main] o.apache.activemq.broker.BrokerService   : Using Persistence Adapter: MemoryPersistenceAdapter
2019-05-30 17:13:59.931  INFO 596 --- [  JMX connector] o.a.a.broker.jmx.ManagementContext       : JMX consoles can connect to service:jmx:rmi:///jndi/rmi://localhost:1099/jmxrmi
2019-05-30 17:14:00.019  INFO 596 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.9 (localhost, ID:DESKTOP-2RB3C1U-59651-1559229239872-0:1) is starting
2019-05-30 17:14:00.026  INFO 596 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.9 (localhost, ID:DESKTOP-2RB3C1U-59651-1559229239872-0:1) started
2019-05-30 17:14:00.026  INFO 596 --- [           main] o.apache.activemq.broker.BrokerService   : For help or more information please see: http://activemq.apache.org
2019-05-30 17:14:00.057  INFO 596 --- [           main] o.a.activemq.broker.TransportConnector   : Connector vm://localhost started
2019-05-30 17:14:00.116  INFO 596 --- [           main] c.c.jms.SpringJmsApplicationTest         : Started SpringJmsApplicationTest in 2.491 seconds (JVM running for 3.467)
2019-05-30 17:14:00.456  INFO 596 --- [           main] com.codenotfound.jms.Sender              : sending person='person[name=John Doe, age=20]' to destination='converter.q'
2019-05-30 17:14:00.508  INFO 596 --- [           main] c.c.jms.PersonMessageConverter           : outbound json='{"name":"John Doe","age":20}'
2019-05-30 17:14:00.519  INFO 596 --- [enerContainer-1] c.c.jms.PersonMessageConverter           : inbound json='{"name":"John Doe","age":20}'
2019-05-30 17:14:00.549  INFO 596 --- [enerContainer-1] com.codenotfound.jms.Receiver            : received person='person[name=John Doe, age=20]'
2019-05-30 17:14:01.566  INFO 596 --- [           main] o.a.activemq.broker.TransportConnector   : Connector vm://localhost stopped
2019-05-30 17:14:01.566  INFO 596 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.9 (localhost, ID:DESKTOP-2RB3C1U-59651-1559229239872-0:1) is shutting down
2019-05-30 17:14:01.581  INFO 596 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.9 (localhost, ID:DESKTOP-2RB3C1U-59651-1559229239872-0:1) uptime 2.090 seconds
2019-05-30 17:14:01.581  INFO 596 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.9 (localhost, ID:DESKTOP-2RB3C1U-59651-1559229239872-0:1) is shutdown
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 4.571 s - in com.codenotfound.jms.SpringJmsApplicationTest
[INFO]
[INFO] Results:
[INFO]
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  9.370 s
[INFO] Finished at: 2019-05-30T17:14:02+02:00
[INFO] ------------------------------------------------------------------------
{% endhighlight %}

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-jms/tree/master/spring-jms-message-converter){:target="_blank"}.
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

In this tutorial, you learned how to create a custom Spring JMS message converter.

Let me know if you found this example useful.

Thanks!
