---
title: JMS - ActiveMQ Message Priority Example
permalink: /2014/01/jms-activemq-message-priority-example.html
excerpt: A detailed example using ActiveMQ that shows how to specify a priority level when sending a JMS message to a queue.
date: 2014-01-18 21:00
categories: [JMS]
tags: [Apache ActiveMQ, ActiveMQ, Example, Java Message Service, JMS, Level, Message, Priority, Queue]
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/jms-logo.png" alt="jms logo">
</figure>

Priority levels are a powerful instrument on JMS messages which allow building robust applications where for example peak traffic will not block important messages (set with a higher priority) from getting through the queue. The following post explains the basics of JMS priority and illustrates them with a code sample using ActiveMQ and Maven.

Tools used:
* ActiveMq 5.13
* Maven 3

# Setting JMS Message Priority Levels

Message priority levels can be used to instruct the JMS provider to deliver urgent messages first. The message’s priority is contained in the [JMSPriority header](http://codenotfound.com/2014/01/jms-message-structure-overview.html). There are ten levels of priority, ranging from 0 (lowest) to 9 (highest). If you do not specify a priority level, the default level is set to 4.

You can set the priority level in either of two ways:
1. You can use the `setPriority()` method of the `MessageProducer` interface to set the priority level for all messages sent by that producer. For example, the following call sets a priority level of '7' for a producer:

``` java
    producer.setPriority(7);
```

2. You can use the long form of the `send()` or the `publish()` method to set the priority level for a specific message. The third argument sets the priority level. For example, the following send call sets the priority level for message to '3':

``` java
    producer.send(message, DeliveryMode.NON_PERSISTENT, 3, 0);
```

> Setting the priority directly on the JMS Message using the `setJMSPriority()` method of the `Message` interface does not work as in that case the priority of the producer is taken!

It is very interesting to have a look at what the [Java Message Service 1.1 Specification](http://www.oracle.com/technetwork/java/docs-136352.html) has to say about priority and the different levels as it gives a better understanding of what can be achieved when used:

``` plaintext
JMS defines a ten-level priority value, with 0 as the lowest priority and 9 as 
the highest. In addition, clients should consider priorities 0-4 as gradations 
of normal priority and priorities 5-9 as gradations of expedited priority.

JMS does not require that a provider strictly implement priority ordering of 
messages; however, it should do its best to deliver expedited messages ahead of 
normal messages.
```

> When implementing JMS priority it is important to realize that correct configuration of the JMS provider and consumers/producers is key in getting higher-priority messages delivered before lower-priority ones. It is also important to note that the JMS specification does not mandate a provider to implement a strict priority ordering of messages.



