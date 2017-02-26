---
title: JSF - PrimeFaces Hello World Example using WildFly and Maven
permalink: /2016/01/jsf-primefaces-hello-world-example-using-wildfly-and-maven.html
excerpt: A PrimeFaces 'Hello World' example using WildFly and Maven.
date: 2016-01-03 21:00
categories: [PrimeFaces]
tags: [Example, JavaServer Faces, JSF, Maven, PrimeFaces, WildFly]
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/primefaces-logo.png" alt="primefaces logo">
</figure>

[WildFly](http://wildfly.org/), formerly known as JBoss, is an application server written in Java developed by JBoss. It implements the Java Platform, Enterprise Edition (Java EE) specification. WildFly runs on multiple platforms and is free and open-source software.

The following post illustrates a basic example in which we will configure, build and run a Hello World PrimeFaces example using WildFly and Maven.

Tools used:
* JSF 2.2
* PrimeFaces 5.3
* WildFly 9
* Maven 3

The code is built and run using [Maven](https://maven.apache.org/). Specified below is the Maven POM file which contains the needed dependencies for JSF and PrimeFaces.

In order to run the Hello World PrimeFaces application, a servlet container is needed and in this example the WildFly implementation will be used. The deployment of the code and application server will be fully automated using the [wildfly-maven-plugin](https://docs.jboss.org/wildfly/plugins/maven/latest/) based on a [sample configuration posted on stackoverflow](http://stackoverflow.com/a/29127121/4201470). It takes as input two configuration parameters which point to the server home directory and configuration XML file.

The <var>'maven-dependency-plugin'</var> is used to unpack a distribution of WildFly in the projects build directory. The <var>'overWrite'</var> flag has been specified as false in order to avoid downloading the distribution on each run. The <var>'maven-resources-plugin'</var> is in turn used to overwrite the default server configuration so we can change the default HTTP listening port value as further explained below.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.codenotfound</groupId>
    <artifactId>jsf-primefaces-wildfly</artifactId>
    <version>1.0</version>
    <packaging>war</packaging>

    <name>JSF - PrimeFaces WildFly Example Using Maven</name>
    <url>http://www.codenotfound.com/2016/01/primeFaces-wildfly-example-using-maven.html</url>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <java.version>1.7</java.version>

        <servlet.version>3.1.0</servlet.version>
        <jsf.version>2.2.12</jsf.version>
        <primefaces.version>5.3</primefaces.version>

        <version.wildfly>9.0.2.Final</version.wildfly>
        <jboss.home>${project.build.directory}/wildfly-${version.wildfly}</jboss.home>
        <server.config>standalone.xml</server.config>

        <maven-compiler-plugin.version>3.3</maven-compiler-plugin.version>
        <maven-dependency-plugin>2.10</maven-dependency-plugin>
        <maven-resources-plugin.version>2.7</maven-resources-plugin.version>
        <wildfly-maven-plugin.version>1.1.0.Alpha5</wildfly-maven-plugin.version>
    </properties>

    <dependencies>
        <!-- Servlet -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>${servlet.version}</version>
            <scope>provided</scope>
        </dependency>
        <!-- JSF -->
        <dependency>
            <groupId>com.sun.faces</groupId>
            <artifactId>jsf-api</artifactId>
            <version>${jsf.version}</version>
            <scope>compile</scope>
        </dependency>
        <dependency>
            <groupId>com.sun.faces</groupId>
            <artifactId>jsf-impl</artifactId>
            <version>${jsf.version}</version>
            <scope>compile</scope>
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
            <!-- Unpack the distribution -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-dependency-plugin</artifactId>
                <version>${maven-resources-plugin.version}</version>
                <executions>
                    <execution>
                        <id>unpack-wildfly</id>
                        <phase>generate-sources</phase>
                        <goals>
                            <goal>unpack</goal>
                        </goals>
                        <configuration>
                            <artifactItems>
                                <artifactItem>
                                    <groupId>org.wildfly</groupId>
                                    <artifactId>wildfly-dist</artifactId>
                                    <version>${version.wildfly}</version>
                                    <type>zip</type>
                                    <overWrite>false</overWrite>
                                    <outputDirectory>${project.build.directory}</outputDirectory>
                                </artifactItem>
                            </artifactItems>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
            <!-- Copy server configuration -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-resources-plugin</artifactId>
                <version>${maven-resources-plugin.version}</version>
                <executions>
                    <execution>
                        <id>copy-standalone-xml</id>
                        <phase>generate-sources</phase>
                        <goals>
                            <goal>copy-resources</goal>
                        </goals>
                        <configuration>
                            <outputDirectory>${jboss.home}/standalone/configuration</outputDirectory>
                            <resources>
                                <resource>
                                    <directory>src/main/resources</directory>
                                </resource>
                            </resources>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
            <!-- Maven WildFly plugin -->
            <plugin>
                <groupId>org.wildfly.plugins</groupId>
                <artifactId>wildfly-maven-plugin</artifactId>
                <version>${wildfly-maven-plugin.version}</version>
                <configuration>
                    <jbossHome>${jboss.home}</jbossHome>
                    <serverConfig>${server.config}</serverConfig>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

In order to change the default HTTP listening port to value <var>'9090'</var> we need to modify the WildFly server configuration XML file which is located in <var>src/main/resources/standalone.xml</var>. The below extract shows the value that was change.

``` xml
    <socket-binding-group name="standard-sockets" default-interface="public" port-offset="${jboss.socket.binding.port-offset:0}">
        <socket-binding name="management-http" interface="management" port="${jboss.management.http.port:9990}"/>
        <socket-binding name="management-https" interface="management" port="${jboss.management.https.port:9993}"/>
        <socket-binding name="ajp" port="${jboss.ajp.port:8009}"/>
        <!-- Change the HTTP listening port to 9090 -->
        <socket-binding name="http" port="${jboss.http.port:9090}"/>
        <socket-binding name="https" port="${jboss.https.port:8443}"/>
        <socket-binding name="txn-recovery-environment" port="4712"/>
        <socket-binding name="txn-status-manager" port="4713"/>
        <outbound-socket-binding name="mail-smtp">
            <remote-destination host="localhost" port="25"/>
        </outbound-socket-binding>
    </socket-binding-group>
```

The context path of our deployed application will be changed to <var>'/codenotfound'</var>. In order to achieve this on WildFly we need to add a <var>jboss-web.xml</var> file in the <var>/WEB-INF</var> directory with a <var>'&lt;context-root&gt;'</var> parameter as explained in this [post](http://stackoverflow.com/a/28475123/4201470).

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<jboss-web xmlns="http://www.jboss.com/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
      http://www.jboss.com/xml/ns/javaee
      http://www.jboss.org/j2ee/schema/jboss-web_5_1.xsd">
    <context-root>/codenotfound</context-root>
</jboss-web>
```

The remaining code of the example is identical to the [PrimeFaces Hello World example detailed in this post]({{ site.url }}/2014/04/jsf-primefaces-hello-world-example.html). Feel free to check it out if you would like to know more details.

In order to run the above example open a command prompt and execute following Maven command: 

``` plaintext
mvn wildfly:run
```

Maven will download the needed dependencies, unpack the WildFly distribution, copy the server configuration, compile the code, and start a WildFly instance on which the web application will be deployed. The result should be the following WildFly startup trace which mentions: 'Deploying application <var>'jsf-primefaces-wildfly-1.0.war'</var> without reporting any errors.


















