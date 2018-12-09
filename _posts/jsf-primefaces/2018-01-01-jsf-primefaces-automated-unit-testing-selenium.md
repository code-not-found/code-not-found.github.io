---
title: "JSF Primefaces Automated Unit Testing using Selenium"
permalink: /jsf-primefaces-automated-unit-testing-selenium.html
excerpt: "A detailed step-by-step tutorial on how to implement an automated unit test for PrimeFaces using Selenium."
date: 2018-01-01
last_modified_at: 2018-12-09
header:
  teaser: "assets/images/jsf-primefaces/jsf-primefaces-unit-testing-example.png"
categories: [PrimeFaces]
tags: [Automated Testing, Example, JUnit, Maven, PrimeFaces, Selenium, Spring Boot, Unit Testing]
published: true
---

<img src="{{ site.url }}/assets/images/jsf-primefaces/jsf-primefaces-unit-testing-example.png" alt="jsf primefaces unit testing example" class="align-right title-image">

Today you're going to see how to automate the unit testing of your [PrimeFaces](http://primefaces.org/){:target="_blank"} application.

The best part?

We will use the **PageObject pattern**.

So let's get to work.

If you want to learn more about PrimeFaces for JSF - head on over to the [JSF PrimeFaces tutorials]({{ site.url }}/jsf-primefaces-tutorials) page.
{: .notice--primary}

## 1. What is Selenium?

