---
title: "Spring WS - Client Server Mutual Authentication Example"
permalink: /2017/04/spring-ws-client-server-mutual-authentication-example.html
excerpt: "A detailed step-by-step tutorial on how to setup mutual authentication using Spring-WS and Spring Boot."
date: 2017-04-25
modified: 2017-04-25
categories: [Spring-WS]
tags: [Certificate, Client, Endpoint, Example, Header, HTTPS, Maven, Mutual Authentication, Server, Spring, Spring Boot, Spring Web Services, Spring-WS, SSL, TLS, Tutorial]
published: false
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/spring-logo.jpg" alt="spring logo" class="logo">
</figure>

[Mutual authentication](https://en.wikipedia.org/wiki/Mutual_authentication) or two-way authentication refers to two parties [authenticating](https://en.wikipedia.org/wiki/Authentication) each other at the same time. In a client server architecture this means that that both client and server check that the other party is, in fact, who they claim to be. When using the TLS protocol (successor to SSL), this check is done via signed certificates.

In the below example we will detail how to implement mutual authentication when using Spring Web Services and Spring Boot. We will start from a simple StockQuote service based on the Spring Web Services Hello World tutorial from a previous post and cover the TLS and mutual authentication setup of both consumer and provider.

Tools used:
* Spring-WS 2.4
* Spring Boot 1.5
* Maven 3.5

# General Project Setup

In this example we will be using `HttpsUrlConnectionMessageSender` on the client in order to set the keystores to by used during the TLS handshake. As this class is not part of the `spring-boot-starter-web-services` starter we need to add an extra dependency to `spring-ws-support` as shown below.

``` xml

```

In addition we will need to generate and add [keystores](https://en.wikipedia.org/wiki/Keystore) for both the client and server to our Maven project. These keystores will contain the [certificates](https://en.wikipedia.org/wiki/Public_key_certificate) that are going to be used to perform the two-way TLS handshake. To generate the keystores and certificates we use [keytool](http://docs.oracle.com/javase/6/docs/technotes/tools/windows/keytool.html) which is a key and certificate management utility that ships with Java.


Open a command prompt at the root of your Maven project and execute following statement to generate a [public/private keypair](https://en.wikipedia.org/wiki/Public-key_cryptography) for the server side. The result will be a <var>server.jks</var> Java Keystore file that contains a key pair caller <var>'server-key'</var>.

``` plaintext
keytool -genkeypair -alias server-key -keyalg RSA -keysize 2048 -validity 3650 -dname "CN=server,O=codenotfound.com" -keypass server-key-p455w0rd -keystore server.jks -storepass server-keystore-p455w0rd
```

If you would like to visualize the content of the keystore you can use a tool like Portecle. Navigate to the JKS file and when prompted enter the keystore password (for the <var>server.jks</var> this is <kbd>"server-keystore-p455w0rd"</kbd>). The result should be should be similar to what is shown below. 

Next, repeat the same command but this time we will be generating the keypair that will be used by the client. The result is a <var>client.jks</var> Java Keystore file that contains a key pair caller <var>'client-key'</var>.

``` plaintext
keytool -genkeypair -alias client-key -keyalg RSA -keysize 2048 -validity 3650 -dname "CN=client,O=codenotfound.com" -keypass client-p455w0rd -keystore client.jks -storepass client-p455w0rd
```

> Notice that for the client the password of the keypair and keystore are identical. This is mandatory as [it is not possible to set the key password when using the default Java key manager](http://stackoverflow.com/a/15182344/4201470).

During the TLS handshake the server will pass it's public key to the client 

# Configuring the Endpoint (Provider)

















