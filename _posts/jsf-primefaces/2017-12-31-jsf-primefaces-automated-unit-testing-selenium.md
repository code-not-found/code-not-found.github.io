---
title: "JSF - Primefaces Automated Unit Testing using Selenium"
permalink: /jsf-primefaces-automated-unit-testing-selenium.html
excerpt: "A detailed step-by-step tutorial on how to implement an automated unit test for PrimeFaces using Selenium."
date: 2018-01-01
last_modified_at: 2018-01-01
header:
  teaser: "assets/images/teaser/primefaces-teaser.png"
categories: [PrimeFaces]
tags: [Automated Testing, Example, JUnit, Maven, PrimeFaces, Selenium, Spring Boot, Unit Testing, Tutorial]
published: true
---

<figure>
  <img src="{{ site.url }}/assets/images/logo/primefaces-logo.png" alt="primefaces logo" class="logo">
</figure>

[Selenium](http://www.seleniumhq.org/){:target="_blank"} is a software-testing framework for web applications. Tests can be run against most modern web browsers and can be controlled by many programming languages. Selenium is available as open-source software, released under the Apache 2.0 license.

The following example shows how to setup an automated unit test for your PrimeFaces web application using Selenium, Spring Boot, and Maven.

# General Project Setup

Tools used:
* PrimeFaces 6.1
* Selenium 3.8
* JoinFaces 2.4
* Spring Boot 1.5
* Maven 3.5

We will start from a previous [Spring Boot Primefaces Tutorial ]({{ site.url }}/jsf-primefaces-example-spring-boot-maven.html) in which we created a greeting dialog based on a first and last name input form.

We add `spring-boot-starter-test` to the existing [Maven](https://maven.apache.org/){:target="_blank"} POM file. This will include the core dependencies for testing Spring Boot applications with libraries that include [JUnit](http://junit.org/junit4/){:target="_blank"}, [Hamcrest](http://hamcrest.org/JavaHamcrest/){:target="_blank"} and [Mockito](http://site.mockito.org/){:target="_blank"}.

In order to create local Selenium scripts, we need to make use of language-specific client drivers. As this example is based on Java we include the `selenium-java` dependency.

The key interface against which tests should be written in Java is the [WebDriver](http://www.seleniumhq.org/projects/webdriver/){:target="_blank"} which has a number of implementing classes.

For this example we will use [HtmlUnitDriver](https://github.com/seleniumhq/htmlunit-driver){:target="_blank"} which is a WebDriver compatible driver for [HtmlUnit](http://htmlunit.sourceforge.net/){:target="_blank"} headless browser. As such we also add the `selenium-htmlunit-driver` dependency.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>jsf-primefaces-unit-testing-selenium</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>jsf-primefaces-unit-testing-selenium</name>
  <description>JSF - Primefaces Automated Unit Testing using Selenium</description>
  <url>https://www.codenotfound.com/jsf-primefaces-automated-unit-testing-selenium.html</url>

  <parent>
    <groupId>org.joinfaces</groupId>
    <artifactId>jsf-spring-boot-parent</artifactId>
    <version>2.4.1</version>
    <relativePath /> <!-- lookup parent from repository -->
  </parent>

  <properties>
    <java.version>1.8</java.version>

    <selenium-java.version>3.8.1</selenium-java.version>
    <selenium-htmlunit-driver.version>2.52.0</selenium-htmlunit-driver.version>
  </properties>

  <dependencies>
    <!-- spring-boot -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
    <!-- joinfaces -->
    <dependency>
      <groupId>org.joinfaces</groupId>
      <artifactId>jsf-spring-boot-starter</artifactId>
    </dependency>
    <!-- selenium -->
    <dependency>
      <groupId>org.seleniumhq.selenium</groupId>
      <artifactId>selenium-java</artifactId>
      <version>${selenium-java.version}</version>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.seleniumhq.selenium</groupId>
      <artifactId>selenium-htmlunit-driver</artifactId>
      <version>${selenium-htmlunit-driver.version}</version>
      <scope>test</scope>
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

# Creating a PrimeFaces Test Page

In order to test the PrimeFaces Hello World page we will use the [PageObject pattern](https://martinfowler.com/bliki/PageObject.html){:target="_blank"}. Within our web app's UI there are areas that our test will interact with (in this example input text fields and a button). A page object simply models these as objects within the test code.

The basic rule of thumb for a page object is that it should allow a software client to do anything and see anything that a human can. It should also provide an interface that's easy to program to and hides the underlying widgetry in the window. So we will create a `HelloWorldPage` class that represents our submit button as a `submit()` method that takes a first and last name as an input parameter.

In order to set the input field values we lookup the corresponding `WebElement`s using the `@FindBy` annotation. Matching is done using the <var>'id'</var> of the HTML elements which are specified in the <var>helloworld.xhtml</var> located under <var>src/main/resources/META-INF/resources</var>.

Note that `@FindBy` is just an alternate way of finding elements. You could also use the `findElement()` method on the `WebDriver`.

> The JSF framework prefixes HTML elements inside a <var>&lt;form&gt;</var> with the [ID of the form](https://stackoverflow.com/a/8279214/4201470){:target="_blank"}. If no ID is present then one will be auto-generated.

We also lookup the submit button and once the input fields are set we trigger it calling the `submit()` method as shown below.

``` java
package com.codenotfound.primefaces.view;

import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;

public class HelloWorldPage {

  @FindBy(id = "hello-world-form:first-name")
  private WebElement firstNameInput;

  @FindBy(id = "hello-world-form:last-name")
  private WebElement lastNameInput;

  @FindBy(id = "hello-world-form:submit")
  private WebElement submitButton;

  public void submit(String firstName, String lastName) {
    firstNameInput.sendKeys(firstName);
    lastNameInput.sendKeys(lastName);
    submitButton.submit();
  }
}
```

The WebDriver's support library contains a [PageFactory](https://github.com/SeleniumHQ/selenium/wiki/PageFactory){:target="_blank"} factory class in order to support the PageObject pattern.

In order to use the above defined `HelloWorldPage` and not have it throw a `NullPointerException`, we need to initialize it using the `PageFactory` as shown below.

We then simply call the `submit()` method and assert that the correct greeting is generated.

> Note we used `getAttribute("textContent")` as using `getText()` [returns an empty string](http://yizeng.me/2014/04/08/get-text-from-hidden-elements-using-selenium-webdriver/){:target="_blank"}.

``` java
package com.codenotfound.primefaces.view;

import static org.assertj.core.api.Assertions.assertThat;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.htmlunit.HtmlUnitDriver;
import org.openqa.selenium.support.PageFactory;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.SpringBootTest.WebEnvironment;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = WebEnvironment.DEFINED_PORT)
public class HelloWorldTest {

  @Test
  public void testSubmit() throws InterruptedException {
    WebDriver driver = new HtmlUnitDriver();
    driver.get("http://localhost:9090/codenotfound/helloworld.xhtml");

    HelloWorldPage page =
        PageFactory.initElements(driver, HelloWorldPage.class);
    page.submit("Jane", "Doe");

    assertThat(driver.findElement(By.id("hello-world-form:greeting"))
        .getAttribute("textContent")).isEqualTo("Hello Jane Doe!");
  }
}
```

Run the above test case by opening a command prompt in the projects root folder and executing following Maven command:

``` plaintext
mvn test
```

The result should be a successful build.

``` plaintext
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.5.7.RELEASE)

09:25:38.666 [main] INFO  c.c.primefaces.view.HelloWorldTest - Starting HelloWorldTest on cnf-pc with PID 2808 (started by CodeNotFound in c:\codenotfound\code\jsf-primefaces\jsf-primefaces-unit-testing-selenium)
09:25:38.668 [main] INFO  c.c.primefaces.view.HelloWorldTest - No active profile set, falling back to default profiles: default
09:25:43.773 [main] INFO  c.c.primefaces.view.HelloWorldTest - Started HelloWorldTest in 5.405 seconds (JVM running for 6.127)
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 7.454 sec - in com.codenotfound.primefaces.view.HelloWorldTest

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 10.405 s
[INFO] Finished at: 2018-01-01T09:25:45+01:00
[INFO] Final Memory: 23M/228M
[INFO] ------------------------------------------------------------------------
```

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/jsf-primefaces/tree/master/jsf-primefaces-unit-testing-selenium){:target="_blank"}.
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

Using the PageObject pattern we were able to unit test a previous PrimeFaces Hello World example using Selenium, Spring Boot, and Maven. Drop a line below in case you enjoyed reading or just to let us know how you are testing your JSF application.
