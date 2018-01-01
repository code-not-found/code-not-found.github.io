---
title: JSF - PrimeFaces login Servlet Filter example using Jetty and Maven
permalink: /2015/01/jsf-primefaces-login-servlet-filter-example-jetty-maven.html
excerpt: A PrimeFaces login servlet filter example using Jetty and Maven.
date: 2015-01-04 21:00
categories: [PrimeFaces]
tags: [Example, Hello World, JavaServer Faces, Jetty, JSF, Maven, PrimeFaces, Project, Tutorial]
redirect_from:
  - /2016/01/jsf-primefaces-login-servlet-filter-example-using-jetty-and-maven.html
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

The <var>login.xhtml</var> page is shown below. The header section contains a `<f:viewAction>` component which will redirect to the home page in case the user is already logged in.

The body contains a `<p:inputText>; and `<p:password>' input component which allow a user to enter the user name and password. Both fields apply some basic validation in that they are both required and need a minimum length of 3 characters. The <var>Login</var> button will pass the entered values and call the `login()` method on the `UserManger` bean. Validation errors detected when submitting the credentials will be displayed using the `&lt;p:messages&gt; component at the top of the page.

At the bottom of the page a number of links are included that make navigating the example easier.

``` html
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"
    xmlns:f="http://java.sun.com/jsf/core"
    xmlns:h="http://java.sun.com/jsf/html"
    xmlns:ui="http://java.sun.com/jsf/facelets"
    xmlns:p="http://primefaces.org/ui">

<h:head>
    <title>Login</title>
    <f:metadata>
        <f:viewAction action="#{userManager.isLoggedInForwardHome()}" />
    </f:metadata>
</h:head>

<h:body>

    <h:form>
        <p:messages id="messages" showDetail="false" />

        <p:panel id="panel" header="Login">

            <h:panelGrid columns="3" cellpadding="5">
                <p:outputLabel for="userName" value="User Name:" />
                <p:inputText id="userName" value="#{userManager.userId}"
                    required="true" label="User name">
                    <f:validateLength minimum="3" />
                </p:inputText>
                <p:message for="userName" display="icon" />

                <p:outputLabel for="password" value="Password:" />
                <p:password id="password"
                    value="#{userManager.userPassword}" required="true"
                    label="Password">
                    <f:validateLength minimum="3" />
                </p:password>
                <p:message for="password" display="icon" />
            </h:panelGrid>

            <p:commandButton value="Login" update="panel messages"
                action="#{userManager.login()}" />
        </p:panel>
    </h:form>

    <ui:include src="/WEB-INF/template/links.xhtml" />

</h:body>
</html>
```

The <var>logout.xhtml</var> page contains two `<p:panel>` components which are rendered based on whether a user is logged in or not. The first panel is shown in case a user is not logged in and contains a confirmation of the fact that the user is logged out. The second panel is shown in case the user is still logged in and provides a <var>Logout</var> button together with a reminder that the user is still logged in.

``` html
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"
    xmlns:f="http://java.sun.com/jsf/core"
    xmlns:h="http://java.sun.com/jsf/html"
    xmlns:ui="http://java.sun.com/jsf/facelets"
    xmlns:p="http://primefaces.org/ui">

<h:head>
    <title>Logout</title>
</h:head>

<h:body>

    <h:form>
        <p:panel header="Logout" rendered="#{!userManager.isLoggedIn()}">
            <p>You are logged out.</p>
        </p:panel>

        <p:panel header="Logout" rendered="#{userManager.isLoggedIn()}">
            <p>You are still logged in, click below button to log
                out!</p>
            <p:commandButton value="Logout"
                action="#{userManager.logout()}" />
        </p:panel>
    </h:form>

    <ui:include src="/WEB-INF/template/links.xhtml" />

</h:body>
</html>
```

The <var>home.xhtml</var> page is located in a <var>/secured</var> folder, to which access will be protected by the `LoginFilter`. The page contains a basic welcome message returning the full name of the user using the `getName()` method. In addition a Logout button is available which allows a user to invalidate the session. 

``` html
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"
    xmlns:f="http://java.sun.com/jsf/core"
    xmlns:h="http://java.sun.com/jsf/html"
    xmlns:ui="http://java.sun.com/jsf/facelets"
    xmlns:p="http://primefaces.org/ui">

<h:head>
    <title>Home</title>
</h:head>

<h:body>

    <h:form>
        <p:panel header="Home">
            <p>
                <h:outputText
                    value="Welcome, #{userManager.currentUser.getName()}!" />
            </p>

            <p:commandButton value="Logout"
                action="#{userManager.logout()}" />
        </p:panel>
    </h:form>

    <ui:include src="/WEB-INF/template/links.xhtml" />

</h:body>
</html>
```

As a workaround for the fact the the `LoginFilter` is not applied to the files in the `<welcome-file-list>` of the <var>web.xml</var>, a <var>redirect.xhtml</var> page is added that does a simple redirect the <var>home.xhtml</var> using the JSF `<f:viewAction>` component.

