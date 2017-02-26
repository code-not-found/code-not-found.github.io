---
title: Java Message Service
permalink: /jms/
---

The [Java Message Service (JMS) API](http://docs.oracle.com/javaee/6/tutorial/doc/bnceh.html) is a Java Message Oriented Middleware (MOM) API for sending messages between two or more clients. It is a Java API that allows applications to create, send, receive, and read messages. The JMS API enables communication that is loosely coupled, asynchronous and reliable. The current version of the JMS specification is version 1.1.

The below links provide many step-by-step examples on how to use the Java Message Service API.

## Quick Start

A number of quick start examples for JMS:

[JMS Hello World example]({{ site.url }}/2014/10/jms-hello-world-activemq-maven.html)
: A Java Message Server Hello World example that explains the basic JMS concepts. It uses the ActiveMQ JMS provider from Apache and offers a first look at sending and receiving messages.

[JMS Point-to-Point messaging example]({{ site.url }}/2014/10/jms-point-to-point-messaging-example-activemq-maven.html)
: A point-to-point (PTP) product or application is built on the concept of message queues, senders and receivers.

[JMS Publish/Subscribe messaging example]({{ site.url }}/2014/11/jms-publish-subscribe-messaging-example-activemq-maven.html)
: In a publish/subscribe (pub/sub) product or application, clients address messages to a topic, which functions somewhat like a bulletin board.

## Core Concepts

How JMS core concepts work:

[JMS Message Structure]({{ site.url }}/2014/01/jms-message-structure-overview.html)
: An overview on the different parts of a JMS message and how to use them.
[JMS Priority]({{ site.url }}/2014/01/jms-priority-activemq-maven.html)
: Explains the JMS priority concepts and provides a code sample which shows how to specify a priority level when sending a JMS message.
    
## Provider Setup

A number of JMS provider installation tutorials:

[Install ActiveMQ]({{ site.url }}/2014/01/jms-apache-activemq-installation.html)
: A step-by-step tutorial on how to install ActiveMQ on Windows.

[Install RabbitMQ]({{ site.url }}/2014/11/amqp-install-rabbitmq-windows.html)
: A step-by-step tutorial on how to install RabbitMQ on Windows.

[Install HermesJMS]({{ site.url }}/2014/01/jms-hermesjms-download-installation.html)
: HermesJMS is an extensible console that helps you interact with JMS providers. This post contains a step-by-step tutorial on how to install HermesJMS on Windows.


## References

Some useful references when studying JMS:

[The Java EE 6 Tutorial](http://docs.oracle.com/javaee/6/tutorial/doc/bncdq.html)