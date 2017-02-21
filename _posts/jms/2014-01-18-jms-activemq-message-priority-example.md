---
title: JMS - ActiveMQ Message Priority Example
permalink: /2014/01/jms-activemq-message-priority-example.html
excerpt: A detailed overview on the different parts of a JMS message.
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






