---
title: Spring Kafka - Avro Bijection Example
permalink: /2017/03/spring-kafka-avro-bijection-example.html
excerpt: A detailed step-by-step tutorial on how to implement an Avro Serializer &amp; Deserializer using Twitter Bijection, Spring Kafka and Spring Boot.
date: 2017-03-16 21:00
categories: [Spring Kafka]
tags: [Apache Kafka, Apache Avro, Avro, Bijection, Deserializer, Example, Maven, Serializer, Spring, Spring Boot, Spring Kafka, Tutorial]
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/spring-logo.png" alt="spring logo">
</figure>

[Twitter Bijection](https://github.com/twitter/bijection) is an invertible function library that converts back and forth between two types. It supports a number of types including Apache Avro. In the following tutorial we will configure, build and run an example in which we will send/receive an Avro message to/from Apache Kafka using Bijection, Spring Kafka, Spring Boot and Maven.

Tools used:
* Twitter Bijection 0.9
* Apache Avro 1.8
* Spring Kafka 1.1
* Spring Boot 1.5
* Maven 3

We base this example on a previous [Spring Kafka Avro serializer/deserializer example]({{ site.url }}//2017/03/spring-kafka-apache-avro-example.html) in which we used the Avro API's to serialize and deserialize objects. As we will see further down below, Bijection allows to convert back and forth in fewer lines of code.

The <var>user.avsc</var> schema from the [Avro getting started guide](https://avro.apache.org/docs/current/gettingstartedjava.html#Defining+a+schema) describes the fields and their types of our `User` type.

``` json
{"namespace": "example.avro",
  "type": "record",
  "name": "User",
  "fields": [
    {"name": "name", "type": "string"},
    {"name": "favorite_number",  "type": ["int", "null"]},
    {"name": "favorite_color", "type": ["string", "null"]}
  ]
}
```














---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-kafka/tree/master/spring-kafka-avro-bijection).
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This concludes the example on how to send/receive Avro messages using Twitter Bijection and Spring Kafka. I created this blog post based on a user request so if you found this tutorial useful or would like to see another variation, let me know.