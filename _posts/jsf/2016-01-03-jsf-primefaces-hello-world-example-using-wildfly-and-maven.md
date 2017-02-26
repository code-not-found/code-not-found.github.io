---
title: JSF - PrimeFaces Hello World Example using WildFly and Maven
permalink: /2016/01/jsf-primefaces-hello-world-example-using-wildfly-and-maven.html
excerpt: A PrimeFaces 'Hello World' example using WildFly and Maven.
date: 2016-01-03 21:00
categories: [PrimeFaces]
tags: [Example, JavaServer Faces, JSF, Maven, PrimeFaces, WildFly]
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/primefaces-logo.png" alt="primefaces logo">
</figure>

[WildFly](http://wildfly.org/), formerly known as JBoss, is an application server written in Java developed by JBoss. It implements the Java Platform, Enterprise Edition (Java EE) specification. WildFly runs on multiple platforms and is free and open-source software.

The following post illustrates a basic example in which we will configure, build and run a Hello World PrimeFaces example using WildFly and Maven.

Tools used:
* JSF 2.2
* PrimeFaces 5.3
* WildFly 9
* Maven 3

The code is built and run using [Maven](https://maven.apache.org/). Specified below is the Maven POM file which contains the needed dependencies for JSF and PrimeFaces.

In order to run the Hello World PrimeFaces application, a servlet container is needed and in this example the WildFly implementation will be used. The deployment of the code and application server will be fully automated using the [wildfly-maven-plugin](https://docs.jboss.org/wildfly/plugins/maven/latest/) based on a [sample configuration posted on stackoverflow](http://stackoverflow.com/a/29127121/4201470). It takes as input two configuration parameters which point to the server home directory and configuration XML file.

The <var>'maven-dependency-plugin'</var> is used to unpack a distribution of WildFly in the projects build directory. The <var>'overWrite'</var> flag has been specified as false in order to avoid downloading the distribution on each run. The <var>'maven-resources-plugin'</var> is in turn used to overwrite the default server configuration so we can change the default HTTP listening port value as further explained below.

``` xml

```





















