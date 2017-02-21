---
title: JSF - PrimeFaces Hello World Example 
permalink: /2014/04/jsf-primefaces-hello-world-example.html
excerpt: A PrimeFaces tutorial in which we build and run a Hello World example using Jetty and Maven.
date: 2016-04-12 21:00
categories: [PrimeFaces]
tags: [Example, Hello World, JavaServer Faces, Jetty, JSF, Maven, PrimeFaces, Project, Tutorial]
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/primefaces-logo.png" alt="primefaces logo">
</figure>

[PrimeFaces](http://primefaces.org/) is an open source component library for JavaServer Faces (JSF). It provides a collection of mostly visual components (widgets) that can be used by JSF programmers to build the UI for a web application. An overview of these widgets can be found at the [PrimeFaces showcase](http://www.primefaces.org/showcase/). In the following tutorial we will configure, build and run a Hello World PrimeFaces example using Jetty and Maven.

Tools used:
* JSF 2.2
* PrimeFaces 6.0
* Jetty 9
* Maven 3

First let's look at the below Maven POM file which contains the needed dependencies for our project. At the bottom of the dependencies list we find the PrimeFaces library. As PrimeFaces is built on top of '<var>JavaServer Faces</var>' we also need to include the JSF dependencies. JSF is a component based Model–view–controller (MVC) framework which is built on top of the '<var>Servlet API</var>' so we also need to include the Servlet dependency.

In order to run our example we need a servlet container (the component of a web server that interacts with Java servlets). There are a number of servlet containers implementations available, in the below example we will use Jetty which is a non-commercial pure Java-based HTTP (Web) server and Java Servlet container from the Eclipse Foundation. There is a Jetty Maven plugin which allows launching a Jetty instance from command line using Maven which has been configured so that the HTTP listener port is set to "<kbd>9090</kbd>" and the context path is set to "<kbd>codenotfound</kbd>".

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.codenotfound</groupId>
    <artifactId>jsf-primefaces-hello-world</artifactId>
    <version>1.0</version>
    <packaging>war</packaging>

    <name>JSF - PrimeFaces Hello World Example</name>
    <url>https://codenotfound.com/2014/04/jsf-primefaces-hello-world-example.html</url>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>

        <servlet.version>3.1.0</servlet.version>
        <jsf.version>2.2.13</jsf.version>
        <primefaces.version>6.0</primefaces.version>

        <maven-compiler-plugin.version>3.5.1</maven-compiler-plugin.version>
        <jetty-maven-plugin.version>9.4.0.M0</jetty-maven-plugin.version>
    </properties>

    <dependencies>
        <!-- Servlet -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>${servlet.version}</version>
            <scope>provided</scope>
        </dependency>
        <!-- JSF -->
        <dependency>
            <groupId>com.sun.faces</groupId>
            <artifactId>jsf-api</artifactId>
            <version>${jsf.version}</version>
            <scope>compile</scope>
        </dependency>
        <dependency>
            <groupId>com.sun.faces</groupId>
            <artifactId>jsf-impl</artifactId>
            <version>${jsf.version}</version>
            <scope>compile</scope>
        </dependency>
        <!-- PrimeFaces -->
        <dependency>
            <groupId>org.primefaces</groupId>
            <artifactId>primefaces</artifactId>
            <version>${primefaces.version}</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>${maven-compiler-plugin.version}</version>
                <configuration>
                    <source>${java.version}</source>
                    <target>${java.version}</target>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.eclipse.jetty</groupId>
                <artifactId>jetty-maven-plugin</artifactId>
                <version>${jetty-maven-plugin.version}</version>
                <configuration>
                    <httpConnector>
                        <port>9090</port>
                    </httpConnector>
                    <webApp>
                        <contextPath>/codenotfound</contextPath>
                    </webApp>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

Next is the `HelloWorld` class which is a simple POJO (Plain Old Java Object) that will provide data for the PrimeFaces (JSF) components. It contains the getters and setters for first and last name fields as well as a method to show a greeting.

In JSF, a class that can be accessed from a JSF page is called Managed Bean. By annotating the `HelloWorld` class with the `@ManagedBean` annotation it becomes a Managed Bean which is accessible and controlled by the JSF framework.

``` java
package com.codenotfound.primefaces;

import javax.faces.bean.ManagedBean;

@ManagedBean
public class HelloWorld {

    private String firstName = "John";
    private String lastName = "Doe";

    public String getFirstName() {
        return firstName;
    }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }

    public String getLastName() {
        return lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }

    public String showGreeting() {
        return "Hello" + " " + firstName + " " + lastName + "!";
    }
}
```

The web page that will be shown is a standard JSF page as defined below. It contains a number of PrimeFaces components which include two <p:inputText> fields, that will be used to enter a first and last name, surrounded by a `<p:panel>`. There is also a `<p:dialog>` component that shows a greeting message. The dialog is triggered by a `<p:commandButton>` that is part of the panel.

In order to use the PrimeFaces components, following namespace needs to be declared: `xmlns:p="http://primefaces.org/ui`.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"
    xmlns:h="http://java.sun.com/jsf/html"
    xmlns:p="http://primefaces.org/ui">

<h:head>
    <title>PrimeFaces Hello World Example</title>
</h:head>

<h:body>
    <h:form>

        <p:panel header="PrimeFaces Hello World Example">
            <h:panelGrid columns="2" cellpadding="4">
                <h:outputText value="First Name: " />
                <p:inputText value="#{helloWorld.firstName}" />

                <h:outputText value="Last Name: " />
                <p:inputText value="#{helloWorld.lastName}" />

                <p:commandButton value="Submit" update="greeting"
                    oncomplete="PF('greetingDialog').show()" />
            </h:panelGrid>
        </p:panel>

        <p:dialog header="Greeting" widgetVar="greetingDialog"
            modal="true" resizable="false">
            <h:panelGrid id="greeting" columns="1" cellpadding="4">
                <h:outputText value="#{helloWorld.showGreeting()}" />
            </h:panelGrid>
        </p:dialog>

    </h:form>
</h:body>
</html>
```

Java web applications use a deployment descriptor file to determine how URLs map to servlets and other information. This file is named <file>web.xml</file>, and resides in the application's WAR under the <file>WEB-INF</file> directory. The below <file>web.xml</file> contains the definition of the `FacesServlet` servlet class that will be used to manage the request processing lifecycle of our web page which contains JSF components. The page is mapped to the servlet by defining a mapping for all files ending with '<var>.xhtml</var>'.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
    version="3.1">

    <!-- File(s) appended to a request for a URL that is not mapped to a web 
        component -->
    <welcome-file-list>
        <welcome-file>helloworld.xhtml</welcome-file>
    </welcome-file-list>

    <!-- Define the JSF servlet (manages the request processing life cycle for 
        JavaServer Faces) -->
    <servlet>
        <servlet-name>faces-servlet</servlet-name>
        <servlet-class>javax.faces.webapp.FacesServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <!-- Map following files to the JSF servlet -->
    <servlet-mapping>
        <servlet-name>faces-servlet</servlet-name>
        <url-pattern>*.xhtml</url-pattern>
    </servlet-mapping>
</web-app>
```
