[Selenium](http://www.seleniumhq.org/){:target="_blank"} is a software testing framework for web applications. Tests can be run against most modern web browsers and can be controlled by many programming languages.

Selenium is available as open-source software, released under the Apache 2.0 license.

The following example shows how to set up an automated unit test for your PrimeFaces web application using Selenium, Spring Boot, and Maven.

## 2. General Project Overview

We will use the following tools/frameworks:
* PrimeFaces 6.2
* JoinFaces 3.3
* Spring Boot 2.1
* Selenium 3.14
* Maven 3.5

Our project has the following directory structure:

<img src="{{ site.url }}/assets/images/jsf-primefaces/jsf-primefaces-unit-testing-maven-project.png" alt="jsf primefaces unit testing maven project">

## 3. Maven Setup

We start from a previous [Spring Boot Primefaces Tutorial]({{ site.url }}/jsf-primefaces-example.html) in which we created a greeting dialog based on a first and last name input form.

We add `spring-boot-starter-test` to the existing [Maven](https://maven.apache.org/){:target="_blank"} POM file. This will include the core dependencies for testing Spring Boot applications with libraries that include [JUnit](http://junit.org/junit4/){:target="_blank"}, [Hamcrest](http://hamcrest.org/JavaHamcrest/){:target="_blank"} and [Mockito](http://site.mockito.org/){:target="_blank"}.

In order to create local Selenium scripts, we need to make use of language-specific client drivers. As this example is based on Java we include the `selenium-java` client dependency.

The key interface against which tests should be written in Java is the [WebDriver](http://www.seleniumhq.org/projects/webdriver/){:target="_blank"} interface which has a number of implementing classes.

For this example, we will use [HtmlUnitDriver](https://github.com/seleniumhq/htmlunit-driver){:target="_blank"} which is a WebDriver compatible driver for the [HtmlUnit](http://htmlunit.sourceforge.net/){:target="_blank"} headless browser. As such we also add the `htmlunit-driver` dependency.

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>jsf-primefaces-unit-testing-selenium</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>jsf-primefaces-unit-testing-selenium</name>
  <description>JSF Primefaces Automated Unit Testing using Selenium</description>
  <url>https://codenotfound.com/jsf-primefaces-automated-unit-testing-selenium.html</url>

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
    <selenium-htmlunit-driver.version>2.52.0</selenium-htmlunit-driver.version>
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
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.seleniumhq.selenium</groupId>
      <artifactId>selenium-java</artifactId>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.seleniumhq.selenium</groupId>
      <artifactId>htmlunit-driver</artifactId>
      <scope>test</scope>
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

## 4. Creating a PrimeFaces Test Page

To test the PrimeFaces Hello World page, we will use the [PageObject pattern](https://martinfowler.com/bliki/PageObject.html){:target="_blank"}. Within our web app's UI there are areas that our test will interact with (in this example input/output text fields and a button). A page object simply models these as objects within the test code.

The basic rule of thumb for a page object is that it should allow a software client to do anything and see anything that a human can. It should also provide an interface that's easy to program to and **hides the underlying widgetry** in the window. In this example, the page object is the `HelloWorldPage` class shown below.

In order to set/get the input/output field values we lookup the corresponding `WebElement`s using the `@FindBy` annotation. Matching is done using the <var>'id'</var> of the HTML elements which are specified in the <var>helloworld.xhtml</var> located under <var>src/main/resources/META-INF/resources</var>. The submit button is also retrieved using its corresponding ID.

Note that `@FindBy` is just an alternate way of finding elements. You could also use the `findElement()` method on the `WebDriver`.

> The JSF framework prefixes HTML elements inside a <var>&lt;form&gt;</var> with the [ID of the form](https://stackoverflow.com/a/8279214/4201470){:target="_blank"}. If no ID is present then one will be auto-generated.

Next, we represent our submit button as a `submit()` method that takes a first and last name as an input parameter. These parameters are set on the corresponding input fields and the submit button is executed. The method finishes by reloading the page elements so that the greeting output field is set with the new value.

The `getGreeting()` method allows to retrieve the greeting shown to the user. Note that we used `getAttribute("textContent")` as using `getText()` [returns an empty string](http://yizeng.me/2014/04/08/get-text-from-hidden-elements-using-selenium-webdriver/){:target="_blank"}.

{% highlight java %}
package com.codenotfound.primefaces;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;
import com.codenotfound.PageObject;

public class HelloWorldPage extends PageObject {

  @FindBy(id = "hello-world-form:first-name")
  private WebElement firstNameInput;

  @FindBy(id = "hello-world-form:last-name")
  private WebElement lastNameInput;

  @FindBy(id = "hello-world-form:submit")
  private WebElement submitButton;

  @FindBy(id = "hello-world-form:greeting")
  private WebElement greetingOutput;

  public HelloWorldPage(WebDriver driver) {
    super(driver);
  }

  public void submit(String firstName, String lastName) {
    // set the input fields
    firstNameInput.sendKeys(firstName);
    lastNameInput.sendKeys(lastName);
    // submit the form
    submitButton.submit();
    // refresh the output field
    PageFactory.initElements(driver, this);
  }

  public String getGreeting() {
    return greetingOutput.getAttribute("textContent");
  }
}
{% endhighlight %}

Notice that the above `HelloWorldPage` page object extends a `PageObject` class. This is a small helper class that uses the [PageFactory](https://github.com/SeleniumHQ/selenium/wiki/PageFactory){:target="_blank"} factory class provided by the WebDriver's support library to help realize the PageObject pattern.

In the constructor, the `PageFactory` is used to instantiate WebElements that we have annotated in the `HelloWorldPage` class.

{% highlight java %}
package com.codenotfound;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.support.PageFactory;

public class PageObject {
  protected WebDriver driver;

  public PageObject(WebDriver driver) {
    this.driver = driver;
    PageFactory.initElements(driver, this);
  }
}
{% endhighlight %}

In our test case, we navigate to the Hello World web page using the `get()` method of the `WebDriver`. We create a new instance of our `HelloWorldPage` page object and call the `submit()` method.

Finally, an assert is used to verify that the correct greeting is generated.

{% highlight java %}
package com.codenotfound.primefaces;

import static org.assertj.core.api.Assertions.assertThat;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.SpringBootTest.WebEnvironment;
import org.springframework.test.context.junit4.SpringRunner;
import com.codenotfound.WebDriverUtil;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = WebEnvironment.DEFINED_PORT)
public class HelloWorldTest extends WebDriverUtil {

  @Test
  public void testSubmit() {
    driver.get("http://localhost:8080/helloworld.xhtml");

    HelloWorldPage page = new HelloWorldPage(driver);
    page.submit("Jane", "Doe");

    assertThat(page.getGreeting()).isEqualTo("Hello Jane Doe!");
  }
}
{% endhighlight %}

Based on following example on [writing functional tests using Selenium](https://www.pluralsight.com/guides/getting-started-with-page-object-pattern-for-your-selenium-tests){:target="_blank"} we have made above test case extend a `WebDriverUtil` class.

This class holds all the `WebDriver` lifecycle management code and assures correct setup and cleanup is done after each test case.

{% highlight java %}
package com.codenotfound;

import java.util.concurrent.TimeUnit;
import org.junit.After;
import org.junit.AfterClass;
import org.junit.BeforeClass;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.htmlunit.HtmlUnitDriver;

public class WebDriverUtil {

  protected static WebDriver driver;

  @BeforeClass
  public static void setUp() {
    driver = new HtmlUnitDriver();
    driver.manage().timeouts().implicitlyWait(10, TimeUnit.SECONDS);
  }

  @After
  public void cleanUp() {
    driver.manage().deleteAllCookies();
  }

  @AfterClass
  public static void tearDown() {
    driver.close();
  }
}
{% endhighlight %}

Let's go ahead and run the `HelloWorldTest` test case. Open a command prompt in the projects root folder and execute following Maven command:

{% highlight plaintext %}
mvn test
{% endhighlight %}

The result should be a successful build during which the test is executed.

{% highlight plaintext %}
.   ____          _            __ _ _
/\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
\\/  ___)| |_)| | | | | || (_| |  ) ) ) )
'  |____| .__|_| |_|_| |_\__, | / / / /
=========|_|==============|___/=/_/_/_/
:: Spring Boot ::        (v2.1.0.RELEASE)

18:15:51.168 [main] INFO  c.c.primefaces.HelloWorldTest - Starting HelloWorldTest on DESKTOP-2RB3C1U with PID 18372 (started by Codenotfound in C:\Users\Codenotfound\repos\jsf-primefaces\jsf-primefaces-unit-testing-selenium)
18:15:51.168 [main] INFO  c.c.primefaces.HelloWorldTest - No active profile set, falling back to default profiles: default
18:15:57.670 [main] INFO  c.c.primefaces.HelloWorldTest - Started HelloWorldTest in 6.862 seconds (JVM running for 9.115)
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 10.551 s - in com.codenotfound.primefaces.HelloWorldTest
[INFO]
[INFO] Results:
[INFO]
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 16.001 s
[INFO] Finished at: 2018-12-08T18:16:00+01:00
[INFO] ------------------------------------------------------------------------
{% endhighlight %}

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/jsf-primefaces/tree/master/jsf-primefaces-unit-testing-selenium){:target="_blank"}.
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

Using the PageObject pattern we were able to unit test a previous PrimeFaces Hello World example using Selenium, Spring Boot, and Maven.

Drop a line below in case you enjoyed reading.

Or to let me know how you are testing your JSF application.
