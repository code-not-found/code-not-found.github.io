---
title: JMS - Hello World using ActiveMQ 
permalink: /2014/10/jms-hello-world-activemq-maven.html
excerpt: A JMS Hello World example using ActiveMQ and Maven.
date: 2014-10-16 21:00
categories: [JMS]
tags: [ActiveMQ, Apache ActiveMQ, Example, Hello World, Java Message Service, JMS, Maven]
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/jms-logo.png" alt="jms logo">
</figure>

The Java Message Service (JMS) API is a Java Message Oriented Middleware (MOM) API for sending messages between two or more clients. It is a Java API that allows applications to create, send, receive, and read messages. The JMS API enables communication that is loosely coupled, asynchronous and reliable. The current version of the JMS specification is version 1.1. The following post introduces the basic JMS concepts and illustrates them with a JMS Hello World example using ActiveMQ and Maven.

To use JMS, one must have a JMS provider that can manage the sessions, queues and topics. Some examples of known JMS providers are [Apache ActiveMQ](http://activemq.apache.org/), WebSphere MQ from IBM or SonicMQ from Aurea Software. Starting from Java EE version 1.4, a JMS provider has to be contained in all Java EE application servers.

# The JMS API Programming Model

<figure>
    <img src="{{ site.url }}/assets/images/jms/jms-api-programming-model.png" alt="jms api programming model">
</figure>

The basic building blocks of the JMS API programming model are shown above. At the top we have the `ConnectionFactory` object which is the object a client uses to create a connection to a JMS provider. A connection factory encapsulates a set of connection configuration parameters like for example the broker URL. A connection factory is a JMS administered object that is typically created by an administrator and later used by JMS clients.

When you have a `ConnectionFactory` object, you can use it to create a connection. A `Connection` object encapsulates a virtual connection with a JMS provider. For example, a connection could represent an open TCP/IP socket between a client and a provider service daemon. Before an application completes, it must close any connections that were created. Failure to close a connection can cause resources not to be released by the JMS provider.

> Closing a connection also closes its sessions and their message producers/message consumers.

A session is a single-threaded context for producing and consuming messages. A session provides a transactional context with which to group a set of sends and receives into an atomic unit of work. `Session` objects are created on top of connections.

> As mentioned above, it is important to note that everything from a session down is single-threaded!

A `MessageProducer` is an object that is created by a session and used for sending messages to a destination. You use a `Session` object to create a message producer for a destination.

> It is possible to create an unidentified producer by specifying a null `Destination` as argument to the `createProducer()` method. When sending a message, overload the send method with the needed destination as the first parameter.

A `MessageConsumer` is an object that is created by a session and used for receiving messages sent to a destination. After you have created a message consumer it becomes active, and you can use it to receive messages. Message delivery does not begin until you start the connection you created by calling its `start()` method.

> Remember to always to call the `start()` method on the `Connection` object in order to receive messages!

A `Destination` is the object a client uses to specify the target of messages it produces and the source of messages it consumes. In the point-to-point messaging domain, destinations are called queues. In the publish/subscribe messaging domain, destinations are called topics.

For more detailed information please check the [JMS API programming model chapter](http://docs.oracle.com/javaee/6/tutorial/doc/bnceh.html) of the [Java EE 6 tutorial](http://docs.oracle.com/javaee/6/tutorial/doc/index.html).

# ActiveMQ Example

Let's illustrate the above by creating a message producer that sends a message containing a first and last name to a Hello World queue. In turn a message consumer will read the message and transform it into a greeting. The example uses Maven and assumes a default [ActiveMQ message broker is up and running]({{ site.url }}/2014/01/jms-install-activemq-windows.html).

Tools used:
* ActiveMQ 5.10
* Maven 3

First let's look at the below Maven POM file which contains the needed dependencies for Logback, JUnit and ActiveMQ.

``` xml

```



























