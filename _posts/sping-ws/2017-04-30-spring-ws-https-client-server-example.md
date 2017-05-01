---
title: "Spring WS - HTTPS Client Server Example"
permalink: /2017/04/spring-ws-https-client-server-example.html
excerpt: "A detailed step-by-step tutorial on how setup HTTPS on client and server side using Spring-WS and Spring Boot."
date: 2017-04-30
modified: 2017-04-30
categories: [Spring-WS]
tags: [Client, Example, HTTPS, Maven, Server, Spring, Spring Boot, Spring Web Services, Spring-WS, Tutorial]
published: true
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/spring-logo.jpg" alt="spring logo" class="logo">
</figure>

[HTTPS](https://en.wikipedia.org/wiki/HTTPS) is a communications protocol for secure communication over a computer network. It consists of communication over Hypertext Transfer Protocol (HTTP) within a connection encrypted by [Transport Layer Security](https://en.wikipedia.org/wiki/Transport_Layer_Security) (TLS), or its predecessor, Secure Sockets Layer (SSL).

A web service exposed on HTTPS provides **authentication** of associated web server with which one is communicating. In addition it provides **bidirectional encryption** of communications between the client and server, which protects against eavesdropping and tampering with or forging the contents of the communication.

The following example shows how to configure both client and server in order to consume and respectively expose a web service over HTTPS using Spring-WS, Spring Boot and Maven. 

Tools used:
* Spring-WS 2.4
* Spring Boot 1.5
* Maven 3.5

# General Project Setup

The setup of the project is based on a previous [Spring WS example]({{ site.url }}/2016/10/spring-ws-soap-web-service-consumer-provider-wsdl-example.html) but the basic <var>helloworld.wsdl</var> has been replaced by a more generic <var>ticketagent.wsdl</var> from the [W3C WSDL 1.1 specification](https://www.w3.org/TR/wsdl11elementidentifiers/#Iri-ref-ex).

Security related features of Spring-WS are not part of the `spring-boot-starter-web-services` Spring Boot starter. As such we have to add two extra dependencies to the Maven POM file in order for the example to work.

First the `spring-ws-security` dependency which contains a `FactoryBean` for setting up the client's `TrustStore`. Second the `spring-ws-support` dependency which contains a `MessageSender` that adds support for (self-signed) HTTPS certificates.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>spring-ws-https</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>spring-ws-https</name>
  <description>Spring WS - HTTPS Client Server Example</description>
  <url>https://www.codenotfound.com/2017/04/spring-ws-https-client-server-example.html</url>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.3.RELEASE</version>
  </parent>

  <properties>
    <java.version>1.8</java.version>

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
    <!-- spring-ws -->
    <dependency>
      <groupId>org.springframework.ws</groupId>
      <artifactId>spring-ws-security</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.ws</groupId>
      <artifactId>spring-ws-support</artifactId>
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

Since applications can communicate either with or without TLS (or SSL), it is necessary for the client to indicate to the server the setup of a TLS connection. One of the main ways of achieving this is to use a different port number for TLS connections. In this example we will use port <var>9443</var> instead of port <var>9090</var>.

Once the client and server have agreed to use TLS, they negotiate a stateful connection by using a handshaking procedure. During this procedure, the server usually then sends back its identification in the form of a [digital certificate](https://en.wikipedia.org/wiki/Public_key_certificate).

Java programs store certificates in a repository called [Java KeyStore](https://en.wikipedia.org/wiki/Keystore) (JKS). To generate the keystore and certificate for this example we use [keytool](http://docs.oracle.com/javase/6/docs/technotes/tools/windows/keytool.html) which is a key and certificate management utility that ships with Java.

Open a command prompt at the root of your Maven project and execute following statement to generate a [public/private keypair](https://en.wikipedia.org/wiki/Public-key_cryptography) for the server side. The result will be a <var>server-keystore.jks</var> Java Keystore file that contains a key pair called <var>'server-keypair'</var>.

``` plaintext
keytool -genkeypair -alias server-keypair -keyalg RSA -keysize 2048 -validity 3650 -dname "CN=server,O=codenotfound.com" -keypass server-key-p455w0rd -keystore server-keystore.jks -storepass server-keystore-p455w0rd
```

If you would like to visualize the content of the keystore you can use a tool like [Portecle](http://portecle.sourceforge.net/). Navigate to the <var>server-keystore.jks</var> JKS file and when prompted enter the keystore password (in the above command we used <kbd>"server-keystore-p455w0rd"</kbd>) and the result should be should be similar to what is shown below.

<figure>
    <img src="{{ site.url }}/assets/images/spring-ws/server-keystore.png" alt="server keystore">
</figure>

For the client we need to create a truststore (also a JKS file) which contains certificates from other parties that you expect to communicate with, or from [Certificate Authorities](https://en.wikipedia.org/wiki/Certificate_authority) that you trust to identify other parties. In this example we will add the server's public certificate to the client's truststore. As a result our client will "trust" and thus allow an HTTPS connection to the server.

To create the truststore we first need to export the public key certificate or digital certificate of the server. Use following command to generate a <var>server-public-key.cer</var> certificate file.

``` plaintext
keytool -exportcert -alias server-keypair -file server-public-key.cer -keystore server-keystore.jks -storepass server-keystore-p455w0rd
```

Use the <var>Examine Certificate</var> menu in Portecle to visualize the certificate.

<figure>
    <img src="{{ site.url }}/assets/images/spring-ws/server-public-key.png" alt="server public key">
</figure>

Now we create a <var>client-truststore.jks</var> that contains the exported certificate by executing following keytool command.

``` plaintext
keytool -importcert -keystore client-truststore.jks -alias server-public-key -file server-public-key.cer -storepass client-truststore-p455w0rd -noprompt
```

Similar to the keystore we can open the truststore using Portecle to inspect its contents.

<figure>
    <img src="{{ site.url }}/assets/images/spring-ws/client-truststore.png" alt="client truststore">
</figure>

As a last step we more the three artifacts we have just generated: <var>client-truststore.jks</var>, <var>server-keystore.jks</var> and <var>server-public-key.cer</var> to the <var>src/main/resources</var> folder so that they are available on the classpath for both client and server setup.

<figure>
    <img src="{{ site.url }}/assets/images/spring-ws/https-jks-files.png" alt="https jks files">
</figure>

# Setup HTTPS on the Client

As the server will expose the ticket agent service on HTTPS we need to change the default URI (service address) that is set on the `WebServiceTemplate`. The `@Value` annotation is used to inject the <var>'client.default-uri'</var> value from the application properties YAML file.

There are two other values that are configured in our <var>application.yml</var> configuration file. These are are the location of the truststore JKS file and it's password as shown below.

``` yml
client:
  default-uri: https://localhost:9443/codenotfound/ws/ticketagent
  ssl:
    trust-store: classpath:jks/client-truststore.jks
    trust-store-password: client-truststore-p455w0rd
```

The `ClientConfig` class 

To easily [load the truststore using Spring configuration](http://docs.spring.io/spring-ws/docs/2.4.0.RELEASE/reference/htmlsingle/#d5e2263), we can use the `KeyStoreFactoryBean` that ships with the `spring-ws-security` dependency that was added to the project's <var>pom.xml</var>. The bean has a resource location property and password, which both need to be set.

``` java
package com.codenotfound.ws.client;

import javax.net.ssl.HostnameVerifier;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.Resource;
import org.springframework.oxm.jaxb.Jaxb2Marshaller;
import org.springframework.ws.client.core.WebServiceTemplate;
import org.springframework.ws.soap.security.support.KeyStoreFactoryBean;
import org.springframework.ws.soap.security.support.TrustManagersFactoryBean;
import org.springframework.ws.transport.http.HttpsUrlConnectionMessageSender;

@Configuration
public class ClientConfig {

  @Value("${client.default-uri}")
  private String defaultUri;

  @Value("${client.ssl.trust-store}")
  private Resource trustStore;

  @Value("${client.ssl.trust-store-password}")
  private String trustStorePassword;

  @Bean
  Jaxb2Marshaller jaxb2Marshaller() {
    Jaxb2Marshaller jaxb2Marshaller = new Jaxb2Marshaller();
    jaxb2Marshaller.setContextPath("org.example.ticketagent");

    return jaxb2Marshaller;
  }

  @Bean
  public WebServiceTemplate webServiceTemplate() throws Exception {
    WebServiceTemplate webServiceTemplate = new WebServiceTemplate();
    webServiceTemplate.setMarshaller(jaxb2Marshaller());
    webServiceTemplate.setUnmarshaller(jaxb2Marshaller());
    webServiceTemplate.setDefaultUri(defaultUri);
    // set a httpsUrlConnectionMessageSender to handle the HTTPS session
    webServiceTemplate.setMessageSender(httpsUrlConnectionMessageSender());

    return webServiceTemplate;
  }

  @Bean
  public HttpsUrlConnectionMessageSender httpsUrlConnectionMessageSender() throws Exception {
    HttpsUrlConnectionMessageSender httpsUrlConnectionMessageSender =
        new HttpsUrlConnectionMessageSender();
    httpsUrlConnectionMessageSender.setTrustManagers(trustManagersFactoryBean().getObject());
    // allows the client to skip host name verification as otherwise following error is thrown:
    // java.security.cert.CertificateException: No name matching localhost found
    httpsUrlConnectionMessageSender.setHostnameVerifier(new HostnameVerifier() {
      @Override
      public boolean verify(String hostname, javax.net.ssl.SSLSession sslSession) {
        if ("localhost".equals(hostname)) {
          return true;
        }
        return false;
      }
    });

    return httpsUrlConnectionMessageSender;
  }

  @Bean
  public KeyStoreFactoryBean trustStore() {
    KeyStoreFactoryBean keyStoreFactoryBean = new KeyStoreFactoryBean();
    keyStoreFactoryBean.setLocation(trustStore);
    keyStoreFactoryBean.setPassword(trustStorePassword);

    return keyStoreFactoryBean;
  }

  @Bean
  public TrustManagersFactoryBean trustManagersFactoryBean() {
    TrustManagersFactoryBean trustManagersFactoryBean = new TrustManagersFactoryBean();
    trustManagersFactoryBean.setKeyStore(trustStore().getObject());

    return trustManagersFactoryBean;
  }
}
```

# Setup Server Basic Authentication

The Spring Boot security starter that we added to our Maven setup has a dependency on Spring Security. If Spring Security is on the classpath then [web applications will be secured by default with HTTP basic authentication](http://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-security) on all HTTP endpoints. In other words our `TicketAgentEndpoint` is now secured with basic auth.

The default user that will be configured has as name <var>'user'</var>. The password is randomly generated at startup.

Typically you will want to configure a custom value for the user and password, in order to do this you need to set the [Spring Boot security properties](https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html) in your application properties file. In this example we set the <var>'user'</var> to <kbd>"codenotfound"</kbd> and the <var>'password'</var> to <kbd>"p455w0rd"</kbd> using the YAML variant as shown below. 

``` yml
security:
  user:
    name: codenotfound
    password: p455w0rd
```

# Testing the Basic Authentication Configuration

In order to test the configuration we just run the `SpringWsApplicationTests` unit test case by issuing the following Maven command.

``` plaintext
mvn test
```

The test case will run successfully as basic authentication is correctly configured on both sides. By default the basic authentication header is not logged but if you want you can add some custom code in order to have [Spring-WS log all the client HTTP headers]({{ site.url }}/2017/04/spring-ws-log-client-server-http-headers.html).

``` plaintext
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.5.3.RELEASE)

21:06:58.016 [main] INFO  c.c.ws.SpringWsApplicationTests - Starting SpringWsApplicationTests on cnf-pc with PID 2176 (started by CodeNotFound in c:\code\st\spring-ws\spring-ws-basic-authentication)
21:06:58.018 [main] INFO  c.c.ws.SpringWsApplicationTests - No active profile set, falling back to default profiles: default
21:07:01.275 [main] INFO  c.c.ws.SpringWsApplicationTests - Started SpringWsApplicationTests in 3.568 seconds (JVM running for 4.234)
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 4.039 sec - in com.codenotfound.ws.SpringWsApplicationTests

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 7.454 s
[INFO] Finished at: 2017-04-29T21:07:01+02:00
[INFO] Final Memory: 27M/217M
[INFO] ------------------------------------------------------------------------
```

Now change the password in the <var>application.yml</var> file to a different value and rerun the test case. This time the test case will fail as a <var>401 Unauthorized</var> is returned by our service.

``` plaintext
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.5.3.RELEASE)

21:52:41.786 [main] INFO  c.c.ws.SpringWsApplicationTests - Starting SpringWsApplicationTests on cnf-pc with PID 5908 (started by CodeNotFound in c:\code\st\spring-ws\spring-ws-basic-authentication)
21:52:41.789 [main] INFO  c.c.ws.SpringWsApplicationTests - No active profile set, falling back to default profiles: default
21:52:45.159 [main] INFO  c.c.ws.SpringWsApplicationTests - Started SpringWsApplicationTests in 3.666 seconds (JVM running for 4.281)
Tests run: 1, Failures: 0, Errors: 1, Skipped: 0, Time elapsed: 4.004 sec <<< FAILURE! - in com.codenotfound.ws.SpringWsApplicationTests
testListFlights(com.codenotfound.ws.SpringWsApplicationTests)  Time elapsed: 0.263 sec  <<< ERROR!
org.springframework.ws.client.WebServiceTransportException:  [401]
        at org.springframework.ws.client.core.WebServiceTemplate.handleError(WebServiceTemplate.java:699)
        at org.springframework.ws.client.core.WebServiceTemplate.doSendAndReceive(WebServiceTemplate.java:609)
        at org.springframework.ws.client.core.WebServiceTemplate.sendAndReceive(WebServiceTemplate.java:555)
        at org.springframework.ws.client.core.WebServiceTemplate.marshalSendAndReceive(WebServiceTemplate.java:390)
        at org.springframework.ws.client.core.WebServiceTemplate.marshalSendAndReceive(WebServiceTemplate.java:383)
        at org.springframework.ws.client.core.WebServiceTemplate.marshalSendAndReceive(WebServiceTemplate.java:373)
        at com.codenotfound.ws.client.TicketAgentClient.listFlights(TicketAgentClient.java:30)
        at com.codenotfound.ws.SpringWsApplicationTests.testListFlights(SpringWsApplicationTests.java:26)


Results :

Tests in error:
  SpringWsApplicationTests.testListFlights:26 ╗ WebServiceTransport  [401]

Tests run: 1, Failures: 0, Errors: 1, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 6.460 s
[INFO] Finished at: 2017-04-29T21:52:45+02:00
[INFO] Final Memory: 18M/227M
[INFO] ------------------------------------------------------------------------
```

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-ws/tree/master/spring-ws-https).
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

Setting up basic authentication on the client side using Spring WS is pretty simple when using the Apache client. The server side is even easier when running on Spring Boot.

Drop me a line if you found the example useful. Or let me know in case of questions.
