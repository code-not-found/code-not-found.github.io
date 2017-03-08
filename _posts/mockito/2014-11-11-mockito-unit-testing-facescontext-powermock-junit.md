---
title: Mockito - unit testing FacesContext using PowerMock, JUnit and Maven
permalink: /2014/11/mockito-unit-testing-facescontext-powermock-junit.html
excerpt: A code sample which shows how to unit test FacesContext using Mockito, PowerMock, JUnit and Maven.
date: 2014-11-11 21:00
categories: [Mockito]
tags: [Code Sample, FacesContext, JSF, JUnit, Maven, Mockito, PowerMock, unit testing]
redirect_from:
  - /2014/11/mockito-mocking-facescontext-using.html
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/mockito-logo.png" alt="mockito logo">
</figure>

JSF defines the `FacesContext` abstract base class for representing all of the contextual information associated with processing an incoming request, and creating the corresponding response. When writing unit test cases for a JSF application there might be a need to mock some of the `FacesContext` static methods. The following post will illustrate how to do this using [PowerMock](https://code.google.com/p/powermock/), a framework that allows you to extend mock libraries like [Mockito](https://code.google.com/p/mockito/) with extra capabilities. In this case the capability to mock the static methods of `FacesContext`.

Tools used:
* JUnit 4.11
* Mockito 1.10
* PowerMock 1.5
* Maven 3

The code sample is built and run using Maven. Specified below is the Maven POM file which contains the needed dependencies for JUnit, Mockito and PowerMock. In addition the PowerMock support module for JUnit (<var>'powermock-module-junit4'</var>) and the PowerMock API for Mockito (<var>'powermock-api-mockito'</var>) dependencies need to be added as specified here.

As the `FacesContext` class is used in this code sample, dependencies to the EL (Expression Language) API and JSF specification API are also included.

Note that the version of JUnit is not the latest as there seems to be a bug where [PowerMock doesn't recognize the correct JUnit version when using JUnit 4.12](http://stackoverflow.com/a/26222732/4201470).

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.codenotfound</groupId>
    <artifactId>mockito-powermock-facescontext</artifactId>
    <version>1.0</version>
    <packaging>jar</packaging>

    <name>Mockito - Mocking FacesContext using PowerMock</name>
    <url>https://codenotfound.com/2014/11/mockito-mocking-facescontext-using.html</url>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <java.version>1.8</java.version>

        <junit.version>4.11</junit.version>
        <mockito.version>1.10.8</mockito.version>
        <powermock.version>1.5.6</powermock.version>

        <el.version>2.2.1-b04</el.version>
        <jsf.version>2.2.8-02</jsf.version>

        <maven-compiler-plugin.version>3.1</maven-compiler-plugin.version>
    </properties>

    <dependencies>
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
        <!-- EL (Unified Expression Language) -->
        <dependency>
            <groupId>javax.el</groupId>
            <artifactId>el-api</artifactId>
            <version>${el.version}</version>
            <scope>test</scope>
        </dependency>
        <!-- JSF -->
        <dependency>
            <groupId>com.sun.faces</groupId>
            <artifactId>jsf-api</artifactId>
            <version>${jsf.version}</version>
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
        </plugins>
    </build>
</project>
```

The `SomeBean` class below contains two methods that make use of `FacesContext`. The first `addMessage()` method will create a new FacesMessage and add it to the `FacesContext`. The second `logout()` method will invalidate the current session.

``` java
package com.codenotfound.mockito;

import javax.faces.application.FacesMessage;
import javax.faces.application.FacesMessage.Severity;
import javax.faces.bean.ManagedBean;
import javax.faces.bean.SessionScoped;
import javax.faces.context.FacesContext;

@ManagedBean
@SessionScoped
public class SomeBean {

    public void addMessage(Severity severity, String summary, String detail) {
        FacesContext.getCurrentInstance().addMessage(null,
                new FacesMessage(severity, summary, detail));
    }

    public String logout() {
        FacesContext.getCurrentInstance().getExternalContext()
                .invalidateSession();

        return "logout?faces-redirect=true";
    }
}
```

The `SomeBeanTest` JUnit test class is used to test the above. The class is annotated using two annotations. The first `@RunWith` annotation tells JUnit to run the test using `PowerMockRunner`. The second `@PrepareForTest` annotation tells PowerMock to prepare to mock the `FacesContext` class. If there are multiple classes to be prepared for mocking, they can be specified using a comma separated list.

Mockito provides the `@Mock` annotation which is a [shorthand for mocks creation](http://docs.mockito.googlecode.com/hg/org/mockito/Mockito.html#9). In the below test class it is used to create the `FacesContext` and `ExternalContext` mocks. Note that the previous `@RunWith(PowerMockRunner.class)` annotation will take care of [initializing fields annotated with Mockito annotations](http://stackoverflow.com/a/22673271/4201470). 

In the `setup()` method a number of objects are specified that are similar for the two test cases. The `mockStatic()` method is called in order to tell PowerMock to mock all static methods of the given `FacesContext` class. We then use the `when()` method to specify what instance to return in case the `getCurrentInstance()` method is called on `FacesContext`. The same is done for the `getExternalContext()` method.

> Note that because of the <var>'org.mockito.Mockito.when'</var> import there is no Mockito class name in front of the static `when()` method.

The first `addMessage()` test case uses the `ArgumentCaptor` capability of Mockito in order to test whether a `FacesMessage` with the correct values was added to the `FacesContext`. The second `testLogout()` test case checks if the correct redirect was returned.

``` java
package com.codenotfound.mockito;

import static org.junit.Assert.assertEquals;
import static org.junit.Assert.assertNull;
import static org.mockito.Mockito.verify;
import static org.mockito.Mockito.when;

import javax.faces.application.FacesMessage;
import javax.faces.context.ExternalContext;
import javax.faces.context.FacesContext;

import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.ArgumentCaptor;
import org.mockito.Mock;
import org.powermock.api.mockito.PowerMockito;
import org.powermock.core.classloader.annotations.PrepareForTest;
import org.powermock.modules.junit4.PowerMockRunner;

@RunWith(PowerMockRunner.class)
@PrepareForTest({ FacesContext.class })
public class SomeBeanTest {

    private SomeBean someBean;

    @Mock
    private FacesContext facesContext;
    @Mock
    private ExternalContext externalContext;

    @Before
    public void setUp() throws Exception {
        someBean = new SomeBean();

        // mock all static methods of FacesContext using PowerMockito
        PowerMockito.mockStatic(FacesContext.class);

        when(FacesContext.getCurrentInstance()).thenReturn(facesContext);
        when(facesContext.getExternalContext()).thenReturn(externalContext);
    }

    @Test
    public void testAddMessage() {
        // create Captor instances for the clientId and FacesMessage parameters
        // that will be added to the FacesContext
        ArgumentCaptor<String> clientIdCaptor = ArgumentCaptor
                .forClass(String.class);
        ArgumentCaptor<FacesMessage> facesMessageCaptor = ArgumentCaptor
                .forClass(FacesMessage.class);

        // run the addMessage() method to be tested
        someBean.addMessage(FacesMessage.SEVERITY_ERROR, "error",
                "something went wrong");

        // verify if the call to addMessage() was made and capture the arguments
        verify(facesContext).addMessage(clientIdCaptor.capture(),
                facesMessageCaptor.capture());

        // check the value of the clientId that was passed
        assertNull(clientIdCaptor.getValue());

        // retrieve the captured FacesMessage
        FacesMessage captured = facesMessageCaptor.getValue();
        // check if the captured FacesMessage contains the expected values
        assertEquals(FacesMessage.SEVERITY_ERROR, captured.getSeverity());
        assertEquals("error", captured.getSummary());
        assertEquals("something went wrong", captured.getDetail());
    }

    @Test
    public void testLogout() {
        assertEquals("logout?faces-redirect=true", someBean.logout());
    }
}
```

In order to run the above test cases, open a command prompt and execute following Maven command: 

``` plaintext
mvn test
```

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/mockito/tree/master/mockito-powermock-facescontext).
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This concludes the mocking FacesContext using Mockito and PowerMock example. If you found this post helpful or have any questions or remarks, please leave a comment.