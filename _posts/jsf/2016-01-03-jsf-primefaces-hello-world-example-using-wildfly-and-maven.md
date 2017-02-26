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

``` plaintext
[INFO] --- wildfly-maven-plugin:1.1.0.Alpha5:run (default-cli) @ jsf-primefaces-wildfly ---
[INFO] JAVA_HOME=C:\Program Files\Java\jdk1.8.0_65\jre
[INFO] JBOSS_HOME=C:\codenotfound\code\jsf-primefaces\jsf-primefaces-wildfly\target\wildfly-9.0.2.Final

[INFO] Server is starting up. Press CTRL + C to stop the server.
jan 03, 2016 7:45:51 AM org.xnio.Xnio 
INFO: XNIO version 3.3.1.Final
jan 03, 2016 7:45:51 AM org.xnio.nio.NioXnio 
INFO: XNIO NIO Implementation Version 3.3.1.Final
jan 03, 2016 7:45:51 AM org.jboss.remoting3.EndpointImpl 
INFO: JBoss Remoting version 4.0.9.Final
07:45:51,929 INFO  [org.jboss.modules] (main) JBoss Modules version 1.4.3.Final
07:45:52,365 INFO  [org.jboss.msc] (main) JBoss MSC version 1.2.6.Final
07:45:52,443 INFO  [org.jboss.as] (MSC service thread 1-6) WFLYSRV0049: WildFly Full 9.0.2.Final (WildFly Core 1.0.2.Final) starting
07:45:54,320 INFO  [org.jboss.as.controller.management-deprecated] (ServerService Thread Pool -- 3) WFLYCTL0028: Attribute 'enabled' in the resource at address '/subsystem=datasources/data-source=ExampleDS' is deprecated, and may be removed in future version. See the attribute description in the output of the read-resource-description operation to learn more about the deprecation.
07:45:54,320 INFO  [org.jboss.as.controller.management-deprecated] (ServerService Thread Pool -- 23) WFLYCTL0028: Attribute 'job-repository-type' in the resource at address '/subsystem=batch' is deprecated, and may be removed in future version. See the attribute description in the output of the read-resource-description operation to learn more about the deprecation.
07:45:54,367 INFO  [org.jboss.as.server] (Controller Boot Thread) WFLYSRV0039: Creating http management service using socket-binding (management-http)
07:45:54,383 INFO  [org.xnio] (MSC service thread 1-5) XNIO version 3.3.1.Final
07:45:54,383 INFO  [org.xnio.nio] (MSC service thread 1-5) XNIO NIO Implementation Version 3.3.1.Final
07:45:54,414 WARN  [org.jboss.as.txn] (ServerService Thread Pool -- 54) WFLYTX0013: Node identifier property is set to the default value. Please make sure it is unique.
07:45:54,414 INFO  [org.jboss.as.security] (ServerService Thread Pool -- 53) WFLYSEC0002: Activating Security Subsystem
07:45:54,414 INFO  [org.jboss.as.jsf] (ServerService Thread Pool -- 44) WFLYJSF0007: Activated the following JSF Implementations: [main]
07:45:54,461 INFO  [org.jboss.as.clustering.infinispan] (ServerService Thread Pool -- 38) WFLYCLINF0001: Activating Infinispan subsystem.
07:45:54,461 INFO  [org.jboss.as.security] (MSC service thread 1-6) WFLYSEC0001: Current PicketBox version=4.9.2.Final
07:45:54,461 INFO  [org.jboss.as.naming] (ServerService Thread Pool -- 46) WFLYNAM0001: Activating Naming Subsystem
07:45:54,476 INFO  [org.jboss.as.webservices] (ServerService Thread Pool -- 56) WFLYWS0002: Activating WebServices Extension
07:45:54,508 INFO  [org.jboss.as.connector] (MSC service thread 1-7) WFLYJCA0009: Starting JCA Subsystem (IronJacamar 1.2.5.Final)
07:45:54,508 INFO  [org.wildfly.extension.io] (ServerService Thread Pool -- 37) WFLYIO001: Worker 'default' has auto-configured to 8 core threads with 64 task threads based on your 4 available processors
07:45:54,554 INFO  [org.wildfly.extension.undertow] (MSC service thread 1-7) WFLYUT0003: Undertow 1.2.9.Final starting
07:45:54,554 INFO  [org.wildfly.extension.undertow] (ServerService Thread Pool -- 55) WFLYUT0003: Undertow 1.2.9.Final starting
07:45:54,554 INFO  [org.jboss.as.mail.extension] (MSC service thread 1-6) WFLYMAIL0001: Bound mail session [java:jboss/mail/Default]
07:45:54,554 INFO  [org.jboss.as.naming] (MSC service thread 1-2) WFLYNAM0003: Starting Naming Service
07:45:54,570 INFO  [org.jboss.as.connector.subsystems.datasources] (ServerService Thread Pool -- 33) WFLYJCA0004: Deploying JDBC-compliant driver class org.h2.Driver (version 1.3)
07:45:54,570 INFO  [org.jboss.as.connector.deployers.jdbc] (MSC service thread 1-7) WFLYJCA0018: Started Driver service with driver-name = h2
07:45:54,809 INFO  [org.wildfly.extension.undertow] (ServerService Thread Pool -- 55) WFLYUT0014: Creating file handler for path C:\codenotfound\code\jsf-primefaces\jsf-primefaces-wildfly\target\wildfly-9.0.2.Final/welcome-content
07:45:54,809 INFO  [org.jboss.remoting] (MSC service thread 1-3) JBoss Remoting version 4.0.9.Final
07:45:54,840 INFO  [org.wildfly.extension.undertow] (MSC service thread 1-6) WFLYUT0012: Started server default-server.
07:45:54,856 INFO  [org.wildfly.extension.undertow] (MSC service thread 1-6) WFLYUT0018: Host default-host starting
07:45:54,996 INFO  [org.wildfly.extension.undertow] (MSC service thread 1-8) WFLYUT0006: Undertow HTTP listener default listening on /127.0.0.1:9090
07:45:55,152 INFO  [org.jboss.as.connector.subsystems.datasources] (MSC service thread 1-6) WFLYJCA0001: Bound data source [java:jboss/datasources/ExampleDS]
07:45:55,183 INFO  [org.jboss.as.server.deployment.scanner] (MSC service thread 1-8) WFLYDS0013: Started FileSystemDeploymentService for directory C:\codenotfound\code\jsf-primefaces\jsf-primefaces-wildfly\target\wildfly-9.0.2.Final\standalone\deployments
07:45:55,199 INFO  [org.jboss.as.server.deployment] (MSC service thread 1-5) WFLYSRV0027: Starting deployment of "jsf-primefaces-wildfly-1.0.war" (runtime-name: "jsf-primefaces-wildfly-1.0.war")
07:45:55,464 INFO  [org.jboss.ws.common.management] (MSC service thread 1-6) JBWS022052: Starting JBoss Web Services - Stack CXF Server 5.0.0.Final
07:45:56,104 INFO  [org.jboss.weld.deployer] (MSC service thread 1-8) WFLYWELD0003: Processing weld deployment jsf-primefaces-wildfly-1.0.war
07:45:56,229 INFO  [org.hibernate.validator.internal.util.Version] (MSC service thread 1-8) HV000001: Hibernate Validator 5.1.3.Final
07:45:56,733 INFO  [org.jboss.weld.deployer] (MSC service thread 1-3) WFLYWELD0006: Starting Services for CDI deployment: jsf-primefaces-wildfly-1.0.war
07:45:56,764 INFO  [org.jboss.weld.Version] (MSC service thread 1-3) WELD-000900: 2.2.16 (SP1)
07:45:56,811 INFO  [org.jboss.weld.deployer] (MSC service thread 1-2) WFLYWELD0009: Starting weld service for deployment jsf-primefaces-wildfly-1.0.war
07:45:57,606 INFO  [javax.enterprise.resource.webcontainer.jsf.config] (ServerService Thread Pool -- 69) Initializing Mojarra 2.2.12-jbossorg-2 20150729-1131 for context '/codenotfound'
07:45:58,542 INFO  [org.primefaces.webapp.PostConstructApplicationEventListener] (ServerService Thread Pool -- 69) Running on PrimeFaces 5.3
07:45:58,558 INFO  [org.wildfly.extension.undertow] (ServerService Thread Pool -- 69) WFLYUT0021: Registered web context: /codenotfound
07:45:58,589 INFO  [org.jboss.as.server] (Controller Boot Thread) WFLYSRV0010: Deployed "jsf-primefaces-wildfly-1.0.war" (runtime-name : "jsf-primefaces-wildfly-1.0.war")
07:45:58,688 INFO  [org.jboss.as] (Controller Boot Thread) WFLYSRV0060: Http management interface listening on http://127.0.0.1:9990/management
07:45:58,688 INFO  [org.jboss.as] (Controller Boot Thread) WFLYSRV0051: Admin console listening on http://127.0.0.1:9990
07:45:58,688 INFO  [org.jboss.as] (Controller Boot Thread) WFLYSRV0025: WildFly Full 9.0.2.Final (WildFly Core 1.0.2.Final) started in 7165ms - Started 555 of 735 services (221 services are lazy, passive or on-demand)
[INFO] Deploying application 'jsf-primefaces-wildfly-1.0.war'

07:45:58,937 INFO  [org.jboss.as.repository] (management-handler-thread - 3) WFLYDR0001: Content added at location C:\codenotfound\code\jsf-primefaces\jsf-primefaces-wildfly\target\wildfly-9.0.2.Final\standalone\data\content\62\18a7ce69de2ad5f6835a1c0edc310c19f9b4a8\content
07:45:58,953 INFO  [org.wildfly.extension.undertow] (ServerService Thread Pool -- 9) WFLYUT0022: Unregistered web context: /codenotfound
07:45:58,953 INFO  [org.jboss.weld.deployer] (MSC service thread 1-1) WFLYWELD0010: Stopping weld service for deployment jsf-primefaces-wildfly-1.0.war
07:45:59,078 INFO  [org.jboss.as.server.deployment] (MSC service thread 1-7) WFLYSRV0028: Stopped deployment jsf-primefaces-wildfly-1.0.war (runtime-name: jsf-primefaces-wildfly-1.0.war) in 140ms
07:45:59,078 INFO  [org.jboss.as.server.deployment] (MSC service thread 1-5) WFLYSRV0027: Starting deployment of "jsf-primefaces-wildfly-1.0.war" (runtime-name: "jsf-primefaces-wildfly-1.0.war")
07:45:59,452 INFO  [org.jboss.weld.deployer] (MSC service thread 1-8) WFLYWELD0003: Processing weld deployment jsf-primefaces-wildfly-1.0.war
07:45:59,593 INFO  [org.jboss.weld.deployer] (MSC service thread 1-1) WFLYWELD0006: Starting Services for CDI deployment: jsf-primefaces-wildfly-1.0.war
07:45:59,593 INFO  [org.jboss.weld.deployer] (MSC service thread 1-1) WFLYWELD0009: Starting weld service for deployment jsf-primefaces-wildfly-1.0.war
07:45:59,749 INFO  [javax.enterprise.resource.webcontainer.jsf.config] (ServerService Thread Pool -- 6) Initializing Mojarra 2.2.12-jbossorg-2 20150729-1131 for context '/codenotfound'
07:46:00,248 INFO  [org.primefaces.webapp.PostConstructApplicationEventListener] (ServerService Thread Pool -- 6) Running on PrimeFaces 5.3
07:46:00,248 INFO  [org.wildfly.extension.undertow] (ServerService Thread Pool -- 6) WFLYUT0021: Registered web context: /codenotfound
07:46:00,295 INFO  [org.jboss.as.server] (management-handler-thread - 3) WFLYSRV0016: Replaced deployment "jsf-primefaces-wildfly-1.0.war" with deployment "jsf-primefaces-wildfly-1.0.war"
07:46:00,295 INFO  [org.jboss.as.repository] (management-handler-thread - 3) WFLYDR0002: Content removed from location C:\codenotfound\code\jsf-primefaces\jsf-primefaces-wildfly\target\wildfly-9.0.2.Final\standalone\data\content\b6\2fa4660581d5b4d8f5703b276b7c2d7a282c58\content
```

Open a web browser and enter following URL: [http://localhost:9090/codenotfound/](http://localhost:9090/codenotfound/). The below page should be displayed:

<figure>
    <img src="{{ site.url }}/assets/images/jsf/wildfly-hello-world-example.png" alt="wildfly hello world example">
</figure>

Enter a first and last name and press the <var>Submit</var> button. A pop-up dialog will be shown with a greeting message. 

<figure>
    <img src="{{ site.url }}/assets/images/jsf/wildfly-hello-world-example-greeting.png" alt="wildfly hello world example greeting">
</figure>

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/jsf-primefaces/tree/master/jsf-primefaces-wildfly).
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This concludes the PrimeFaces WildFly example. If you found this post helpful or have any questions or remarks, please leave a comment.