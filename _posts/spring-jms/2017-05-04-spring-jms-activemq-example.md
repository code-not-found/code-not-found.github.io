---
title: "Spring JMS ActiveMQ Example"
permalink: /spring-jms-activemq-example.html
excerpt: "A detailed step-by-step tutorial on how to connect to Apache ActiveMQ using Spring JMS and Spring Boot."
date: 2017-05-04
last_modified_at: 2019-05-30
header:
  teaser: "assets/images/spring-jms/spring-jms-activemq.png"
categories: [Spring JMS]
tags: [ActiveMQ, Apache ActiveMQ, Consumer, Example, Hello World, Maven, Producer, Spring, Spring Boot, Spring JMS, Tutorial]
redirect_from:
  - /2017/05/spring-jms-activemq-example.html
  - /2017/05/spring-jms-activemq-consumer-producer-example.html
  - /spring-jms-activemq-consumer-producer-example.html
published: true
---

<img src="{{ site.url }}/assets/images/spring-jms/spring-jms-activemq.png" alt="spring jms activemq" class="align-right title-image">

I'm going to show you EXACTLY how to create a [Spring JMS](https://docs.spring.io/spring/docs/5.1.7.RELEASE/spring-framework-reference/integration.html#jms){:target="_blank"} _Hello World_ example that uses [Spring Boot](https://spring.io/projects/spring-boot){:target="_blank"}, [ActiveMQ](http://activemq.apache.org/){:target="_blank"}, and [Maven](https://maven.apache.org/){:target="_blank"}.

(Step-by-step)

So if you're a Spring JMS beginner, **you'll love this guide**.

Let's get this show on the road!

If you want to learn more about Spring JMS - head on over to the [Spring JMS tutorials]({{ site.url }}/spring-jms-tutorials) page.
{: .notice--primary}

## 1. What is Spring JMS?

Spring provides a JMS integration framework that **simplifies the use of the JMS API**. Much like Spring's integration does for the JDBC API.

