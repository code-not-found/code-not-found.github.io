---
title: JSF - PrimeFaces login Servlet Filter example using Jetty and Maven
permalink: /2015/01/jsf-primefaces-login-servlet-filter-example-jetty-maven.html
excerpt: A PrimeFaces login servlet filter example using Jetty and Maven.
date: 2015-01-04 21:00
categories: [PrimeFaces]
tags: [Example, Hello World, JavaServer Faces, Jetty, JSF, Maven, PrimeFaces, Project, Tutorial]
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/primefaces-logo.png" alt="primefaces logo">
</figure>

When creating a Java Server Faces application that needs to ensure only authenticated users can access certain pages, a `Servlet Filter` in combination with a session managed bean could be used to achieve this. The following post illustrates how to implement a basic PrimeFaces login using Jetty and Maven. It is largely based on this excellent [post by BalusC at the stackoverflow forum](https://stackoverflow.com/questions/3841361/jsf-http-session-login/3842060#3842060).

Tools used:
* JSF 2.2
* PrimeFaces 5.1
* Jetty 8
* Maven 3

The code is built and run using Maven. Specified below is the Maven POM file which contains the needed dependencies for JSF and PrimeFaces. In addition all classes contain corresponding unit test cases which use [Mockito and PowerMock for mocking FacesContext]({{ site.url }}/2014/11/mockito-unit-testing-facescontext-powermock-junit.html). The JSF application will run on a Jetty instance launched from from command line using the jetty-maven-plugin. 




















