---
title: "JSF Primefaces Themes Example"
permalink: /jsf-primefaces-themes-example.html
excerpt: "A code sample on how to configure the PrimeFaces Bootstrap theme using Spring Boot."
date: 2018-01-02
last_modified_at: 2018-12-06
header:
  teaser: "assets/images/jsf-primefaces/jsf-primefaces-themes-example.png"
categories: [PrimeFaces]
tags: [Bootstrap, Code Sample, Example, JavaServer Faces, JSF, Maven, PrimeFaces, Spring Boot, Theme, PrimeFaces Theme]
redirect_from:
  - /jsf-primefaces-theme-spring-boot.html
published: true
---

<img src="{{ site.url }}/assets/images/jsf-primefaces/jsf-primefaces-themes-example.png" alt="jsf primefaces themes example" class="align-right title-image">

Today you’re going to learn how to setup a [PrimeFaces](https://www.primefaces.org/){:target="_blank"} Theme.

(FAST)

Let’s jump right in…

If you want to learn more about PrimeFaces for JSF - head on over to the [JSF PrimeFaces tutorials]({{ site.url }}/jsf-primefaces-tutorials) page.
{: .notice--primary}

## 1. What are PrimeFaces Themes?

PrimeFaces is integrated with the **ThemeRoller CSS Framework** in order to support different themes. There are three types of themes you can configure:

1. You can [purchase premium themes](https://www.primefaces.org/themes/){:target="_blank"}.
2. You can choose from the [free community themes](https://www.primefaces.org/themes/){:target="_blank"} (click the Community tab).
3. You can create your own theme using the online theme generator tool of [ThemeRoller](http://jqueryui.com/themeroller/){:target="_blank"}.

The following example shows how to set up the Bootstrap PrimeFaces theme using Spring Boot and Maven.

## 2. General Project Overview

We will use the following tools/frameworks:
* PrimeFaces 6.2
* JoinFaces 3.3
* Spring Boot 2.1
* Maven 3.5

Our project has the following directory structure:

<img src="{{ site.url }}/assets/images/jsf-primefaces/jsf-primefaces-themes-maven-project.png" alt="jsf primefaces themes maven project">

## 3. Maven Setup

As the PrimeFaces community themes are not available in the Maven central repository we need to specify the [PrimeFaces Maven Repository](http://repository.primefaces.org){:target="_blank"} in our Maven POM file as shown below.

Once this is done we need to include the [dependency to the specific PrimeFaces theme](https://repository.primefaces.org/org/primefaces/themes/){:target="_blank"} we want to use.

Alternatively, we can include all available themes by specifying the `all-themes` dependency. We will do the later in this example.

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>jsf-primefaces-themes</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>jsf-primefaces-themes</name>
  <description>JSF Primefaces Themes Example</description>
  <url>https://codenotfound.com/jsf-primefaces-themes-example.html</url>

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
      <groupId>org.primefaces.themes</groupId>
      <artifactId>all-themes</artifactId>
      <version>1.0.10</version>
    </dependency>
  </dependencies>

  <repositories>
    <repository>
      <id>primefaces-maven-repository</id>
      <name>PrimeFaces Maven Repository</name>
      <url>http://repository.primefaces.org</url>
    </repository>
  </repositories>

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

We will start from a previous [JSF Spring Boot Example]({{ site.url }}/jsf-primefaces-example.html) in which we created a greeting dialog based on a first and last name input field.

## 4. PrimeFaces Bootstrap Theme Setup

As we are using [JoinFaces](https://github.com/joinfaces/joinfaces#joinfaces){:target="_blank"} to setup Spring Boot we can use an [application property](https://github.com/joinfaces/joinfaces#jsf-properties-configuration-via-applicationproperties-or-applicationyml){:target="_blank"} in order to configure a PrimeFaces theme.

We set the <var>Bootstrap</var> theme by specifying the <var>jsf:primefaces:theme</var> property in <var>src/main/resources/application.yml</var> as shown below.

{% highlight yaml %}
jsf:
  primefaces:
    theme: bootstrap
{% endhighlight %}

Let's check the new look of our web application. Execute following Maven command:

{% highlight plaintext %}
mvn spring-boot:run
{% endhighlight %}

Once Spring Boot has started, open a web browser and enter the following URL: [http://localhost:8080/helloworld.xhtml](http://localhost:8080/helloworld.xhtml){:target="_blank"}.

The web page should now be displayed in the newly configured theme.

<img src="{{ site.url }}/assets/images/jsf-primefaces/jsf-primefaces-bootstrap-theme.png" alt="jsf primefaces bootstrap theme">

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/jsf-primefaces/tree/master/jsf-primefaces-themes){:target="_blank"}.
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

In this post, we illustrated how to configure the PrimeFaces Bootstrap theme using Spring Boot and Maven.

Drop a line below if you enjoyed reading this post.
