---
title: JMS - Message Structure Overview
permalink: /2014/01/jms-message-structure-overview.html
excerpt: A detailed overview on the different parts of a JMS message.
date: 2014-01-13 21:00
categories: [JMS]
tags: [Format, Java Message Service, JMS, Message, Properties, Structure]
redirect_from:
  - /2014/01/jms-message-structure.html
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/jms-logo.png" alt="jms logo" class="logo">
</figure>

The basic structure of a JMS message consists out of three parts: **headers**, **properties** and **body**. In this blog post we will explore the different elements of the JMS message format and explain their use. For an exhaustive overview please check the [Java Message Service Concepts](http://docs.oracle.com/javaee/6/tutorial/doc/bncdq.html) chapter of The Java EE 6 Tutorial.

The JMS API defines the standard form of a JMS message, which should be portable across all JMS providers. The picture below illustrates the high level structure of a JMS message.

<figure>
    <img src="{{ site.url }}/assets/images/jms/jms-message-structure.png" alt="jms message structure">
</figure>

# JMS Message Headers

The JMS message header part, contains a number of predefined fields that must be present in every JMS message. Most of the values in the header are set by the JMS provider (which overrides any client-set values) when the message is put on a JMS destination. The values are used by both clients and providers to identify and route messages.

The table below lists the JMS message header fields, indicates how their values are set and describes the content of each header field.

| Header Field                | Set By                         | Description                                                                                                                                                                                         |
| --------------------------- | ------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
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

# JMS Message Properties

You can create and set properties for JMS messages if you need values in addition to those provided by the header fields. Properties are optional and stored as standard Java name/value pairs. Property fields are most often used for message selection and filtering.

There are three kinds of message properties:
1. **Application-related properties**: A Java application can assign application-related properties, which are set before the message is delivered.
2. **Provider-related properties**: Every JMS provider can define proprietary properties that can be set either by the client or automatically by the provider. Provider-related properties are prefixed with <var>'JMS_'</var> followed by the vendor name and the specific property name; for example: `JMS_IBM_MsgType` or `JMS_SonicMQ_XQ.isMultipart`
3. **Standard properties:** These standardized properties are set by the JMS provider (if supported) when a message is sent. Standard property names start with <var>'JMSX'</var>; for example: `JMSXUserid` or `JMSXDeliveryCount`.

# JMS Message Body

The message body contains the main information that is being exchanged by the JMS message. The JMS API defines five message body formats, also called message types, which allow you to send and receive data in a number of forms. JMS specifies only the interface and does not specify the implementation. This approach allows for vendor-specific implementation and transportation of messages while using a common interface.

| Message Type             | Body Contains                                                                                                                                                                                                                              |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| <var>TextMessage</var>   | A java.lang.String object (for example, the contents of an XML file).                                                                                                                                                                      |
| <var>MapMessage</var>    | A set of name-value pairs, with names as String objects and values as primitive types in the Java programming language. The entries can be accessed sequentially by enumerator or randomly by name. The order of the entries is undefined. |
| <var>BytesMessage</var>  | A stream of uninterpreted bytes. This message type is for literally encoding a body to match an existing message format.                                                                                                                   |
| <var>StreamMessage</var> | A stream of primitive values in the Java programming language, filled and read sequentially.                                                                                                                                               |
| <var>ObjectMessage</var> | A Serializable object in the Java programming language.                                                                                                                                                                                    |

> Some JMS vendor implementations have added additional non-standard messages types; for example SonicMQ provides a **MultipartMessage** message type.

The JMS API provides methods for creating messages of each type and for filling in their contents. For example, to create and send a `TextMessage`, you might use the following statements:

``` java
    TextMessage message = session.createTextMessage();
    message.setText(msg_text);  // msg_text is a String
    producer.send(message);
```

At the consuming end, a message arrives as a generic `Message` object and must be cast to the appropriate message type. You can use one or more getter methods to extract the message contents. The following code fragment uses the `getText()` method:

``` java
    Message m = consumer.receive();
    if (m instanceof TextMessage) {
        TextMessage message = (TextMessage) m;
        System.out.println("Reading message: " + message.getText());
    } else {
        // Handle error
    }
```

---

This concludes the overview of the JMS message structure. If you found this post helpful or have any questions or remarks, please leave a comment.