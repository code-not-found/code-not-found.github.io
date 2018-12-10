---
title: "Spring JMS Boot Configuration Example"
permalink: /spring-jms-boot-configuration-example.html
excerpt: "A detailed step-by-step tutorial on how to autoconfigure ActiveMQ and Spring JMS using Spring Boot."
date: 2017-05-08
modified: 2018-12-10
header:
  teaser: "assets/images/spring-jms/spring-jms-boot-configuration.png"
categories: [Spring JMS]
tags: [Autoconfig, Autoconfiguration, ActiveMQ, Apache ActiveMQ, Configuration, Example, Maven, Spring, Spring Boot, Spring JMS, Tutorial]
redirect_from:
  - /2017/04/spring-jms-boot-example.html
  - /2017/05/spring-jms-activemq-boot-example.html
  - /spring-jms-activemq-boot-example.html
published: true
---

<img src="{{ site.url }}/assets/images/spring-jms/spring-jms-boot-configuration.png" alt="spring jms boot configuration" class="align-right title-image">




If you want to learn more about Spring JMS - head on over to the [Spring JMS tutorials]({{ site.url }}/spring-jms-tutorials) page.
{: .notice--primary}

## 1. What is Spring Boot Auto-Configuration

[Spring Boot auto-configuration](http://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-auto-configuration.html){:target="_blank"} will try to automatically configure your Spring application based on the JAR dependencies that are available. In other words, if the `spring-jms` and `activemq-broker` dependencies are on the classpath and you have not manually configured any `ConnectionFactory`, `JmsTemplate` or `JmsListenerContainerFactory` beans, then Spring Boot will auto-configure them for you using default values.

To show this behavior we will start from a previous [Spring JMS tutorial]({{ site.url }}/spring-jms-activemq-consumer-producer-example.html) in which we send/receive messages to/from ActiveMQ using Spring JMS. The original code will be reduced to a bare minimum in order to demonstrate Spring Boot's autoconfiguration capabilities.

# 2. General Project Setup

If you want to learn more about Spring JMS - head on over to the [Spring JMS tutorials page]({{ site.url }}/spring-jms/).
{: .notice--primary}

## 2. General Project Overview

We will use the following tools/frameworks:
* Spring JMS 5.1
* Spring Boot 2.1
* ActiveMQ 5.14
* Maven 3.5

Our project has the following directory structure:

<img src="{{ site.url }}/assets/images/spring-jms/spring-jms-activemq-boot-maven-project.png" alt="spring jms activemq boot maven project">

## 3. Maven Setup

The example project is managed using [Maven](https://maven.apache.org/){:target="_blank"}. Needed dependencies like [Spring Boot](https://projects.spring.io/spring-boot/){:target="_blank"} and [Spring JMS](http://projects.spring.io/spring-framework/){:target="_blank"} are included by declaring the `spring-boot-starter-activemq` [Spring Boot starter](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters){:target="_blank"} in the POM file as shown below.

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>spring-jms-activemq-boot</artifactId>
  <version>0.0.1-SNAPSHOT</version>

  <name>spring-jms-activemq-boot</name>
  <description>Spring JMS Boot Configuration Example</description>
  <url>https://codenotfound.com/spring-jms-boot-configuration-example.html</url>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.1.RELEASE</version>
    <relativePath /><!-- lookup parent from repository -->
  </parent>

  <properties>
    <java.version>1.8</java.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-activemq</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>
</project>
{% endhighlight %}

## 4. Spring Boot Setup

The `SpringJmsApplication` remains untouched. Note that in order for the auto-configuration to work we need to opt-in by adding the `@EnableAutoConfiguration` or `@SpringBootApplication` (which is same as adding `@Configuration` `@EnableAutoConfiguration` `@ComponentScan`) annotation to one of our `@Configuration` classes.

> Only ever add one `@EnableAutoConfiguration` annotation. It is recommended to add it to your primary `@Configuration` class.

{% highlight java %}
package com.codenotfound;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringJmsApplication {

  public static void main(String[] args) {
    SpringApplication.run(SpringJmsApplication.class, args);
  }
}
{% endhighlight %}

# Autoconfigure the Spring JMS Message Producer

The setup and creation of the `JmsTemplate` and `ConnectionFactory` beans are automatically done by Spring Boot. We just need to autowire the `JmsTemplate` and use it in the `send()` method.

> By annotating the `Sender` class with `@Component`, Spring will instantiate this class as a bean that we will use in a below test case. In order for this to work, we also need the `@EnableAutoConfiguration` which was indirectly specified on `SpringKafkaApplication` by using the `@SpringBootApplication` annotation.

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

  public void send(String destination, String message) {
    LOGGER.info("sending message='{}' to destination='{}'", message,
        destination);
    jmsTemplate.convertAndSend(destination, message);
  }
}
{% endhighlight %}

# Autoconfigure the Spring Kafka Message Consumer

Similar to the `Sender`, the setup and creation of the `ConnectionFactory` and `JmsListenerContainerFactory` bean is automatically done by Spring Boot. The `@JmsListener` annotation creates a message listener container for the annotated `receive()` method.

The destination name is specified using the <var>'${destination.boot}'</var> placeholder for which the value will be automatically fetched from the <var>application.yml</var> properties file.

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

  private CountDownLatch latch = new CountDownLatch(1);

  public CountDownLatch getLatch() {
    return latch;
  }

  @JmsListener(destination = "${queue.boot}")
  public void receive(String message) {
    LOGGER.info("received message='{}'", message);
    latch.countDown();
  }
}
{% endhighlight %}

