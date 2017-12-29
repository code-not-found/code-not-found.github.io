---
title: "EasyMock - Unit Testing FacesContext using PowerMock, JUnit and Maven"
permalink: /easymock-unit-testing-facescontext-powermock-junit.html
excerpt: "A code sample which shows how to unit test FacesContext using EasyMock, PowerMock, JUnit and Maven."
date: 2014-11-12
last_modified_at: 2014-11-12
header:
  teaser: "assets/images/teaser/easymock-teaser.png"
categories: [EasyMock]
tags: [Code Sample, EasyMock, FacesContext, JSF, JUnit, Maven, PowerMock, Unit Testing]
redirect_from:
  - /2014/11/easymock-mocking-facescontext-using.html
  - /2014/11/easymock-unit-testing-facescontext-powermock-junit.html
published: true
---

<figure>
    <img src="{{ site.url }}/assets/images/logo/easymock-logo.png" alt="easymock logo" class="logo">
</figure>

JSF defines the `FacesContext` abstract base class for representing all of the contextual information associated with processing an incoming request and creating the corresponding response. When writing unit test cases for a JSF application there might be a need to mock some of `FacesContext` static methods.

The following post will illustrate how to do this using [PowerMock](https://code.google.com/p/powermock/){:target="_blank"}, which is a framework that allows you to extend mock libraries like [EasyMock](http://easymock.org/){:target="_blank"} with extra capabilities. In this case the capability to mock the static methods of `FacesContext`.

Tools used:
* JUnit 4.12
* EasyMock 3.5
* PowerMock 1.7
* Maven 3.5

The code sample is built and run using Maven. Specified below is the Maven POM file which contains the needed dependencies for JUnit, EasyMock, and PowerMock.

In addition, the PowerMock support module for JUnit `powermock-module-junit4` and the PowerMock API for EasyMock `powermock-api-easymock` dependencies need to be added as specified below.

As the `FacesContext` class is used in this code sample, dependencies to the EL (Expression Language) API and JSF specification API are also included.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>easymock-powermock-facescontext</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>EasyMock - Unit Testing FacesContext using PowerMock, JUnit and Maven</name>
  <url>https://codenotfound.com/easymock-unit-testing-facescontext-powermock-junit.html</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <java.version>1.8</java.version>
    <junit.version>4.12</junit.version>
    <easymock.version>3.5.1</easymock.version>
    <powermock.version>1.7.3</powermock.version>
    <el.version>3.0.0</el.version>
    <jsf.version>2.2.15</jsf.version>
    <maven-compiler-plugin.version>3.7.0</maven-compiler-plugin.version>
  </properties>

  <dependencies>
    <!-- JUnit -->
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>${junit.version}</version>
      <scope>test</scope>
    </dependency>
    <!-- EasyMock -->
    <dependency>
      <groupId>org.easymock</groupId>
      <artifactId>easymock</artifactId>
      <version>${easymock.version}</version>
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
      <artifactId>powermock-api-easymock</artifactId>
      <version>${powermock.version}</version>
      <scope>test</scope>
    </dependency>
    <!-- EL (Unified Expression Language) -->
    <dependency>
      <groupId>javax.el</groupId>
      <artifactId>javax.el-api</artifactId>
      <scope>test</scope>
      <version>${el.version}</version>
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

The `SomeBean` class below contains two methods that make use of `FacesContext`. The first `addMessage()` method will create a new `FacesMessage` and add it to the `FacesContext`. The second `logout()` method will invalidate the current session.

``` java
package com.codenotfound.easymock;

import javax.faces.application.FacesMessage;
import javax.faces.application.FacesMessage.Severity;
import javax.faces.bean.ManagedBean;
import javax.faces.bean.SessionScoped;
import javax.faces.context.FacesContext;

@ManagedBean
@SessionScoped
public class SomeBean {

  public void addMessage(Severity severity, String summary,
      String detail) {
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

Next is the `SomeBeanTest` JUnit test class. The class is annotated using two annotations. The first `@RunWith` annotation tells JUnit to run the test using `PowerMockRunner`. The second `@PrepareForTest` annotation tells PowerMock to prepare to mock the `FacesContext` class. If there are multiple classes to be prepared for mocking, they can be specified using a comma-separated list.

In the `setup()` method a number of objects are specified that are similar for the two test cases. The `mockStatic()` method is called in order to tell PowerMock to mock all static methods of the given `FacesContext` class. In addition the `FacesContext` and `ExternalContext` mock objects are created.

There are two test cases specified which follow the basic EasyMock testing steps:

| Step | Action                                                                               
| ---- | -------------------------------------------------------------------------------------
| 1    | Call `expect(mock.[method call]).andReturn([result])` for each expected call         
| 2    | Call `mock.[method call], then EasyMock.expectLastCall()` for each expected void call
| 3    | Call `replay(mock)` to switch from "record" mode to "playback" mode                  
| 4    | Call the test method                                                                 
| 5    | Call `verify(mock)` to assure that all expected calls happened                       

In addition to this, the first `addMessage()` test case uses the `Capture` capability of EasyMock in order to test whether a `FacesMessage` with the correct values was added to the `FacesContext`. The second `testLogout()` test case checks if the correct redirect was returned.

``` java
package com.codenotfound.easymock;

import static org.easymock.EasyMock.capture;
import static org.easymock.EasyMock.createMock;
import static org.easymock.EasyMock.expect;
import static org.easymock.EasyMock.expectLastCall;
import static org.easymock.EasyMock.replay;
import static org.easymock.EasyMock.verify;
import static org.junit.Assert.assertEquals;
import static org.junit.Assert.assertNull;

import javax.faces.application.FacesMessage;
import javax.faces.context.ExternalContext;
import javax.faces.context.FacesContext;

import org.easymock.Capture;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.powermock.api.easymock.PowerMock;
import org.powermock.core.classloader.annotations.PrepareForTest;
import org.powermock.modules.junit4.PowerMockRunner;

@RunWith(PowerMockRunner.class)
@PrepareForTest({FacesContext.class})
public class SomeBeanTest {

  private SomeBean someBean;

  private FacesContext facesContext;
  private ExternalContext externalContext;

  @Before
  public void setUp() throws Exception {
    someBean = new SomeBean();

    // mock all static methods of FacesContext using PowerMockito
    PowerMock.mockStatic(FacesContext.class);

    facesContext = createMock(FacesContext.class);
    externalContext = createMock(ExternalContext.class);
  }

  @Test
  public void testAddMessage() {
    // create Capture instances for the clientId and FacesMessage
    // parameters that will be added to the FacesContext
    Capture<String> clientIdCapture = new Capture<String>();
    Capture<FacesMessage> facesMessageCapture =
        new Capture<FacesMessage>();

    expect(FacesContext.getCurrentInstance()).andReturn(facesContext)
        .once();
    // expect the call to the addMessage() method and capture the
    // arguments
    facesContext.addMessage(capture(clientIdCapture),
        capture(facesMessageCapture));
    expectLastCall().once();

    // replay the class (not the instance)
    PowerMock.replay(FacesContext.class);
    replay(facesContext);

    someBean.addMessage(FacesMessage.SEVERITY_ERROR, "error",
        "something went wrong");

    // verify the class (not the instance)
    PowerMock.verify(FacesContext.class);
    verify(facesContext);

    // check the value of the clientId that was passed
    assertNull(clientIdCapture.getValue());

    // retrieve the captured FacesMessage
    FacesMessage captured = facesMessageCapture.getValue();
    // check if the captured FacesMessage contains the expected values
    assertEquals(FacesMessage.SEVERITY_ERROR, captured.getSeverity());
    assertEquals("error", captured.getSummary());
    assertEquals("something went wrong", captured.getDetail());
  }

  @Test
  public void testLogout() {
    expect(FacesContext.getCurrentInstance()).andReturn(facesContext)
        .once();
    expect(facesContext.getExternalContext())
        .andReturn(externalContext).once();
    // expect the call to the invalidateSession() method
    externalContext.invalidateSession();
    expectLastCall().once();

    // replay the class (not the instance)
    PowerMock.replay(FacesContext.class);
    replay(facesContext);
    replay(externalContext);

    assertEquals("logout?faces-redirect=true", someBean.logout());

    // verify the class (not the instance)
    PowerMock.verify(FacesContext.class);
    verify(facesContext);
    verify(externalContext);
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
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/easymock/tree/master/easymock-powermock-facescontext){:target="_blank"}.
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This concludes the mocking FacesContext using EasyMock and PowerMock example. If you found this post helpful or have any questions or remarks, please leave a comment.
