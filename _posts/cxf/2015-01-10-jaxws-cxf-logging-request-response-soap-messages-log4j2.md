---
title: JAX-WS - CXF logging request and response SOAP messages using Log4j2
permalink: /2015/01/jaxws-cxf-logging-request-response-soap-messages-log4j2.html
excerpt: A code sample which shows how to configure CXF to log the request and response SOAP messages using Log4j2.
date: 2015-01-10 21:00
categories: [Apache CXF]
tags: [Apache CXF, Code Sample, CXF, JAX-WS, Jetty, log4j2, logging, Maven, SOAP message]
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/apache-cxf-logo.png" alt="apache cxf logo">
</figure>

CXF uses Java SE Logging for both client- and server-side logging of SOAP requests and responses. Logging is activated by use of separate in/out interceptors that can be attached to the requester and/or provider as required. These interceptors can be specified either programmatically (via Java code and/or annotations) or via use of configuration files. The following code sample shows how to configure CXF interceptors using Log4j2 for the Hello World web service from a previous [post]({{ site.url }}/2014/08/jaxws-cxf-contract-first-hello-world-jetty-maven.html).

Tools used:
* CXF 3.0
* Spring 4.1
* Log4j2 2.1
* Jetty 8
* Maven 3

Specifying the interceptors via configuration files offers two benefits over programmatic configuration:
1. Logging requirements **can be altered without needing to recompile the code**.
2. **No Apache CXF-specific APIs need to be added to your code**, which helps it remain interoperable with other JAX-WS compliant web service stacks.

