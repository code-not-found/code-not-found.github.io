---
title: "JSF - PrimeFaces Example using Spring Boot and Maven"
permalink: /jsf-primefaces-example-spring-boot-maven.html
excerpt: "A detailed step-by-step tutorial on how to implement a PrimeFaces Example with Spring Boot and Maven."
date: 2017-12-30
last_modified_at: 2017-12-30
header:
  teaser: "assets/images/teaser/primefaces-teaser.png"
categories: [PrimeFaces]
tags: [Example, Hello World, JavaServer Faces, JSF, Maven, PrimeFaces, Spring Boot, Tutorial]
redirect_from:
  - /2017/04/jsf-primefaces-spring-boot-example.html
published: true
---

<figure>
  <img src="{{ site.url }}/assets/images/logo/primefaces-logo.png" alt="primefaces logo" class="logo">
</figure>

[PrimeFaces](http://primefaces.org/){:target="_blank"} is an open source component library for [JavaServer Faces](http://www.oracle.com/technetwork/java/javaee/javaserverfaces-139869.html){:target="_blank"} (JSF). It provides a collection of mostly visual components (widgets) that can be used by JSF programmers to build the UI for a web application.

An overview of these widgets can be found at the [PrimeFaces showcase](http://www.primefaces.org/showcase/){:target="_blank"}.

In the following tutorial, we will configure, build and run a Hello World example using PrimeFaces, Spring Boot, and Maven.

# General Project Setup

Tools used:
* PrimeFaces 6.2
* JoinFaces 3.2
* Spring Boot 2.0
* Maven 3.5

We will be building and running our example using [Apache Maven](https://maven.apache.org/){:target="_blank"}. Shown below is the XML representation of our Maven project in a POM file. It contains the needed dependencies for compiling and running our example.

The [JoinFaces](https://github.com/joinfaces/joinfaces#joinfaces){:target="_blank"} project enables JSF usage inside a JAR packaged [Spring Boot](https://projects.spring.io/spring-boot/){:target="_blank"} application. It can auto-configure PrimeFaces, PrimeFaces Extensions, BootsFaces, ButterFaces, RichFaces, OmniFaces, AngularFaces, Mojarra and MyFaces libraries to run on embedded Tomcat, Jetty or Undertow servlet containers.

We use the `joinfaces-parent` dependency to define all modules except for the joinfaces dependencies.

To facilitate the management of the different Spring JSF dependencies, [JoinFaces Spring Boot Starters](https://github.com/joinfaces/joinfaces/wiki/JoinFaces-Starters-3.x){:target="_blank"} can be used which are a set of convenient dependency descriptors that you can include in your application. There are sixteen JoinFaces Starters available: six basic starters, two utility starters, one meta starter, five component starters, one theme starter and one extra starter.

In this example, we will use the `primefaces-spring-boot-starter` which imports the needed dependencies for PrimeFaces and Spring Boot.

In the plugins section, we include the `spring-boot-maven-plugin` Maven plugin so that we can build a single, runnable "uber-jar". This will also allow us to start the example via a Maven command.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>jsf-primefaces-spring-boot</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>jsf-primefaces-spring-boot</name>
  <description>JSF - PrimeFaces Example using Spring Boot and Maven</description>
  <url>https://www.codenotfound.com/jsf-primefaces-example-spring-boot-maven.html</url>

  <parent>
    <groupId>org.joinfaces</groupId>
    <artifactId>joinfaces-parent</artifactId>
    <version>3.2.0</version>
    <relativePath /> <!-- lookup parent from repository -->
  </parent>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>1.8</java.version>
  </properties>

  <dependencies>
    <!-- joinfaces -->
    <dependency>
      <groupId>org.joinfaces</groupId>
      <artifactId>primefaces-spring-boot-starter</artifactId>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <!-- spring-boot-maven-plugin -->
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>
</project>
```

We also create a `SpringPrimeFacesApplication` that contains a `main()` method that uses Spring Boot's `SpringApplication.run()` method to bootstrap the application, starting Spring. For more information on Spring Boot, we refer to the [Spring Boot getting started guide](https://spring.io/guides/gs/spring-boot/){:target="_blank"}.

``` java
package com.codenotfound.primefaces;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringPrimeFacesApplication {

  public static void main(String[] args) {
    SpringApplication.run(SpringPrimeFacesApplication.class, args);
  }
}
```

# Creating the PrimeFaces Hello World Example

The `HelloWorld` class which is a simple POJO (Plain Old Java Object) that will provide data for the PrimeFaces (JSF) components. It contains the getters and setters for first and last name fields as well as a method to show a greeting.

We annotated the Bean with `@Named` so that it becomes a CDI managed bean with an EL name that is accessible by the JSF framework.

``` java
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

```

The web page that will be shown is a standard JSF page as defined below. It contains a number of PrimeFaces components which include two <p:inputText> fields, that will be used to enter a first and last name, surrounded by a <var>&lt;p:panel&gt;</var>.

There is also a <var>&lt;p:dialog&gt;</var> component that shows a greeting message. The dialog is triggered by a <var>&lt;p:commandButton&gt;</var> that is part of the panel.

In order to use the PrimeFaces components, the following namespace needs to be declared: `xmlns:p="http://primefaces.org/ui`.

> Note that JSF artifiacts like <var>.xhtml</var> and <var>.jsf</var> files need to be placed at the <var>src/main/resources/META-INF/resources</var> folder.

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
```

# Running the PrimeFaces Hello World Example

In order to run the above example open a command prompt and execute following Maven command:

``` plaintext
mvn spring-boot:run
```

Maven will download the needed dependencies, compile the code and start an Apache Tomcat instance on which the PrimeFaces application will be deployed.

``` plaintext
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.0.1.RELEASE)

2018-04-30 20:49:09.666  INFO 5400 --- [           main] c.c.p.SpringPrimeFacesApplication        : Starting SpringPrimeFacesApplication on cnf-pc with PID 5400 (c:\blogs\codenotfound\code\jsf-primefaces\jsf-primefaces-spring-boot\target\classes started by CodeNotFound in c:\blogs\codenotfound\code\jsf-primefaces\jsf-primefaces-spring-boot)
2018-04-30 20:49:09.670  INFO 5400 --- [           main] c.c.p.SpringPrimeFacesApplication        : No active profile set, falling back to default profiles: default
2018-04-30 20:49:09.746  INFO 5400 --- [           main] ConfigServletWebServerApplicationContext : Refreshing org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@18fffd14: startup date [Mon Apr 30 20:49:09 CEST 2018]; root of context hierarchy
2018-04-30 20:49:10.901  INFO 5400 --- [           main] f.a.AutowiredAnnotationBeanPostProcessor : JSR-330 'javax.inject.Inject' annotation found and supported for autowiring
2018-04-30 20:49:11.327  INFO 5400 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2018-04-30 20:49:11.355  INFO 5400 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2018-04-30 20:49:11.355  INFO 5400 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet Engine: Apache Tomcat/8.5.29
2018-04-30 20:49:11.367  INFO 5400 --- [ost-startStop-1] o.a.catalina.core.AprLifecycleListener   : The APR based Apache Tomcat Native library which allows optimal performance in production environments was not found on the java.library.path: [C:\Program Files\Java\jdk1.8.0_172\bin;C:\Windows\Sun\Java\bin;C:\Windows\system32;C:\Windows;C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShell\v1.0\;C:\Program Files\Java\jdk1.8.0_172\bin;C:\code\tools\apache-maven-3.5.3\bin;.]
2018-04-30 20:49:11.701  INFO 5400 --- [ost-startStop-1] org.apache.jasper.servlet.TldScanner     : At least one JAR was scanned for TLDs yet contained no TLDs. Enable debug logging for this logger for a complete list of JARs that were scanned but no TLDs were found in them. Skipping unneeded JARs during scanning can improve startup time and JSP compilation time.
2018-04-30 20:49:11.717  INFO 5400 --- [ost-startStop-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2018-04-30 20:49:11.718  INFO 5400 --- [ost-startStop-1] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 1976 ms
2018-04-30 20:49:11.942  INFO 5400 --- [ost-startStop-1] o.s.b.w.servlet.ServletRegistrationBean  : Servlet FacesServlet mapped to [/faces/*, *.jsf, *.faces, *.xhtml]
2018-04-30 20:49:11.943  INFO 5400 --- [ost-startStop-1] o.s.b.w.servlet.ServletRegistrationBean  : Servlet dispatcherServlet mapped to [/]
2018-04-30 20:49:11.949  INFO 5400 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'characterEncodingFilter' to: [/*]
2018-04-30 20:49:11.950  INFO 5400 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'hiddenHttpMethodFilter' to: [/*]
2018-04-30 20:49:11.950  INFO 5400 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'httpPutFormContentFilter' to: [/*]
2018-04-30 20:49:11.951  INFO 5400 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'requestContextFilter' to: [/*]
2018-04-30 20:49:12.470  INFO 5400 --- [ost-startStop-1] org.reflections.Reflections              : Reflections took 457 ms to scan 6 urls, producing 752 keys and 4129 values
2018-04-30 20:49:12.809  INFO 5400 --- [ost-startStop-1] j.e.resource.webcontainer.jsf.config     : Initializing Mojarra 2.3.4 ( 20180405-0442 7a05fbdf38c7d725ccac9237e44de6dbc77bee7d) for context ''
2018-04-30 20:49:12.962  INFO 5400 --- [ost-startStop-1] j.e.r.webcontainer.jsf.application       : JSF1048: PostConstruct/PreDestroy annotations present.  ManagedBeans methods marked with these annotations will have said annotations processed.
2018-04-30 20:49:13.552  INFO 5400 --- [ost-startStop-1] .w.PostConstructApplicationEventListener : Running on PrimeFaces 6.2
2018-04-30 20:49:13.553  INFO 5400 --- [ost-startStop-1] .a.PostConstructApplicationEventListener : Running on PrimeFaces Extensions 6.2.4
2018-04-30 20:49:13.553  INFO 5400 --- [ost-startStop-1] o.omnifaces.VersionLoggerEventListener   : Using OmniFaces version 1.14.1
2018-04-30 20:49:13.892  INFO 5400 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**/favicon.ico] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2018-04-30 20:49:14.101  INFO 5400 --- [           main] s.w.s.m.m.a.RequestMappingHandlerAdapter : Looking for @ControllerAdvice: org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@18fffd14: startup date [Mon Apr 30 20:49:09 CEST 2018]; root of context hierarchy
2018-04-30 20:49:14.178  INFO 5400 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/error]}" onto public org.springframework.http.ResponseEntity<java.util.Map<java.lang.String, java.lang.Object>> org.springframework.boot.autoconfigure.web.servlet.error.BasicErrorController.error(javax.servlet.http.HttpServletRequest)
2018-04-30 20:49:14.181  INFO 5400 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/error],produces=[text/html]}" onto public org.springframework.web.servlet.ModelAndView org.springframework.boot.autoconfigure.web.servlet.error.BasicErrorController.errorHtml(javax.servlet.http.HttpServletRequest,javax.servlet.http.HttpServletResponse)
2018-04-30 20:49:14.211  INFO 5400 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/webjars/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2018-04-30 20:49:14.211  INFO 5400 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2018-04-30 20:49:14.540  INFO 5400 --- [           main] o.s.j.e.a.AnnotationMBeanExporter        : Registering beans for JMX exposure on startup
2018-04-30 20:49:14.592  INFO 5400 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2018-04-30 20:49:14.596  INFO 5400 --- [           main] c.c.p.SpringPrimeFacesApplication        : Started SpringPrimeFacesApplication in 5.303 seconds (JVM running for 8.969)
```

Open a web browser and enter following URL: [http://localhost:8080/helloworld.xhtml](http://localhost:8080/helloworld.xhtml){:target="_blank"}. The below web page should now be displayed.

<figure>
  <img src="{{ site.url }}/assets/images/posts/jsf-primefaces/jsf-primefaces-spring-boot-hello-world-example.png" alt="jsf primefaces spring boot hello world example">
</figure>

Enter a first and last name and press the <var>Submit</var> button. A pop-up dialog will be shown with a greeting message.

<figure>
  <img src="{{ site.url }}/assets/images/posts/jsf-primefaces/jsf-primefaces-spring-boot-hello-world-example-greeting.png" alt="jsf primefaces spring boot hello world example greeting">
</figure>

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/jsf-primefaces/tree/master/jsf-primefaces-spring-boot){:target="_blank"}.
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

JoinFaces makes it easy to set up a PrimeFaces example using Spring Boot and Maven.

If you found this post helpful or have any questions or remarks, please leave a comment below.
