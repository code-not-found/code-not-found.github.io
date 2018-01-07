---
title: "JAX-WS - CXF logging request and response SOAP messages using Log4j"
permalink: /jaxws-cxf-logging-request-response-soap-messages-log4j.html
excerpt: "A code sample which shows how to configure CXF to log the request and response SOAP messages using Log4j."
date: 2015-01-11
last_modified_at: 2015-01-11
header:
  teaser: "assets/images/teaser/apache-cxf-teaser.png"
categories: [Apache CXF]
tags: [Apache CXF, Code Sample, CXF, JAX-WS, Jetty, log4j, Logging, Maven, SOAP Message]
redirect_from:
  - /2014/09/jax-ws-cxf-logging-inbound-and-outbound.html
  - /2015/01/jaxws-cxf-logging-request-response-soap-messages-log4j.html
published: true
---

<figure>
    <img src="{{ site.url }}/assets/images/logo/apache-cxf-logo.png" alt="apache cxf logo" class="logo">
</figure>

[Apache CXF](https://cxf.apache.org/){:target="_blank"} uses Java SE Logging for both client- and server-side logging of SOAP requests and responses. Logging is activated by use of separate in/out interceptors that can be attached to the requester and/or provider as required. These interceptors can be specified either programmatically (via Java code and/or annotations) or via the use of configuration files.

The following code sample shows how to configure CXF interceptors using Log4j for the [Hello World web service from a previous post]({{ site.url }}/jaxws-cxf-contract-first-hello-world-webservice-tutorial.html).

Tools used:
* CXF 3.0
* Spring 4.1
* Log4j2 2.1
* Jetty 8.1
* Maven 3.5

Specifying the interceptors via configuration files offers two benefits over programmatic configuration:
1. Logging requirements **can be altered without needing to recompile the code**.
2. **No Apache CXF-specific APIs need to be added to your code**, which helps it remain interoperable with other JAX-WS compliant web service stacks.

For this example [Log4j](https://logging.apache.org/log4j/1.2/){:target="_blank"} will be used which is is a Java-based logging utility. As a best practice the CXF `java.util.logging` calls will first be redirected to [SLF4J](http://www.slf4j.org/){:target="_blank"} (Simple Logging Facade for Java) as described here. In other words a <var>META-INF/cxf/org.apache.cxf.Logger</var> file will be created on the classpath containing the following:

``` plaintext
org.apache.cxf.common.logging.Slf4jLogger
```

As the Hello World example uses Spring, the commons-logging calls from the Spring framework will also be redirected to SLF4J using [jcl-over-slf4j](http://www.slf4j.org/legacy.html){:target="_blank"}. Now that all logging calls of both CXF and Spring are redirected to SLF4J, Log4j will be plugged into SLF4J using the [slf4j-log4j12](http://www.slf4j.org/legacy.html){:target="_blank"} adaptation layer. The picture below illustrates the described approach.

<figure>
    <img src="{{ site.url }}/assets/images/posts/cxf/cxf-logging-using-log4j.png" alt="cxf logging using log4j">
</figure>

The below Maven POM file contains the needed dependencies for the SLF4 bridge `jcl-over-slf4j`, Log4j adaptation layer `slf4j-log4j12` and Log4j `log4j`. In addition, it contains all other needed dependencies and plugins needed in order to run the example.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>cxf-jaxws-jetty-logging-log4j</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>war</packaging>

  <name>cxf-jaxws-jetty-logging-log4j</name>
  <description>JAX-WS - CXF logging request and response SOAP messages using Log4j</description>
  <url>https://codenotfound.com/jaxws-cxf-logging-request-response-soap-messages-log4j.html</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <java.version>1.8</java.version>

    <slf4j.version>1.7.10</slf4j.version>
    <log4j.version>1.2.17</log4j.version>
    <junit.version>4.12</junit.version>
    <cxf.version>3.0.3</cxf.version>
    <spring.version>4.1.4.RELEASE</spring.version>

    <maven-compiler-plugin.version>3.1</maven-compiler-plugin.version>
    <jetty-maven-plugin.version>8.1.16.v20140903</jetty-maven-plugin.version>
    <maven-failsafe-plugin.version>2.17</maven-failsafe-plugin.version>
  </properties>

  <dependencies>
    <!-- Logging -->
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>jcl-over-slf4j</artifactId>
      <version>${slf4j.version}</version>
    </dependency>
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-log4j12</artifactId>
      <version>${slf4j.version}</version>
    </dependency>
    <dependency>
      <groupId>log4j</groupId>
      <artifactId>log4j</artifactId>
      <version>${log4j.version}</version>
    </dependency>
    <!-- JUnit -->
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>${junit.version}</version>
      <scope>test</scope>
    </dependency>
    <!-- CXF -->
    <dependency>
      <groupId>org.apache.cxf</groupId>
      <artifactId>cxf-rt-frontend-jaxws</artifactId>
      <version>${cxf.version}</version>
    </dependency>
    <dependency>
      <groupId>org.apache.cxf</groupId>
      <artifactId>cxf-rt-transports-http</artifactId>
      <version>${cxf.version}</version>
    </dependency>
    <dependency>
      <groupId>org.apache.cxf</groupId>
      <!-- Jetty is needed if you're are not using the CXFServlet -->
      <artifactId>cxf-rt-transports-http-jetty</artifactId>
      <version>${cxf.version}</version>
    </dependency>
    <!-- Spring -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>${spring.version}</version>
      <exclusions>
        <exclusion>
          <artifactId>commons-logging</artifactId>
          <groupId>commons-logging</groupId>
        </exclusion>
      </exclusions>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-web</artifactId>
      <version>${spring.version}</version>
      <exclusions>
        <exclusion>
          <artifactId>commons-logging</artifactId>
          <groupId>commons-logging</groupId>
        </exclusion>
      </exclusions>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-test</artifactId>
      <version>${spring.version}</version>
      <scope>test</scope>
      <exclusions>
        <exclusion>
          <artifactId>commons-logging</artifactId>
          <groupId>commons-logging</groupId>
        </exclusion>
      </exclusions>
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
      <!-- jetty-maven-plugin -->
      <plugin>
        <groupId>org.mortbay.jetty</groupId>
        <artifactId>jetty-maven-plugin</artifactId>
        <version>${jetty-maven-plugin.version}</version>
        <configuration>
          <connectors>
            <connector
              implementation="org.eclipse.jetty.server.nio.SelectChannelConnector">
              <port>9090</port>
            </connector>
          </connectors>
          <stopPort>8005</stopPort>
          <stopKey>STOP</stopKey>
          <daemon>true</daemon>
        </configuration>
        <executions>
          <execution>
            <id>start-jetty</id>
            <phase>pre-integration-test</phase>
            <goals>
              <goal>start</goal>
            </goals>
          </execution>
          <execution>
            <id>stop-jetty</id>
            <phase>post-integration-test</phase>
            <goals>
              <goal>stop</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
      <!-- maven-failsafe-plugin -->
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-failsafe-plugin</artifactId>
        <version>${maven-failsafe-plugin.version}</version>
        <executions>
          <execution>
            <goals>
              <goal>integration-test</goal>
              <goal>verify</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
      <!-- cxf-codegen-plugin -->
      <plugin>
        <groupId>org.apache.cxf</groupId>
        <artifactId>cxf-codegen-plugin</artifactId>
        <version>${cxf.version}</version>
        <executions>
          <execution>
            <id>generate-sources</id>
            <phase>generate-sources</phase>
            <configuration>
              <sourceRoot>${project.build.directory}/generated/cxf</sourceRoot>
              <wsdlOptions>
                <wsdlOption>
                  <wsdl>${project.basedir}/src/main/resources/wsdl/helloworld.wsdl</wsdl>
                  <wsdlLocation>classpath:wsdl/helloworld.wsdl</wsdlLocation>
                </wsdlOption>
              </wsdlOptions>
            </configuration>
            <goals>
              <goal>wsdl2java</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
</project>
```

# Log4j configuration
 The log4j environment is fully configurable programmatically. However, it is far more flexible to configure log4j using configuration files. Currently, configuration files can be written in XML or in Java properties (key=value) format. For this example, a configuration script in XML is used as shown below. The script will write all logging events to a file except for the request and response SOAP messages that will be written to a different file.

Two appenders are configured to write logging events to a rolling file. The first file <var>jaxws-jetty-cxf-logging.log</var> will contain all log events except for the ones emitted by the CXF logging interceptors as these are going to be written to <var>jaxws-jetty-cxf-logging-ws.log</var>.

The last section contains the different loggers and the level at which information is logged. The log level of the <var>'org.apache.cxf.services'</var> logger needs to be set to INFO in order to activate the SOAP logging events. In addition the <var>WS_LOG_FILE</var> appender needs to be referenced in order to write the SOAP logging events to a different file. Note the <var>'additivity="false"'</var> which makes sure the log events are not written to appenders attached to its ancestors (in this case <var>APP_LOG_FILE</var>). 

``` xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE log4j:configuration SYSTEM "log4j.dtd">
<log4j:configuration debug="false"
  xmlns:log4j='http://jakarta.apache.org/log4j/'>

  <!-- APPENDERS -->
  <appender name="APP_LOG_FILE" class="org.apache.log4j.DailyRollingFileAppender">
    <param name="file" value="jaxws-jetty-cxf-logging.log" />
    <param name="DatePattern" value="'.'yyyy-MM-dd" />
    <layout class="org.apache.log4j.PatternLayout">
      <param name="ConversionPattern" value="%d{HH:mm:ss.SSS} %-5p [%t][%c{1}] %m%n" />
    </layout>
  </appender>

  <appender name="WS_LOG_FILE" class="org.apache.log4j.DailyRollingFileAppender">
    <param name="file" value="jaxws-jetty-cxf-logging-ws.log" />
    <param name="DatePattern" value="'.'yyyy-MM-dd" />
    <layout class="org.apache.log4j.PatternLayout">
      <param name="ConversionPattern" value="%d{HH:mm:ss.SSS} %-5p [%t][%c{1}] %m%n" />
    </layout>
  </appender>

  <!-- LOGGERS -->
  <logger name="com.codenotfound.soap.http.cxf">
    <level value="INFO" />
  </logger>

  <logger name="org.springframework">
    <level value="WARN" />
  </logger>
  <logger name="org.apache.cxf">
    <level value="WARN" />
  </logger>

  <!-- level INFO needed to log SOAP messages -->
  <logger name="org.apache.cxf.services" additivity="false">
    <level value="INFO" />
    <!-- specify a dedicated appender for the SOAP messages -->
    <appender-ref ref="WS_LOG_FILE" />
  </logger>

  <root>
    <level value="WARN" />
    <appender-ref ref="APP_LOG_FILE" />
  </root>

</log4j:configuration>
```

# Requester
 In order to activate the logging interceptors provided by the CXF framework, there are two options. For the requester(client) the option where all logging interceptors are configured manually will be illustrated.
 
 The other option, where the logging feature is used to configure all interceptors, will be shown in the provider section down below. Check out following post for more information on [cxf feature vs interceptor]({{ site.url }}/cxf-feature-vs-interceptor.html).

First, an instance of the abstract `AbstractLoggingInterceptor` class is created to enable pretty printing of the SOAP messages. Next, instances of the `LoggingInInterceptor` and `LoggingOutInterceptor` are specified which have as parent the previously defined `abstractLoggingInterceptor` instance.

In a last step, the interceptors are added to the CXF bus, which is the backbone of the CXF architecture that manages the respective inbound and outbound message and fault interceptor chains for all client and server endpoints. The interceptors are added to both in/out and respective fault interceptor chains as shown below.

> Note that interceptors can be added to a client, server or bus.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:cxf="http://cxf.apache.org/core"
  xmlns:jaxws="http://cxf.apache.org/jaxws"
  xsi:schemaLocation="
http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
http://cxf.apache.org/core http://cxf.apache.org/schemas/core.xsd
http://cxf.apache.org/jaxws http://cxf.apache.org/schemas/jaxws.xsd">

  <jaxws:client id="helloWorldRequesterBean"
    serviceClass="com.codenotfound.services.helloworld.HelloWorldPortType"
    address="${helloworld.address}">
  </jaxws:client>

  <!-- abstractLoggingInterceptor that will enable pretty printing of messages -->
  <bean id="abstractLoggingInterceptor" abstract="true">
    <property name="prettyLogging" value="true" />
  </bean>

  <!-- logging interceptors that will log all received/sent messages -->
  <bean id="loggingInInterceptor" class="org.apache.cxf.interceptor.LoggingInInterceptor"
    parent="abstractLoggingInterceptor">
  </bean>
  <bean id="loggingOutInterceptor" class="org.apache.cxf.interceptor.LoggingOutInterceptor"
    parent="abstractLoggingInterceptor">
  </bean>

  <!-- add the logging interceptors to the CXF bus -->
  <cxf:bus>
    <cxf:inInterceptors>
      <ref bean="loggingInInterceptor" />
    </cxf:inInterceptors>
    <cxf:inFaultInterceptors>
      <ref bean="loggingInInterceptor" />
    </cxf:inFaultInterceptors>
    <cxf:outInterceptors>
      <ref bean="loggingOutInterceptor" />
    </cxf:outInterceptors>
    <cxf:outFaultInterceptors>
      <ref bean="loggingOutInterceptor" />
    </cxf:outFaultInterceptors>
  </cxf:bus>

</beans>
```

# Provider

Activating the interceptors at provider(server) side will be done using the `LoggingFeature` that is supplied with the CXF framework.

First, an instance of the abstract `LoggingFeature` class is created with the `prettyLogging` set to true in order to enable pretty printing of the SOAP messages. As with the interceptors, the feature is added to the CXF bus in order to activate them as shown below.

> Note that features can be added to a client, server or a bus.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:cxf="http://cxf.apache.org/core"
  xmlns:jaxws="http://cxf.apache.org/jaxws"
  xsi:schemaLocation="
http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
http://cxf.apache.org/core http://cxf.apache.org/schemas/core.xsd
http://cxf.apache.org/jaxws http://cxf.apache.org/schemas/jaxws.xsd">

  <jaxws:endpoint id="helloWorldProviderBean"
    implementor="com.codenotfound.soap.http.cxf.HelloWorldImpl"
    address="/helloworld">
  </jaxws:endpoint>

  <!-- loggingFeature that will log all received/sent messages -->
  <bean id="loggingFeature" class="org.apache.cxf.feature.LoggingFeature">
    <property name="prettyLogging" value="true" />
  </bean>

  <!-- add the loggingFeature to the cxf bus -->
  <cxf:bus>
    <cxf:features>
      <ref bean="loggingFeature" />
    </cxf:features>
  </cxf:bus>

</beans>
```

# Testing

Testing of the service is done using a unit and an integration test case. For the unit test case, the provider is created without Spring (using `JaxWsServerFactoryBean`), as such the logging feature needs to be added programmatically as shown below.

``` java
package com.codenofound.soap.http.cxf;

import static org.junit.Assert.assertEquals;

import org.apache.cxf.feature.LoggingFeature;
import org.apache.cxf.jaxws.JaxWsServerFactoryBean;
import org.junit.BeforeClass;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import com.codenotfound.services.helloworld.Person;
import com.codenotfound.soap.http.cxf.HelloWorldClient;
import com.codenotfound.soap.http.cxf.HelloWorldImpl;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
    locations = {"classpath:/META-INF/spring/context-requester.xml"})
public class HelloWorldImplTest {

  private static String ENDPOINT_ADDRESS =
      "http://localhost:9090/cnf/services/helloworld";

  @Autowired
  private HelloWorldClient helloWorldClient;

  @BeforeClass
  public static void setUpBeforeClass() throws Exception {
    // start the HelloWorld service using jaxWsServerFactoryBean
    JaxWsServerFactoryBean jaxWsServerFactoryBean =
        new JaxWsServerFactoryBean();

    // adding loggingFeature to print the received/sent messages
    LoggingFeature loggingFeature = new LoggingFeature();
    loggingFeature.setPrettyLogging(true);

    jaxWsServerFactoryBean.getFeatures().add(loggingFeature);

    // setting the implementation
    HelloWorldImpl implementor = new HelloWorldImpl();
    jaxWsServerFactoryBean.setServiceBean(implementor);
    // setting the endpoint
    jaxWsServerFactoryBean.setAddress(ENDPOINT_ADDRESS);
    jaxWsServerFactoryBean.create();
  }

  @Test
  public void testSayHello() {
    Person person = new Person();
    person.setFirstName("John");
    person.setLastName("Doe");

    assertEquals("Hello John Doe!",
        helloWorldClient.sayHello(person));
  }
}
```

Run the example by opening a command prompt and executing following Maven command:

``` plaintext
mvn verify
```

The result should be that a number of log files are created in the project root directory. Amongst these files there should be <var>jaxws-jetty-cxf-logging-ws.log</var> and <var>jaxws-jetty-cxf-logging-ws-test.log</var> which contain the exchanged SOAP messages.

``` plaintext
06:26:41.718 INFO  [qtp1766911175-27][HelloWorld_PortType] Inbound Message
----------------------------
ID: 1
Address: http://localhost:9090/cnf/services/helloworld
Encoding: UTF-8
Http-Method: POST
Content-Type: text/xml; charset=UTF-8
Headers: {Accept=[*/*], Cache-Control=[no-cache], connection=[keep-alive], Content-Length=[229], content-type=[text/xml; charset=UTF-8], Host=[localhost:9090], Pragma=[no-cache], SOAPAction=[""], User-Agent=[Apache CXF 3.0.3]}
Payload: <soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
  <soap:Body>
    <person xmlns="http://codenotfound.com/services/helloworld">
      <firstName>John</firstName>
      <lastName>Doe</lastName>
    </person>
  </soap:Body>
</soap:Envelope>

--------------------------------------
06:26:41.786 INFO  [qtp1766911175-27][HelloWorld_PortType] Outbound Message
---------------------------
ID: 1
Response-Code: 200
Encoding: UTF-8
Content-Type: text/xml
Headers: {}
Payload: <soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
  <soap:Body>
    <greeting xmlns="http://codenotfound.com/services/helloworld">
      <text>Hello John Doe!</text>
    </greeting>
  </soap:Body>
</soap:Envelope>
```

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/cxf-jaxws/tree/master/cxf-jaxws-jetty-logging-log4j){:target="_blank"}.
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This concludes the CXF logging request and response messages using Log4j example. If you found this post helpful or have any questions or remarks, please leave a comment.
