---
title: "JSF - PrimeFaces Hello World Example using WildFly and Maven"
permalink: /jsf-primefaces-hello-world-example-wildfly-maven.html
excerpt: "A detailed step-by-step tutorial in which we build and run a Hello World PrimeFaces example using WildFly and Maven"
date: 2016-01-03
last_modified_at: 2016-01-03
header:
  teaser: "assets/images/teaser/primefaces-teaser.png"
categories: [PrimeFaces]
tags: [Example, Hello World, JavaServer Faces, JSF, Maven, PrimeFaces, WildFly, Tutorial]
redirect_from:
  - /2016/01/jsf-primefaces-wildfly-example-using-maven.html
  - /2016/01/jsf-primefaces-hello-world-example-using-wildfly-and-maven.html
published: true
---

<figure>
    <img src="{{ site.url }}/assets/images/logo/primefaces-logo.png" alt="primefaces logo" class="logo">
</figure>

[PrimeFaces](http://primefaces.org/){:target="_blank"} is an open source component library for JavaServer Faces (JSF). It provides a collection of mostly visual components (widgets) that can be used by JSF programmers to build the UI for a web application. An overview of these widgets can be found at the [PrimeFaces showcase](http://www.primefaces.org/showcase/){:target="_blank"}.

[WildFly](http://wildfly.org/){:target="_blank"}, formerly known as [JBoss AS](http://www.jboss.org/){:target="_blank"}, is an application server written in Java developed by JBoss. It implements the Java Platform, Enterprise Edition (Java EE) specification. WildFly runs on multiple platforms and is free and open-source software.

The following post illustrates a basic example in which we will configure, build and run a Hello World PrimeFaces example using WildFly and Maven.

Tools used:
* JSF 2.2
* PrimeFaces 5.3
* WildFly 9
* Maven 3.5

The code is built and run using [Maven](https://maven.apache.org/){:target="_blank"}. Specified below is the Maven POM file which contains the needed dependencies for JSF and PrimeFaces.

In order to run the Hello World PrimeFaces application, a servlet container is needed and in this example, the WildFly implementation will be used. The deployment of the code and application server will be fully automated using the [wildfly-maven-plugin](https://docs.jboss.org/wildfly/plugins/maven/latest/){:target="_blank"}. The plugin has been configured so that the HTTP listener port is set to "<kbd>9090</kbd>".

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>jsf-primefaces-wildfly</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>war</packaging>

  <name>JSF - PrimeFaces Hello World Example using WildFly and Maven</name>
  <url>https://www.codenotfound.com/jsf-primefaces-hello-world-example-wildfly-maven.html</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <java.version>1.8</java.version>

    <servlet.version>3.1.0</servlet.version>
    <jsf.version>2.2.12</jsf.version>
    <primefaces.version>6.1</primefaces.version>

    <maven-compiler-plugin.version>3.7.0</maven-compiler-plugin.version>
    <wildfly-maven-plugin.version>1.2.1.Final</wildfly-maven-plugin.version>
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
      <scope>provided</scope>
    </dependency>
    <dependency>
      <groupId>com.sun.faces</groupId>
      <artifactId>jsf-impl</artifactId>
      <version>${jsf.version}</version>
      <scope>provided</scope>
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
      <!-- wildfly-maven-plugin -->
      <plugin>
        <groupId>org.wildfly.plugins</groupId>
        <artifactId>wildfly-maven-plugin</artifactId>
        <version>${wildfly-maven-plugin.version}</version>
        <configuration>
          <server-args>
            <server-arg>-Djboss.http.port=9090</server-arg>
          </server-args>
        </configuration>
      </plugin>
    </plugins>
  </build>
</project>
```

The context path of our deployed application will be changed to <var>'/codenotfound'</var>. In order to achieve this on WildFly we need to add a <var>jboss-web.xml</var> file in the <var>/WEB-INF</var> directory with a <var>'&lt;context-root&gt;'</var> parameter as explained in this [Stack Overflow post](http://stackoverflow.com/a/28475123/4201470){:target="_blank"}.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<jboss-web version="11.0" xmlns="http://www.jboss.com/xml/ns/javaee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.jboss.com/xml/ns/javaee http://www.jboss.org/j2ee/schema/jboss-web_11_0.xsd">
  <context-root>/codenotfound</context-root>
</jboss-web>
```

The remaining code of the example is identical to a previous [PrimeFaces Hello World example running on Jetty]({{ site.url }}/jsf-primefaces-hello-world-example-jetty-maven.html). Feel free to check it out if you would like to know more details.

In order to run the above example open a command prompt and execute following Maven command: 

``` plaintext
mvn wildfly:run
```

Maven will download the needed dependencies, compile the code, and start a WildFly instance on which the web application will be deployed. The result should be the following WildFly startup trace which mentions: <var>Deployed "jsf-primefaces-wildfly-0.0.1-SNAPSHOT.war</var> without reporting any errors.

> Note that the WildFly distribution is 159 MB to download.

``` plaintext
[INFO] JAVA_HOME=C:\Program Files\Java\jdk1.8.0_152\jre
[INFO] JBOSS_HOME=c:\code\jsf-primefaces\jsf-primefaces-wildfly\target\wildfly-run\wildfly-11.0.0.Final

[INFO] Server is starting up. Press CTRL + C to stop the server.
dec 30, 2017 6:31:27 AM org.jboss.remoting3.EndpointImpl <clinit>
INFO: JBoss Remoting version 5.0.0.Final
dec 30, 2017 6:31:27 AM org.xnio.Xnio <clinit>
INFO: XNIO version 3.5.1.Final
dec 30, 2017 6:31:27 AM org.xnio.nio.NioXnio <clinit>
INFO: XNIO NIO Implementation Version 3.5.1.Final
dec 30, 2017 6:31:27 AM org.wildfly.security.Version <clinit>
INFO: ELY00001: WildFly Elytron version 1.1.0.Final
06:31:27,549 INFO  [org.jboss.modules] (main) JBoss Modules version 1.6.1.Final
06:31:28,103 INFO  [org.jboss.msc] (main) JBoss MSC version 1.2.7.SP1
06:31:28,233 INFO  [org.jboss.as] (MSC service thread 1-7) WFLYSRV0049: WildFly Full 11.0.0.Final (WildFly Core 3.0.8.Final) starting
06:31:29,790 INFO  [org.jboss.as.controller.management-deprecated] (Controller Boot Thread) WFLYCTL0028: Attribute 'security-realm' in the resource at address '/core-service=management/management-interface=http-interface' is deprecated, and may be removed in future version. See the attribute description in the output of the read-resource-description operation to learn more about the deprecation.
06:31:29,821 INFO  [org.wildfly.security] (ServerService Thread Pool -- 19) ELY00001: WildFly Elytron version 1.1.6.Final
06:31:29,818 INFO  [org.jboss.as.controller.management-deprecated] (ServerService Thread Pool -- 18) WFLYCTL0028: Attribute 'security-realm' in the resource at address '/subsystem=undertow/server=default-server/https-listener=https' is deprecated, and may be removed in future version. See the attribute description in the output of the read-resource-description operation to learn more about the deprecation.
06:31:29,887 INFO  [org.jboss.as.server] (Controller Boot Thread) WFLYSRV0039: Creating http management service using socket-binding (management-http)
06:31:29,899 INFO  [org.xnio] (MSC service thread 1-7) XNIO version 3.5.4.Final
06:31:29,905 INFO  [org.xnio.nio] (MSC service thread 1-7) XNIO NIO Implementation Version 3.5.4.Final
06:31:29,930 INFO  [org.jboss.as.jaxrs] (ServerService Thread Pool -- 43) WFLYRS0016: RESTEasy version 3.0.24.Final
06:31:29,940 WARN  [org.jboss.as.txn] (ServerService Thread Pool -- 58) WFLYTX0013: The node-identifier attribute on the /subsystem=transactions is set to the default value. This is a danger for environments running multiple servers. Please make sure the attribute value is unique.
06:31:29,941 INFO  [org.jboss.as.security] (ServerService Thread Pool -- 57) WFLYSEC0002: Activating Security Subsystem
06:31:29,946 INFO  [org.wildfly.extension.io] (ServerService Thread Pool -- 41) WFLYIO001: Worker 'default' has auto-configured to 8 core threads with 64 task threads based on your 4 available processors
06:31:29,949 INFO  [org.jboss.as.security] (MSC service thread 1-4) WFLYSEC0001: Current PicketBox version=5.0.2.Final
06:31:29,958 INFO  [org.jboss.as.webservices] (ServerService Thread Pool -- 60) WFLYWS0002: Activating WebServices Extension
06:31:29,965 INFO  [org.jboss.as.naming] (ServerService Thread Pool -- 50) WFLYNAM0001: Activating Naming Subsystem
06:31:29,968 INFO  [org.jboss.as.clustering.infinispan] (ServerService Thread Pool -- 42) WFLYCLINF0001: Activating Infinispan subsystem.
06:31:29,970 INFO  [org.jboss.as.connector] (MSC service thread 1-2) WFLYJCA0009: Starting JCA Subsystem (WildFly/IronJacamar 1.4.6.Final)
06:31:29,979 INFO  [org.jboss.as.jsf] (ServerService Thread Pool -- 48) WFLYJSF0007: Activated the following JSF Implementations: [main]
06:31:30,009 INFO  [org.jboss.as.naming] (MSC service thread 1-2) WFLYNAM0003: Starting Naming Service
06:31:30,011 INFO  [org.jboss.as.mail.extension] (MSC service thread 1-2) WFLYMAIL0001: Bound mail session [java:jboss/mail/Default]
06:31:30,071 INFO  [org.jboss.as.connector.subsystems.datasources] (ServerService Thread Pool -- 36) WFLYJCA0004: Deploying JDBC-compliant driver class org.h2.Driver (version 1.4)
06:31:30,106 INFO  [org.jboss.as.connector.deployers.jdbc] (MSC service thread 1-2) WFLYJCA0018: Started Driver service with driver-name = h2
06:31:30,173 INFO  [org.jboss.remoting] (MSC service thread 1-7) JBoss Remoting version 5.0.5.Final
06:31:30,196 INFO  [org.wildfly.extension.undertow] (MSC service thread 1-2) WFLYUT0003: Undertow 1.4.18.Final starting
06:31:30,300 INFO  [org.jboss.as.ejb3] (MSC service thread 1-2) WFLYEJB0482: Strict pool mdb-strict-max-pool is using a max instance size of 16 (per class), which is derived from the number of CPUs on this host.
06:31:30,301 INFO  [org.jboss.as.ejb3] (MSC service thread 1-3) WFLYEJB0481: Strict pool slsb-strict-max-pool is using a max instance size of 64 (per class), which is derived from thread worker pool sizing.
06:31:30,386 INFO  [org.wildfly.extension.undertow] (ServerService Thread Pool -- 59) WFLYUT0014: Creating file handler for path 'c:\code\jsf-primefaces\jsf-primefaces-wildfly\target\wildfly-run\wildfly-11.0.0.Final/welcome-content' with options [directory-listing: 'false', follow-symlink: 'false', case-sensitive: 'true', safe-symlink-paths: '[]']
06:31:30,407 INFO  [org.wildfly.extension.undertow] (MSC service thread 1-6) WFLYUT0012: Started server default-server.
06:31:30,409 INFO  [org.wildfly.extension.undertow] (MSC service thread 1-6) WFLYUT0018: Host default-host starting
06:31:30,417 INFO  [org.jboss.as.ejb3] (MSC service thread 1-7) WFLYEJB0493: EJB subsystem suspension complete
06:31:30,497 INFO  [org.wildfly.extension.undertow] (MSC service thread 1-4) WFLYUT0006: Undertow HTTP listener default listening on 127.0.0.1:9090
06:31:30,595 INFO  [org.jboss.as.connector.subsystems.datasources] (MSC service thread 1-1) WFLYJCA0001: Bound data source [java:jboss/datasources/ExampleDS]
06:31:30,629 INFO  [org.jboss.as.patching] (MSC service thread 1-5) WFLYPAT0050: WildFly Full cumulative patch ID is: base, one-off patches include: none
06:31:30,644 WARN  [org.jboss.as.domain.management.security] (MSC service thread 1-5) WFLYDM0111: Keystore c:\code\jsf-primefaces\jsf-primefaces-wildfly\target\wildfly-run\wildfly-11.0.0.Final\standalone\configuration\application.keystore not found, it will be auto generated on first use with a self signed certificate for host localhost
06:31:30,650 INFO  [org.jboss.as.server.deployment.scanner] (MSC service thread 1-1) WFLYDS0013: Started FileSystemDeploymentService for directory c:\code\jsf-primefaces\jsf-primefaces-wildfly\target\wildfly-run\wildfly-11.0.0.Final\standalone\deployments
06:31:30,964 INFO  [org.wildfly.extension.undertow] (MSC service thread 1-6) WFLYUT0006: Undertow HTTPS listener https listening on 127.0.0.1:8443
06:31:31,132 INFO  [org.jboss.ws.common.management] (MSC service thread 1-6) JBWS022052: Starting JBossWS 5.1.9.Final (Apache CXF 3.1.12)
06:31:31,308 INFO  [org.jboss.as.server] (Controller Boot Thread) WFLYSRV0212: Resuming server
06:31:31,316 INFO  [org.jboss.as] (Controller Boot Thread) WFLYSRV0060: Http management interface listening on http://127.0.0.1:9990/management
06:31:31,317 INFO  [org.jboss.as] (Controller Boot Thread) WFLYSRV0051: Admin console listening on http://127.0.0.1:9990
06:31:31,317 INFO  [org.jboss.as] (Controller Boot Thread) WFLYSRV0025: WildFly Full 11.0.0.Final (WildFly Core 3.0.8.Final) started in 4284ms - Started 292 of 553 services (347 services are lazy, passive or on-demand)
06:31:31,589 INFO  [org.jboss.as.repository] (management-handler-thread - 1) WFLYDR0001: Content added at location c:\code\jsf-primefaces\jsf-primefaces-wildfly\target\wildfly-run\wildfly-11.0.0.Final\standalone\data\content\15\5d6df8fb371d9a0239a10abdbc0d307ce809b9\content
06:31:31,605 INFO  [org.jboss.as.server.deployment] (MSC service thread 1-2) WFLYSRV0027: Starting deployment of "jsf-primefaces-wildfly-0.0.1-SNAPSHOT.war" (runtime-name: "jsf-primefaces-wildfly-0.0.1-SNAPSHOT.war")
06:31:32,523 INFO  [org.infinispan.factories.GlobalComponentRegistry] (MSC service thread 1-3) ISPN000128: Infinispan version: Infinispan 'Chakra' 8.2.8.Final
06:31:32,667 WARN  [org.jboss.weld.deployer] (MSC service thread 1-2) WFLYWELD0013: Deployment jsf-primefaces-wildfly-0.0.1-SNAPSHOT.war contains CDI annotations but no bean archive was found (no beans.xml or class with bean defining annotations was present).
06:31:32,832 INFO  [org.jboss.as.clustering.infinispan] (ServerService Thread Pool -- 19) WFLYCLINF0002: Started client-mappings cache from ejb container
06:31:32,971 INFO  [javax.enterprise.resource.webcontainer.jsf.config] (ServerService Thread Pool -- 8) Initializing Mojarra 2.2.13.SP4  for context '/codenotfound'
06:31:33,708 INFO  [org.primefaces.webapp.PostConstructApplicationEventListener] (ServerService Thread Pool -- 8) Running on PrimeFaces 6.1
06:31:33,725 INFO  [org.wildfly.extension.undertow] (ServerService Thread Pool -- 8) WFLYUT0021: Registered web context: '/codenotfound' for server 'default-server'
06:31:33,781 INFO  [org.jboss.as.server] (management-handler-thread - 1) WFLYSRV0010: Deployed "jsf-primefaces-wildfly-0.0.1-SNAPSHOT.war" (runtime-name : "jsf-primefaces-wildfly-0.0.1-SNAPSHOT.war")
06:31:46,918 INFO  [org.hibernate.validator.internal.util.Version] (default task-1) HV000001: Hibernate Validator 5.3.5.Final
```

Open a web browser and enter following URL: [http://localhost:9090/codenotfound/](http://localhost:9090/codenotfound/){:target="_blank"}. The below page should be displayed:

<figure>
    <img src="{{ site.url }}/assets/images/posts/jsf-primefaces/jsf-primefaces-hello-world-example.png" alt="jsf primefaces hello world example">
</figure>

Enter a first and last name and press the <var>Submit</var> button. A pop-up dialog will be shown with a greeting message. 

<figure>
    <img src="{{ site.url }}/assets/images/posts/jsf-primefaces/jsf-primefaces-hello-world-example-greeting.png" alt="jsf primefaces hello world example greeting">
</figure>

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/jsf-primefaces/tree/master/jsf-primefaces-wildfly){:target="_blank"}.
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This concludes the PrimeFaces WildFly example. If you found this post helpful or have any questions or remarks, please leave a comment.
