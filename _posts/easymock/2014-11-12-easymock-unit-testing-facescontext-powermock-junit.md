---
title: EasyMock - unit testing FacesContext using PowerMock, JUnit and Maven
permalink: /2014/11/easymock-unit-testing-facescontext-powermock-junit.html
excerpt: A code sample which shows how to unit test FacesContext using EasyMock, PowerMock, JUnit and Maven.
date: 2014-11-12 21:00
categories: [EasyMock]
tags: [Code Sample, EasyMock, FacesContext, JSF, JUnit, Maven, PowerMock, unit testing]
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/easymock-logo.png" alt="easymock logo">
</figure>

JSF defines the `FacesContext` abstract base class for representing all of the contextual information associated with processing an incoming request, and creating the corresponding response. When writing unit test cases for a JSF application there might be a need to mock some of `FacesContext` static methods. The following post will illustrate how to do this using [PowerMock](https://code.google.com/p/powermock/), which is a framework that allows you to extend mock libraries like [EasyMock](http://easymock.org/) with extra capabilities. In this case the capability to mock the static methods of `FacesContext`.

Tools used:
* JUnit 4.11
* EasyMock 3.2
* PowerMock 1.5
* Maven 3

The code sample is built and run using Maven. Specified below is the Maven POM file which contains the needed dependencies for JUnit, EasyMock and PowerMock. In addition the PowerMock support module for JUnit (<var>'powermock-module-junit4'</var>) and the PowerMock API for EasyMock (<var>'powermock-api-easymock'</var>) dependencies need to be added as specified here.

As the `FacesContext` class is used in this code sample, dependencies to the EL (Expression Language) API and JSF specification API are also included.

Note that the version of JUnit is not the latest as there seems to be a bug where [PowerMock doesn't recognize the correct JUnit version when using JUnit 4.12](http://stackoverflow.com/a/26222732/4201470).

``` xml

```

















