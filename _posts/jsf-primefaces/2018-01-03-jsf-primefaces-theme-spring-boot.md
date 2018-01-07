---
title: "JSF - Primefaces Theme using Spring Boot"
permalink: /jsf-primefaces-theme-spring-boot.html
excerpt: "A code sample on how to configure a Primefaces theme using Spring Boot."
date: 2018-01-02
last_modified_at: 2018-01-02
header:
  teaser: "assets/images/teaser/primefaces-teaser.png"
categories: [PrimeFaces]
tags: [Code Sample, Example, JavaServer Faces, JSF, Maven, PrimeFaces, Spring Boot, Theme, PrimeFaces Theme]
published: true
---

<figure>
  <img src="{{ site.url }}/assets/images/logo/primefaces-logo.png" alt="primefaces logo" class="logo">
</figure>

[PrimeFaces](http://primefaces.org/){:target="_blank"} is integrated with the ThemeRoller CSS Framework in order to support different themes. There are three types of themes you can configure:

1. You can [purchase premium themes](https://www.primefaces.org/themes/){:target="_blank"}
2. You can choose from the [free community themes](https://www.primefaces.org/themes/){:target="_blank"}
3. You can create your own theme using the online theme generator tool of [ThemeRoller](http://jqueryui.com/themeroller/){:target="_blank"}.

The following example shows how to setup a PrimeFaces theme using Spring Boot and Maven.

# General Project Setup

Tools used:
* Spring Boot 1.5
* PrimeFaces 6.1
* JoinFaces 2.4
* Maven 3.5

As the PrimeFaces community themes are not available in the [Maven central repository](http://repo1.maven.org/){:target="_blank"} we need to specify the [PrimeFaces Maven Repository](http://repository.primefaces.org){:target="_blank"} in our Maven POM file as shown below.

Once this is done we need to include the [dependency to the specific PrimeFaces theme](https://repository.primefaces.org/org/primefaces/themes/){:target="_blank"} we want to use. Alternatively, we can include all available themes by specifying the `all-themes` dependency. We will do the later in this example.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>jsf-primefaces-theme-spring-boot</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>jsf-primefaces-theme-spring-boot</name>
  <description>JSF - Primefaces Theme using Spring Boot</description>
  <url>https://www.codenotfound.com/jsf-primefaces-theme-spring-boot.html</url>

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
    <!-- primefaces -->
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
      <!-- spring-boot-maven-plugin -->
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>
</project>
```

We will start from a previous [JSF Spring Boot Example ]({{ site.url }}/jsf-primefaces-example-spring-boot-maven.html) in which we created a greeting dialog based on a first and last name input field.

As we are using [JoinFaces](https://github.com/joinfaces/joinfaces#joinfaces){:target="_blank"} to setup Spring Boot we can use an [application property](https://github.com/joinfaces/joinfaces#jsf-properties-configuration-via-applicationproperties-or-applicationyml){:target="_blank"} in order to configure a PrimeFaces theme.

We set the <var>'Afterdark'</var> theme by specifying the <var>'jsf.primefaces.theme'</var> property in <var>src/main/resources/application.yml</var> as shown below.

``` yaml
jsf:
  primefaces: 
    theme: afterdark

server:
  context-path: /codenotfound
  port: 9090
```

Let's checkout our web applicatiosn new look by running following Maven command:

``` plaintext
mvn spring-boot:run
```

Once Spring Boot has started, open a web browser and enter the following URL: [http://localhost:9090/codenotfound/helloworld.xhtml](http://localhost:9090/codenotfound/helloworld.xhtml){:target="_blank"}.

The below web page should now be displayed in the newly configured theme.

<figure>
  <img src="{{ site.url }}/assets/images/posts/jsf-primefaces/jsf-primefaces-theme-spring-boot.png" alt="jsf primefaces theme spring boot">
</figure>

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/jsf-primefaces/tree/master/jsf-primefaces-theme-spring-boot){:target="_blank"}.
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

In this post, we illustrated how to configure a Spring Boot PrimeFaces theme.

Drop a line below if you enjoyed reading this post.