The Spring Framework will take care of some low-level details when working with the [JMS API](http://docs.oracle.com/javaee/6/tutorial/doc/bncdr.html){:target="_blank"}.

In this tutorial, we will create a Hello World example in which we will send/receive a message to/from Apache ActiveMQ using Spring JMS, Spring Boot, and Maven.

## 2. General Project Overview

We will use the following tools/frameworks:
* Spring JMS 5.1
* Spring Boot 2.1
* ActiveMQ 5.15
* Maven 3.6

Our project has the following directory structure:

<img src="{{ site.url }}/assets/images/spring-jms/spring-jms-activemq-hello-world-maven-project.png" alt="spring jms activemq hello world maven project">

## 3. Maven Setup

We build and run our example using **Maven**. If not already the case, [download and install Apache Maven](https://downlinko.com/download-install-apache-maven-windows.html){:target="_blank"}.

Let's use [Spring Initializr](https://start.spring.io/){:target="_blank"} to generate our Maven project. Make sure to select <var>JMS (ActiveMQ)</var> as a dependency.

<img src="{{ site.url }}/assets/images/spring-jms/spring-jms-activemq-hello-world-initializr.png" alt="spring jms activemq hello world initializr">

Click <var>Generate Project</var> to generate and download the Spring Boot project template. At the root of the project, you'll find a <var>pom.xml</var> file which is the XML representation of the Maven project.

To avoid having to manage the version compatibility of the different Spring dependencies, we will inherit the defaults from the `spring-boot-starter-parent` parent POM.

The generated project contains [Spring Boot Starters](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters){:target="_blank"} that manage the different Spring dependencies.

> You can find back the exact dependency versions in Appendix F of the reference documentation. For Spring Boot 2.1.5 the [ActiveMQ dependency is version 5.15.9](https://docs.spring.io/spring-boot/docs/2.1.5.RELEASE/reference/html/appendix-dependency-versions.html#appendix-dependency-versions){:target="_blank"}.

The `spring-boot-starter-activemq` dependency includes the needed dependencies for using Spring JMS in combination with ActiveMQ.

The `spring-boot-starter-test` includes the dependencies for testing Spring Boot applications with libraries that include [JUnit](http://junit.org/junit4/){:target="_blank"}, [Hamcrest](http://hamcrest.org/JavaHamcrest/){:target="_blank"} and [Mockito](http://site.mockito.org/){:target="_blank"}.

In the plugins section, you'll find the [Spring Boot Maven Plugin](https://docs.spring.io/spring-boot/docs/2.1.5.RELEASE/reference/html/build-tool-plugins-maven-plugin.html){:target="_blank"}. `spring-boot-maven-plugin` allows us to build a single, runnable "uber-jar". This is a convenient way to execute and transport code.

Also, the plugin allows you to start the example via a Maven command.

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>spring-jms-activemq-hello-world</artifactId>
  <version>0.0.1-SNAPSHOT</version>

  <name>spring-jms-activemq-hello-world</name>
  <description>Spring JMS ActiveMQ Example</description>
  <url>https://codenotfound.com/spring-jms-activemq-example.html</url>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.5.RELEASE</version>
    <relativePath /><!-- lookup parent from repository -->
  </parent>

  <properties>
    <java.version>11</java.version>
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

Spring Boot is used in order to make a Spring JMS example application that you can "just run".

We start by creating a `SpringJmsApplication` which contains the `main()` method that uses Spring Boot's `SpringApplication.run()` method to launch the application.

> The `@SpringBootApplication` annotation is a convenience annotation that adds: `@Configuration`, `@EnableAutoConfiguration` and `@ComponentScan`.

For more information on Spring Boot check out the [Spring Boot getting started guide](https://spring.io/guides/gs/spring-boot/){:target="_blank"}.

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

The below sections will detail how to create a sender and receiver together with their respective configurations.

> It is also possible to have [Spring Boot autoconfigure Spring JMS]({{ site.url }}/spring-jms-annotations-example.html) using default values so that actual code that needs to be written is reduced to a bare minimum.

## 5. Create a Spring JMS Message Producer

For sending messages we will be using the `JmsTemplate` which requires a reference to a `ConnectionFactory`. The template provides convenience methods which handle the creation and release of resources when sending or synchronously receiving messages.

In the below `Sender` class, the `JmsTemplate` is auto-wired as the actual creation of the `Bean` will be done in a separate `SenderConfig` class.

In this tutorial we will use the `convertAndSend()` method which sends the given object to the <var>helloworld.q</var> destination, converting the object to a JMS message.

The [type of JMS message]({{ site.url }}/jms-message-types-properties-overview.html#jms-message-body) depends on the type of the object being passed. In the case of a `String`, a JMS `TextMessage` will be created.

> For more detailed information on how to send JMS messages, check the [Spring JmsTemplate Example]({{ site.url }}/spring-jms-jmstemplate-example.html).

{% highlight java %}
package com.codenotfound.jms;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jms.core.JmsTemplate;

public class Sender {

  private static final Logger LOGGER =
      LoggerFactory.getLogger(Sender.class);

  @Autowired
  private JmsTemplate jmsTemplate;

  public void send(String message) {
    LOGGER.info("sending message='{}'", message);
    jmsTemplate.convertAndSend("helloworld.q", message);
  }
}
{% endhighlight %}

The creation of the `JmsTemplate` and `Sender` is handled in the `SenderConfig` class. This class is annotated with [@Configuration](https://docs.spring.io/spring/docs/5.1.7.RELEASE/spring-framework-reference/core.html#beans-java-basic-concepts){:target="_blank"} which indicates that the class can be used by the Spring IoC container as a source of bean definitions.

In order to be able to use the Spring JMS template, we need to provide a reference to a `ConnectionFactory` which is used to [create connections with the JMS provider](http://docs.oracle.com/javaee/6/tutorial/doc/bnceh.html){:target="_blank"}. In addition, it encapsulates various configuration parameters, many of which are vendor specific.

In the case of ActiveMQ, we use the `ActiveMQConnectionFactory`.

On the `ActiveMQConnectionFactory` we set the broker URL which is fetched from the <var>application.yml</var> properties file using the `@Value` annotation.

{% highlight yaml %}
activemq:
  broker-url: tcp://localhost:61616
{% endhighlight %}

The `JmsTemplate` was originally designed to be used in combination with a J2EE container where the container would provide the necessary pooling of the JMS resources.

As we are running this example on Spring Boot, we will wrap `ActiveMQConnectionFactory` using Spring's `CachingConnectionFactory` in order to still have the benefit of caching of sessions, connections, and producers as well as automatic connection recovery.

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
    return new JmsTemplate(cachingConnectionFactory());
  }

  @Bean
  public Sender sender() {
    return new Sender();
  }
}
{% endhighlight %}

## 6. Create a Spring JMS Message Consumer

Like with any messaging-based application, you need to create a receiver that will handle the messages that have been sent. The below `Receiver` is nothing more than a simple POJO that defines a method for receiving messages. In this guide we named the method `receive()`, but you can name it anything you like.

The `@JmsListener` annotation creates a message listener container behind the scenes for each annotated method, using a `JmsListenerContainerFactory`. By default, a bean with name <var>jmsListenerContainerFactory</var> is expected that we will set up in the next section.

Using the `destination` element, we specify the destination for this listener. In the below example we set the destination to <var>helloworld.q</var>.

> For testing convenience, we added a `CountDownLatch`. This allows the POJO to signal that a message is received. This is something you are not likely to implement in a production application.

{% highlight java %}
package com.codenotfound.jms;

import java.util.concurrent.CountDownLatch;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.jms.annotation.JmsListener;

public class Receiver {

  private static final Logger LOGGER =
      LoggerFactory.getLogger(Receiver.class);

  private CountDownLatch latch = new CountDownLatch(1);

  public CountDownLatch getLatch() {
    return latch;
  }

  @JmsListener(destination = "helloworld.q")
  public void receive(String message) {
    LOGGER.info("received message='{}'", message);
    latch.countDown();
  }
}
{% endhighlight %}

The creation and configuration of the different Spring Beans needed for the `Receiver` POJO are grouped in the `ReceiverConfig` class.

> Note that we need to add the `@EnableJms` annotation to enable support for the `@JmsListener` annotation that was used on the `Receiver`.

The `jmsListenerContainerFactory()` is expected by the `@JmsListener` annotation from the `Receiver`.

Contrary to the `JmsTemplate` ideally [don't use Spring's CachingConnectionFactory with a message listener container](http://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/jms/listener/DefaultMessageListenerContainer.html){:target="_blank"} at all. Reason for this is that it is generally preferable to let the listener container itself handle appropriate caching within its lifecycle.

As we are connecting to ActiveMQ, an `ActiveMQConnectionFactory` is created and passed in the constructor of the `DefaultJmsListenerContainerFactory`.

> More details on how to receive JMS messages can be found in the [Spring JMS Listener Example]({{ site.url }}/spring-jms-listener-example.html).

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

    return factory;
  }

  @Bean
  public Receiver receiver() {
    return new Receiver();
  }
}
{% endhighlight %}

## 7. Testing the Spring JMS Example

Spring Boot will automatically start an embedded broker if the following conditions are met:
* ActiveMQ is on the classpath
* No broker URL is specified through <var>spring.activemq.broker-url</var>

Let's use the embedded broker for testing. We add a dedicated <var>application.yml</var> properties file under <var>src/test/resources</var>. Inside we specify the VM URI as broker connection URL.

{% highlight yaml %}
activemq:
  broker-url: vm://embedded-broker?broker.persistent=false
{% endhighlight %}

Next, we create a basic `SpringJmsApplicationTest` class to verify that we are able to send and receive a message to and from ActiveMQ.

It contains a `testReceive()` unit test case that uses the `Sender` to send a message to the <var>'helloworld.q'</var> queue on the ActiveMQ message broker. We then use the `CountDownLatch` from the `Receiver` to verify that a message was received.

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
    sender.send("Hello Spring JMS ActiveMQ!");

    receiver.getLatch().await(10000, TimeUnit.MILLISECONDS);
    assertThat(receiver.getLatch().getCount()).isEqualTo(0);
  }
}
{% endhighlight %}

In order to execute the above test, open a command prompt in the project root directory and run following Maven command:

{% highlight plaintext %}
mvn test
{% endhighlight %}

Maven will download the dependencies, compile the code and run the unit test case. The result should be a successful build as shown below:

{% highlight plaintext %}
 .   ____          _            __ _ _
/\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
\\/  ___)| |_)| | | | | || (_| |  ) ) ) )
'  |____| .__|_| |_|_| |_\__, | / / / /
=========|_|==============|___/=/_/_/_/
:: Spring Boot ::        (v2.1.5.RELEASE)

