---
title: "Spring JMS Integration Gateway Example"
permalink: /spring-jms-integration-gateway-example.html
excerpt: "A detailed step-by-step tutorial on how to connect to a JMS broker using a Spring Integration Gateway and Spring Boot."
date: 2019-01-14
last_modified_at: 2019-05-30
header:
  teaser: "assets/images/spring-jms/spring-jms-integration-gateway.png"
categories: [Spring JMS]
tags: [ActiveMQ, Apache ActiveMQ, Example, Integration Gateway, Maven, Spring, Spring Boot, Spring Integration, Spring JMS, Tutorial]
published: true
---

<img src="{{ site.url }}/assets/images/spring-jms/spring-jms-integration-gateway.png" alt="spring jms integration gateway" class="align-right title-image">

This is the most comprehensive guide on setting up a [Spring JMS Integration Gateway](https://docs.spring.io/spring-integration/docs/5.1.5.RELEASE/reference/html/#jms-inbound-gateway){:target="_blank"}.

The best part?

It includes a working code sample.

Keep reading…

If you want to learn more about Spring JMS - head on over to the [Spring JMS tutorials]({{ site.url }}/spring-jms-tutorials) page.
{: .notice--primary}

## 1. What is a Spring JMS Integration Gateway?

A [Spring Integration Gateway](https://docs.spring.io/spring-integration/docs/5.1.5.RELEASE/reference/html/#gateway){:target="_blank"} is a component that hides the messaging API provided by Spring Integration.

In other words a _Messaging Gateway_ [encapsulates messaging-specific code](https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessagingGateway.html){:target="_blank"} and separates it from the rest of the application.

Gateways provide **request-reply behavior**. This is contrary to [channel adapters](https://docs.spring.io/spring-integration/docs/5.1.5.RELEASE/reference/html/#overview-endpoints-channeladapter){:target="_blank"} that only provide unidirectional send or receive behavior.

## 2. Example Setup

We use the same setup Maven project setup as a previous [Spring JMS Integration example]({{ site.url }}/spring-jms-integration-example.html).

However this time we will build a request/reply flow where the gateway also handles replies.

<img src="{{ site.url }}/assets/images/spring-jms/spring-jms-integration-gateway-example-overview.png" alt="spring jms integration gateway example overview">

There are two outbound channels that connect to the `JmsOutboundGateway`.

The _OutboundRequestChannel_ sends messages to the outbound gateway. This gateway creates JMS messages from Spring Integration messages and sends them to a JMS destination (in this case the request queue). It then handles the JMS reply message and sends it to the _OutboundResponseChannel_.

The `JmsInboundGateway` connects to two inbound channels.

The inbound gateway receives from a JMS destination and sends to the _InboundRequestChannel_. It also handles the reply on the _InboundResponseChannel_ and sends it to the JMS reply destination (in this case the response queue).

The `OrderService` implements a basic request/reply behavior. It listens on the _InboundRequestChannel_ and replies on the _InboundResponseChannel_.

## 3. General Project Overview

We will use the following tools/frameworks:
* Spring JMS 5.1
* Spring Integration 5.1
* Spring Boot 2.1
* ActiveMQ 5.15
* Maven 3.6

Our project has the following directory structure:

<img src="{{ site.url }}/assets/images/spring-jms/spring-jms-integration-gateway-maven-project.png" alt="spring jms integration gateway maven project">

## 4. Configuring the Spring JMS Outbound Gateway

First, we create an `OutboundGatewayConfig` class and annotate it with `@Configuration`.

Define an _OutboundOrderRequestChannel_ as a `DirectChannel` bean. This is the default channel provided by the framework, but you can use any of the [message channels Spring Integration provides](https://docs.spring.io/spring-integration/docs/5.1.5.RELEASE/reference/html/#messaging-channels-section){:target="_blank"}.

The _OutboundOrderResponseChannel_ is defined as a `QueueChannel`. This allows us to receive the response messages using the `receive()` method.

The <var>jmsOutboundGateway</var> [outbound gateway](https://docs.spring.io/spring-integration/docs/5.1.5.RELEASE/reference/html/#jms-outbound-gateway){:target="_blank"} constructor requires a `ConnectionFactory`. We just pass the Bean that [Spring Boot auto-configures]({{ site.url }}/spring-jms-annotations-example.html) for us.

Use `setRequestDestinationName()` to set the destination to which the request JMS messages needs to be sent (<var>request.q</var>). The `setReplyDestinationName()` method is used to specify the JMS destination to which the response JMS messages need to be sent (<var>response.q</var>).

The gateway receives the requests messages from the _OutboundOrderRequestChannel_. The link between the two is made via the `@ServiceActivator`. This annotation allows connecting any Spring-managed Object to an input channel.

The _OutboundOrderResponseChannel_ is specified using `setReplyChannel()`.

{% highlight java %}
package com.codenotfound.jms;

import javax.jms.ConnectionFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.integration.annotation.ServiceActivator;
import org.springframework.integration.channel.DirectChannel;
import org.springframework.integration.channel.QueueChannel;
import org.springframework.integration.jms.JmsOutboundGateway;
import org.springframework.messaging.MessageChannel;

@Configuration
public class OutboundGatewayConfig {

  @Value("${destination.order.request}")
  private String orderRequestDestination;

  @Value("${destination.order.response}")
  private String orderResponseDestination;

  @Bean
  public MessageChannel outboundOrderRequestChannel() {
    return new DirectChannel();
  }

  @Bean
  public MessageChannel outboundOrderResponseChannel() {
    return new QueueChannel();
  }

  @Bean
  @ServiceActivator(inputChannel = "outboundOrderRequestChannel")
  public JmsOutboundGateway jmsOutboundGateway(
      ConnectionFactory connectionFactory) {
    JmsOutboundGateway gateway = new JmsOutboundGateway();
    gateway.setConnectionFactory(connectionFactory);
    gateway.setRequestDestinationName(orderRequestDestination);
    gateway.setReplyDestinationName(orderResponseDestination);
    gateway.setReplyChannel(outboundOrderResponseChannel());

    return gateway;
  }
}
{% endhighlight %}

## 5. Configuring the Spring JMS Inbound Gateway

In the `InboundGatewayConfig` class we configure the [inbound gateway](https://docs.spring.io/spring-integration/docs/5.1.5.RELEASE/reference/html/#jms-inbound-gateway){:target="_blank"}.

We define the _InboundOrderRequestChannel_ and _InboundOrderResponseChannel_ message channels.

Next, we create the `OrderService` Bean that will handle the incoming request messages. It is connected to the _InboundOrderRequestChannel_ using `@ServiceActivator`.

The <var>jmsInboundGateway</var> constructor [requires](https://docs.spring.io/spring-integration/api/org/springframework/integration/jms/JmsInboundGateway.html){:target="_blank"} a `MessageListenerContainer` and a `ChannelPublishingJmsMessageListener`.

On the gateway, we specify the _InboundOrderRequestChannel_ as the request message channel.

> We do not specify a reply-channel. The Gateway auto-creates a temporary, anonymous reply channel, where it listens for the reply.

We create `SimpleMessageListenerContainer` that allows us to receive messages from the request destination (<var>request.q</var>). For more details on the setup check out the [Spring JMS listener example]({{ site.url }}/spring-jms-listener-example.html).

The [ChannelPublishingJmsMessageListener](https://docs.spring.io/spring-integration/api/org/springframework/integration/jms/ChannelPublishingJmsMessageListener.html){:target="_blank"} converts a JMS Message into a Spring Integration Message and then sends that message to a channel. If the `expectReply` value is true, it will also wait for a Spring Integration reply Message and convert that into a JMS reply Message.



> By default the `setExpectReply` property is set to <var>false</var> so don't forget to set it to <var>true</var>!

{% highlight java %}
package com.codenotfound.jms;

import javax.jms.ConnectionFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.integration.annotation.ServiceActivator;
import org.springframework.integration.channel.DirectChannel;
import org.springframework.integration.config.EnableIntegration;
import org.springframework.integration.jms.ChannelPublishingJmsMessageListener;
import org.springframework.integration.jms.JmsInboundGateway;
import org.springframework.jms.listener.SimpleMessageListenerContainer;
import org.springframework.messaging.MessageChannel;

@Configuration
@EnableIntegration
public class InboundGatewayConfig {

  @Value("${destination.order.request}")
  private String orderRequestDestination;

  @Bean
  public MessageChannel inboundOrderRequestChannel() {
    return new DirectChannel();
  }

  @Bean
  public MessageChannel inboundOrderResponseChannel() {
    return new DirectChannel();
  }

  @Bean
  @ServiceActivator(inputChannel = "inboundOrderRequestChannel")
  public OrderService orderService() {
    return new OrderService();
  }

  @Bean
  public JmsInboundGateway jmsInboundGateway(
      ConnectionFactory connectionFactory) {
    JmsInboundGateway gateway = new JmsInboundGateway(
        simpleMessageListenerContainer(connectionFactory),
        channelPublishingJmsMessageListener());
    gateway.setRequestChannel(inboundOrderRequestChannel());

    return gateway;
  }

  @Bean
  public SimpleMessageListenerContainer simpleMessageListenerContainer(
      ConnectionFactory connectionFactory) {
    SimpleMessageListenerContainer container =
        new SimpleMessageListenerContainer();
    container.setConnectionFactory(connectionFactory);
    container.setDestinationName(orderRequestDestination);
    return container;
  }

  @Bean
  public ChannelPublishingJmsMessageListener channelPublishingJmsMessageListener() {
    ChannelPublishingJmsMessageListener channelPublishingJmsMessageListener =
        new ChannelPublishingJmsMessageListener();
    channelPublishingJmsMessageListener.setExpectReply(true);

    return channelPublishingJmsMessageListener;
  }
}
{% endhighlight %}

To wrap up the example we need to create the `OrderService`.

This class receives an order message. It then logs the content and the creates a simple status message using a [MessageBuilder](https://docs.spring.io/spring-integration/api/org/springframework/integration/support/MessageBuilder.html){:target="_blank"}.

In order for the <var>jmsOutboundGateway</var> to be able to [correlate a response to a previous request](https://docs.spring.io/spring-integration/docs/5.1.5.RELEASE/reference/html/#gateway-reply-correlation){:target="_blank"} we set the <var>jms_correlationId</var> property.

We also specify the <var>inboundOrderResponseChannel</var> as the message channel to which responses need to be sent.

{% highlight java %}
package com.codenotfound.jms;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.messaging.Message;
import org.springframework.messaging.support.MessageBuilder;

public class OrderService {

  private static final Logger LOGGER =
      LoggerFactory.getLogger(OrderService.class);

  public Message<?> order(Message<?> order) {
    LOGGER.info("received order='{}'", order);

    Message<?> status = MessageBuilder.withPayload("Accepted")
        .setHeader("jms_correlationId",
            order.getHeaders().get("jms_messageId"))
        .setReplyChannelName("inboundOrderResponseChannel").build();
    LOGGER.info("sending status='{}'", status);

    return status;
  }
}
{% endhighlight %}

## 6. Testing the Spring JMS Integration Gateways

Now that we have configured both gateways it is time to test them.

Create a `SpringJmsApplicationTest` unit test case. Then, use the `ApplicationContext` to get a reference to the _OutboundOrderRequestChannel_.

Send in a Spring Integration message and then wait on the _OutboundOrderResponseChannel_ for the response to arrive.

{% highlight java %}
package com.codenotfound.jms;

import static org.assertj.core.api.Assertions.assertThat;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.context.ApplicationContext;
import org.springframework.integration.channel.QueueChannel;
import org.springframework.messaging.MessageChannel;
import org.springframework.messaging.support.GenericMessage;
import org.springframework.test.annotation.DirtiesContext;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
@DirtiesContext
public class SpringJmsApplicationTest {

  @Autowired
  private ApplicationContext applicationContext;

  @Test
  public void testIntegrationGateway() {
    MessageChannel outboundOrderRequestChannel =
        applicationContext.getBean("outboundOrderRequestChannel",
            MessageChannel.class);
    QueueChannel outboundOrderResponseChannel = applicationContext
        .getBean("outboundOrderResponseChannel", QueueChannel.class);

    outboundOrderRequestChannel
        .send(new GenericMessage<>("order-001"));

    assertThat(
        outboundOrderResponseChannel.receive(5000).getPayload())
            .isEqualTo("Accepted");;
  }
}
{% endhighlight %}

Open a command prompt in the root directory and start the test case using below Maven command:

{% highlight plaintext %}
mvn test
{% endhighlight %}

In the output logs, we can see that the order message is received by the inbound gateway.

A <var>TemporaryReplyChannel</var> is used to receive the response from the `OrderService`.

The flow ends with the status message being received by the outbound gateway.

{% highlight plaintext %}
 .   ____          _            __ _ _
/\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
\\/  ___)| |_)| | | | | || (_| |  ) ) ) )
'  |____| .__|_| |_|_| |_\__, | / / / /
=========|_|==============|___/=/_/_/_/
:: Spring Boot ::        (v2.1.5.RELEASE)

2019-05-30 17:10:10.203  INFO 9624 --- [           main] c.c.jms.SpringJmsApplicationTest         : Starting SpringJmsApplicationTest on DESKTOP-2RB3C1U with PID 9624 (started by Codenotfound in C:\Users\Codenotfound\repos\spring-jms\spring-jms-integration-gateway)
2019-05-30 17:10:10.204  INFO 9624 --- [           main] c.c.jms.SpringJmsApplicationTest         : No active profile set, falling back to default profiles: default
2019-05-30 17:10:10.891  INFO 9624 --- [           main] faultConfiguringBeanFactoryPostProcessor : No bean named 'errorChannel' has been explicitly defined. Therefore, a default PublishSubscribeChannel will be created.
2019-05-30 17:10:10.897  INFO 9624 --- [           main] faultConfiguringBeanFactoryPostProcessor : No bean named 'taskScheduler' has been explicitly defined. Therefore, a default ThreadPoolTaskScheduler will be created.
2019-05-30 17:10:10.903 DEBUG 9624 --- [           main] faultConfiguringBeanFactoryPostProcessor : SpEL function '#xpath' isn't registered: there is no spring-integration-xml.jar on the classpath.
2019-05-30 17:10:10.909  INFO 9624 --- [           main] faultConfiguringBeanFactoryPostProcessor : No bean named 'integrationHeaderChannelRegistry' has been explicitly defined. Therefore, a default DefaultHeaderChannelRegistry will be created.
2019-05-30 17:10:11.009  INFO 9624 --- [           main] trationDelegate$BeanPostProcessorChecker : Bean 'integrationDisposableAutoCreatedBeans' of type [org.springframework.integration.config.annotation.Disposables] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
2019-05-30 17:10:11.046  INFO 9624 --- [           main] trationDelegate$BeanPostProcessorChecker : Bean 'org.springframework.integration.config.IntegrationManagementConfiguration' of type [org.springframework.integration.config.IntegrationManagementConfiguration$$EnhancerBySpringCGLIB$$e7ebec34] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
2019-05-30 17:10:11.906  INFO 9624 --- [           main] o.apache.activemq.broker.BrokerService   : Using Persistence Adapter: MemoryPersistenceAdapter
2019-05-30 17:10:11.992  INFO 9624 --- [  JMX connector] o.a.a.broker.jmx.ManagementContext       : JMX consoles can connect to service:jmx:rmi:///jndi/rmi://localhost:1099/jmxrmi
2019-05-30 17:10:12.093  INFO 9624 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.9 (localhost, ID:DESKTOP-2RB3C1U-59598-1559229011932-0:1) is starting
2019-05-30 17:10:12.098  INFO 9624 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.9 (localhost, ID:DESKTOP-2RB3C1U-59598-1559229011932-0:1) started
2019-05-30 17:10:12.098  INFO 9624 --- [           main] o.apache.activemq.broker.BrokerService   : For help or more information please see: http://activemq.apache.org
2019-05-30 17:10:12.132  INFO 9624 --- [           main] o.a.activemq.broker.TransportConnector   : Connector vm://localhost started
2019-05-30 17:10:12.554  INFO 9624 --- [           main] o.s.s.c.ThreadPoolTaskScheduler          : Initializing ExecutorService 'taskScheduler'
2019-05-30 17:10:12.625 DEBUG 9624 --- [           main] faultConfiguringBeanFactoryPostProcessor :
Spring Integration global properties:

spring.integration.channels.autoCreate=true
spring.integration.taskScheduler.poolSize=10
spring.integration.readOnly.headers=
spring.integration.channels.maxUnicastSubscribers=0x7fffffff
spring.integration.endpoints.noAutoStartup=
spring.integration.messagingTemplate.throwExceptionOnLateReply=false
spring.integration.channels.maxBroadcastSubscribers=0x7fffffff

2019-05-30 17:10:12.633 DEBUG 9624 --- [           main] .s.i.c.GlobalChannelInterceptorProcessor : No global channel interceptors.
2019-05-30 17:10:12.639  INFO 9624 --- [           main] o.s.i.endpoint.EventDrivenConsumer       : Adding {logging-channel-adapter:_org.springframework.integration.errorLogger} as a subscriber to the 'errorChannel' channel
2019-05-30 17:10:12.640  INFO 9624 --- [           main] o.s.i.channel.PublishSubscribeChannel    : Channel 'application.errorChannel' has 1 subscriber(s).
2019-05-30 17:10:12.641  INFO 9624 --- [           main] o.s.i.endpoint.EventDrivenConsumer       : started _org.springframework.integration.errorLogger
2019-05-30 17:10:12.641  INFO 9624 --- [           main] o.s.i.endpoint.EventDrivenConsumer       : Adding {service-activator:inboundGatewayConfig.orderService.serviceActivator} as a subscriber to the 'inboundOrderRequestChannel' channel
2019-05-30 17:10:12.641  INFO 9624 --- [           main] o.s.integration.channel.DirectChannel    : Channel 'application.inboundOrderRequestChannel' has 1 subscriber(s).
2019-05-30 17:10:12.641  INFO 9624 --- [           main] o.s.i.endpoint.EventDrivenConsumer       : started inboundGatewayConfig.orderService.serviceActivator
2019-05-30 17:10:12.642  INFO 9624 --- [           main] o.s.i.endpoint.EventDrivenConsumer       : Adding {jms:outbound-gateway:outboundGatewayConfig.jmsOutboundGateway.serviceActivator} as a subscriber to the 'outboundOrderRequestChannel' channel
2019-05-30 17:10:12.642  INFO 9624 --- [           main] o.s.integration.channel.DirectChannel    : Channel 'application.outboundOrderRequestChannel' has 1 subscriber(s).
2019-05-30 17:10:12.642  INFO 9624 --- [           main] o.s.i.endpoint.EventDrivenConsumer       : started outboundGatewayConfig.jmsOutboundGateway.serviceActivator
2019-05-30 17:10:12.642  INFO 9624 --- [           main] ishingJmsMessageListener$GatewayDelegate : started org.springframework.integration.jms.ChannelPublishingJmsMessageListener$GatewayDelegate@7fda2001
2019-05-30 17:10:12.643  INFO 9624 --- [           main] o.s.i.jms.JmsMessageDrivenEndpoint       : started org.springframework.integration.jms.JmsMessageDrivenEndpoint@162c1dfb
2019-05-30 17:10:12.643  INFO 9624 --- [           main] o.s.integration.jms.JmsInboundGateway    : started jmsInboundGateway
2019-05-30 17:10:12.672  INFO 9624 --- [           main] c.c.jms.SpringJmsApplicationTest         : Started SpringJmsApplicationTest in 2.878 seconds (JVM running for 3.83)
2019-05-30 17:10:13.050 DEBUG 9624 --- [           main] o.s.integration.channel.DirectChannel    : preSend on channel 'outboundOrderRequestChannel', message: GenericMessage [payload=order-001, headers={id=cc373c3d-4828-3ac8-eff0-961cf89896d1, timestamp=1559229013049}]
2019-05-30 17:10:13.051 DEBUG 9624 --- [           main] o.s.integration.jms.JmsOutboundGateway   : jmsOutboundGateway received message: GenericMessage [payload=order-001, headers={id=cc373c3d-4828-3ac8-eff0-961cf89896d1, timestamp=1559229013049}]
2019-05-30 17:10:13.058 DEBUG 9624 --- [           main] o.s.integration.jms.JmsOutboundGateway   : ReplyTo: queue://response.q
2019-05-30 17:10:13.102 DEBUG 9624 --- [ Session Task-1] .i.j.ChannelPublishingJmsMessageListener : converted JMS Message [ActiveMQTextMessage {commandId = 7, responseRequired = true, messageId = ID:DESKTOP-2RB3C1U-59598-1559229011932-4:1:2:1:1, originalDestination = null, originalTransactionId = null, producerId = ID:DESKTOP-2RB3C1U-59598-1559229011932-4:1:2:1, destination = queue://request.q, transactionId = null, expiration = 0, timestamp = 1559229013072, arrival = 0, brokerInTime = 1559229013073, brokerOutTime = 1559229013081, correlationId = null, replyTo = queue://response.q, persistent = true, type = null, priority = 4, groupID = null, groupSequence = 0, targetConsumerId = null, compressed = false, userID = null, content = null, marshalledProperties = null, dataStructure = null, redeliveryCounter = 0, size = 1042, properties = {timestamp=1559229013049}, readOnlyProperties = true, readOnlyBody = true, droppable = false, jmsXGroupFirstForConsumer = false, text = order-001}] to integration Message payload [order-001]
2019-05-30 17:10:13.108 DEBUG 9624 --- [ Session Task-1] o.s.integration.channel.DirectChannel    : preSend on channel 'inboundOrderRequestChannel', message: GenericMessage [payload=order-001, headers={replyChannel=org.springframework.messaging.core.GenericMessagingTemplate$TemporaryReplyChannel@528ed261, jms_redelivered=false, errorChannel=org.springframework.messaging.core.GenericMessagingTemplate$TemporaryReplyChannel@528ed261, jms_replyTo=queue://response.q, jms_destination=queue://request.q, id=5aa2de11-c75d-5cad-347d-cde8edd8add3, priority=4, jms_timestamp=1559229013072, jms_messageId=ID:DESKTOP-2RB3C1U-59598-1559229011932-4:1:2:1:1, timestamp=1559229013108}]
2019-05-30 17:10:13.108 DEBUG 9624 --- [ Session Task-1] o.s.i.handler.ServiceActivatingHandler   : ServiceActivator for [org.springframework.integration.handler.MethodInvokingMessageProcessor@549bfc9a] (inboundGatewayConfig.orderService.serviceActivator.handler) received message: GenericMessage [payload=order-001, headers={replyChannel=org.springframework.messaging.core.GenericMessagingTemplate$TemporaryReplyChannel@528ed261, jms_redelivered=false, errorChannel=org.springframework.messaging.core.GenericMessagingTemplate$TemporaryReplyChannel@528ed261, jms_replyTo=queue://response.q, jms_destination=queue://request.q, id=5aa2de11-c75d-5cad-347d-cde8edd8add3, priority=4, jms_timestamp=1559229013072, jms_messageId=ID:DESKTOP-2RB3C1U-59598-1559229011932-4:1:2:1:1, timestamp=1559229013108}]
2019-05-30 17:10:13.116  INFO 9624 --- [ Session Task-1] com.codenotfound.jms.OrderService        : received order='GenericMessage [payload=order-001, headers={replyChannel=org.springframework.messaging.core.GenericMessagingTemplate$TemporaryReplyChannel@528ed261, jms_redelivered=false, errorChannel=org.springframework.messaging.core.GenericMessagingTemplate$TemporaryReplyChannel@528ed261, jms_replyTo=queue://response.q, jms_destination=queue://request.q, id=5aa2de11-c75d-5cad-347d-cde8edd8add3, priority=4, jms_timestamp=1559229013072, jms_messageId=ID:DESKTOP-2RB3C1U-59598-1559229011932-4:1:2:1:1, timestamp=1559229013108}]'
2019-05-30 17:10:13.116  INFO 9624 --- [ Session Task-1] com.codenotfound.jms.OrderService        : sending status='GenericMessage [payload=Accepted, headers={replyChannel=inboundOrderResponseChannel, jms_correlationId=ID:DESKTOP-2RB3C1U-59598-1559229011932-4:1:2:1:1, id=0457eb0e-6e39-0ba0-e1e9-6ed3e950511c, timestamp=1559229013116}]'
2019-05-30 17:10:13.116 DEBUG 9624 --- [ Session Task-1] o.s.integration.channel.DirectChannel    : postSend (sent=true) on channel 'inboundOrderRequestChannel', message: GenericMessage [payload=order-001, headers={replyChannel=org.springframework.messaging.core.GenericMessagingTemplate$TemporaryReplyChannel@528ed261, jms_redelivered=false, errorChannel=org.springframework.messaging.core.GenericMessagingTemplate$TemporaryReplyChannel@528ed261, jms_replyTo=queue://response.q, jms_destination=queue://request.q, id=5aa2de11-c75d-5cad-347d-cde8edd8add3, priority=4, jms_timestamp=1559229013072, jms_messageId=ID:DESKTOP-2RB3C1U-59598-1559229011932-4:1:2:1:1, timestamp=1559229013108}]
2019-05-30 17:10:13.117 DEBUG 9624 --- [ Session Task-1] .i.j.ChannelPublishingJmsMessageListener : Reply destination: queue://response.q
2019-05-30 17:10:13.127 DEBUG 9624 --- [           main] o.s.integration.jms.JmsOutboundGateway   : converted JMS Message [ActiveMQTextMessage {commandId = 10, responseRequired = true, messageId = ID:DESKTOP-2RB3C1U-59598-1559229011932-4:1:1:1:1, originalDestination = null, originalTransactionId = null, producerId = ID:DESKTOP-2RB3C1U-59598-1559229011932-4:1:1:1, destination = queue://response.q, transactionId = null, expiration = 0, timestamp = 1559229013119, arrival = 0, brokerInTime = 1559229013119, brokerOutTime = 1559229013121, correlationId = ID:DESKTOP-2RB3C1U-59598-1559229011932-4:1:2:1:1, replyTo = queue://response.q, persistent = true, type = null, priority = 4, groupID = null, groupSequence = 0, targetConsumerId = null, compressed = false, userID = null, content = null, marshalledProperties = null, dataStructure = null, redeliveryCounter = 0, size = 1040, properties = {priority=4, timestamp=1559229013117}, readOnlyProperties = true, readOnlyBody = true, droppable = false, jmsXGroupFirstForConsumer = false, text = Accepted}] to integration Message payload [Accepted]
2019-05-30 17:10:13.129 DEBUG 9624 --- [           main] o.s.integration.channel.QueueChannel     : preSend on channel 'outboundOrderResponseChannel', message: GenericMessage [payload=Accepted, headers={jms_redelivered=false, jms_replyTo=queue://response.q, jms_destination=queue://response.q, jms_correlationId=ID:DESKTOP-2RB3C1U-59598-1559229011932-4:1:2:1:1, id=e64b09f8-fe5b-e612-7d44-bd1d118cfdc9, priority=4, jms_timestamp=1559229013119, jms_messageId=ID:DESKTOP-2RB3C1U-59598-1559229011932-4:1:1:1:1, timestamp=1559229013129}]
2019-05-30 17:10:13.130 DEBUG 9624 --- [           main] o.s.integration.channel.QueueChannel     : postSend (sent=true) on channel 'outboundOrderResponseChannel', message: GenericMessage [payload=Accepted, headers={jms_redelivered=false, jms_replyTo=queue://response.q, jms_destination=queue://response.q, jms_correlationId=ID:DESKTOP-2RB3C1U-59598-1559229011932-4:1:2:1:1, id=e64b09f8-fe5b-e612-7d44-bd1d118cfdc9, priority=4, jms_timestamp=1559229013119, jms_messageId=ID:DESKTOP-2RB3C1U-59598-1559229011932-4:1:1:1:1, timestamp=1559229013129}]
2019-05-30 17:10:13.130 DEBUG 9624 --- [           main] o.s.integration.channel.DirectChannel    : postSend (sent=true) on channel 'outboundOrderRequestChannel', message: GenericMessage [payload=order-001, headers={id=cc373c3d-4828-3ac8-eff0-961cf89896d1, timestamp=1559229013049}]
2019-05-30 17:10:13.131 DEBUG 9624 --- [           main] o.s.integration.channel.QueueChannel     : postReceive on channel 'outboundOrderResponseChannel', message: GenericMessage [payload=Accepted, headers={jms_redelivered=false, jms_replyTo=queue://response.q, jms_destination=queue://response.q, jms_correlationId=ID:DESKTOP-2RB3C1U-59598-1559229011932-4:1:2:1:1, id=e64b09f8-fe5b-e612-7d44-bd1d118cfdc9, priority=4, jms_timestamp=1559229013119, jms_messageId=ID:DESKTOP-2RB3C1U-59598-1559229011932-4:1:1:1:1, timestamp=1559229013129}]
2019-05-30 17:10:13.198  INFO 9624 --- [           main] ishingJmsMessageListener$GatewayDelegate : stopped org.springframework.integration.jms.ChannelPublishingJmsMessageListener$GatewayDelegate@7fda2001
2019-05-30 17:10:13.199  INFO 9624 --- [           main] o.s.i.jms.JmsMessageDrivenEndpoint       : stopped org.springframework.integration.jms.JmsMessageDrivenEndpoint@162c1dfb
2019-05-30 17:10:13.199  INFO 9624 --- [           main] o.s.integration.jms.JmsInboundGateway    : stopped jmsInboundGateway
2019-05-30 17:10:13.200  INFO 9624 --- [           main] o.s.i.endpoint.EventDrivenConsumer       : Removing {logging-channel-adapter:_org.springframework.integration.errorLogger} as a subscriber to the 'errorChannel' channel
2019-05-30 17:10:13.200  INFO 9624 --- [           main] o.s.i.channel.PublishSubscribeChannel    : Channel 'application.errorChannel' has 0 subscriber(s).
2019-05-30 17:10:13.200  INFO 9624 --- [           main] o.s.i.endpoint.EventDrivenConsumer       : stopped _org.springframework.integration.errorLogger
2019-05-30 17:10:13.200  INFO 9624 --- [           main] o.s.i.endpoint.EventDrivenConsumer       : Removing {service-activator:inboundGatewayConfig.orderService.serviceActivator} as a subscriber to the 'inboundOrderRequestChannel' channel
2019-05-30 17:10:13.200  INFO 9624 --- [           main] o.s.integration.channel.DirectChannel    : Channel 'application.inboundOrderRequestChannel' has 0 subscriber(s).
2019-05-30 17:10:13.200  INFO 9624 --- [           main] o.s.i.endpoint.EventDrivenConsumer       : stopped inboundGatewayConfig.orderService.serviceActivator
2019-05-30 17:10:13.200  INFO 9624 --- [           main] o.s.i.endpoint.EventDrivenConsumer       : Removing {jms:outbound-gateway:outboundGatewayConfig.jmsOutboundGateway.serviceActivator} as a subscriber to the 'outboundOrderRequestChannel' channel
2019-05-30 17:10:13.200  INFO 9624 --- [           main] o.s.integration.channel.DirectChannel    : Channel 'application.outboundOrderRequestChannel' has 0 subscriber(s).
2019-05-30 17:10:13.200  INFO 9624 --- [           main] o.s.i.endpoint.EventDrivenConsumer       : stopped outboundGatewayConfig.jmsOutboundGateway.serviceActivator
2019-05-30 17:10:13.201  INFO 9624 --- [           main] o.s.s.c.ThreadPoolTaskScheduler          : Shutting down ExecutorService 'taskScheduler'
2019-05-30 17:10:13.207  INFO 9624 --- [           main] o.a.activemq.broker.TransportConnector   : Connector vm://localhost stopped
2019-05-30 17:10:13.207  INFO 9624 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.9 (localhost, ID:DESKTOP-2RB3C1U-59598-1559229011932-0:1) is shutting down
2019-05-30 17:10:13.213  INFO 9624 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.9 (localhost, ID:DESKTOP-2RB3C1U-59598-1559229011932-0:1) uptime 1.448 seconds
2019-05-30 17:10:13.213  INFO 9624 --- [           main] o.apache.activemq.broker.BrokerService   : Apache ActiveMQ 5.15.9 (localhost, ID:DESKTOP-2RB3C1U-59598-1559229011932-0:1) is shutdown
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 4.186 s - in com.codenotfound.jms.SpringJmsApplicationTest
[INFO]
[INFO] Results:
[INFO]
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  7.691 s
[INFO] Finished at: 2019-05-30T17:10:13+02:00
[INFO] ------------------------------------------------------------------------
{% endhighlight %}

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-jms/tree/master/spring-jms-integration-gateway){:target="_blank"}.
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

In this tutorial, we build an end-to-end messaging example that uses an outbound and inbound Spring JMS Integration gateway.

Drop a line below in case of any questions.

Thanks!
