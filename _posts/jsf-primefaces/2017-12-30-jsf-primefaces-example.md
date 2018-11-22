---
title: "JSF PrimeFaces Example"
permalink: /jsf-primefaces-example.html
excerpt: "A detailed step-by-step tutorial on how to implement a Hello World PrimeFaces for JSF example using Spring Boot and Maven."
date: 2017-12-30
last_modified_at: 2018-11-22
header:
  teaser: "assets/images/jsf-primefaces/jsf-primefaces-example.png"
categories: [PrimeFaces]
tags: [Example, Hello World, JavaServer Faces, JSF, Maven, PrimeFaces, Spring Boot, Tutorial]
redirect_from:
  - /2017/04/jsf-primefaces-spring-boot-example.html
  - /jsf-primefaces-example-spring-boot-maven.html
published: true
---

<img src="{{ site.url }}/assets/images/jsf-primefaces/jsf-primefaces-example.png" alt="jsf primefaces example" class="align-right title-image">

I'm going to show you exactly how to create a [PrimeFaces](http://primefaces.org/){:target="_blank"} _Hello World_ example that uses [Spring Boot](https://spring.io/projects/spring-boot){:target="_blank"} and [Maven](https://maven.apache.org/){:target="_blank"}.

(Step-by-step)

So if you're a PrimeFaces for JSF beginner, **you'll love this guide**.

Ready?

If you want to learn more about PrimeFaces for JSF - head on over to the [JSF PrimeFaces tutorials page]({{ site.url }}/jsf-primefaces-tutorials).
{: .notice--primary}

## 1. What is PrimeFaces for JSF?

