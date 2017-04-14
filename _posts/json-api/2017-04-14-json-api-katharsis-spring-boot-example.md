---
title: "JSON API - Katharsis Spring Boot Hello World Example"
permalink: /2017/04/json-api-katharsis-spring-boot-example.html
excerpt: "A detailed step-by-step tutorial on setting up Spring Kafka using Spring Boot autoconfiguration."
date: 2017-04-14
modified: 2017-04-14
categories: [Spring Kafka]
tags: [Autoconfig, Autoconfiguration, Apache Kafka, Example, Maven, Spring, Spring Boot, Spring Kafka, Tutorial]
published: false
---

{% include figure image_path="/assets/images/logos/spring-logo.jpg" alt="spring logo" %}

[JSON API](http://jsonapi.org/) is a specification for building APIs in JSON. It details how clients should request resources to be fetched or modified, and how servers should respond to those requests. JSON API is designed to minimize both the number of requests and the amount of data transmitted between clients and servers.

[Katharsis](http://katharsis.io) is a Java library that implements the JSON API specification. It is an additional layer that can be plugged on top of existing server side Java implementations in order to provide easy HATEOAS support. Katharsis defines resources which can be shared over a RESTful interface and a repository for handling them.

---

The example will be built and run using Apache Maven. In order to use the Katharsis framework we need to include the katharsis-core dependency. As we will be using Spring Boot for running the server part we also need to include katharsis-spring. For the client part the katharsis-client depenency is needed.

As Katharsis aims to have as little dependencies as possible we need to provide a HTTP client library on the classpath that will be picked up automatically. Both [OkHttp](https://square.github.io/okhttp) and [Apache Http Client](https://hc.apache.org/httpcomponents-client-ga/index.html) are supported. For this example we will use okhttp.

Running and testing the example will be done using spring-boot-starter and spring-boot-starter-test. The spring-boot-maven-plugin plugin will allow spinning up an embedded tomcat on which we can explore our RESTful Hello World API.

---

Text for spring boot application

---

We start by defining a simple model that will represent our Greeting resource. It contains an id in addition to the actual greeting content. We also define the constructors and getters/setters for the two fields.

The @JsonApiResource annotation defines a resource. It requires the type parameter to be defined which will be used to form the URI and populate the type field which is passed as part of the JSON message. According to JSON API standard, the name defined in type can be either plural or singular

The @JsonApiId defines a field which will be used as an identifier of the Greeting resource. Each resource requires this annotation to be present on a field which is of primitive type or which type implements the Serializable interface.

---

Modelled resources must be complemented by a corresponding repository implementation. This can be achieved by implementing one of those two repository interfaces:
1. ResourceRepositoryV2 for a resource
2. RelationshipRepositoryV2 resp. BulkRelationshipRepositoryV2 for resource relationships

For our Greeting resource we will create a GreetingRepositoryImpl which extends the ResourceRepositoryBase implmentation of the ResourceRepositoryV2 repository interface. This base repository will be used by Katharsis to operate on the resource. The implementation consists of five basic methods which provide CRUD operation for a resource and two parameters: the first is a type of a resource and the second is the type of the resource’s identifier.

For this example we will use a HashMap to store the greetings and in the constructor we already populate the Map with a greeting that contains <var>'Hello World!</var> as content.

The ResourceRepositoryBase is a base class that takes care of some boiler-plate, like implementing findOne with findAll. An implementation can then look as simple as:











---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-kafka/tree/master/spring-kafka-boot).
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

Using Spring Boot's autoconfiguration we were able to setup a sender and receiver using only a couple of lines of code. Hopefully this example will kick-start your Spring Kafka development. Drop a line below in case something was not clear or just to let me know if everything worked.