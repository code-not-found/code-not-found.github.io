---
title: "Spring JMS Artemis Example"
permalink: /spring-jms-artemis-example.html
excerpt: "A detailed step-by-step tutorial on how to connect to Apache ActiveMQ Artemis using Spring JMS and Spring Boot."
date: 2018-12-22
last_modified_at: 2019-05-30
header:
  teaser: "assets/images/spring-jms/spring-jms-artemis.png"
categories: [Spring JMS]
tags: [Artemis, Apache ActiveMQ Artemis, Example, Maven, Spring, Spring Boot, Spring JMS, Tutorial]
published: true
---

<img src="{{ site.url }}/assets/images/spring-jms/spring-jms-artemis.png" alt="spring jms artemis" class="align-right title-image">

In this post I'm going to show you how to connect to [ActiveMQ Artemis](https://activemq.apache.org/artemis/){:target="_blank"} using [Spring JMS](https://docs.spring.io/spring/docs/5.1.7.RELEASE/spring-framework-reference/integration.html#jms){:target="_blank"}.

In fact:

Artemis is the successor of ActiveMQ.

So if you're starting a new project, give it a try.

Let's take a closer look.

If you want to learn more about Spring JMS - head on over to the [Spring JMS tutorials]({{ site.url }}/spring-jms-tutorials) page.
{: .notice--primary}

## 1. What is Apache ActiveMQ Artemis?

