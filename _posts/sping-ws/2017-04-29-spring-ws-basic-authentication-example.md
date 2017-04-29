---
title: "Spring WS - Basic Authentication Example"
permalink: /2017/04/spring-ws-basic-authentication-example.html
excerpt: "A detailed step-by-step tutorial on how to configure basic authentication using Spring-WS and Spring Boot."
date: 2017-04-24
modified: 2017-04-24
categories: [Spring-WS]
tags: [Basic Authentication, Client, Endpoint, Example, HTTP, Maven, Spring, Spring Boot, Spring Web Services, Spring-WS, Tutorial]
published: true
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/spring-logo.jpg" alt="spring logo" class="logo">
</figure>

[Basic Authentication](https://en.wikipedia.org/wiki/Basic_access_authentication) (BA) is a method for a HTTP client to provide a user name and password when making a request. There is no [confidentiality](https://en.wikipedia.org/wiki/Confidentiality) protection for the transmitted credentials. therefore it is strongly advised to use it in conjunction with HTTPS.

The credentials are provided as an HTTP header field called <var>'Authorization'</var> which is constructed as follows:

1. The username and password are combined with a single colon.

    ``` plaintext
    codenotofound:p455w0rd
    ```

2. The resulting string is encoded into an [octet sequence](https://tools.ietf.org/html/rfc7617#section-2) and then [Base64 encoded](https://tools.ietf.org/html/rfc4648#section-4). You can use an [online Base64 decoder](https://www.base64decode.org/) to check below value.

    ``` plaintext
    Y29kZW5vdGZvdW5kOnA0NTV3MHJk
    ```
3. The authorization method and a space i.e. <kbd>"Basic "</kbd> is then put before the encoded string.

    ``` plaintext
    Authorization: Basic Y29kZW5vdGZvdW5kOnA0NTV3MHJk
    ```

Instead of writing custom code to create and check the HTTP authorization header we will configure Spring WS to do the work for us. The below example illustrates how a client and server can be configured to apply basic access authentication using Spring-WS, Spring Boot and Maven. 

Tools used:
* Spring-WS 2.4
* Spring Boot 1.5
* Maven 3.5

# General Project Setup

The setup of the project is based on a previous [Spring WS tutorial]({{ site.url }}/2016/10/spring-ws-soap-web-service-consumer-provider-wsdl-example.html) in which we have swapped out the basic <var>helloworld.wsdl</var> for a more generic <var>ticketagent.wsdl</var> from the [W3C WSDL 1.1 specification](https://www.w3.org/TR/wsdl11elementidentifiers/#Iri-ref-ex).

There are two additional dependencies that we need to add to the Maven POM file in order for our example to work.

The first one is `spring-boot-starter-security` [Spring Boot starter](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters) dependency which will be used for the server setup.

The second one is the Apache `httpclient` dependency that we need for the client setup part.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>spring-ws-basic-authentication</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>spring-ws-basic-authentication</name>
  <description>Spring WS - Basic Authentication Example</description>
  <url>https://www.codenotfound.com/spring-ws-basic-authentication-example.html</url>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.3.RELEASE</version>
  </parent>

  <properties>
    <java.version>1.8</java.version>

    <httpclient.version>4.5.3</httpclient.version>
    <maven-jaxb2-plugin.version>0.13.2</maven-jaxb2-plugin.version>
  </properties>

  <dependencies>
    <!-- spring-boot -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web-services</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <!-- httpclient -->
    <dependency>
      <groupId>org.apache.httpcomponents</groupId>
      <artifactId>httpclient</artifactId>
      <version>${httpclient.version}</version>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <!-- spring-boot-maven-plugin -->
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
      <!-- maven-jaxb2-plugin -->
      <plugin>
        <groupId>org.jvnet.jaxb2.maven2</groupId>
        <artifactId>maven-jaxb2-plugin</artifactId>
        <version>${maven-jaxb2-plugin.version}</version>
        <executions>
          <execution>
            <goals>
              <goal>generate</goal>
            </goals>
          </execution>
        </executions>
        <configuration>
          <schemaDirectory>${project.basedir}/src/main/resources/wsdl</schemaDirectory>
          <schemaIncludes>
            <include>*.wsdl</include>
          </schemaIncludes>
        </configuration>
      </plugin>
    </plugins>
  </build>
</project>
```



# Setup Client Basic Authentication 




# Setup Server Basic Authentication

The Spring Boot security starter that we added to our Maven setup has a dependency on Spring Security. If Spring Security is on the classpath then [web applications will be secured by default with HTTP basic authentication](http://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-security) on all HTTP endpoints. In other words our `TicketAgentEndpoint` is secured with basic auth.

The default user that will be configured has as name <var>user</var>. The password is randomly generated at startup. Typically you will want to configure a custom value for the user and password, in order to do this you need to set the [Spring Boot security properties](https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html) in your application properties file.

In this example set <var>'security:user'</var> name and password properties using the YAML variant as shown below. 

``` yml
security:
  user:
    name: codenotfound
    password: p455w0rd
```

# Testing the Basic Authentication Configuration



---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-ws/tree/master/spring-ws-basic-authentication).
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

Spring WS provides good support for setting and mapping the SOAPAction header as we have illustrated in above example.

If you have any additional thoughts let me know down below. Thanks!
