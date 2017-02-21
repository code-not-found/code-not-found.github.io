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

The basic structure of a JMS message consists out of three parts: **headers**, **properties** and **body*. In this blog post we will explore the different elements of the JMS message format and explain their use. For an exhaustive overview please check the [Java Message Service Concepts](http://docs.oracle.com/javaee/6/tutorial/doc/bncdq.html) chapter of The Java EE 6 Tutorial.

The JMS API defines the standard form of a JMS message, which should be portable across all JMS providers. The picture below illustrates the high level structure of a JMS message.

<figure>
    <img src="{{ site.url }}/assets/images/jms/jms_message_structure.png" alt="jms message structure">
</figure>

# JMS Message Headers

The JMS message header part, contains a number of predefined fields that must be present in every JMS message. Most of the values in the header are set by the JMS provider (which overrides any client-set values) when the message is put on a JMS destination. The values are used by both clients and providers to identify and route messages.

The table below lists the JMS message header fields, indicates how their values are set and describes the content of each header field.



