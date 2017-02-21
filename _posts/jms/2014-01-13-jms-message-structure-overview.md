---
title: JMS - Message Structure Overview
permalink: /2014/01/jms-message-structure-overview.html
excerpt: A detailed overview on the different parts of a JMS message.
date: 2014-01-13 21:00
categories: [JMS]
tags: [Format, Java Message Service, JMS, Message, Properties, Structure]
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/jms-logo.png" alt="jms logo">
</figure>

The basic structure of a JMS message consists out of three parts: **headers**, **properties** and **body**. In this blog post we will explore the different elements of the JMS message format and explain their use. For an exhaustive overview please check the [Java Message Service Concepts](http://docs.oracle.com/javaee/6/tutorial/doc/bncdq.html) chapter of The Java EE 6 Tutorial.

The JMS API defines the standard form of a JMS message, which should be portable across all JMS providers. The picture below illustrates the high level structure of a JMS message.

<figure>
    <img src="{{ site.url }}/assets/images/jms/jms-message-structure.png" alt="jms message structure">
</figure>

# JMS Message Headers

The JMS message header part, contains a number of predefined fields that must be present in every JMS message. Most of the values in the header are set by the JMS provider (which overrides any client-set values) when the message is put on a JMS destination. The values are used by both clients and providers to identify and route messages.

The table below lists the JMS message header fields, indicates how their values are set and describes the content of each header field.

| Header Field                | Set By                         | Description                                                  |
| --------------------------- | ------------------------------ | ------------------------------------------------------------ |
| <var>JMSDestination</var>   | `send()` or `publish()` method | Returns a Destination object (a Topic or a Queue, or their temporary version) describing where the message was directed.                                                                            |
| <var>JMSDeliveryMode</var>  | `send()` or `publish()` method | Can be `DeliveryMode.NON_PERSISTENT` or `DeliveryMode.PERSISTENT`; only persistent messages guarantee delivery in case of a crash of the brokers that transport it.                                 |
| <var>JMSExpiration</var>    | `send()` or `publish()` method | Returns a timestamp indicating the expiration time of the message; it can be 0 on a message without a defined expiration.                                                                           |
| <var>JMSPriority</var>      | `send()` or `publish()` method | Returns a 0-9 integer value (higher is better) defining the priority for delivery. It is only a best-effort value.                                                                                  |
| <var>JMSMessageID</var>     | `send()` or `publish()` method | Contains a generated ID for identifying a message, unique at least for the current broker. All generated IDs start with the prefix 'ID:', but you can override it with the corresponding setter.    |
| <var>JMSTimestamp</var>     | `send()` or `publish()` method | Returns a long indicating the time of sending.                                                                                                                                                      |
| <var>JMSCorrelationID</var> | Client                         | Can link a message with another, usually one that has been sent previously (typically used for a request/response scenario). For example, a reply can carry the ID of the original request message. |
| <var>JMSReplyTo</var>       | Client                         | Is a Destination object where replies should be sent, it can be null.                                                                                                                               |
| <var>JMSType</var>          | Client                         | Defines a field for provider-specific or application-specific message types.                                                                                                                        |
| <var>JMSRedelivered</var>   | JMS provider                   | Returns a boolean indicating if the message is being delivered again after a delivery which was not acknowledge.                                                                                    |















