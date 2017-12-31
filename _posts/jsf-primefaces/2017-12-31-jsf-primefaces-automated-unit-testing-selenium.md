---
title: "JSF - Primefaces Automated Unit Testing using Selenium"
permalink: /jsf-primefaces-automated-unit-testing-selenium.html
excerpt: "A detailed step-by-step tutorial on how to implement an automated unit test for PrimeFaces using Selenium."
date: 2017-12-31
last_modified_at: 2017-12-31
header:
  teaser: "assets/images/teaser/primefaces-teaser.png"
categories: [PrimeFaces]
tags: [Automated Testing, Example, JUnit, Maven, PrimeFaces, Selenium, Spring Boot, Unit Testing, Tutorial]
published: false
---

<figure>
  <img src="{{ site.url }}/assets/images/logo/primefaces-logo.png" alt="primefaces logo" class="logo">
</figure>

[Selenium](http://www.seleniumhq.org/){:target="_blank"} is a software-testing framework for web applications. Tests can be run against most modern web browsers and can be controlled by many programming languages. Selenium is open-source software, released under the Apache 2.0 license.

The following example shows how to setup an automated unit test using Selenium, Spring Boot, and Maven.

# General Project Setup

Tools used:
* JSF 2.2
* PrimeFaces 6.1
* Selenium 3.8
* Spring Boot 1.5
* Maven 3.5

We will start from a previous [Spring Boot Primefaces Tutorial ]({{ site.url }}/jsf-primefaces-example-spring-boot-maven.html) in which we created a greeting dialog based on a first and last name input form.







We will be building and running our example using [Apache Maven](https://maven.apache.org/){:target="_blank"}. Shown below is the XML representation of our Maven project in a POM file. It contains the needed dependencies for compiling and running our example.

This [JoinFaces](https://github.com/joinfaces/joinfaces){:target="_blank"} project enables JSF usage inside a JAR packaged [Spring Boot](https://projects.spring.io/spring-boot/){:target="_blank"} application. It autoconfigures PrimeFaces, PrimeFaces Extensions, BootsFaces, ButterFaces, RichFaces, OmniFaces, AngularFaces, Mojarra and MyFaces libraries to run on embedded Tomcat, Jetty or Undertow servlet containers.

To facilitate the management of the different Spring dependencies, [JoinFaces Spring Boot Starters](https://github.com/joinfaces/joinfaces/wiki/JoinFaces-Starters-2.x){:target="_blank"} are used which are a set of convenient dependency descriptors that you can include in your application. There are twelve JSF Spring Boot Starters available: six basic starters, one utility starter, one meta starter and five component starters.

The `spring-boot-starter` dependency is the core starter, which includes auto-configuration support, logging and YAML. 

We use the `jsf-spring-boot-parent` project to obtain the right version for any of the dependencies in our build configuration. As a result, we do not need to specify a version for any of the dependencies in our build configuration as Spring Boot is managing this for us.

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
    <artifactId>jsf-spring-boot-parent</artifactId>
    <version>2.4.1</version>
    <relativePath /> <!-- lookup parent from repository -->
  </parent>

  <properties>
    <java.version>1.8</java.version>
  </properties>

  <dependencies>
    <!-- spring-boot -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <!-- joinfaces -->
    <dependency>
      <groupId>org.joinfaces</groupId>
      <artifactId>jsf-spring-boot-starter</artifactId>
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

In JSF, a class that can be accessed from a JSF page is called Managed Bean. By annotating the `HelloWorld` class with the `@ManagedBean` annotation it becomes a Managed Bean which is accessible and controlled by the JSF framework.

``` java
package com.codenotfound.primefaces.model;

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
    return "Hello " + firstName + " " + lastName + "!";
  }
}
```


The web page that will be shown is a standard JSF page as defined below. It contains a number of PrimeFaces components which include two <p:inputText> fields, that will be used to enter a first and last name, surrounded by a <var>&lt;p:panel&gt;</var>.

There is also a <var>&lt;p:dialog&gt;</var> component that shows a greeting message. The dialog is triggered by a <var>&lt;p:commandButton&gt;</var> that is part of the panel.

In order to use the PrimeFaces components, the following namespace needs to be declared: `xmlns:p="http://primefaces.org/ui`.

> JSF files need to be placed at the <var>src/main/resources/META-INF/resources</var> folder.

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
  <h:form id="hello-world">

    <p:panel header="PrimeFaces Hello World Example">
      <h:panelGrid columns="2" cellpadding="4">
        <h:outputText value="First Name: " />
        <p:inputText id="first-name" value="#{helloWorld.firstName}" />

        <h:outputText value="Last Name: " />
        <p:inputText id="last-name" value="#{helloWorld.lastName}" />

        <p:commandButton id="submit" value="Submit" update="greeting"
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

There are a lot of [JSF properties](https://github.com/joinfaces/joinfaces#jsf-properties-configuration-via-applicationproperties-or-applicationyml){:target="_blank"} available that allow you to configure your project.

In this example we will change the default Spring Boot server HTTP port value from <var>'8080'</var> to <var>'9090'</var> by explicitly setting it in the <var>application.yml</var> properties file located under <var>src/main/resources</var> as shown below. We also set the context path to <var>'codenotfound'</var>.

``` yaml
server:
  context-path: /codenotfound
  port: 9090
```

# Testing the PrimeFaces Hello World Example


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
 :: Spring Boot ::        (v1.5.7.RELEASE)

2017-12-30 16:20:16.551  INFO 5112 --- [           main] c.c.p.SpringPrimeFacesApplication        : Starting SpringPrimeFacesApplication on cnf-pc with PID 5112 (c:\codenotfound\code\jsf-primefaces\jsf-primefaces-spring-boot\target\classes started by CodeNotFound in c:\codenotfound\code\jsf-primefaces\jsf-primefaces-spring-boot)
2017-12-30 16:20:16.555  INFO 5112 --- [           main] c.c.p.SpringPrimeFacesApplication        : No active profile set, falling back to default profiles: default
2017-12-30 16:20:16.599  INFO 5112 --- [           main] ationConfigEmbeddedWebApplicationContext : Refreshing org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@424e77fe: startup date [Sat Dec 30 16:20:16 CET 2017]; root of context hierarchy
2017-12-30 16:20:17.532  INFO 5112 --- [           main] f.a.AutowiredAnnotationBeanPostProcessor : JSR-330 'javax.inject.Inject' annotation found and supported for autowiring
2017-12-30 16:20:17.585  INFO 5112 --- [           main] trationDelegate$BeanPostProcessorChecker : Bean 'org.joinfaces.javaxfaces.ProjectStageAutoConfiguration' of type [org.joinfaces.javaxfaces.ProjectStageAutoConfiguration] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
2017-12-30 16:20:17.947  INFO 5112 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat initialized with port(s): 9090 (http)
2017-12-30 16:20:17.957  INFO 5112 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2017-12-30 16:20:17.958  INFO 5112 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet Engine: Apache Tomcat/8.5.20
2017-12-30 16:20:18.250  INFO 5112 --- [ost-startStop-1] org.apache.jasper.servlet.TldScanner     : At least one JAR was scanned for TLDs yet contained no TLDs. Enable debug logging for this logger for a complete list of JARs that were scanned but no TLDs were found in them. Skipping unneeded JARs during scanning can improve startup time and JSP compilation time.
2017-12-30 16:20:18.270  INFO 5112 --- [ost-startStop-1] o.a.c.c.C.[.[localhost].[/codenotfound]  : Initializing Spring embedded WebApplicationContext
2017-12-30 16:20:18.270  INFO 5112 --- [ost-startStop-1] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 1674 ms
2017-12-30 16:20:19.033  INFO 5112 --- [ost-startStop-1] org.reflections.Reflections              : Reflections took 364 ms to scan 6 urls, producing 712 keys and 3778 values
2017-12-30 16:20:19.291  INFO 5112 --- [ost-startStop-1] o.s.b.w.servlet.ServletRegistrationBean  : Mapping servlet: 'dispatcherServlet' to [/]
2017-12-30 16:20:19.294  INFO 5112 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'characterEncodingFilter' to: [/*]
2017-12-30 16:20:19.294  INFO 5112 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'hiddenHttpMethodFilter' to: [/*]
2017-12-30 16:20:19.294  INFO 5112 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'httpPutFormContentFilter' to: [/*]
2017-12-30 16:20:19.295  INFO 5112 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'requestContextFilter' to: [/*]
2017-12-30 16:20:19.345  INFO 5112 --- [ost-startStop-1] j.e.resource.webcontainer.jsf.config     : Initializing Mojarra 2.2.14 ( 20161114-2152 unable to get svn info) for context '/codenotfound'
2017-12-30 16:20:19.512  INFO 5112 --- [ost-startStop-1] j.e.r.webcontainer.jsf.application       : JSF1048: PostConstruct/PreDestroy annotations present.  ManagedBeans methods marked with these annotations will have said annotations processed.
2017-12-30 16:20:20.140  INFO 5112 --- [ost-startStop-1] .w.PostConstructApplicationEventListener : Running on PrimeFaces 6.1
2017-12-30 16:20:20.140  INFO 5112 --- [ost-startStop-1] .a.PostConstructApplicationEventListener : Running on PrimeFaces Extensions 6.1
2017-12-30 16:20:20.140  INFO 5112 --- [ost-startStop-1] o.omnifaces.VersionLoggerEventListener   : Using OmniFaces version 1.14
2017-12-30 16:20:20.407  INFO 5112 --- [           main] s.w.s.m.m.a.RequestMappingHandlerAdapter : Looking for @ControllerAdvice: org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@424e77fe: startup date [Sat Dec 30 16:20:16 CET 2017]; root of context hierarchy
2017-12-30 16:20:20.468  INFO 5112 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/error]}" onto public org.springframework.http.ResponseEntity<java.util.Map<java.lang.String, java.lang.Object>> org.springframework.boot.autoconfigure.web.BasicErrorController.error(javax.servlet.http.HttpServletRequest)
2017-12-30 16:20:20.471  INFO 5112 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/error],produces=[text/html]}" onto public org.springframework.web.servlet.ModelAndView org.springframework.boot.autoconfigure.web.BasicErrorController.errorHtml(javax.servlet.http.HttpServletRequest,javax.servlet.http.HttpServletResponse)
2017-12-30 16:20:20.498  INFO 5112 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/webjars/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2017-12-30 16:20:20.499  INFO 5112 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2017-12-30 16:20:20.533  INFO 5112 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**/favicon.ico] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2017-12-30 16:20:20.711  INFO 5112 --- [           main] o.s.j.e.a.AnnotationMBeanExporter        : Registering beans for JMX exposure on startup
2017-12-30 16:20:20.775  INFO 5112 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat started on port(s): 9090 (http)
2017-12-30 16:20:20.781  INFO 5112 --- [           main] c.c.p.SpringPrimeFacesApplication        : Started SpringPrimeFacesApplication in 4.627 seconds (JVM running for 7.815)
```

Open a web browser and enter following URL: [http://localhost:9090/codenotfound/helloworld.xhtml](http://localhost:9090/codenotfound/helloworld.xhtml){:target="_blank"}. The below web page should now be displayed.

<figure>
    <img src="{{ site.url }}/assets/images/posts/jsf-primefaces/primefaces-spring-boot-hello-world-example.png" alt="primefaces spring boot hello world example">
</figure>

Enter a first and last name and press the <var>Submit</var> button. A pop-up dialog will be shown with a greeting message.

<figure>
    <img src="{{ site.url }}/assets/images/posts/jsf-primefaces/primefaces-spring-boot-hello-world-example-greeting.png" alt="primefaces spring boot hello world example greeting">
</figure>

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/jsf-primefaces/tree/master/jsf-primefaces-spring-boot){:target="_blank"}.
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This concludes the PrimeFaces Hello World example using Spring Boot and Maven. If you found this post helpful or have any questions or remarks, please leave a comment below.