For this example [Log4j2](https://logging.apache.org/log4j/2.x/) will be used which is an upgrade of the [Log4j](https://logging.apache.org/log4j/1.2/) project. As a best practice the CXF `java.util.logging` calls will first be redirected to [SLF4J](http://www.slf4j.org/) (Simple Logging Facade for Java) as described [here](https://cxf.apache.org/docs/debugging-and-logging.html#DebuggingandLogging-UsingSLF4JInsteadofjava.util.logging%28since2.2.8%29). In other words a <var>META-INF/cxf/org.apache.cxf.Logger</var> file will be created on the classpath containing the following:

``` plaintext
org.apache.cxf.common.logging.Slf4jLogger
```

As the Hello World example uses Spring, the commons-logging calls from the Spring framework will also be redirected to SLF4J using [jcl-over-slf4j](http://www.slf4j.org/legacy.html). Now that all logging calls of both CXF and Spring are redirected to SLF4J, Log4j2 will be plugged into SLF4J using the [Log4j 2 SLF4J Binding](https://logging.apache.org/log4j/2.0/log4j-slf4j-impl/index.html). The picture below illustrates the described approach. 

<figure>
    <img src="{{ site.url }}/assets/images/cxf/cxf-logging-using-log4j2.png" alt="cxf logging using log4j2">
</figure>

The below Maven POM file contains the needed dependencies for the SLF4 bridge (<var>'jcl-over-slf4j'</var>), Log4j 2 SLF4J Binding (<var>'log4j-slf4j-impl'</var>) and Log4j2 (<var>'log4j-core'</var>). In addition it contains all other needed dependencies and plugins needed in order to run the example.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.codenotfound</groupId>
    <artifactId>jaxws-jetty-cxf-logging-log4j2</artifactId>
    <version>1.0</version>
    <packaging>war</packaging>

    <name>JAX-WS - CXF logging request and response SOAP messages using Log4j2</name>
    <url>https://codenotfound.com/2015/01/jaxws-cxf-logging-request-response-soap-messages-log4j2.html</url>

    <prerequisites>
        <maven>3.0</maven>
    </prerequisites>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <java.version>1.8</java.version>

        <slf4j.version>1.7.10</slf4j.version>
        <log4j2.version>2.1</log4j2.version>
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
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-slf4j-impl</artifactId>
            <version>${log4j2.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
            <version>${log4j2.version}</version>
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

# Log4j2 configuration

Log4j2 can be configured in 1 of 4 ways:
1. Through a configuration file written in XML, JSON, or YAML.
2. Programmatically, by creating a `ConfigurationFactory` and `Configuration` implementation.
3. Programmatically, by calling the APIs exposed in the `Configuration` interface to add components to the default configuration.
4. Programmatically, by calling methods on the internal `Logger` class.

For this example a configuration script in XML is used as shown below. The script will write all logging events to a file except for the request and response SOAP messages that will be written to a different file.

After defining some properties that contain the layout pattern and file names, two appenders are configured to write logging events to a rolling file. The first file <var>jaxws-jetty-cxf-logging.log</var> will contain all log events except for the ones emitted by the CXF logging interceptors as these are going to be written to <var>jaxws-jetty-cxf-logging-ws.log</var>.

The last section contains the different loggers and the level at which information is logged. The log level of the <var>'org.apache.cxf.services'</var> logger needs to be set to INFO in order to activate the SOAP logging events. In addition the <var>'WS_LOG_FILE'</var> appender needs to be referenced in order to write the SOAP logging events to a different file. Note the <var>'additivity="false"'</var> which makes sure the log events are not written to appenders attached to its ancestors (in this case <var>'APP_LOG_FILE'</var>).

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration monitorInterval="60">

    <!-- PROPERTIES -->
    <Properties>
        <Property name="layout">%d{HH:mm:ss.SSS} %-5level
            [%thread][%logger{0}] %m%n</Property>
        <Property name="logFile">jaxws-jetty-cxf-logging</Property>
        <Property name="logFile-ws">jaxws-jetty-cxf-logging-ws</Property>
    </Properties>

    <!-- APPENDERS -->
    <Appenders>
        <RollingFile name="APP_LOG_FILE" fileName="${logFile}.log"
            filePattern="${logFile}.%d{yyyy-MM-dd}.log">
            <TimeBasedTriggeringPolicy
                interval="1" modulate="true" />
            <DefaultRolloverStrategy max="30" />
            <PatternLayout pattern="${layout}" />
        </RollingFile>

        <RollingFile name="WS_LOG_FILE" fileName="${logFile-ws}.log"
            filePattern="${logFile-ws}.%d{yyyy-MM-dd}.log">
            <TimeBasedTriggeringPolicy
                interval="1" modulate="true" />
            <DefaultRolloverStrategy max="30" />
            <PatternLayout pattern="${layout}" />
        </RollingFile>
    </Appenders>

    <!-- LOGGERS -->
    <Loggers>
        <Logger name="com.codenotfound.soap.http.cxf" level="INFO" />

        <Logger name="org.springframework" level="WARN" />
        <Logger name="org.apache.cxf" level="WARN" />

        <!-- level INFO needed to log SOAP messages -->
        <Logger name="org.apache.cxf.services" level="INFO"
            additivity="false">
            <!-- specify a dedicated appender for the SOAP messages -->
            <AppenderRef ref="WS_LOG_FILE" />
        </Logger>

        <Root level="WARN">
            <AppenderRef ref="APP_LOG_FILE" />
        </Root>
    </Loggers>

</Configuration>
```

# Requester

In order to activate the logging interceptors provided by the CXF framework, there are two options. For the requester(client) the option where all logging interceptors are configured manually will be illustrated. The other option, where the logging feature is used to configure all interceptors, will be shown in the provider section down below. For more information on the difference between interceptors and features check out this post.

First an instance of the abstract AbstractLoggingInterceptor class is created to enable pretty printing of the SOAP messages. Next, instances of the LoggingInInterceptor and LoggingOutInterceptor are specified which have as parent the previously defined abstractLoggingInterceptor instance. In a last step the interceptors are added to the CXF bus, which is the backbone of the CXF architecture that manages the respective inbound and outbound message and fault interceptor chains for all client and server endpoints. The interceptors are added to both in/out and respective fault interceptor chains as shown below.

> Note that interceptors can be added to a client, server or bus.
















