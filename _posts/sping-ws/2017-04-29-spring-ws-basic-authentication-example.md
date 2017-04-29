---
title: "Spring WS - Basic Authentication Example"
permalink: /2017/04/spring-ws-basic-authentication-example.html
excerpt: "A detailed step-by-step tutorial on how to configure basic authentication using Spring-WS and Spring Boot."
date: 2017-04-24
modified: 2017-04-24
categories: [Spring-WS]
tags: [Basic Authentication, Client, Endpoint, Example, Maven, Spring, Spring Boot, Spring Web Services, Spring-WS, Tutorial]
published: false
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/spring-logo.jpg" alt="spring logo" class="logo">
</figure>

[Basic Authentication](https://en.wikipedia.org/wiki/Basic_access_authentication) (BA) is a method for a HTTP client to provide a user name and password when making a request. There is no [confidentiality](https://en.wikipedia.org/wiki/Confidentiality) protection for the transmitted credentials. therefore it is strongly advised to use it in conjunction with HTTPS.

The credentials are provided as an HTTP header field called <var>'Authorization'</var> which is constructed as follows:

1. The username and password are combined with a single colon.

``` plaintext
codenotofound:p455w0rd
```

2. The resulting string is encoded into an [octet sequence](https://tools.ietf.org/html/rfc7617#section-2) and then [Base64 encoded](https://tools.ietf.org/html/rfc4648#section-4). You can use an [online Base64 decoder](https://www.base64decode.org/) to check below value.

``` plaintext
Y29kZW5vdG9mb3VuZDpwNDU1dzByZA==
```
3. The authorization method and a space i.e. <kbd>"Basic "</kbd> is then put before the encoded string.

``` plaintext
Authorization: Basic Y29kZW5vdG9mb3VuZDpwNDU1dzByZA==
```

Instead of writing custom code to create and check the HTTP authorization header we will configure Spring WS to do the work for us. The below example illustrates how a client and server can be configured to apply basic access authentication using Spring-WS, Spring Boot and Maven. 

Tools used:
* Spring-WS 2.4
* Spring Boot 1.5
* Maven 3.5

# General Project Setup

The setup of the project is based on a previous [Spring WS example]({{ site.url }}/2016/10/spring-ws-soap-web-service-consumer-provider-wsdl-example.html) in which we have swapped out the basic <var>helloworld.wsdl</var> for a more generic <var>ticketagent.wsdl</var> from the [W3C WSDL 1.1 specification](https://www.w3.org/TR/wsdl11elementidentifiers/#Iri-ref-ex).



---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-ws/tree/master/spring-ws-basic-authentication).
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

Spring WS provides good support for setting and mapping the SOAPAction header as we have illustrated in above example.

If you have any additional thoughts let me know down below. Thanks!