Apache ActiveMQ Artemis a JMS Broker that is based on the [HornetQ](https://en.wikipedia.org/wiki/HornetQ){:target="_blank"} code base.

Artemis is a separate product to [ActiveMQ](http://activemq.apache.org/){:target="_blank"}. At the moment of writing the development team is working toward feature parity between ActiveMQ 5.x and Artemis.

The goal is that **Artemis eventually becomes ActiveMQ 6.x**.

In this guide, we will create a _Hello World_ example that receives a greeting message from an Artemis JMS broker using Spring JMS, Spring Boot, and Maven.

## 2. General Project Overview

We will use the following tools/frameworks:
* Spring JMS 5.1
* Spring Boot 2.1
* Artemis 2.6
* Maven 3.6

Our project has the following directory structure:

<img src="{{ site.url }}/assets/images/spring-jms/spring-jms-artemis-maven-project.png" alt="spring jms artemis maven project">

## 3. Maven Setup

We build and run our example using **Maven**. If not already the case, [download and install Apache Maven](https://downlinko.com/download-install-apache-maven-windows.html){:target="_blank"}.

Let's use [Spring Initializr](https://start.spring.io/){:target="_blank"} to generate our Maven project. Make sure to select <var>JMS (Artemis)</var> as a dependency.

<img src="{{ site.url }}/assets/images/spring-jms/spring-jms-artemis-hello-world-initializr.png" alt="spring jms artemis hello world initializr">

Click <var>Generate Project</var> to generate and download the Spring Boot project template. At the root of the project, you'll find a <var>pom.xml</var> file which is the XML representation of the Maven project.

To avoid having to manage the version compatibility of the different Spring dependencies, we will inherit the defaults from the `spring-boot-starter-parent` parent POM.

> You can find back the exact dependency versions in Appendix F of the reference documentation. For Spring Boot 2.1.5 the [Artemis dependency is version 2.6.6](https://docs.spring.io/spring-boot/docs/2.1.5.RELEASE/reference/html/appendix-dependency-versions.html#appendix-dependency-versions){:target="_blank"}.

The generated project contains [Spring Boot Starters](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters){:target="_blank"} that manage the different Spring dependencies.

The `spring-boot-starter-artemis` dependency includes the needed dependencies for using Spring JMS in combination with Artemis.

The `spring-boot-starter-test` includes the dependencies for testing Spring Boot applications with libraries that include [JUnit](http://junit.org/junit4/){:target="_blank"}, [Hamcrest](http://hamcrest.org/JavaHamcrest/){:target="_blank"} and [Mockito](http://site.mockito.org/){:target="_blank"}.

We also add a dependency on `artemis-junit`. It provides tools that allow us to have access to an embedded Artemis server when running our unit test.

In the plugins section, you'll find the [Spring Boot Maven Plugin](https://docs.spring.io/spring-boot/docs/2.1.5.RELEASE/reference/html/build-tool-plugins-maven-plugin.html){:target="_blank"}. `spring-boot-maven-plugin` allows us to build a single, runnable "uber-jar". This is a convenient way to execute and transport code.

Also, the plugin allows you to start the example via a Maven command.

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>spring-jms-artemis-hello-world</artifactId>
  <version>0.0.1-SNAPSHOT</version>

  <name>spring-jms-artemis-hello-world</name>
  <description>Spring JMS Artemis Example</description>
  <url>https://codenotfound.com/spring-jms-artemis-example.html</url>

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
      <artifactId>spring-boot-starter-artemis</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.apache.activemq</groupId>
      <artifactId>artemis-junit</artifactId>
      <version>${artemis.version}</version>
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

The project is for a large part identical to a previous [Spring JMS ActiveMQ example]({{ site.url }}/spring-jms-activemq-example.html). As such we will only detail the changes that are needed to connect to Artemis.

## 4. Create a Spring JMS Message Producer

We still use an `ActiveMQConnectionFactory` but this time it is part of the `org.apache.activemq.artemis.jms.client` package.

We pass a <var>brokerUrl</var> to the constructor as shown below.

The value is specified in the <var>application.yml</var> properties file located under <var>src/main/resources</var>.

{% highlight java %}
package com.codenotfound.jms;

import org.apache.activemq.artemis.jms.client.ActiveMQConnectionFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.jms.connection.CachingConnectionFactory;
import org.springframework.jms.core.JmsTemplate;

@Configuration
public class SenderConfig {

  @Value("${artemis.broker-url}")
  private String brokerUrl;

  @Bean
  public ActiveMQConnectionFactory senderActiveMQConnectionFactory() {
    return new ActiveMQConnectionFactory(brokerUrl);
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

## 5. Create a Spring JMS Message Consumer

Similar to the `SenderConfig`, we use the `ActiveMQConnectionFactory` from the `org.apache.activemq.artemis.jms.client` package.

{% highlight java %}
package com.codenotfound.jms;

import org.apache.activemq.artemis.jms.client.ActiveMQConnectionFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.jms.annotation.EnableJms;
import org.springframework.jms.config.DefaultJmsListenerContainerFactory;

@Configuration
@EnableJms
public class ReceiverConfig {

  @Value("${artemis.broker-url}")
  private String brokerUrl;

  @Bean
  public ActiveMQConnectionFactory receiverActiveMQConnectionFactory() {
    return new ActiveMQConnectionFactory(brokerUrl);
  }

  @Bean
  public DefaultJmsListenerContainerFactory jmsListenerContainerFactory() {
    DefaultJmsListenerContainerFactory factory =
        new DefaultJmsListenerContainerFactory();
    factory
        .setConnectionFactory(receiverActiveMQConnectionFactory());
    factory.setConcurrency("3-10");

    return factory;
  }

  @Bean
  public Receiver receiver() {
    return new Receiver();
  }
}
{% endhighlight %}

And that's it! We can now test our connection to an Artemis JMS broker.

## 6. Testing the JMS listener

The [artemis-junit](https://activemq.apache.org/artemis/docs/latest/unit-testing.html){:target="_blank"} package provides some JUnit rules. These make it easy to start a server for our tests.

Use the `@Rule` annotation to create an `EmbeddedJMSResource` that will run an Artemis server.

Then use the `Sender` to send a message.

The `getLatch()` on the `Receiver` allows us to check if the message was received.

{% highlight java %}
package com.codenotfound;

import static org.assertj.core.api.Assertions.assertThat;
import java.util.concurrent.TimeUnit;
import org.apache.activemq.artemis.junit.EmbeddedJMSResource;
import org.junit.Rule;
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

  @Rule
  public EmbeddedJMSResource resource = new EmbeddedJMSResource();

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

Let's run the unit test to check if everything is working.

Open a command prompt in the root directory and execute the following Maven command.

{% highlight plaintext %}
mvn test
{% endhighlight %}

In the output logs, we can see that the <var>Hello Spring JMS ActiveMQ!</var> greeting is received.

{% highlight plaintext %}
.   ____          _            __ _ _
/\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
\\/  ___)| |_)| | | | | || (_| |  ) ) ) )
'  |____| .__|_| |_|_| |_\__, | / / / /
=========|_|==============|___/=/_/_/_/
:: Spring Boot ::        (v2.1.5.RELEASE)

2019-05-30 10:37:50.255  INFO 9420 --- [           main] c.codenotfound.SpringJmsApplicationTest  : Starting SpringJmsApplicationTest on DESKTOP-2RB3C1U with PID 9420 (started by Codenotfound in C:\Users\Codenotfound\repos\spring-jms\spring-jms-artemis-hello-world)
2019-05-30 10:37:50.256  INFO 9420 --- [           main] c.codenotfound.SpringJmsApplicationTest  : No active profile set, falling back to default profiles: default
2019-05-30 10:37:51.677  INFO 9420 --- [           main] c.codenotfound.SpringJmsApplicationTest  : Started SpringJmsApplicationTest in 1.777 seconds (JVM running for 2.949)
2019-05-30 10:37:51.689  INFO 9420 --- [           main] o.a.a.artemis.junit.EmbeddedJMSResource  : Starting EmbeddedJMSResource: embedded-jms-server
2019-05-30 10:37:51.690  INFO 9420 --- [           main] o.a.a.artemis.junit.EmbeddedJMSResource  : Starting EmbeddedJMSResource: embedded-jms-server
2019-05-30 10:37:51.825  INFO 9420 --- [           main] org.apache.activemq.artemis.core.server  : AMQ221000: live Message Broker is starting with configuration Broker Configuration (clustered=false,journalDirectory=data/journal,bindingsDirectory=data/bindings,largeMessagesDirectory=data/largemessages,pagingDirectory=data/paging)
2019-05-30 10:37:51.838  INFO 9420 --- [           main] org.apache.activemq.artemis.core.server  : AMQ221045: libaio is not available, switching the configuration into NIO
2019-05-30 10:37:51.853  INFO 9420 --- [           main] org.apache.activemq.artemis.core.server  : AMQ221057: Global Max Size is being adjusted to 1/2 of the JVM max size (-Xmx). being defined as 1.067.450.368
2019-05-30 10:37:51.871  INFO 9420 --- [           main] org.apache.activemq.artemis.core.server  : AMQ221043: Protocol module found: [artemis-server]. Adding protocol support for: CORE
2019-05-30 10:37:51.934  INFO 9420 --- [           main] org.apache.activemq.artemis.core.server  : AMQ221007: Server is now live
2019-05-30 10:37:51.935  INFO 9420 --- [           main] org.apache.activemq.artemis.core.server  : AMQ221001: Apache ActiveMQ Artemis Message Broker version 2.6.4 [embedded-jms-server, nodeID=364c2807-82b6-11e9-83ff-bc5ff48510d9]
2019-05-30 10:37:52.266  INFO 9420 --- [           main] com.codenotfound.jms.Sender              : sending message='Hello Spring JMS ActiveMQ!'
2019-05-30 10:37:52.501  WARN 9420 --- [mpl$5@2373ad99)] org.apache.activemq.artemis.core.server  : AMQ222165: No Dead Letter Address configured for queue helloworld.q in AddressSettings
2019-05-30 10:37:52.502  WARN 9420 --- [mpl$5@2373ad99)] org.apache.activemq.artemis.core.server  : AMQ222166: No Expiry Address configured for queue helloworld.q in AddressSettings
2019-05-30 10:37:56.709  INFO 9420 --- [enerContainer-4] com.codenotfound.jms.Receiver            : received message='Hello Spring JMS ActiveMQ!'
2019-05-30 10:37:56.750  INFO 9420 --- [           main] o.a.a.artemis.junit.EmbeddedJMSResource  : Stopping EmbeddedJMSResource: embedded-jms-server
2019-05-30 10:37:56.751  INFO 9420 --- [           main] o.a.a.artemis.junit.EmbeddedJMSResource  : Stopping EmbeddedJMSResource: embedded-jms-server
2019-05-30 10:37:56.785  INFO 9420 --- [           main] org.apache.activemq.artemis.core.server  : AMQ221002: Apache ActiveMQ Artemis Message Broker version 2.6.4 [364c2807-82b6-11e9-83ff-bc5ff48510d9] stopped, uptime 4.974 seconds
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 7.816 s - in com.codenotfound.SpringJmsApplicationTest
[INFO]
[INFO] Results:
[INFO]
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  12.435 s
[INFO] Finished at: 2019-05-30T10:37:57+02:00
[INFO] ------------------------------------------------------------------------
{% endhighlight %}

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-jms/tree/master/spring-jms-artemis-hello-world){:target="_blank"}.
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

In this guide, we showed how to use Spring JMS to connect to ActiveMQ Artemis.

Drop a line below if this example was useful.

Or if you have a question.

Thanks!
