---
title: "JSF Primefaces Spring Security Example"
permalink: /jsf-primefaces-spring-security-example.html
excerpt: "A detailed step-by-step tutorial on how to implement a PrimeFaces login page using Spring Security, Spring Boot, and Maven."
date: 2018-01-17
last_modified_at: 2018-12-07
header:
  teaser: "assets/images/jsf-primefaces/jsf-primefaces-spring-security-example.png"
categories: [PrimeFaces]
tags: [Example, JavaServer Faces, JSF, Login, Maven, PrimeFaces, Spring Boot, Spring Security]
redirect_from:
  - /jsf-primefaces-spring-security.html
published: true
---

<img src="{{ site.url }}/assets/images/jsf-primefaces/jsf-primefaces-spring-security-example.png" alt="jsf primefaces spring security example" class="align-right title-image">

This is a complete guide to setup a PimeFaces login using [Spring Security](https://spring.io/projects/spring-security){:target="_blank"}.

So if you want to secure your site, you'll love the step-by-step approach in this guide.

We've got a lot to cover, so let's get started.

If you want to learn more about PrimeFaces for JSF - head on over to the [JSF PrimeFaces tutorials]({{ site.url }}/jsf-primefaces-tutorials) page.
{: .notice--primary}

## 1. What is Spring Security?

[Spring Security](https://en.wikipedia.org/wiki/Spring_Security){:target="_blank"} is a framework that focuses on providing both [authentication](https://en.wikipedia.org/wiki/Authentication){:target="_blank"} and [authorization](https://en.wikipedia.org/wiki/Authorization){:target="_blank"} to Java applications.

The project was started in late 2003 as 'Acegi Security'. Subsequently, it was incorporated into the Spring portfolio as Spring Security, an official Spring sub-project.

The following example shows how to set up a PrimeFaces login page in combination with Spring Security, Spring Boot, and Maven.

## 2. General Project Overview

We will use the following tools/frameworks:
* PrimeFaces 6.2
* JoinFaces 3.3
* Spring Boot 2.1
* Spring Security 5.1
* Maven 3.5

Our project has the following directory structure:

<img src="{{ site.url }}/assets/images/jsf-primefaces/jsf-primefaces-spring-security-maven-project.png" alt="jsf primefaces spring security maven project">

## 3. Maven Setup

The example is based on a previous [Hello World Primefaces Tutorial]({{ site.url }}/jsf-primefaces-example.html) in which we created a greeting dialog based on a first and last name input form.

We also include the setup of a welcome page using the [PrimeFaces redirect example]({{ site.url }}/jsf-welcome-page-redirect-example-spring-boot.html).

To use Spring Security we add `spring-boot-starter-security` to the existing [Maven](https://maven.apache.org/){:target="_blank"} POM file. This will include the core security dependencies that are needed for securing our JSF application.

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>jsf-primefaces-spring-security</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>jsf-primefaces-spring-security</name>
  <description>JSF PrimeFaces Spring Security Example</description>
  <url>https://codenotfound.com/jsf-primefaces-spring-security-example.html</url>

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
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-security</artifactId>
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

## 4. JSF Security Authorization and Authentication

Spring Security ships with a [default login page generator](https://github.com/spring-projects/spring-security/blob/master/web/src/main/java/org/springframework/security/web/authentication/ui/DefaultLoginPageGeneratingFilter.java){:target="_blank"} but in this example we will configure a custom login page using [PrimeFaces](https://www.primefaces.org/){:target="_blank"} components.

The <var>login.xhtml</var> page is located under <var>src/main/resources/META-INF/resources</var> and consists out of an <var>&lt;p:inputText&gt;</var> for the user name and a <var>&lt;p:password&gt;</var> for the password.

A <var>&lt;p:commandButton&gt;</var> is used to submit the form.

> Note that the IDs of the input fields need to be <var>'username'</var> and <var>'password'</var> respectively as by default Spring Security will look for parameters with these names.

We also specify `prependId="false"` on the form as otherwise the JSF framework will prefix the parameters with the ID of the form. Another option would be to override the default field names using the `usernameParameter()` and `passwordParameter()` methods on [formLogin()](https://docs.spring.io/spring-security/site/docs/5.1.1.RELEASE/reference/html/jc.html#jc-form){:target="_blank"} which we use further below.

{% highlight html %}
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"
  xmlns:h="http://java.sun.com/jsf/html"
  xmlns:f="http://xmlns.jcp.org/jsf/core"
  xmlns:p="http://primefaces.org/ui"
  xmlns:pe="http://primefaces.org/ui/extensions">

<h:head>
  <title>Login</title>
  <h:outputStylesheet name="/css/login.css" />
</h:head>

<h:body>
  <h:form prependId="false">

    <p:panelGrid columns="1" styleClass="ui-fluid center ui-noborder">
      <h2>Please login</h2>

      <p:outputLabel value="Login failed!" styleClass="red"
        rendered="${!empty param['error']}" />

      <p:inputText id="username" placeholder="User name" />
      <p:password id="password" placeholder="Password" />

      <p:commandButton value="Login" ajax="false" />
    </p:panelGrid>

  </h:form>
</h:body>
</html>
{% endhighlight %}

If Spring Security is on the classpath Spring Boot automatically [configures a number of basic security features](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/reference/htmlsingle/#boot-features-security){:target="_blank"} for a web application.

We will customize the web application security configuration by creating a `SecurityConfig` class that extends `WebSecurityConfigurerAdapter` which is a convenient base class that provides a default security configuration. The class is annotated with `@EnableWebSecurity` to enable Spring Security's web security support.

Override the `configure(HttpSecurity http)` method in order to define when and how users need to be authenticated.

Specifying `authorizeRequests().anyRequest().authenticated()` ensures that any request to our application requires the user to be authenticated.

The static resources (CSS, JavaScript, ...) need to be accessible to anyone otherwise the look and feel of the login page won't be the same as the rest of the application. Adding `antMatchers("/javax.faces.resource/**").permitAll()` allows anyone to access a URL that begins with <var>/javax.faces.resource/</var> (which is where the static resources in a JSF application are served).

As we have defined a custom PrimeFaces login page we need to specify its location using `formLogin().loginPage("/login.xhtml")` so that when authentication is required, the browser is redirected to <var>/login.xhtml</var>. We also need `permitAll()` on the login page so that anyone has access otherwise we end up in a [redirect loop](https://docs.spring.io/spring-security/site/docs/5.1.1.RELEASE/guides/html5/form-javaconfig.html#granting-access-to-unauthenticated-users){:target="_blank"}.

If the login fails we redirect to the same login page with a <var>error=true</var> HTTP parameter in the URL using `failureUrl("/login.xhtml?error=true")`. This allows us to display and error message to the user.

Using `WebSecurityConfigurerAdapter`, logout capabilities are automatically applied. The default is that accessing the URL <var>/logout</var> will [log the user out](https://docs.spring.io/spring-security/site/docs/5.1.1.RELEASE/reference/html/jc.html#jc-logout){:target="_blank"}. By specifying `logout().logoutSuccessUrl("/login.xhtml")` we redirect the user to the login page once he/she is successfully logged out.

Spring Security applies measures to prevents [CSRF attacks](https://en.wikipedia.org/wiki/Cross-site_request_forgery){:target="_blank"} by requiring a [randomly generated token](https://docs.spring.io/spring-security/site/docs/5.1.1.RELEASE/reference/html/csrf.html#synchronizer-token-pattern){:target="_blank"} as an HTTP parameter. However as JSF 2.2 already contains an [explicit protection against CSRF attacks](http://arjan-tijms.omnifaces.org/p/jsf-22.html#869){:target="_blank"} we disable the Spring Security protection by specifying `http.csrf().disable()`.

We will override the default single user `AuthenticationManager` that Spring Boot sets by auto-wiring an `AuthenticationManagerBuilder` into the `configureGlobal()` of our `SecurityConfig` `@Configuration` class.

For this example we use in-memory authentication in which two users (<var>'john.doe'</var> and <var>'jane.doe'</var>) with different roles (<var>'USER'</var> and <var>'ADMIN'</var>) are defined.

Password storage with Spring Security has undergone a [major overhaul since version 5](https://spring.io/blog/2017/11/01/spring-security-5-0-0-rc1-released#password-storage-updated){:target="_blank"}. Because of these changes we prefix <var>{noop}</var> to the passwords in order for the `DelegatingPasswordEncoder` to use the `NoOpPasswordEncoder` [to validate them](https://stackoverflow.com/a/47150363/4201470){:target="_blank"}.

{% highlight java %}
package com.codenotfound.primefaces;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

  @Override
  protected void configure(HttpSecurity http) throws Exception {
    // require all requests to be authenticated except for the resources
    http.authorizeRequests().antMatchers("/javax.faces.resource/**")
        .permitAll().anyRequest().authenticated();
    // login
    http.formLogin().loginPage("/login.xhtml").permitAll()
        .failureUrl("/login.xhtml?error=true");
    // logout
    http.logout().logoutSuccessUrl("/login.xhtml");
    // not needed as JSF 2.2 is implicitly protected against CSRF
    http.csrf().disable();
  }

  @Autowired
  public void configureGlobal(AuthenticationManagerBuilder auth)
      throws Exception {
    auth.inMemoryAuthentication().withUser("john.doe")
        .password("{noop}1234").roles("USER").and()
        .withUser("jane.doe").password("{noop}5678").roles("ADMIN");
  }
}
{% endhighlight %}

> When debugging Spring Security it is sometimes useful to enable the <var>DEBUG</var> level on the `org.springframework.security` logger. We have added this logger to the <var>logback.xml</var> logging configuration file located under <var>src/main/resources</var>.

## 5. Role Based Access Control

Now that access to our application is secured we will adapt the Hello World example to illustrate role-based access control.

Spring Security has its own [taglib](https://docs.spring.io/spring-security/site/docs/5.1.1.RELEASE/reference/html/taglibs.html){:target="_blank"} which provides basic support for accessing security information and applying security constraints in JSPs.

On the <var>helloworld.xhtml</var> page we add a <var>&lt;div&gt;</var> element in which we use the authorize tag in order to display a message in case the user has the <var>'USER'</var> or <var>'ADMIN'</var> role.

We also add a logout <var>&lt;p:commandButton&gt;</var> at the bottom of the page. Note that we do not use <var>&lt;h:form&gt;</var> as [JSF sets the form action automatically to the current page](https://stackoverflow.com/a/19200679/4201470){:target="_blank"} and what we want to do is to navigate to the default logout URL offered by Spring Security.

{% highlight html %}
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"
  xmlns:h="http://java.sun.com/jsf/html"
  xmlns:p="http://primefaces.org/ui"
  xmlns:sec="http://www.springframework.org/security/tags">

<h:head>
  <title>PrimeFaces Hello World Example</title>
  <h:outputStylesheet name="/css/main.css" />
</h:head>

<h:body>

  <div class="authorization-div">
    <sec:authorize access="hasRole('ROLE_USER')">
      <p:outputLabel value="You have the USER role" />
    </sec:authorize>
    <sec:authorize access="hasRole('ROLE_ADMIN')">
      <p:outputLabel value="You have the ADMIN role" />
    </sec:authorize>
  </div>

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

  <h:form onsubmit="this.action='#{request.contextPath}/logout';"
    class="logout-form">
    <p:commandButton value="Logout" ajax="false" />
  </h:form>

</h:body>
</html>
{% endhighlight %}

To wrap up we illustrate how you can access the current `Authentication` object stored in the security context.

We change the `showGreeting()` method to display the name of the authenticated user by calling `SecurityContextHolder.getContext().getAuthentication()` as shown below.

{% highlight java %}
package com.codenotfound.primefaces;

import javax.inject.Named;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;

@Named
public class HelloWorld {

  private String firstName = "";
  private String lastName = "";

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
    Authentication authentication =
        SecurityContextHolder.getContext().getAuthentication();

    return "Hello " + authentication.getName() + "!";
  }
}
{% endhighlight %}

## 6. Testing the JSF Security Roles

Let's test our secured PrimeFace login example by running following Maven command:

{% highlight plaintext %}
mvn spring-boot:run
{% endhighlight %}

Once Spring Boot has started, open a web browser and enter the following URL: [http://localhost:8080/helloworld.xhtml](http://localhost:8080/helloworld.xhtml){:target="_blank"}.

As we are not authenticated, Spring Security will redirect us to the login page.

<img src="{{ site.url }}/assets/images/jsf-primefaces/jsf-primefaces-login-page.png" alt="jsf primefaces login page">

Go ahead and enter the following user name=<kbd>john.doe</kbd> and password=<kbd>1234</kbd> and click the <var>Login</var> button.

We now see the Hello World page and the role of the user is displayed up top as shown below.

<img src="{{ site.url }}/assets/images/jsf-primefaces/jsf-primefaces-user-role.png" alt="jsf primefaces user role">

If we click on <var>Submit</var> then the username that was used to log in will be displayed.

<img src="{{ site.url }}/assets/images/jsf-primefaces/jsf-primefaces-authentication-user-name.png" alt="jsf primefaces authentication user name">

Press the <var>Logout</var> button in order to be redirected to the login page.

If we now enter the same user but with an incorrect password an error message will be displayed.

<img src="{{ site.url }}/assets/images/jsf-primefaces/jsf-primefaces-login-error.png" alt="jsf primefaces login error">

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/jsf-primefaces/tree/master/jsf-primefaces-spring-security){:target="_blank"}.
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

The Spring Security framework is packed with out-of-the-box features that allow you to secure your JSF web application. All that is needed is a bit of configuration in order to set up authentication and authorization of users accessing our PrimeFaces example.

Let me know if you found the tutorial helpful.

Or in case you have some questions or remarks.

Thanks!
