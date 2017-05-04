---
title: "Spring WS - Mutual Authentication Example"
permalink: /2017/04/spring-ws-mutual-authentication-example.html
excerpt: "A detailed step-by-step tutorial on how setup mutual certificate authentication using Spring-WS and Spring Boot."
date: 2017-05-01
modified: 2017-05-01
categories: [Spring-WS]
tags: [Client, Example, Maven, Mutual Authentication, Server, Spring, Spring Boot, Spring Web Services, Spring-WS, Tutorial, Two-Way Authentication]
published: false
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/spring-logo.jpg" alt="spring logo" class="logo">
</figure>

[Mutual authentication or two-way authentication](https://en.wikipedia.org/wiki/Mutual_authentication) refers to **two parties authenticating each other at the same time**. In other words the client must prove its identity to the server, and the server must prove its identity to the client, before any traffic is sent over the client-to-server connection.

This example shows how to configure both client and server so that mutual authentication is enabled on a web service using Spring-WS, Spring Boot and Maven. 

Tools used:
* Spring-WS 2.4
* Spring Boot 1.5
* Maven 3.5

# General Project Setup

The setup of the project is based on a previous [Spring WS HTTPS example]({{ site.url }}/2017/04/spring-ws-https-client-server-example.html) in which we configured the server authentication part. We will extend this setup so that the client also authenticates itself towards the server.

We will use [keytool](http://docs.oracle.com/javase/6/docs/technotes/tools/windows/keytool.html) to generate the different [Java KeyStores](https://en.wikipedia.org/wiki/Keystore) (JKS) which contain the key pairs and public certificates for both client and server.

Subsequently execute the following three commands in order to generate the <var>server-keystore.jks</var> and <var>client-truststore.jks</var>.

> Note that we are specifying a DNS subject alternative name entry (<kbd>"-ext san=dns:localhost"</kbd>) matching the <var>'localhost'</var> hostname on the first keytool command. This way we do not need to override the `HostnameVerifier` like we did in the [HTTPS client example]({{ site.url }}/2017/04/spring-ws-https-client-server-example.html).

``` plaintext
keytool -genkeypair -alias server-keypair -keyalg RSA -keysize 2048 -validity 3650 -dname "CN=server,O=codenotfound.com" -keypass server-key-p455w0rd -keystore server-keystore.jks -storepass server-keystore-p455w0rd -ext san=dns:localhost
```

``` plaintext
keytool -exportcert -alias server-keypair -file server-public-key.cer -keystore server-keystore.jks -storepass server-keystore-p455w0rd
```

``` plaintext
keytool -importcert -keystore client-truststore.jks -alias server-public-key -file server-public-key.cer -storepass client-truststore-p455w0rd -noprompt
```

Next execute following three commands to generate the <var>client-keystore.jks</var> and <var>server-truststore.jks</var>.

``` plaintext
keytool -genkeypair -alias client-keypair -keyalg RSA -keysize 2048 -validity 3650 -dname "CN=client,O=codenotfound.com" -keypass client-key-p455w0rd -keystore client-keystore.jks -storepass client-keystore-p455w0rd
```

``` plaintext
keytool -exportcert -alias client-keypair -file client-public-key.cer -keystore client-keystore.jks -storepass client-keystore-p455w0rd
```

``` plaintext
keytool -importcert -keystore server-truststore.jks -alias client-public-key -file client-public-key.cer -storepass server-truststore-p455w0rd -noprompt
```

Now (if needed) move the created JKS files into <var>src/main/resources</var>. The result should be as shown below.

<figure>
    <img src="{{ site.url }}/assets/images/spring-ws/mutual-authentication-jks-files.png" alt="mutual authentication jks files">
</figure>

# Setup the Client Keystore and Truststore

The details on the keystore and trustore are injected in the `ClientConfig` class using the `@Value` annotation. The values are defined in the <var>application.yml</var> properties file which is located under <var>src/main/resources</var>.

``` yaml
client:
  default-uri: https://localhost:9443/codenotfound/ws/ticketagent
  ssl:
    key-store: classpath:jks/client-keystore.jks
    key-store-password: client-keystore-p455w0rd
    key-password: client-key-p455w0rd
    trust-store: classpath:jks/client-truststore.jks
    trust-store-password: client-truststore-p455w0rd
```

As the client needs to authenticate itself

``` java
package com.codenotfound.ws.client;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.Resource;
import org.springframework.oxm.jaxb.Jaxb2Marshaller;
import org.springframework.ws.client.core.WebServiceTemplate;
import org.springframework.ws.soap.security.support.KeyManagersFactoryBean;
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

  @Value("${client.ssl.key-store}")
  private Resource keyStore;

  @Value("${client.ssl.key-store-password}")
  private String keyStorePassword;

  @Value("${client.ssl.key-password}")
  private String keyPassword;

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
    // set the trust store(s)
    httpsUrlConnectionMessageSender.setTrustManagers(trustManagersFactoryBean().getObject());
    // set the key store(s)
    httpsUrlConnectionMessageSender.setKeyManagers(keyManagersFactoryBean().getObject());

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

  @Bean
  public KeyStoreFactoryBean keyStore() {
    KeyStoreFactoryBean keyStoreFactoryBean = new KeyStoreFactoryBean();
    keyStoreFactoryBean.setLocation(keyStore);
    keyStoreFactoryBean.setPassword(keyStorePassword);

    return keyStoreFactoryBean;
  }

  @Bean
  public KeyManagersFactoryBean keyManagersFactoryBean() {
    KeyManagersFactoryBean keyManagersFactoryBean = new KeyManagersFactoryBean();
    keyManagersFactoryBean.setKeyStore(keyStore().getObject());
    // set the password of the key pair to be used
    keyManagersFactoryBean.setPassword(keyPassword);

    return keyManagersFactoryBean;
  }
}
```

# Setup the Server Keystore and Truststore

As our web service runs on Spring Boot, we just need to configure the [underlying web server with the correct parameters](http://docs.spring.io/spring-boot/docs/current/reference/html/howto-embedded-servlet-containers.html#howto-configure-ssl). This is done via the [Spring boot web properties](https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html).

Configure the server's keystore (that was generated at the beginning of this tutorial) and corresponding password in addition to the alias of the key pair to be used and it's password. Also change the port to <var>'9443'</var>.

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

Open below URL in your browser and the ticket agent service WSDL definition will now be served over HTTPS.

``` plaintext
https://localhost:9443/codenotfound/ws/ticketagent.wsdl
```

> Notice that your browser will probably flag the connection as being not secure (go ahead and accept an exception). The reason for this is that we are using self-signed certificates which are by default untrusted by your browser.

<figure>
    <img src="{{ site.url }}/assets/images/spring-ws/https-ticketagent-wsdl.png" alt="https ticketagent wsdl">
</figure>

# Testing Spring WS over HTTPS

In order to test the example we can trigger the existing `SpringWsApplicationTests` unit test case by running following Maven command.

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
 :: Spring Boot ::        (v1.5.3.RELEASE)

08:42:52.929 [main] INFO  c.c.ws.SpringWsApplicationTests - Starting SpringWsApplicationTests on cnf-pc with PID 5352 (started by CodeNotFound in c:\code\st\spring-ws\spring-ws-https)
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

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-ws/tree/master/spring-ws-mutual-authentication).
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

Although setting up HTTPs using Spring WS is not extensively covered in the reference documentation, it can be done quite easily using configuration and some support classes.

If you found this tutorial helpful or if you run into some problems let me know in the comments section below.