[PrimeFaces](https://en.wikipedia.org/wiki/PrimeFaces){:target="_blank"} is an open source component library for [JavaServer Faces](http://www.oracle.com/technetwork/java/javaee/javaserverfaces-139869.html){:target="_blank"} (JSF).

It provides a collection of **mostly visual components (widgets)** that can be used by JSF programmers to build the UI for a web application.

An overview of these widgets can be found at the [PrimeFaces showcase](http://www.primefaces.org/showcase/){:target="_blank"}.

To show PrimeFaces in action, we will build and run a Hello World example using Spring Boot, and Maven.

The example consists out of a basic web page that contains two input fields and a submit button. When you click on the button a greeting popup is displayed that uses the values from the input fields.

## 2. General Project Overview

We will use the following tools/frameworks:
* PrimeFaces 6.2
* JoinFaces 3.3
* Spring Boot 2.1
* Maven 3.5

Our project has the following directory structure:

<img src="{{ site.url }}/assets/images/jsf-primefaces/jsf-primefaces-hello-world-maven-project.png" alt="jsf primefaces hello world maven project">

## 3. Maven Setup

We build and run our example using **Maven**. If not already the case make sure to [download and install Apache Maven](https://downlinko.com/download-install-apache-maven-windows.html){:target="_blank"}.

Shown below is the XML representation of our Maven project in a <var>pom.xml</var> file. It contains the needed dependencies to compile and run our example.

The [JoinFaces](https://github.com/joinfaces/joinfaces#joinfaces){:target="_blank"} project enables JSF usage inside a JAR packaged [Spring Boot](https://projects.spring.io/spring-boot/){:target="_blank"} application.

It can auto-configure PrimeFaces, PrimeFaces Extensions, BootsFaces, ButterFaces, RichFaces, OmniFaces, AngularFaces, Mojarra and MyFaces libraries to run on embedded Tomcat, Jetty or Undertow servlet containers.

JoinFaces imports its dependency versions using [dependency management](https://github.com/spring-projects/spring-boot/wiki/Building-On-Spring-Boot#dependency-management){:target="_blank"}.

To facilitate the management of the different Spring JSF dependencies, [JoinFaces Spring Boot Starters](https://github.com/joinfaces/joinfaces/wiki/JoinFaces-Starters-3.x){:target="_blank"} can be used. These are a set of convenient dependency descriptors that you can include in your application. There are several JoinFaces Starters available: basic starters, utility starters, meta starter, component starters, theme starters, and extra starters.

In this example, we will use the `primefaces-spring-boot-starter` which imports the needed dependencies for PrimeFaces and Spring Boot.

Starting with JoinFaces 3.2.2, the `cdi-api` dependency is [no longer pulled by default](https://github.com/joinfaces/joinfaces/issues/503){:target="_blank"}. As we need the `@Named` annotation, we explicitly define it.

In the plugins section, we include the `spring-boot-maven-plugin` Maven plugin so that we can build a single, runnable "uber-jar". This will also allow us to start the example via a Maven command.

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>jsf-primefaces-hello-world</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>jsf-primefaces-hello-world</name>
  <description>JSF PrimeFaces Hello World Example</description>
  <url>https://codenotfound.com/jsf-primefaces-example.html</url>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.0.RELEASE</version>
    <relativePath /> <!-- lookup parent from repository -->
  </parent>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>1.8</java.version>
    <joinfaces.version>3.3.0-rc2</joinfaces.version>
  </properties>

  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>org.joinfaces</groupId>
        <artifactId>joinfaces-dependencies</artifactId>
        <version>${joinfaces.version}</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
    </dependencies>
  </dependencyManagement>

  <dependencies>
    <dependency>
      <groupId>org.joinfaces</groupId>
      <artifactId>primefaces-spring-boot-starter</artifactId>
    </dependency>

    <dependency>
      <groupId>javax.enterprise</groupId>
      <artifactId>cdi-api</artifactId>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>

</project>
{% endhighlight %}

We create a `SpringPrimeFacesApplication` that contains a `main()` method that uses Spring Boot's `SpringApplication.run()` method to bootstrap Spring to the application. For more information on Spring Boot, we refer to the [Spring Boot getting started guide](https://spring.io/guides/gs/spring-boot/){:target="_blank"}.

{% highlight java %}
package com.codenotfound.primefaces;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringPrimeFacesApplication {

  public static void main(String[] args) {
    SpringApplication.run(SpringPrimeFacesApplication.class, args);
  }
}
{% endhighlight %}

## Creating the PrimeFaces Hello World Example

Define a `HelloWorld` class which is a simple POJO (Plain Old Java Object) that will provide data for the PrimeFaces (JSF) components. The class contains the getters and setters for first and last name fields as well as a method to show a greeting.

We annotated the Bean with `@Named` so that it becomes a [CDI managed bean](https://stackoverflow.com/a/12012663/4201470){:target="_blank"} with an [EL](https://docs.oracle.com/javaee/6/tutorial/doc/gjddd.html){:target="_blank"} name that is accessible by the JSF framework.

{% highlight java %}
package com.codenotfound.primefaces.model;

import javax.inject.Named;

@Named
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
    return "Hello " + firstName + " " + lastName + "!";
  }
}
{% endhighlight %}

The web page that will be shown is a standard JSF page as defined below. It contains some PrimeFaces components which include two <var>&lt;p:inputText&gt;</var> fields. These are used to enter a first and last name.

There is also a <var>&lt;p:dialog&gt;</var> component that shows a greeting message. The dialog is triggered by a <var>&lt;p:commandButton&gt;</var>.

TSo use the PrimeFaces components, the following namespace needs to be declared: `xmlns:p="http://primefaces.org/ui`.

> Note that JSF artifiacts like <var>.xhtml</var> and <var>.jsf</var> files need to be placed under the <var>src/main/resources/META-INF/resources</var> folder.

{% highlight html %}
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"
  xmlns:h="http://java.sun.com/jsf/html"
  xmlns:p="http://primefaces.org/ui">

<h:head>
  <title>PrimeFaces Hello World Example</title>
</h:head>

<h:body>
  <h:form id="helloworld-form">

    <p:panel header="PrimeFaces Hello World Example">
      <h:panelGrid columns="2" cellpadding="4">
        <h:outputText value="First Name: " />
        <p:inputText id="first-name" value="#{helloWorld.firstName}" />

        <h:outputText value="Last Name: " />
        <p:inputText id="last-name" value="#{helloWorld.lastName}" />

        <p:commandButton id="submit" value="Submit"
          update="greeting-panel"
          oncomplete="PF('greetingDialog').show()" />
      </h:panelGrid>
    </p:panel>

    <p:dialog header="Greeting" widgetVar="greetingDialog"
      modal="true" resizable="false">
      <h:panelGrid id="greeting-panel" columns="1" cellpadding="4">
        <h:outputText value="#{helloWorld.showGreeting()}" />
      </h:panelGrid>
    </p:dialog>

  </h:form>
</h:body>
</html>
{% endhighlight %}

## Running the PrimeFaces Hello World Example

In order to run the above example open a command prompt in the project root folder and execute following Maven command:

{% highlight plaintext %}
mvn spring-boot:run
{% endhighlight %}

Maven will download the needed dependencies, compile the code and start an Apache Tomcat instance on which the PrimeFaces application will be deployed.

{% highlight plaintext %}
.   ____          _            __ _ _
/\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
\\/  ___)| |_)| | | | | || (_| |  ) ) ) )
'  |____| .__|_| |_|_| |_\__, | / / / /
=========|_|==============|___/=/_/_/_/
:: Spring Boot ::        (v2.1.0.RELEASE)

2018-11-21 20:12:18.288  INFO 12892 --- [           main] c.c.p.SpringPrimeFacesApplication        : Starting SpringPrimeFacesApplication on DESKTOP-2RB3C1U with PID 12892 (C:\Users\Codenotfound\repos\jsf-primefaces\jsf-primefaces-hello-world\target\classes started by Codenotfound in C:\Users\Codenotfound\repos\jsf-primefaces\jsf-primefaces-hello-world)
2018-11-21 20:12:18.288  INFO 12892 --- [           main] c.c.p.SpringPrimeFacesApplication        : No active profile set, falling back to default profiles: default
2018-11-21 20:12:19.256  INFO 12892 --- [           main] trationDelegate$BeanPostProcessorChecker : Bean 'org.joinfaces.autoconfigure.javaxfaces.JsfBeansAutoConfiguration$Jsf2_3AutoConfiguration' of type [org.joinfaces.autoconfigure.javaxfaces.JsfBeansAutoConfiguration$Jsf2_3AutoConfiguration$$EnhancerBySpringCGLIB$$90749af6] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
2018-11-21 20:12:19.256  INFO 12892 --- [           main] trationDelegate$BeanPostProcessorChecker : Bean 'org.joinfaces.autoconfigure.javaxfaces.JsfBeansAutoConfiguration' of type [org.joinfaces.autoconfigure.javaxfaces.JsfBeansAutoConfiguration$$EnhancerBySpringCGLIB$$44a31ffc] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
2018-11-21 20:12:19.631  INFO 12892 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2018-11-21 20:12:19.678  INFO 12892 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2018-11-21 20:12:19.678  INFO 12892 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet Engine: Apache Tomcat/9.0.12
2018-11-21 20:12:19.694  INFO 12892 --- [           main] o.a.catalina.core.AprLifecycleListener   : The APR based Apache Tomcat Native library which allows optimal performance in production environments was not found on the java.library.path: [C:\Program Files\Java\jdk1.8.0_181\bin;C:\Windows\Sun\Java\bin;C:\Windows\system32;C:\Windows;C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShell\v1.0\;C:\Windows\System32\OpenSSH\;C:\Go\bin;C:\Users\Codenotfound\AppData\Local\Microsoft\WindowsApps;C:\Program Files\Java\jdk1.8.0_181\bin;C:\Users\Codenotfound\tools\apache-maven-3.5.4\bin;C:\Users\Codenotfound\AppData\Local\GitHubDesktop\bin;C:\Users\Codenotfound\AppData\Local\atom\bin;C:\Users\Codenotfound\go\bin;C:\Users\Codenotfound\AppData\Local\Programs\Microsoft VS Code\bin;C:\Users\Codenotfound\AppData\Local\Programs\Git\cmd;;.]
2018-11-21 20:12:19.975  INFO 12892 --- [           main] org.apache.jasper.servlet.TldScanner     : At least one JAR was scanned for TLDs yet contained no TLDs. Enable debug logging for this logger for a complete list of JARs that were scanned but no TLDs were found in them. Skipping unneeded JARs during scanning can improve startup time and JSP compilation time.
2018-11-21 20:12:20.006  INFO 12892 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2018-11-21 20:12:20.006  INFO 12892 --- [           main] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 1672 ms
2018-11-21 20:12:20.116  INFO 12892 --- [           main] o.s.b.w.servlet.ServletRegistrationBean  : Servlet FacesServlet mapped to [/faces/*, *.jsf, *.faces, *.xhtml]
2018-11-21 20:12:20.131  INFO 12892 --- [           main] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'characterEncodingFilter' to: [/*]
2018-11-21 20:12:20.131  INFO 12892 --- [           main] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'requestContextFilter' to: [/*]
2018-11-21 20:12:20.678  INFO 12892 --- [           main] org.reflections.Reflections              : Reflections took 437 ms to scan 6 urls, producing 757 keys and 4157 values
2018-11-21 20:12:21.116  INFO 12892 --- [           main] j.e.resource.webcontainer.jsf.config     : Initializing Mojarra 2.3.7 ( 20180822-0020 fb5578e991d03fa881315e4c7beb52869a5e664b) for context ''
2018-11-21 20:12:21.334  INFO 12892 --- [           main] j.e.r.webcontainer.jsf.application       : JSF1048: PostConstruct/PreDestroy annotations present.  ManagedBeans methods marked with these annotations will have said annotations processed.
2018-11-21 20:12:22.100  INFO 12892 --- [           main] .w.PostConstructApplicationEventListener : Running on PrimeFaces 6.2
2018-11-21 20:12:22.100  INFO 12892 --- [           main] .a.PostConstructApplicationEventListener : Running on PrimeFaces Extensions 6.2.9
2018-11-21 20:12:22.100  INFO 12892 --- [           main] o.omnifaces.VersionLoggerEventListener   : Using OmniFaces version 1.14.1
2018-11-21 20:12:22.647  INFO 12892 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2018-11-21 20:12:22.662  INFO 12892 --- [           main] c.c.p.SpringPrimeFacesApplication        : Started SpringPrimeFacesApplication in 4.874 seconds (JVM running for 11.745)
{% endhighlight %}

Open a web browser and enter the following URL: [http://localhost:8080/helloworld.xhtml](http://localhost:8080/helloworld.xhtml){:target="_blank"}. The below web page should now be displayed.

<img src="{{ site.url }}/assets/images/jsf-primefaces/jsf-primefaces-hello-world-example.png" alt="jsf primefaces hello world example">

Enter a first and last name and press the <var>Submit</var> button. A pop-up dialog will be shown with a greeting message.

<img src="{{ site.url }}/assets/images/jsf-primefaces/jsf-primefaces-hello-world-example-greeting.png" alt="jsf primefaces hello world example greeting">

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/jsf-primefaces/tree/master/jsf-primefaces-hello-world){:target="_blank"}.
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

In this getting started tutorial you learned how to configure PrimeFaces using JoinFaces, Spring Boot and Maven.

If you found this post helpful or have any questions:

Please leave a comment below.