Using application properties we can further fine tune the different settings of the `ConnectionFactory`, `JmsTemplate` and `JmsListenerContainerFactory` beans. Scroll down to <var># ACTIVEMQ</var> and <var># JMS</var> sections in the following link in order to get a [complete overview on all the available ActiveMQ and JMS properties](https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html){:target="_blank"}.

In this example, we use default values and only specify the destination and broker URL in the included <var>application.yml</var> properties file.

{% highlight yaml %}
spring:
  activemq:
    broker-url: tcp://localhost:61616

queue:
  boot: boot.q
{% endhighlight %}

# Testing the Sender and Receiver

In order to verify that our code works, a simple `SpringJmsApplicationTest` test case is used. It contains a `testReceive()` unit test case that uses the `Sender` to send a message to the <var>'boot.q'</var> queue on the ActiveMQ broker. We then use the `CountDownLatch` from the `Receiver` to verify that a message was successfully received.

We include a dedicated <var>application.yml</var> properties file for testing under <var>src/test/resources</var> that does not contain a broker URL. If Spring Boot does not find a broker URL, auto-configuration will automatically start an embedded ActiveMQ broker instance.

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
    sender.send("boot.q", "Hello Boot!");

    receiver.getLatch().await(10000, TimeUnit.MILLISECONDS);
    assertThat(receiver.getLatch().getCount()).isEqualTo(0);
  }
}
{% endhighlight %}

Let's run the test case by executing following Maven command at the command prompt:

{% highlight plaintext %}
mvn test
{% endhighlight %}

The test case will be triggered resulting in following log statements:

{% highlight plaintext %}
.   ____          _            __ _ _
/\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
\\/  ___)| |_)| | | | | || (_| |  ) ) ) )
'  |____| .__|_| |_|_| |_\__, | / / / /
=========|_|==============|___/=/_/_/_/
:: Spring Boot ::        (v2.1.1.RELEASE)

2018-12-10 15:15:18.931  INFO 18428 --- [           main] c.c.jms.SpringJmsApplicationTest         : Starting SpringJmsApplicationTest on DESKTOP-2RB3C1U with PID 18428 (started by Codenotfound in C:\Users\Codenotfound\repos\spring-jms\spring-jms-activemq-boot)
2018-12-10 15:15:18.932  INFO 18428 --- [           main] c.c.jms.SpringJmsApplicationTest         : No active profile set, falling back to default profiles: default
2018-12-10 15:15:20.096  INFO 18428 --- [           main] o.apache.activemq.broker.BrokerService   : Using Persistence Adapter: MemoryPersistenceAdapter
2018-12-10 15:15:20.133  INFO 18428 --- [  JMX connector] o.a.a.broker.jmx.ManagementContext       : JMX consoles can connect to service:jmx:rmi:///jndi/rmi://localhost:1099/jmxrmi
2018-12-10 15:15:20.263  INFO 18428 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.8 (localhost, ID:DESKTOP-2RB3C1U-62308-1544451320126-0:1) is starting
2018-12-10 15:15:20.271  INFO 18428 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.8 (localhost, ID:DESKTOP-2RB3C1U-62308-1544451320126-0:1) started
2018-12-10 15:15:20.272  INFO 18428 --- [           main] o.apache.activemq.broker.BrokerService   : For help or more information please see: http://activemq.apache.org
2018-12-10 15:15:20.314  INFO 18428 --- [           main] o.a.activemq.broker.TransportConnector   : Connector vm://localhost started
2018-12-10 15:15:20.356  INFO 18428 --- [           main] c.c.jms.SpringJmsApplicationTest         : Started SpringJmsApplicationTest in 1.795 seconds (JVM running for 2.836)
2018-12-10 15:15:20.586  INFO 18428 --- [           main] com.codenotfound.jms.Sender              : sending message='Hello Boot!' to destination='boot.q'
2018-12-10 15:15:20.611  INFO 18428 --- [enerContainer-1] com.codenotfound.jms.Receiver            : received message='Hello Boot!'
2018-12-10 15:15:21.623  INFO 18428 --- [           main] o.a.activemq.broker.TransportConnector   : Connector vm://localhost stopped
2018-12-10 15:15:21.624  INFO 18428 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.8 (localhost, ID:DESKTOP-2RB3C1U-62308-1544451320126-0:1) is shutting down
2018-12-10 15:15:21.637  INFO 18428 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.8 (localhost, ID:DESKTOP-2RB3C1U-62308-1544451320126-0:1) uptime 1.687 seconds
2018-12-10 15:15:21.638  INFO 18428 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.8 (localhost, ID:DESKTOP-2RB3C1U-62308-1544451320126-0:1) is shutdown
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 3.785 s - in com.codenotfound.jms.SpringJmsApplicationTest
[INFO]
[INFO] Results:
[INFO]
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 8.314 s
[INFO] Finished at: 2018-12-10T15:15:22+01:00
[INFO] ------------------------------------------------------------------------
{% endhighlight %}

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-jms/tree/master/spring-jms-activemq-boot){:target="_blank"}.
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

In above example, we were able to autoconfigure a JMS connection to ActiveMQ using Spring Boot and a couple of lines of code.

Drop a comment in case you thought the example was helpful or if you found something was missing.
