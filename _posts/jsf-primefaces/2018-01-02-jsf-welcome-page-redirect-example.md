---
title: "JSF - Welcome Page Redirect Example using Spring Boot"
permalink: /jsf-welcome-page-redirect-example-spring-boot.html
excerpt: "A detailed code sample on how to implement a JSF welcome page redirect using Spring Boot."
date: 2018-01-02
last_modified_at: 2018-01-02
header:
  teaser: "assets/images/teaser/primefaces-teaser.png"
categories: [PrimeFaces]
tags: [Code Sample, Example, JavaServer Faces, JSF, Maven, PrimeFaces, Redirect, Spring Boot, Welcome File, Welcome Page]
published: true
---

<figure>
  <img src="{{ site.url }}/assets/images/logo/primefaces-logo.png" alt="primefaces logo" class="logo">
</figure>

A [welcome-file-list](https://docs.oracle.com/cd/E19798-01/821-1841/bnaer/index.html){:target="_blank"} allows you to specify a list of files that the web container will use for appending to a request for a URL that is not mapped to a web component.

The following example shows how to setup a welcome file redirect for your JSF web application using Spring Boot and Maven.

# General Project Setup

Tools used:
* Spring Boot 1.5
* PrimeFaces 6.1
* JoinFaces 2.4
* Selenium 3.8
* Maven 3.5

We will start from a previous [JSF Spring Boot Tutorial ]({{ site.url }}/jsf-primefaces-example-spring-boot-maven.html) in which we created a greeting dialog based on a first and last name input form.

As we are running on Spring Boot we no longer have a <var>web.xml</var> in which we can specify a <var>&lt;welcome-file-list&gt;</var>.

A way to solve this is to [extend the WebMvcConfigurerAdapter](https://stackoverflow.com/a/29054676/4201470){:target="_blank"} and then forward the default mapping to the target web page. In this example we will forward to <var>helloworld.xhtml</var> as shown below.

The `WelcomePageRedirect` class is annotated with `@Configuration` which [indicates](https://docs.spring.io/spring/docs/4.3.13.RELEASE/spring-framework-reference/html/beans.html#beans-java-basic-concepts){:target="_blank"} that the class can be used by the Spring IoC container as a source of bean definitions. In other words we can specify the page redirect using Java-configuration instead of XML.

``` java
package com.codenotfound.primefaces.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.core.Ordered;
import org.springframework.web.servlet.config.annotation.ViewControllerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;

@Configuration
public class DefaultView extends WebMvcConfigurerAdapter {

  @Override
  public void addViewControllers(ViewControllerRegistry registry) {
    registry.addViewController("/")
        .setViewName("forward:/helloworld.xhtml");
    registry.setOrder(Ordered.HIGHEST_PRECEDENCE);

    super.addViewControllers(registry);
  }
}
```

Let's test above configuration by opening a command prompt. Execute following Maven command in order to start the JSF Hello World web application.

``` plaintext
mvn spring-boot:run
```

The resulting console log should mention <var>'Started SpringPrimeFacesApplication'</var> which indicates our web application is up and running.

``` plaintext
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.5.7.RELEASE)

09:53:47.137 [main] INFO  c.c.p.SpringPrimeFacesApplication - Starting SpringPrimeFacesApplication on cnf-pc with PID 4748 (c:\codenotfound\code\jsf-primefaces\jsf-welcome-page-redirect\target\classes started by CodeNotFound in c:\codenotfound\code\jsf-primefaces\jsf-welcome-page-redirect)
09:53:47.141 [main] INFO  c.c.p.SpringPrimeFacesApplication - No active profile set, falling back to default profiles: default
09:53:51.586 [main] INFO  c.c.p.SpringPrimeFacesApplication - Started SpringPrimeFacesApplication in 4.756 seconds (JVM running for 8.77)
```

Open a web browser and enter following URL: [http://localhost:9090/codenotfound/](http://localhost:9090/codenotfound/){:target="_blank"}. The below web page should now be displayed.

<figure>
  <img src="{{ site.url }}/assets/images/posts/jsf-primefaces/jsf-welcome-page-redirect.png" alt="jsf welcome page redirect">
</figure>

This means that the redirect was successfully applied as otherwise a <var>'Whitelabel Error Page'</var> error would have been returned as shown below.

<figure>
  <img src="{{ site.url }}/assets/images/posts/jsf-primefaces/jsf-welcome-page-redirect-error.png" alt="jsf welcome page redirect error">
</figure>

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/jsf-primefaces/tree/master/jsf-welcome-page-redirect){:target="_blank"}.
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This concludes the short code sample on how to setup a JSF welcome file XHTML using Spring Boot. Let me know if this example was helpful.
