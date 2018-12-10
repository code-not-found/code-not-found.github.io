---
title: "JSF PrimeFaces Welcome Page Redirect Example"
permalink: /jsf-primefaces-welcome-page-redirect-example.html
excerpt: "A code sample on how to implement a PrimeFaces for JSF welcome page redirect using Spring Boot."
date: 2018-01-02
last_modified_at: 2018-12-08
header:
  teaser: "assets/images/jsf-primefaces/jsf-primefaces-welcome-page-redirect-example.png"
categories: [PrimeFaces]
tags: [Code Sample, Example, JavaServer Faces, JSF, Maven, PrimeFaces, Redirect, Spring Boot, Welcome File, Welcome Page]
redirect_from:
  - /jsf-welcome-page-redirect-example-spring-boot.html
published: true
---

<img src="{{ site.url }}/assets/images/jsf-primefaces/jsf-primefaces-welcome-page-redirect-example.png" alt="jsf primefaces welcome page redirect example" class="align-right title-image">

When it comes to web application navigation, I'm sure at one point you would like to set up a home page redirect.

If you're looking for a practical example using [Spring Boot](https://spring.io/projects/spring-boot){:target="_blank"}, then you'll love this code sample.

So here we go.

If you want to learn more about PrimeFaces for JSF - head on over to the [JSF PrimeFaces tutorials]({{ site.url }}/jsf-primefaces-tutorials) page.
{: .notice--primary}

## 1. What is a Welcome-File-List?

A [welcome-file-list](https://docs.oracle.com/cd/E19798-01/821-1841/bnaer/index.html){:target="_blank"} allows you to specify a list of files that the web container will use for appending to a request for a URL that is not mapped to a web component.

The following example shows how to set a welcome file redirect for your [PrimeFaces](http://primefaces.org/){:target="_blank"} JSF web application using Spring Boot and Maven.

## 2. General Project Overview

We will use the following tools/frameworks:
* PrimeFaces 6.2
* JoinFaces 3.3
* Spring Boot 2.1
* Maven 3.5

Our project has the following directory structure:

<img src="{{ site.url }}/assets/images/jsf-primefaces/jsf-primefaces-welcome-page-redirect-maven-project.png" alt="jsf primefaces welcome page redirect maven project">

## 3. Changing the Default Welcome Page for Spring Boot

We start from a previous [JSF Spring Boot Tutorial ]({{ site.url }}/jsf-primefaces-example.html) in which we created a greeting dialog using a first and last name input form.

As we are running on Spring Boot we no longer have a <var>web.xml</var> in which we can specify a <var>&lt;welcome-file-list&gt;</var>.

A way to solve this is to [extend the WebMvcConfigurerAdapter](https://stackoverflow.com/a/29054676/4201470){:target="_blank"}. Yet, as of Spring 5, the [WebMvcConfigurerAdapter is deprecated](https://www.baeldung.com/web-mvc-configurer-adapter-deprecated){:target="_blank"}. A solution for this is to use the `WebMvcConfigurer` interface directly.

Create a `WelcomePageRedirect` class that implements `WebMvcConfigurer`.

Annotate it with `@Configuration`. This [indicates](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/core.html#beans-java-basic-concepts){:target="_blank"} that the class can be used by the Spring IoC container as a source of bean definitions. In other words, we can specify the page redirect using Java-configuration instead of XML.

Override the `addViewControllers()` method and forward the default mapping to the target web page. In this example, we will forward to <var>helloworld.xhtml</var> as shown below.

{% highlight java %}
package com.codenotfound.primefaces;

import org.springframework.context.annotation.Configuration;
import org.springframework.core.Ordered;
import org.springframework.web.servlet.config.annotation.ViewControllerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WelcomePageRedirect implements WebMvcConfigurer {

  @Override
  public void addViewControllers(ViewControllerRegistry registry) {
    registry.addViewController("/")
        .setViewName("forward:/helloworld.xhtml");
    registry.setOrder(Ordered.HIGHEST_PRECEDENCE);
  }
}
{% endhighlight %}

To test the above configuration, open a command prompt. Execute following Maven command to start the JSF Hello World web application.

{% highlight plaintext %}
mvn spring-boot:run
{% endhighlight %}

The resulting console log should mention <var>'Started SpringPrimeFacesApplication'</var> which indicates our web application is up and running.

{% highlight plaintext %}
.   ____          _            __ _ _
/\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
\\/  ___)| |_)| | | | | || (_| |  ) ) ) )
'  |____| .__|_| |_|_| |_\__, | / / / /
=========|_|==============|___/=/_/_/_/
:: Spring Boot ::        (v2.1.0.RELEASE)

