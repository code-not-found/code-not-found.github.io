---
title: JMS - Point-to-Point messaging example using ActiveMQ and Maven  
permalink: /2014/10/jms-point-to-point-messaging-example-activemq-maven.html
excerpt: A JMS Point-to-Point messaging example using ActiveMQ and Maven.
date: 2014-10-16 21:00
categories: [JMS]
tags: [ActiveMQ, Apache ActiveMQ, Example, Java Message Service, JMS, Maven, point-to-point, ptp]
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/jms-logo.png" alt="jms logo">
</figure>

A point-to-point (PTP) product or application is built on the concept of message queues, senders and receivers. Each message is addressed to a specific queue and receiving clients extract messages from the queues established to hold their messages. Queues retain all messages sent to them until the messages are consumed. The following post introduces the basic concepts of JMS point-to-point messaging and illustrates them with a code sample using ActiveMQ and Maven.

# Point-to-Point Messaging

<figure>
    <img src="{{ site.url }}/assets/images/jms/jms-point-to-point-messaging" alt="jms point-to-point messaging">
</figure>

PTP messaging has the following characteristics:
* Each message has **only one consumer**.
* A sender and a receiver of a message have **no timing dependencies**. The receiver can fetch the message whether or not it was running when the client sent the message.
* The receiver **acknowledges the successful processing** of a message.

# ActiveMQ Example

Let's illustrate the above characteristics by creating a message producer that sends a message containing a first and last name to a queue. In turn a message consumer will read the message and transform it into a greeting. The code is very similar to the JMS Hello World example but contains a few key differences explained below.

Tools used:
* ActiveMQ 5.10
* Maven 3

The code is built and run using Maven. Specified below is the Maven POM file which contains the needed dependencies for Logback, JUnit and ActiveMQ.
































