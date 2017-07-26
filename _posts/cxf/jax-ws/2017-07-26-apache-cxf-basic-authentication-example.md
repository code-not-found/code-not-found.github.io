---
title: Apache CXF - Basic Authentication Example
permalink: /apache-cxf-basic-authentication-example.html
excerpt: "A detailed step-by-step tutorial on how to configure basic authentication using Apache CXF and Spring Boot."
date: 2017-07-26
modified: 2017-07-26
header:
  teaser: "assets/images/apache-cxf-teaser.png"
categories: [Apache CXF - JAX-WS]
tags: [Apache CXF, Basic Authentication, Client, CXF, Endpoint, Example, HTTP, Maven, Spring Boot, Tutorial]
published: true
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/apache-cxf-logo.png" alt="apache cxf logo" class="logo">
</figure>

[Basic Authentication](https://en.wikipedia.org/wiki/Basic_access_authentication){:target="_blank"} (BA) is a method for a HTTP client to provide a user name and password when making a request. There is no [confidentiality](https://en.wikipedia.org/wiki/Confidentiality){:target="_blank"} protection for the transmitted credentials. therefore it is strongly advised to use it in conjunction with HTTPS.

The credentials are provided as an HTTP header field called <var>'Authorization'</var> which is constructed as follows:

1. The username and password are combined with a single colon.

    ``` plaintext
    codenotofound:p455w0rd
    ```

2. The resulting string is encoded into an [octet sequence](https://tools.ietf.org/html/rfc7617#section-2){:target="_blank"} and then [Base64 encoded](https://tools.ietf.org/html/rfc4648#section-4){:target="_blank"}. You can use an [online Base64 decoder](https://www.base64decode.org/){:target="_blank"} to decode below value.

    ``` plaintext
    Y29kZW5vdGZvdW5kOnA0NTV3MHJk
    ```

3. The authorization method and a space (<kbd>"Basic "</kbd>) are then put before the encoded string.

    ``` plaintext
    Basic Y29kZW5vdGZvdW5kOnA0NTV3MHJk
    ```

Instead of writing custom code to create and check the HTTP authorization header we will configure Apache CXF and Spring Boot to do the work for us. The below example illustrates how a client and server can be configured to apply basic access authentication using Apache CXF, Spring Boot, and Maven.

If you want to learn more about Apache CXF for JAX-WS - head on over to the [Apache CXF - JAX-WS tutorials page]({{ site.url }}/cxf-jaxws/).
{: .notice--primary}

# General Project Setup

Tools used:
* Apache CXF 3.1
* Spring Boot 1.5
* Maven 3.5

The setup of the sample is based on a previous [Apache CXF tutorial]({{ site.url }}/apache-cxf-spring-boot-soap-web-service-client-server-example.html) in which we have swapped out the basic <var>helloworld.wsdl</var> for a more generic <var>ticketagent.wsdl</var> from the [W3C WSDL 1.1 specification](https://www.w3.org/TR/wsdl11elementidentifiers/#Iri-ref-ex){:target="_blank"}.

In order for this example to work we need to add one additional dependency to the Maven POM file which is the `spring-boot-starter-security` [Spring Boot starter](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters){:target="_blank"} dependency that will be used for the server setup.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>cxf-jaxws-basic-authentication</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>cxf-jaxws-basic-authentication</name>
  <description>Apache CXF - Basic Authentication Example</description>
  <url>https://www.codenotfound.com/cxf-jaxws/</url>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.4.RELEASE</version>
  </parent>

  <properties>
    <java.version>1.8</java.version>

    <cxf.version>3.1.12</cxf.version>
  </properties>

  <dependencies>
    <!-- cxf -->
    <dependency>
      <groupId>org.apache.cxf</groupId>
      <artifactId>cxf-spring-boot-starter-jaxws</artifactId>
      <version>${cxf.version}</version>
    </dependency>
    <!-- spring-boot -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <!-- spring-boot-maven-plugin -->
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
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
                  <wsdl>${project.basedir}/src/main/resources/wsdl/ticketagent.wsdl</wsdl>
                  <wsdlLocation>classpath:wsdl/ticketagent.wsdl</wsdlLocation>
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

# CXF Basic Authentication Client 

The CXF framework ships with an `AuthorizationPolicy` class that can be set on the `HTTPConduit` which handles the HTTP(S) transport protocols.

We update the `ClientConfig` by adding a <var>'basicAuthorization'</var> `Bean` on which we set the username and password that are both retrieved from the <var>application.yml</var> properties file. As a basic authentication HTTP header needs to be added we set the type to <var>'Basic'</var>.

The `HTTPConduit` is retieved from the <var>'ticketAgentProxy'</var> `Bean` and using the `setAuthorization()` method the policy is set.

``` java
package com.codenotfound.client;

import org.apache.cxf.configuration.security.AuthorizationPolicy;
import org.apache.cxf.endpoint.Client;
import org.apache.cxf.frontend.ClientProxy;
import org.apache.cxf.jaxws.JaxWsProxyFactoryBean;
import org.apache.cxf.transport.http.HTTPConduit;
import org.example.ticketagent_wsdl11.TicketAgent;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class ClientConfig {

  @Value("${client.ticketagent.address}")
  private String address;
  
  @Value("${client.ticketagent.user-name}")
  private String userName;
  
  @Value("${client.ticketagent.password}")
  private String password;

  @Bean(name = "ticketAgentProxy")
  public TicketAgent ticketAgentProxy() {
    JaxWsProxyFactoryBean jaxWsProxyFactoryBean = new JaxWsProxyFactoryBean();
    jaxWsProxyFactoryBean.setServiceClass(TicketAgent.class);
    jaxWsProxyFactoryBean.setAddress(address);

    return (TicketAgent) jaxWsProxyFactoryBean.create();
  }

  @Bean
  public Client ticketAgentClientProxy() {
    return ClientProxy.getClient(ticketAgentProxy());
  }

  @Bean
  public HTTPConduit ticketAgentConduit() {
    HTTPConduit httpConduit = (HTTPConduit) ticketAgentClientProxy().getConduit();
    httpConduit.setAuthorization(basicAuthorization());

    return httpConduit;
  }

  @Bean
  public AuthorizationPolicy basicAuthorization() {
    AuthorizationPolicy authorizationPolicy = new AuthorizationPolicy();
    authorizationPolicy.setUserName(userName);
    authorizationPolicy.setPassword(password);
    authorizationPolicy.setAuthorizationType("Basic");

    return authorizationPolicy;
  }
}
```

# CXF Basic Authentication Server

The Spring Boot security starter that was added to our Maven setup has a dependency on Spring Security. If Spring Security is on the classpath then [web applications will automatically be secured with HTTP basic authentication](http://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-security){:target="_blank"} on all HTTP endpoints. In other words our, `TicketAgentEndpoint` is now secured with basic auth.

The default user that will be configured has as name <var>'user'</var>. The password is randomly generated at startup (it is displayed in the startup logs).

Typically you will want to configure a custom value for the user and password, in order to do this you need to set the [Spring Boot security properties](https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html){:target="_blank"} in the application properties file. In this example we set the <var>'user'</var> to <kbd>"codenotfound"</kbd> and the <var>'password'</var> to <kbd>"p455w0rd"</kbd> in <var>application.yml</var> using the YAML variant as shown below.

``` yaml
security:
  user:
    name: codenotfound
    password: p455w0rd
```

# Testing the Basic Authentication Configuration

In order to test above configuration, we just run the `SpringWsApplicationTests` unit test case by executing the following Maven command.

``` plaintext
mvn test
```

The test case will run successfully as basic authentication is correctly configured on both sides. The `TicketAgentImpl` was annotated with the `LoggingFeature` and the <var>'org.apache.cxf.services'</var> log level was set to <var>'INFO'</var> so that the HTTP headers are logged including the authorization header as shown below.

``` plaintext
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.5.4.RELEASE)

20:01:07.938 [main] INFO  c.c.SpringCxfApplicationTests - Starting SpringCxfApplicationTests on cnf-pc with PID 4104 (started by CodeNotFound in c:\codenotfound\code\cxf-jaxws\cxf-jaxws-basic-authenticat
ion)
20:01:07.941 [main] INFO  c.c.SpringCxfApplicationTests - No active profile set, falling back to default profiles: default
20:01:12.494 [main] INFO  c.c.SpringCxfApplicationTests - Started SpringCxfApplicationTests in 4.864 seconds (JVM running for 5.557)
20:01:12.713 [http-nio-9090-exec-1] INFO  o.a.c.s.T.T.TicketAgent - Inbound Message
----------------------------
ID: 1
Address: http://localhost:9090/codenotfound/ws/ticketagent
Encoding: UTF-8
Http-Method: POST
Content-Type: text/xml; charset=UTF-8
Headers: {Accept=[*/*], Authorization=[Basic Y29kZW5vdGZvdW5kOnA0NTV3MHJk], cache-control=[no-cache], connection=[keep-alive], Content-Length=[181], content-type=[text/xml; charset=UTF-8], host=[local
host:9090], pragma=[no-cache], SOAPAction=[""], user-agent=[Apache-CXF/3.1.12]}
Payload: <soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"><soap:Body><ns2:listFlightsRequest xmlns:ns2="http://example.org/TicketAgent.xsd"/></soap:Body></soap:Envelope>
--------------------------------------
20:01:12.779 [http-nio-9090-exec-1] INFO  o.a.c.s.T.T.TicketAgent - Outbound Message
---------------------------
ID: 1
Response-Code: 200
Encoding: UTF-8
Content-Type: text/xml
Headers: {}
Payload: <soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"><soap:Body><ns2:listFlightsResponse xmlns:ns2="http://example.org/TicketAgent.xsd"><flightNumber>101</flightNumber></ns2:
listFlightsResponse></soap:Body></soap:Envelope>
--------------------------------------
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 5.263 sec - in com.codenotfound.SpringCxfApplicationTests

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 7.960 s
[INFO] Finished at: 2017-07-26T20:01:13+02:00
[INFO] Final Memory: 21M/227M
[INFO] ------------------------------------------------------------------------
```

Now change the password in the <var>application.yml</var> file to a different value and rerun the test case. This time the test case will fail as a <var>'401'</var> is returned by the server.

``` plaintext
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.5.4.RELEASE)

21:33:07.861 [main] INFO  c.c.SpringCxfApplicationTests - Starting SpringCxfApplicationTests on cnf-pc with PID 3308 (started by CodeNotFound in c:\codenotfound\code\cxf-jaxws\cxf-jaxws-basic-authenticat
ion)
21:33:07.863 [main] INFO  c.c.SpringCxfApplicationTests - No active profile set, falling back to default profiles: default
21:33:11.392 [main] INFO  c.c.SpringCxfApplicationTests - Started SpringCxfApplicationTests in 3.825 seconds (JVM running for 4.459)
21:33:11.657 [main] WARN  o.a.cxf.phase.PhaseInterceptorChain - Interceptor for {http://example.org/TicketAgent.wsdl11}TicketAgentService#{http://example.org/TicketAgent.wsdl11}listFlights has thrown
exception, unwinding now
org.apache.cxf.interceptor.Fault: Could not send Message.
        at org.apache.cxf.interceptor.MessageSenderInterceptor$MessageSenderEndingInterceptor.handleMessage(MessageSenderInterceptor.java:64)
        at org.apache.cxf.phase.PhaseInterceptorChain.doIntercept(PhaseInterceptorChain.java:308)
        at org.apache.cxf.endpoint.ClientImpl.doInvoke(ClientImpl.java:518)
        at org.apache.cxf.endpoint.ClientImpl.invoke(ClientImpl.java:427)
        at org.apache.cxf.endpoint.ClientImpl.invoke(ClientImpl.java:328)
        at org.apache.cxf.endpoint.ClientImpl.invoke(ClientImpl.java:281)
        at org.apache.cxf.frontend.ClientProxy.invokeSync(ClientProxy.java:96)
        at org.apache.cxf.jaxws.JaxWsClientProxy.invoke(JaxWsClientProxy.java:139)
        at com.sun.proxy.$Proxy109.listFlights(Unknown Source)
        at com.codenotfound.client.TicketAgentClient.listFlights(TicketAgentClient.java:23)
        at com.codenotfound.SpringCxfApplicationTests.testListFlights(SpringCxfApplicationTests.java:26)
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:498)
        at org.junit.runners.model.FrameworkMethod$1.runReflectiveCall(FrameworkMethod.java:50)
        at org.junit.internal.runners.model.ReflectiveCallable.run(ReflectiveCallable.java:12)
        at org.junit.runners.model.FrameworkMethod.invokeExplosively(FrameworkMethod.java:47)
        at org.junit.internal.runners.statements.InvokeMethod.evaluate(InvokeMethod.java:17)
        at org.springframework.test.context.junit4.statements.RunBeforeTestMethodCallbacks.evaluate(RunBeforeTestMethodCallbacks.java:75)
        at org.springframework.test.context.junit4.statements.RunAfterTestMethodCallbacks.evaluate(RunAfterTestMethodCallbacks.java:86)
        at org.springframework.test.context.junit4.statements.SpringRepeat.evaluate(SpringRepeat.java:84)
        at org.junit.runners.ParentRunner.runLeaf(ParentRunner.java:325)
        at org.springframework.test.context.junit4.SpringJUnit4ClassRunner.runChild(SpringJUnit4ClassRunner.java:252)
        at org.springframework.test.context.junit4.SpringJUnit4ClassRunner.runChild(SpringJUnit4ClassRunner.java:94)
        at org.junit.runners.ParentRunner$3.run(ParentRunner.java:290)
        at org.junit.runners.ParentRunner$1.schedule(ParentRunner.java:71)
        at org.junit.runners.ParentRunner.runChildren(ParentRunner.java:288)
        at org.junit.runners.ParentRunner.access$000(ParentRunner.java:58)
        at org.junit.runners.ParentRunner$2.evaluate(ParentRunner.java:268)
        at org.springframework.test.context.junit4.statements.RunBeforeTestClassCallbacks.evaluate(RunBeforeTestClassCallbacks.java:61)
        at org.springframework.test.context.junit4.statements.RunAfterTestClassCallbacks.evaluate(RunAfterTestClassCallbacks.java:70)
        at org.junit.runners.ParentRunner.run(ParentRunner.java:363)
        at org.springframework.test.context.junit4.SpringJUnit4ClassRunner.run(SpringJUnit4ClassRunner.java:191)
        at org.apache.maven.surefire.junit4.JUnit4Provider.execute(JUnit4Provider.java:283)
        at org.apache.maven.surefire.junit4.JUnit4Provider.executeWithRerun(JUnit4Provider.java:173)
        at org.apache.maven.surefire.junit4.JUnit4Provider.executeTestSet(JUnit4Provider.java:153)
        at org.apache.maven.surefire.junit4.JUnit4Provider.invoke(JUnit4Provider.java:128)
        at org.apache.maven.surefire.booter.ForkedBooter.invokeProviderInSameClassLoader(ForkedBooter.java:203)
        at org.apache.maven.surefire.booter.ForkedBooter.runSuitesInProcess(ForkedBooter.java:155)
        at org.apache.maven.surefire.booter.ForkedBooter.main(ForkedBooter.java:103)
Caused by: org.apache.cxf.transport.http.HTTPException: HTTP response '401: null' when communicating with http://localhost:9090/codenotfound/ws/ticketagent
        at org.apache.cxf.transport.http.HTTPConduit$WrappedOutputStream.doProcessResponseCode(HTTPConduit.java:1609)
        at org.apache.cxf.transport.http.HTTPConduit$WrappedOutputStream.handleResponseInternal(HTTPConduit.java:1616)
        at org.apache.cxf.transport.http.HTTPConduit$WrappedOutputStream.handleResponse(HTTPConduit.java:1560)
        at org.apache.cxf.transport.http.HTTPConduit$WrappedOutputStream.close(HTTPConduit.java:1361)
        at org.apache.cxf.transport.AbstractConduit.close(AbstractConduit.java:56)
        at org.apache.cxf.transport.http.HTTPConduit.close(HTTPConduit.java:658)
        at org.apache.cxf.interceptor.MessageSenderInterceptor$MessageSenderEndingInterceptor.handleMessage(MessageSenderInterceptor.java:62)
        ... 40 common frames omitted
Tests run: 1, Failures: 0, Errors: 1, Skipped: 0, Time elapsed: 4.193 sec <<< FAILURE! - in com.codenotfound.SpringCxfApplicationTests
testListFlights(com.codenotfound.SpringCxfApplicationTests)  Time elapsed: 0.293 sec  <<< ERROR!
javax.xml.ws.WebServiceException: Could not send Message.
        at org.apache.cxf.transport.http.HTTPConduit$WrappedOutputStream.doProcessResponseCode(HTTPConduit.java:1609)
        at org.apache.cxf.transport.http.HTTPConduit$WrappedOutputStream.handleResponseInternal(HTTPConduit.java:1616)
        at org.apache.cxf.transport.http.HTTPConduit$WrappedOutputStream.handleResponse(HTTPConduit.java:1560)
        at org.apache.cxf.transport.http.HTTPConduit$WrappedOutputStream.close(HTTPConduit.java:1361)
        at org.apache.cxf.transport.AbstractConduit.close(AbstractConduit.java:56)
        at org.apache.cxf.transport.http.HTTPConduit.close(HTTPConduit.java:658)
        at org.apache.cxf.interceptor.MessageSenderInterceptor$MessageSenderEndingInterceptor.handleMessage(MessageSenderInterceptor.java:62)
        at org.apache.cxf.phase.PhaseInterceptorChain.doIntercept(PhaseInterceptorChain.java:308)
        at org.apache.cxf.endpoint.ClientImpl.doInvoke(ClientImpl.java:518)
        at org.apache.cxf.endpoint.ClientImpl.invoke(ClientImpl.java:427)
        at org.apache.cxf.endpoint.ClientImpl.invoke(ClientImpl.java:328)
        at org.apache.cxf.endpoint.ClientImpl.invoke(ClientImpl.java:281)
        at org.apache.cxf.frontend.ClientProxy.invokeSync(ClientProxy.java:96)
        at org.apache.cxf.jaxws.JaxWsClientProxy.invoke(JaxWsClientProxy.java:139)
        at com.sun.proxy.$Proxy109.listFlights(Unknown Source)
        at com.codenotfound.client.TicketAgentClient.listFlights(TicketAgentClient.java:23)
        at com.codenotfound.SpringCxfApplicationTests.testListFlights(SpringCxfApplicationTests.java:26)


Results :

Tests in error:
  SpringCxfApplicationTests.testListFlights:26 ╗ WebService Could not send Messa...

Tests run: 1, Failures: 0, Errors: 1, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 6.859 s
[INFO] Finished at: 2017-07-26T21:33:11+02:00
[INFO] Final Memory: 21M/227M
[INFO] ------------------------------------------------------------------------
```

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/cxf-jaxws/tree/master/cxf-jaxws-basic-authentication).
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This concludes the CXF Basic Authentication example using Spring and Spring Boot.

Feel free to add a comment if you enjoyed the tutorial or if you still have a question you would like to ask.
