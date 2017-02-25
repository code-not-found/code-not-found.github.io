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

The code is built and run using Maven. Specified below is the Maven POM file which contains the needed dependencies for JSF and PrimeFaces. In addition all classes contain corresponding unit test cases which use [Mockito and PowerMock for mocking `FacesContext`]({{ site.url }}/2014/11/mockito-unit-testing-facescontext-powermock-junit.html). The JSF application will run on a Jetty instance launched from from command line using the <var>'jetty-maven-plugin'</var>. 

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.codenotfound</groupId>
    <artifactId>jsf-jetty-primefaces-login-servlet-filter</artifactId>
    <version>1.0</version>
    <packaging>war</packaging>

    <name>JSF - PrimeFaces Login Servlet Filter example using Jetty and Maven</name>
    <url>https://codenotfound.com/2015/01/jsf-primefaces-login-servlet-filter-example-jetty-maven.html</url>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <java.version>1.8</java.version>

        <slf4j.version>1.7.7</slf4j.version>
        <logback.version>1.1.2</logback.version>
        <junit.version>4.11</junit.version>
        <mockito.version>1.10.8</mockito.version>
        <powermock.version>1.5.6</powermock.version>
        <el.version>2.2.1-b04</el.version>
        <servlet.version>3.0.1</servlet.version>
        <jsf.version>2.2.8-02</jsf.version>
        <primefaces.version>5.1</primefaces.version>

        <maven-compiler-plugin.version>3.1</maven-compiler-plugin.version>
        <jetty-maven-plugin.version>8.1.14.v20131031</jetty-maven-plugin.version>
    </properties>

    <dependencies>
        <!-- Logging -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>${logback.version}</version>
        </dependency>
        <!-- JUnit -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>
        <!-- Mockito -->
        <dependency>
            <groupId>org.mockito</groupId>
            <artifactId>mockito-core</artifactId>
            <version>${mockito.version}</version>
            <scope>test</scope>
        </dependency>
        <!-- PowerMock -->
        <dependency>
            <groupId>org.powermock</groupId>
            <artifactId>powermock-core</artifactId>
            <version>${powermock.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.powermock</groupId>
            <artifactId>powermock-module-junit4</artifactId>
            <version>${powermock.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.powermock</groupId>
            <artifactId>powermock-api-mockito</artifactId>
            <version>${powermock.version}</version>
            <scope>test</scope>
        </dependency>
        <!-- Unified Expression Language (EL) for unit testing -->
        <dependency>
            <groupId>javax.el</groupId>
            <artifactId>el-api</artifactId>
            <version>${el.version}</version>
            <scope>test</scope>
        </dependency>
        <!-- Servlet -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>${servlet.version}</version>
        </dependency>
        <!-- JSF -->
        <dependency>
            <groupId>com.sun.faces</groupId>
            <artifactId>jsf-api</artifactId>
            <version>${jsf.version}</version>
        </dependency>
        <dependency>
            <groupId>com.sun.faces</groupId>
            <artifactId>jsf-impl</artifactId>
            <version>${jsf.version}</version>
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
                <groupId>org.mortbay.jetty</groupId>
                <artifactId>jetty-maven-plugin</artifactId>
                <version>${jetty-maven-plugin.version}</version>
                <configuration>
                    <webAppConfig>
                        <contextPath>/cnf</contextPath>
                    </webAppConfig>
                    <connectors>
                        <connector
                            implementation="org.eclipse.jetty.server.nio.SelectChannelConnector">
                            <port>9090</port>
                        </connector>
                    </connectors>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

# Model

First a `User` class is defined which models some basic information on a user. 

``` java
package com.codenotfound.jsf.primefaces.model;

import java.io.Serializable;

public class User implements Serializable {

    private static final long serialVersionUID = 1L;

    private String userId;
    private String firstName;
    private String lastName;

    public User(String userId, String firstName, String lastName) {
        super();

        this.userId = userId;
        this.firstName = firstName;
        this.lastName = lastName;
    }

    public String getUserId() {
        return userId;
    }

    public void setUserId(String userId) {
        this.userId = userId;
    }

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

    public String getName() {
        return firstName + " " + lastName;
    }

    public String toString() {
        return "user[userId=" + userId + ", firstName=" + firstName
                + ", lastName=" + lastName + "]";
    }
}
```

# Controller

Next we have the `UserManager`, which is a a session scoped managed bean that will handle all login related activities. It contains a `login()` method that will try to look-up a user based on a `userName` and `userPassword` combination entered on the login page. When the user is found it is assigned to the `currentUser` variable and a redirect to the home page is returned. If the user is not found, a redirect to the login page is returned.

A `logout()` method will invalidate the session and the redirect to the logout page will make sure that the previous data is longer longer available as explained in following [post](http://stackoverflow.com/a/5620582/4201470).

The `isLoggedIn()` method will be used by the `LoginFilter` to check if a user is logged in. It checks the value of the `currentUser` which is only set after a successful login. The `isLoggedInForwardHome()` method will return a redirect to the home page in case a user is already logged in.

> Do not provide a setter for the `currentUser` variable, as this potentially allows a way to circumvent the `login()` method! 

``` java
package com.codenotfound.jsf.primefaces.controller;

import com.codenotfound.jsf.primefaces.model.User;

import javax.faces.application.FacesMessage;
import javax.faces.bean.ManagedBean;
import javax.faces.bean.SessionScoped;
import javax.faces.context.FacesContext;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@ManagedBean
@SessionScoped
public class UserManager {

    private static final Logger LOGGER = LoggerFactory
            .getLogger(UserManager.class);

    public static final String HOME_PAGE_REDIRECT = "/secured/home.xhtml?faces-redirect=true";
    public static final String LOGOUT_PAGE_REDIRECT = "/logout.xhtml?faces-redirect=true";

    private String userId;
    private String userPassword;
    private User currentUser;

    public String login() {
        // lookup the user based on user name and user password
        currentUser = find(userId, userPassword);

        if (currentUser != null) {
            LOGGER.info("login successful for '{}'", userId);

            return HOME_PAGE_REDIRECT;
        } else {
            LOGGER.info("login failed for '{}'", userId);
            FacesContext.getCurrentInstance().addMessage(null,
                    new FacesMessage(FacesMessage.SEVERITY_WARN,
                            "Login failed",
                            "Invalid or unknown credentials."));

            return null;
        }
    }

    public String logout() {
        String identifier = userId;

        // invalidate the session
        LOGGER.debug("invalidating session for '{}'", identifier);
        FacesContext.getCurrentInstance().getExternalContext()
                .invalidateSession();

        LOGGER.info("logout successful for '{}'", identifier);
        return LOGOUT_PAGE_REDIRECT;
    }

    public boolean isLoggedIn() {
        return currentUser != null;
    }

    public String isLoggedInForwardHome() {
        if (isLoggedIn()) {
            return HOME_PAGE_REDIRECT;
        }

        return null;
    }

    private User find(String userId, String password) {
        User result = null;

        // code block to be replaced with actual retrieval of user
        if ("john.doe".equalsIgnoreCase(userId)
                && "1234".equals(password)) {
            result = new User(userId, "John", "Doe");
        }

        return result;
    }

    public String getUserId() {
        return userId;
    }

    public void setUserId(String userId) {
        this.userId = userId;
    }

    public String getUserPassword() {
        return userPassword;
    }

    public void setUserPassword(String userPassword) {
        this.userPassword = userPassword;
    }

    public User getCurrentUser() {
        return currentUser;
    }

    // do not provide a setter for currentUser!
}
```

# View

In the example a number of web pages will be used which realize the view part of the JSF web application. The following picture show how the different pages are structured in the webapp directory of the application WAR.

<figure>
    <img src="{{ site.url }}/assets/images/jsf/jsf-login-pages-overview.png" alt="jsf login pages overview">
</figure>

























