---
title: Spring Kafka - JSON Example
permalink: /2017/03/spring-kafka-json-example.html
excerpt: A detailed step-by-step tutorial on how to send/receive JSON messages using Spring Kafka and Spring Boot.
date: 2017-03-21 21:00
categories: [Spring Kafka]
tags: [Apache Kafka, Example, Maven, JSON, JsonDeserializer, JsonSerializer, Spring, Spring Boot, Spring Kafka, Tutorial]
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/spring-logo.png" alt="spring logo">
</figure>

[JSON](http://www.json.org/) (JavaScript Object Notation) is a lightweight data-interchange format that uses human-readable text to transmit data objects. It is built on two structures: a collection of name/value pairs and an ordered list of values. The following tutorial illustrates how to send/receive a Java object as a JSON `byte[]` array to/from Apache Kafka using Spring Kafka, Spring Boot and Maven.

Tools used:
* Spring Boot 1.5
* Spring Kafka 1.1
* Maven 3

[Apache Kafka](https://kafka.apache.org/) stores and transports `Byte` arrays in its topics. It ships with a number of [built in (de)serializers](https://kafka.apache.org/0100/javadoc/org/apache/kafka/common/serialization/Serializer.html) but a JSON one is not included. Luckily for us, [Spring Kafka](https://projects.spring.io/spring-kafka/) includes a support package that contains a [JSON (de)serializer](https://github.com/spring-projects/spring-kafka/tree/master/spring-kafka/src/main/java/org/springframework/kafka/support/serializer) that uses a [Jackson](https://github.com/FasterXML/jackson) `ObjectMapper` under the covers.

We start from a previous [Spring Kafka tutorial]({{ site.url }}/2016/09/spring-kafka-consumer-producer-example.html). The only thing that needs to be added to the Maven POM file is the `spring-boot-starter-web` dependency which will indirectly include the needed `jackson-*` JAR dependencies.




---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-kafka/tree/master/spring-kafka-json).
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This concludes the example on how to send/receive JSON messages using Spring Kafka. If you have some questions or remarks, drop me a line below.