``` html
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"
    xmlns:f="http://java.sun.com/jsf/core"
    xmlns:h="http://java.sun.com/jsf/html"
    xmlns:ui="http://java.sun.com/jsf/facelets">

<h:head>
    <title>Redirect</title>
    <f:metadata>
        <f:viewAction action="/secured/home.xhtml?faces-redirect=true" />
    </f:metadata>
</h:head>

<h:body>

</h:body>
</html>
```

The <var>unsecured.xhtml</var> page is an example of a page that can be accesses without needing to login. The <var>links.xhtml</var> is a `ui:composition` component that is referenced from all the above page which contains links to make navigating the example easier. 

# Security

The `LoginFilter` class is a `Servlet Filter` that will be used to restrict access to the home page. When called, it will try to retrieve the `UserManager` from the `ServletRequest`. Note that the session attribute name used to retrieve the `UserManager` is the name of the managed bean. If the `isLoggedIn()` returns true then the call is allowed through. In all other cases, a redirect is done to the login page.

``` java
package com.codenotfound.jsf.primefaces.filter;

import com.codenotfound.jsf.primefaces.controller.UserManager;

import java.io.IOException;

import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class LoginFilter implements Filter {

    private static final Logger LOGGER = LoggerFactory
            .getLogger(LoginFilter.class);

    public static final String LOGIN_PAGE = "/login.xhtml";

    @Override
    public void doFilter(ServletRequest servletRequest,
            ServletResponse servletResponse, FilterChain filterChain)
            throws IOException, ServletException {

        HttpServletRequest httpServletRequest = (HttpServletRequest) servletRequest;
        HttpServletResponse httpServletResponse = (HttpServletResponse) servletResponse;

        // managed bean name is exactly the session attribute name
        UserManager userManager = (UserManager) httpServletRequest
                .getSession().getAttribute("userManager");

        if (userManager != null) {
            if (userManager.isLoggedIn()) {
                LOGGER.debug("user is logged in");
                // user is logged in, continue request
                filterChain.doFilter(servletRequest, servletResponse);
            } else {
                LOGGER.debug("user is not logged in");
                // user is not logged in, redirect to login page
                httpServletResponse.sendRedirect(
                        httpServletRequest.getContextPath()
                                + LOGIN_PAGE);
            }
        } else {
            LOGGER.debug("userManager not found");
            // user is not logged in, redirect to login page
            httpServletResponse.sendRedirect(
                    httpServletRequest.getContextPath() + LOGIN_PAGE);
        }
    }

    @Override
    public void init(FilterConfig arg0) throws ServletException {
        LOGGER.debug("LoginFilter initialized");
    }

    @Override
    public void destroy() {
        // close resources
    }
}
```

The <var>web.xml</var> deployment descriptor file is shown below. It contains the definition of the `LoginFilter` and the resources to which it needs to be applied. In this example the filter will be applied to all pages inside the <var>/secured</var> directory.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns="http://java.sun.com/xml/ns/javaee" xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
    version="3.0">

    <display-name>PrimeFaces Login Example</display-name>

    <!-- define the JSF listener class when using the jetty-maven-plugin 
        with Jetty8 -->
    <listener>
        <listener-class>com.sun.faces.config.ConfigureListener</listener-class>
    </listener>

    <!-- login filter -->
    <filter>
        <filter-name>login</filter-name>
        <filter-class>com.codenotfound.jsf.primefaces.filter.LoginFilter</filter-class>
    </filter>
    <!-- set the login filter to secure all the pages in the /secured/* path 
        of the application -->
    <filter-mapping>
        <filter-name>login</filter-name>
        <url-pattern>/secured/*</url-pattern>
    </filter-mapping>

    <welcome-file-list>
        <welcome-file>redirect.xhtml</welcome-file>
    </welcome-file-list>

    <servlet>
        <servlet-name>faces</servlet-name>
        <servlet-class>javax.faces.webapp.FacesServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>faces</servlet-name>
        <url-pattern>*.xhtml</url-pattern>
    </servlet-mapping>
</web-app>
```

# Running the Example
In order to run the above example open a command prompt and execute following Maven command:

``` plaintext
mvn jetty:run
```

Maven will download the needed dependencies, compile the code and start a Jetty instance on which the web application will be deployed. Open a web browser and enter following URL: [http://localhost:9090/cnf/](http://localhost:9090/cnf/). The result should be that below page is displayed:

<figure>
    <img src="{{ site.url }}/assets/images/jsf/jsf-login-page.png" alt="jsf login page">
</figure>

Enter following credentials: User name="<kbd>john.doe</kbd>" and Password="<kbd>1234</kbd>" and a welcome page will be displayed as shown below. 

<figure>
    <img src="{{ site.url }}/assets/images/jsf/jsf-home-page.png" alt="jsf home page">
</figure>

Click the <var>Logout</var> button and a redirect to the logout page should happen as shown below. 

<figure>
    <img src="{{ site.url }}/assets/images/jsf/jsf-logout-page.png" alt="jsf logout page">
</figure>

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/jsf-primefaces/tree/master/jsf-jetty-primefaces-login-servlet-filter).
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This concludes the PrimeFaces login example using Jetty. If you found this post helpful or have any questions or remarks, please leave a comment. 