2018-12-07 14:53:49.417  INFO 19904 --- [           main] c.c.SpringPrimeFacesApplication          : Starting SpringPrimeFacesApplication on DESKTOP-2RB3C1U with PID 19904 (C:\Users\Codenotfound\repos\jsf-primefaces\jsf-primefaces-welcome-page-redirect\target\classes started by Codenotfound in C:\Users\Codenotfound\repos\jsf-primefaces\jsf-primefaces-welcome-page-redirect)
2018-12-07 14:53:49.425  INFO 19904 --- [           main] c.c.SpringPrimeFacesApplication          : No active profile set, falling back to default profiles: default
2018-12-07 14:53:51.129  INFO 19904 --- [           main] trationDelegate$BeanPostProcessorChecker : Bean 'org.joinfaces.autoconfigure.javaxfaces.JsfBeansAutoConfiguration$Jsf2_3AutoConfiguration' of type [org.joinfaces.autoconfigure.javaxfaces.JsfBeansAutoConfiguration$Jsf2_3AutoConfiguration$$EnhancerBySpringCGLIB$$573efe5b] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
2018-12-07 14:53:51.137  INFO 19904 --- [           main] trationDelegate$BeanPostProcessorChecker : Bean 'org.joinfaces.autoconfigure.javaxfaces.JsfBeansAutoConfiguration' of type [org.joinfaces.autoconfigure.javaxfaces.JsfBeansAutoConfiguration$$EnhancerBySpringCGLIB$$b6d8361] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
2018-12-07 14:53:51.714  INFO 19904 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2018-12-07 14:53:51.742  INFO 19904 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2018-12-07 14:53:51.743  INFO 19904 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet Engine: Apache Tomcat/9.0.12
2018-12-07 14:53:51.758  INFO 19904 --- [           main] o.a.catalina.core.AprLifecycleListener   : The APR based Apache Tomcat Native library which allows optimal performance in production environments was not found on the java.library.path: [C:\Program Files\Java\jdk1.8.0_181\bin;C:\Windows\Sun\Java\bin;C:\Windows\system32;C:\Windows;C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShell\v1.0\;C:\Windows\System32\OpenSSH\;C:\Go\bin;C:\Users\Codenotfound\AppData\Local\Microsoft\WindowsApps;C:\Program Files\Java\jdk1.8.0_181\bin;C:\Users\Codenotfound\tools\apache-maven-3.5.4\bin;C:\Users\Codenotfound\AppData\Local\GitHubDesktop\bin;C:\Users\Codenotfound\AppData\Local\atom\bin;C:\Users\Codenotfound\go\bin;C:\Users\Codenotfound\AppData\Local\Programs\Microsoft VS Code\bin;C:\Users\Codenotfound\AppData\Local\Programs\Git\cmd;;.]
2018-12-07 14:53:52.106  INFO 19904 --- [           main] org.apache.jasper.servlet.TldScanner     : At least one JAR was scanned for TLDs yet contained no TLDs. Enable debug logging for this logger for a complete list of JARs that were scanned but no TLDs were found in them. Skipping unneeded JARs during scanning can improve startup time and JSP compilation time.
2018-12-07 14:53:52.129  INFO 19904 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2018-12-07 14:53:52.129  INFO 19904 --- [           main] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 2647 ms
2018-12-07 14:53:52.259  INFO 19904 --- [           main] o.s.b.w.servlet.ServletRegistrationBean  : Servlet FacesServlet mapped to [/faces/*, *.jsf, *.faces, *.xhtml]
2018-12-07 14:53:52.261  INFO 19904 --- [           main] o.s.b.w.servlet.ServletRegistrationBean  : Servlet dispatcherServlet mapped to [/]
2018-12-07 14:53:52.268  INFO 19904 --- [           main] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'characterEncodingFilter' to: [/*]
2018-12-07 14:53:52.271  INFO 19904 --- [           main] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'hiddenHttpMethodFilter' to: [/*]
2018-12-07 14:53:52.271  INFO 19904 --- [           main] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'formContentFilter' to: [/*]
2018-12-07 14:53:52.272  INFO 19904 --- [           main] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'requestContextFilter' to: [/*]
2018-12-07 14:53:52.860  INFO 19904 --- [           main] org.reflections.Reflections              : Reflections took 508 ms to scan 6 urls, producing 758 keys and 4159 values
2018-12-07 14:53:53.313  INFO 19904 --- [           main] j.e.resource.webcontainer.jsf.config     : Initializing Mojarra 2.3.7 ( 20180822-0020 fb5578e991d03fa881315e4c7beb52869a5e664b) for context ''
2018-12-07 14:53:53.529  INFO 19904 --- [           main] j.e.r.webcontainer.jsf.application       : JSF1048: PostConstruct/PreDestroy annotations present.  ManagedBeans methods marked with these annotations will have said annotations processed.
2018-12-07 14:53:54.511  INFO 19904 --- [           main] .w.PostConstructApplicationEventListener : Running on PrimeFaces 6.2
2018-12-07 14:53:54.511  INFO 19904 --- [           main] .a.PostConstructApplicationEventListener : Running on PrimeFaces Extensions 6.2.9
2018-12-07 14:53:54.511  INFO 19904 --- [           main] o.omnifaces.VersionLoggerEventListener   : Using OmniFaces version 1.14.1
2018-12-07 14:53:54.746  INFO 19904 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2018-12-07 14:53:55.160  INFO 19904 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2018-12-07 14:53:55.160  INFO 19904 --- [           main] c.c.SpringPrimeFacesApplication          : Started SpringPrimeFacesApplication in 6.156 seconds (JVM running for 10.557)
{% endhighlight %}

Open a web browser and enter the following URL: [http://localhost:8080/](http://localhost:8080/){:target="_blank"}. The below web page should now be displayed.

<img src="{{ site.url }}/assets/images/jsf-primefaces/jsf-welcome-page-redirect.png" alt="jsf welcome page redirect">

This means that the redirect was successfully applied as otherwise a <var>'Whitelabel Error Page'</var> error would have been returned as shown below.

<img src="{{ site.url }}/assets/images/jsf-primefaces/jsf-welcome-page-redirect-error.png" alt="jsf welcome page redirect error">

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/jsf-primefaces/tree/master/jsf-welcome-page-redirect){:target="_blank"}.
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This concludes the short code sample on how to set up a JSF welcome file XHTML using Spring Boot.

Let me know if this example was helpful.

Thanks!