2019-05-30 08:39:06.606  INFO 13060 --- [           main] c.codenotfound.SpringJmsApplicationTest  : Starting SpringJmsApplicationTest on DESKTOP-2RB3C1U with PID 13060 (started by Codenotfound in C:\Users\Codenotfound\repos\spring-jms\spring-jms-activemq-hello-world)
2019-05-30 08:39:06.608  INFO 13060 --- [           main] c.codenotfound.SpringJmsApplicationTest  : No active profile set, falling back to default profiles: default
2019-05-30 08:39:08.007  INFO 13060 --- [           main] o.apache.activemq.broker.BrokerService   : Using Persistence Adapter: MemoryPersistenceAdapter
2019-05-30 08:39:08.082  INFO 13060 --- [  JMX connector] o.a.a.broker.jmx.ManagementContext       : JMX consoles can connect to service:jmx:rmi:///jndi/rmi://localhost:1099/jmxrmi
2019-05-30 08:39:08.154  INFO 13060 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.9 (embedded-broker, ID:DESKTOP-2RB3C1U-56942-1559198348025-0:1) is starting
2019-05-30 08:39:08.160  INFO 13060 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.9 (embedded-broker, ID:DESKTOP-2RB3C1U-56942-1559198348025-0:1) started
2019-05-30 08:39:08.161  INFO 13060 --- [           main] o.apache.activemq.broker.BrokerService   : For help or more information please see: http://activemq.apache.org
2019-05-30 08:39:08.191  INFO 13060 --- [           main] o.a.activemq.broker.TransportConnector   : Connector vm://embedded-broker started
2019-05-30 08:39:08.234  INFO 13060 --- [           main] c.codenotfound.SpringJmsApplicationTest  : Started SpringJmsApplicationTest in 1.994 seconds (JVM running for 3.161)
2019-05-30 08:39:08.591  INFO 13060 --- [           main] com.codenotfound.jms.Sender              : sending message='Hello Spring JMS ActiveMQ!'
2019-05-30 08:39:08.624  INFO 13060 --- [enerContainer-1] com.codenotfound.jms.Receiver            : received message='Hello Spring JMS ActiveMQ!'
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 3.048 s - in com.codenotfound.SpringJmsApplicationTest
2019-05-30 08:39:08.709  INFO 13060 --- [MQ ShutdownHook] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.9 (embedded-broker, ID:DESKTOP-2RB3C1U-56942-1559198348025-0:1) is shutting down
2019-05-30 08:39:08.721  INFO 13060 --- [MQ ShutdownHook] o.a.activemq.broker.TransportConnector   : Connector vm://embedded-broker stopped
2019-05-30 08:39:08.731  INFO 13060 --- [MQ ShutdownHook] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.9 (embedded-broker, ID:DESKTOP-2RB3C1U-56942-1559198348025-0:1) uptime 0.965 seconds
2019-05-30 08:39:08.731  INFO 13060 --- [MQ ShutdownHook] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.9 (embedded-broker, ID:DESKTOP-2RB3C1U-56942-1559198348025-0:1) is shutdown
[INFO]
[INFO] Results:
[INFO]
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  13.707 s
[INFO] Finished at: 2019-05-30T08:39:09+02:00
[INFO] ------------------------------------------------------------------------
{% endhighlight %}

Above test case can also be executed after you [install Apache ActiveMQ]({{ site.url }}/jms-apache-activemq-installation.html) on your local system.

Simply change the <var>'activemq:broker-url'</var> property to point to <var>'tcp://localhost:61616'</var> in case the broker is running on the default URL.

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-jms/tree/master/spring-jms-activemq-hello-world){:target="_blank"}.
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This concludes our example in which we used Spring JMS and Spring Boot in order to connect to ActiveMQ.

If you found this sample useful or have a question you would like to ask.

Leave a comment below!
