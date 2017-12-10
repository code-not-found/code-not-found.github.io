---
title: "Spring WS - HTTPS Client-Server Example"
permalink: /spring-ws-https-client-server-example.html
excerpt: "A detailed step-by-step tutorial on how to setup HTTPS on client and server side using Spring-WS and Spring Boot."
date: 2017-04-30
modified: 2017-04-30
header:
  teaser: "assets/images/header/spring-ws-teaser.png"
categories: [Spring-WS]
tags: [Client, Example, HTTPS, Maven, Server, Spring, Spring Boot, Spring Web Services, Spring-WS, SSL, TLS, Tutorial]
redirect_from:
  - /2017/04/spring-ws-https-client-server-example.html
published: true
---

<figure>
    <img src="{{ site.url }}/assets/images/logo/spring-logo.png" alt="spring logo" class="logo">
</figure>

[HTTPS](https://en.wikipedia.org/wiki/HTTPS){:target="_blank"} is a protocol for secure communication over a computer network. It consists of communication over Hypertext Transfer Protocol (HTTP) within a connection encrypted by [Transport Layer Security](https://en.wikipedia.org/wiki/Transport_Layer_Security){:target="_blank"} (TLS), or its predecessor, Secure Sockets Layer (SSL).

A web service exposed on HTTPS provides **authentication** of the associated web server with which one is communicating. In addition, it provides **bidirectional encryption** of communications between the client and server, that protects against eavesdropping and tampering with the contents of the communication.

The following example shows how to configure both client and server in order to consume and respectively expose a web service over HTTPS using Spring-WS, Spring Boot, and Maven.

If you want to learn more about Spring WS - head on over to the [Spring WS tutorials page]({{ site.url }}/spring-ws/).
{: .notice--primary}

# General Project Setup

Tools used:
* Spring-WS 2.4
* HttpClient 4.5
* Spring Boot 1.5
* Maven 3.5

The setup of the project is based on a previous [Spring WS example]({{ site.url }}/spring-ws-soap-web-service-consumer-provider-wsdl-example.html) but the basic <var>helloworld.wsdl</var> has been replaced by a more generic <var>ticketagent.wsdl</var> from the [W3C WSDL 1.1 specification](https://www.w3.org/TR/wsdl11elementidentifiers/#Iri-ref-ex){:target="_blank"}.

There are two implementations of the `WebServiceMessageSender` interface for sending messages via HTTPS. The default implementation is the `HttpsUrlConnectionMessageSender`, which uses the facilities provided by Java itself. The alternative is the `HttpComponentsMessageSender`, which uses the [Apache HttpComponents HttpClient](https://hc.apache.org/httpcomponents-client-ga){:target="_blank"}.

We will use the `HttpComponentsMessageSender` implementation in below example as it contains more advanced and easy-to-use functionality. On GitHub, however, we have also added a [HTTPS example that uses the HttpsUrlConnectionMessageSender implementation](https://github.com/code-not-found/spring-ws/tree/master/spring-ws-https){:target="_blank"} in case a dependency on the `HttpClient` is not desired.

In order to use the `HttpComponentsMessageSender` implementation, we need to add the Apache `httpclient` dependency to the Maven POM file.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>spring-ws-https-httpclient</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>spring-ws-https-httpclient</name>
  <description>Spring WS - HTTPS Client Server Example</description>
  <url>https://www.codenotfound.com/spring-ws-https-client-server-example.html</url>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.9.RELEASE</version>
  </parent>

  <properties>
    <java.version>1.8</java.version>
    <httpclient.version>4.5.4</httpclient.version>
    <maven-jaxb2-plugin.version>0.13.3</maven-jaxb2-plugin.version>
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

Since applications can communicate either with or without TLS (or SSL), it is necessary for the client to indicate to the server the setup of a TLS connection. One of the main ways of achieving this is to use a different port number for TLS connections. In this example we will use port <var>9443</var> instead of port <var>9090</var>.

Once the client and server have agreed to use TLS, they negotiate a stateful connection by using a handshaking procedure. During this procedure, the server usually sends back its identification in the form of a [digital certificate](https://en.wikipedia.org/wiki/Public_key_certificate){:target="_blank"}.

Java programs store certificates in a repository called [Java KeyStore](https://en.wikipedia.org/wiki/Keystore){:target="_blank"} (JKS). To generate the keystore and certificate for this example we use [keytool](http://docs.oracle.com/javase/6/docs/technotes/tools/windows/keytool.html){:target="_blank"} which is a key and certificate management utility that ships with Java.

Open a command prompt at the root of your Maven project and execute following statement to generate a [public/private keypair](https://en.wikipedia.org/wiki/Public-key_cryptography){:target="_blank"} for the server side. The result will be a <var>server-keystore.jks</var> Java Keystore file that contains a key pair called <var>'server-keypair'</var>.

``` plaintext
keytool -genkeypair -alias server-keypair -keyalg RSA -keysize 2048 -validity 3650 -dname "CN=server,O=codenotfound.com" -keypass server-key-p455w0rd -keystore server-keystore.jks -storepass server-keystore-p455w0rd
```

If you would like to visualize the content of the keystore you can use a tool like [Portecle](http://portecle.sourceforge.net/){:target="_blank"}. Using the <var>File</var> menu, navigate to the <var>server-keystore.jks</var> JKS file and when prompted enter the keystore password (in the above command we used <kbd>"server-keystore-p455w0rd"</kbd>) and the result should be should be similar to what is shown below.

<figure>
    <img src="{{ site.url }}/assets/images/posts/spring-ws/server-keystore.png" alt="server keystore">
</figure>

For the client, we need to create a truststore (also a JKS file) which contains certificates from other parties that you expect to communicate with, or from [Certificate Authorities](https://en.wikipedia.org/wiki/Certificate_authority){:target="_blank"} (CA) that you trust to identify other parties. In this example, we will add the server's public certificate to the client's truststore. As a result, our client will "trust" and thus allow an HTTPS connection to the server.

To create the truststore we first need to export the public key certificate or digital certificate of the server. Use following command to generate a <var>server-public-key.cer</var> certificate file.

``` plaintext
keytool -exportcert -alias server-keypair -file server-public-key.cer -keystore server-keystore.jks -storepass server-keystore-p455w0rd
```

If you want you can use the <var>Examine Certificate</var> menu in Portecle to visualize the certificate.

<figure>
    <img src="{{ site.url }}/assets/images/posts/spring-ws/server-public-key.png" alt="server public key">
</figure>

Now we create a <var>client-truststore.jks</var> that contains the exported certificate by executing following keytool command.

``` plaintext
keytool -importcert -keystore client-truststore.jks -alias server-public-key -file server-public-key.cer -storepass client-truststore-p455w0rd -noprompt
```

Similar to the keystore we can open the truststore using Portecle to inspect its contents.

<figure>
    <img src="{{ site.url }}/assets/images/posts/spring-ws/client-truststore.png" alt="client truststore">
</figure>

Finally, we move the three artifacts we have just generated: <var>client-truststore.jks</var>, <var>server-keystore.jks</var> and <var>server-public-key.cer</var> to the <var>src/main/resources</var> folder so that they are available on the classpath for both client and server setup.

<figure>
    <img src="{{ site.url }}/assets/images/posts/spring-ws/https-jks-files.png" alt="https jks files">
</figure>

# Setup HTTPS on the Client

As the server will expose the Ticket Agent service on HTTPS we need to change the default URI (service address) that is set on the `WebServiceTemplate` used by the client. The `@Value` annotation is used to inject the <var>'client.default-uri'</var> value from the application properties YAML file.

There are two other values that are configured in the <var>application.yml</var> properties file. These are are the location of the truststore JKS file and its password as shown below.

``` yaml
client:
  default-uri: https://localhost:9443/codenotfound/ws/ticketagent
  ssl:
    trust-store: classpath:jks/client-truststore.jks
    trust-store-password: client-truststore-p455w0rd
```

In the `ClientConfig` class we need to enable the `WebServiceTemplate` to connect using the HTTPS protocol. This is done by creating and setting a `HttpComponentsMessageSender` on which we then configure the HttpClient which provides full [support for HTTP over Secure Sockets Layer (SSL) or Transport Layer Security (TLS) protocols](https://hc.apache.org/httpcomponents-client-4.5.x/tutorial/html/connmgmt.html#d5e449){:target="_blank"}.

`HttpClient` makes use of `SSLConnectionSocketFactory` to create SSL connections. `SSLConnectionSocketFactory` allows for a high degree of customization. It can take an instance of `SSLContext` as a parameter and use it to create custom configured TLS/SSL connections.

During the TLS handshaking procedure, the client needs to decide whether it trusts the public key certificate that the server provides. This is done based on whether or not this certificate (or one of its issuing CA's) is present in (one of) the client's truststores. We specify a `TrustManagersFactoryBean` to handle the configured truststores.

In order to trust the server certificate, create an `sslContext()` bean on which we set the truststore file and its corresponding password. This context is then passed to the `sslConnectionSocketFactory()` bean which is in turn set on the `httpClient()`.

If we were to test the client with above settings we would run into the following exception

``` plaintext
javax.net.ssl.SSLHandshakeException: java.security.cert.CertificateException: No name matching localhost found
```

The reason for this is that when the HTTPS client connects to a server, it's not enough for a certificate to be trusted, it also has to match the server you want to talk to. In other words, the client verifies that the hostname in the certificate matches the hostname of the server. For more detailed information check [this answer on Stack Overflow](http://stackoverflow.com/a/3093650/4201470){:target="_blank"}.

In order to fix this problem, we could regenerate the server keypair so it contains <var>'localhost'</var>. You can find the needed keytool command in the [Spring WS mutual authentication tutorial]({{ site.url }}/spring-ws-mutual-authentication-example.html). 

Another option, which we will use in this example, is to turn hostname verification off. Apache ships a `NoopHostnameVerifier` that can be used for this. Simply pass an instance to the `SSLConnectionSocketFactory` constructor. Note that this is not something you would want to do in production!

There is one last problem we need to take care of. The `HttpComponentsMessageSender` has [two constructors](https://github.com/spring-projects/spring-ws/blob/master/spring-ws-core/src/main/java/org/springframework/ws/transport/http/HttpComponentsMessageSender.java){:target="_blank"}, with and without `HttpClient`, and the one with `HttpClient` omits adding a `SoapRemoveHeaderInterceptor`. The `HttpClient` throws an exception if <var>Content-Length</var> or <var>Transfer-Encoding</var> headers have been set. 

So in order to make sure those headers are not present we add the `SoapRemoveHeaderInterceptor` by using the `addInterceptorFirst()` method on the `HttpClientBuilder`.
 

``` java
package com.codenotfound.ws.client;

import javax.net.ssl.SSLContext;

import org.apache.http.client.HttpClient;
import org.apache.http.conn.ssl.NoopHostnameVerifier;
import org.apache.http.conn.ssl.SSLConnectionSocketFactory;
import org.apache.http.impl.client.HttpClientBuilder;
import org.apache.http.ssl.SSLContextBuilder;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.Resource;
import org.springframework.oxm.jaxb.Jaxb2Marshaller;
import org.springframework.ws.client.core.WebServiceTemplate;
import org.springframework.ws.transport.http.HttpComponentsMessageSender;
import org.springframework.ws.transport.http.HttpComponentsMessageSender.RemoveSoapHeadersInterceptor;

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
    webServiceTemplate.setMessageSender(httpComponentsMessageSender());

    return webServiceTemplate;
  }

  @Bean
  public HttpComponentsMessageSender httpComponentsMessageSender() throws Exception {
    HttpComponentsMessageSender httpComponentsMessageSender = new HttpComponentsMessageSender();
    httpComponentsMessageSender.setHttpClient(httpClient());

    return httpComponentsMessageSender;
  }

  public HttpClient httpClient() throws Exception {
    return HttpClientBuilder.create().setSSLSocketFactory(sslConnectionSocketFactory())
        .addInterceptorFirst(new RemoveSoapHeadersInterceptor()).build();
  }

  public SSLConnectionSocketFactory sslConnectionSocketFactory() throws Exception {
    // NoopHostnameVerifier essentially turns hostname verification off as otherwise following error
    // is thrown: java.security.cert.CertificateException: No name matching localhost found
    return new SSLConnectionSocketFactory(sslContext(), NoopHostnameVerifier.INSTANCE);
  }

  public SSLContext sslContext() throws Exception {
    return SSLContextBuilder.create()
        .loadTrustMaterial(trustStore.getFile(), trustStorePassword.toCharArray()).build();
  }
}
```

# Setup HTTPS on the Server

As our web service runs on Spring Boot, we just need to configure the [underlying web server with the correct parameters](http://docs.spring.io/spring-boot/docs/current/reference/html/howto-embedded-servlet-containers.html#howto-configure-ssl){:target="_blank"}. This is done via the [Spring Boot web properties](https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html){:target="_blank"} (look for the <var># EMBEDDED SERVER CONFIGURATION</var> heading).

In this example, we use the YAML format to specify the different parameters in the application properties file as shown below. The server HTTP port is set to <var>'9443'</var> in order to indicate the usage of HTTPS. The server's keystore (that was generated at the beginning of this tutorial) and the corresponding password are also configured in addition to the alias of the key pair to be used and the corresponding password.

``` yaml
server:
  port: 9443
  ssl:
    key-store: classpath:jks/server-keystore.jks
    key-store-password: server-keystore-p455w0rd
    key-alias: server-keypair
    key-password: server-key-p455w0rd
```

In order to quickly test if the setup was successful, start Spring Boot by running below Maven command at the command prompt.

``` plaintext
mvn spring-boot:run
```

Open below URL in your browser and the ticket agent service WSDL definition will now be served over HTTPS: [https://localhost:9443/codenotfound/ws/ticketagent.wsdl](https://localhost:9443/codenotfound/ws/ticketagent.wsdl)

> Notice that your browser will probably flag the connection as being not secure (go ahead and accept an exception). The reason for this is that we are using self-signed certificates which are by default untrusted by your browser.

<figure>
    <img src="{{ site.url }}/assets/images/posts/spring-ws/https-ticketagent-wsdl.png" alt="https ticketagent wsdl">
</figure>

# Testing Spring WS over HTTPS

In order to test the example, we can trigger the existing `SpringWsApplicationTests` unit test case by running following Maven command.

``` plaintext
mvn test
```

This will result in a successful test run as shown below.

``` plaintext
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.5.9.RELEASE)

08:42:52.929 [main] INFO  c.c.ws.SpringWsApplicationTests - Starting SpringWsApplicationTests on cnf-pc with PID 5352 (started by CodeNotFound in c:\code\spring-ws\spring-ws-https)
08:42:52.932 [main] INFO  c.c.ws.SpringWsApplicationTests - No active profile set, falling back to default profiles: default
08:42:55.727 [main] INFO  c.c.ws.SpringWsApplicationTests - Started SpringWsApplicationTests in 3.096 seconds (JVM running for 3.798)
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 3.484 sec - in com.codenotfound.ws.SpringWsApplicationTests

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 6.275 s
[INFO] Finished at: 2017-05-01T08:42:56+02:00
[INFO] Final Memory: 20M/227M
[INFO] ------------------------------------------------------------------------
```

> If you would like more information when debugging SL/TLS connections add <kbd>"-Djavax.net.debug=ssl,handshake"</kbd> at the end of your Maven command as shown below.

``` plaintext
mvn test -Djavax.net.debug=ssl,handshake
```

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-ws/tree/master/spring-ws-https-httpclient){:target="_blank"}.
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

Although setting up HTTPS using Spring WS is not extensively covered in the reference documentation, it can be done quite easily using configuration and some builder classes.

If you found this tutorial helpful or if you run into some problems let me know in the comments section below.
