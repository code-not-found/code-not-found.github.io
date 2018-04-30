---
title: "JSF - Primefaces DataTable Example"
permalink: /jsf-primefaces-datatable-example.html
excerpt: "A detailed step-by-step tutorial on how to implement a PrimeFaces DataTable using Spring Data JPA, Spring Boot, and Maven."
date: 2018-03-03
last_modified_at: 2018-03-03
header:
  teaser: "assets/images/teaser/primefaces-teaser.png"
categories: [PrimeFaces]
tags: [DataTable, Example, JavaServer Faces, JSF, Maven, PrimeFaces, Spring Data JPA, Spring Boot]
published: true
---

<figure>
  <img src="{{ site.url }}/assets/images/logo/primefaces-logo.png" alt="primefaces logo" class="logo">
</figure>

[PrimeFaces](https://www.primefaces.org/){:target="_blank"} ships with a <var>DataTable</var> component that displays data in tabular format.

The following example shows how to implement a PrimeFaces DataTable in combination with Spring Data JPA, Spring Boot, and Maven.

# General Project Setup

Tools used:
* PrimeFaces 6.2
* JoinFaces 3.2
* Spring Boot 2.0
* Spring Data JPA 2.0
* Maven 3.5

The example builds on a previous [Hello World Primefaces Tutorial]({{ site.url }}/jsf-primefaces-example-spring-boot-maven.html) in which we configured a basic PrimeFaces application to run on Spring Boot.

Instead of hard coding the data that will be displayed in the DataTable, we will fetch it from a [H2 database](http://www.h2database.com/html/main.html){:target="_blank"}. By adding the `h2` Maven dependency to the POM file, [Spring Boot will auto-configure an embedded H2 database](https://docs.spring.io/spring-boot/docs/2.0.1.RELEASE/reference/html/boot-features-sql.html#boot-features-embedded-database-support){:target="_blank"} that we can use.

In addition the [Spring Data JPA](https://projects.spring.io/spring-data-jpa/){:target="_blank"} project is used in order to reduce the amount of boilerplate code required to implement the data access layer for the persistence store (in this case H2). The `spring-boot-starter-data-jpa` includes `spring-data-jpa` and other needed dependencies.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>jsf-primefaces-datatable</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>jsf-primefaces-datatable</name>
  <description>JSF - PrimeFaces DataTable Example</description>
  <url>https://www.codenotfound.com/jsf-primefaces-datatable-example.html</url>

  <parent>
    <groupId>org.joinfaces</groupId>
    <artifactId>joinfaces-parent</artifactId>
    <version>3.2.0</version>
    <relativePath /> <!-- lookup parent from repository -->
  </parent>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>1.8</java.version>
  </properties>

  <dependencies>
    <!-- joinfaces -->
    <dependency>
      <groupId>org.joinfaces</groupId>
      <artifactId>primefaces-spring-boot-starter</artifactId>
    </dependency>
    <!-- spring-boot -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <!-- h2 -->
    <dependency>
      <groupId>com.h2database</groupId>
      <artifactId>h2</artifactId>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <!-- spring-boot-maven-plugin -->
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>
</project>
```

# Prepare the Database

Similar to the [PrimeFaces ShowCase example](https://www.primefaces.org/showcase/ui/data/datatable/basic.xhtml){:target="_blank"} we will be displaying different cars in the DataTable. The below class represents a car with a basic structure.

We will use the [Java Persistence API](http://www.oracle.com/technetwork/java/javaee/tech/persistence-jsp-140049.html){:target="_blank"} to define how the `Car` class maps to a relational database table.

The class is annotated with the `@Entity` annotation, which marks the class as an entity class. Typically, an entity represents a table in a relational database, and each entity instance corresponds to a row in that table.

So for the `Car` class, a table named <var>CAR</var> will be created with the `id`, `brand`, `year` and `color` fields as columns.

The `id` filed is annotated with `@Id` to mark it as a primary key. The `@GeneratedValue` annotation specifies that the primary key will be automatically generated.

``` java
package com.codenotfound.primefaces.model;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Entity(name = "Car")
public class Car {

  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long id;

  private String brand;

  private int year;

  private String color;

  public Car() {}

  public Car(Long id, String brand, int year, String color) {
    this.id = id;
    this.brand = brand;
    this.year = year;
    this.color = color;
  }

  public Long getId() {
    return id;
  }

  public void setId(Long id) {
    this.id = id;
  }

  public String getBrand() {
    return brand;
  }

  public void setBrand(String brand) {
    this.brand = brand;
  }

  public int getYear() {
    return year;
  }

  public void setYear(int year) {
    this.year = year;
  }

  public String getColor() {
    return color;
  }

  public void setColor(String color) {
    this.color = color;
  }
}
```

Next we create a [Spring Data Repository](https://docs.spring.io/spring-data/jpa/docs/2.0.6.RELEASE/reference/html/#repositories.core-concepts){:target="_blank"} which will auto-generate the implementation for our `Car` domain object. Simply extend the `JpaRepository` and pass the domain class to manage in addition to the id type of the domain class as type arguments.

We annotated the below interface with `@Repository` which is a marker for any class that fulfills the role or stereotype (also known as Data Access Object or DAO) of a repository. It is also a specialization of `@Component` annotation which means that Spring will automatically create a Bean for this class in case a component scan is performed. 

> As our `SpringPrimeFacesApplication` is annotated with `@SpringBootApplication` an implicit component scan is performed at startup.

``` java
package com.codenotfound.primefaces.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import com.codenotfound.primefaces.model.Car;

@Repository
public interface CarRepository extends JpaRepository<Car, Long> {
}
```

In addition to setting up the H2 database, [Spring Boot will also initialize it with data](https://docs.spring.io/spring-boot/docs/2.0.1.RELEASE/reference/html/howto-database-initialization.html#howto-initialize-a-database-using-spring-jdbc){:target="_blank"} if the needed script is found on the classpath.

Simply add an <var>data.sql</var> file under <var>src/main/resources/META-INF/resources</var> that contains the data to be inserted.

``` sql
INSERT INTO car (brand, year, color) VALUES
  ('Audi', 1992, 'Red'),
  ('Fiat', 2001, 'Red'),
  ('Mercedes', 1991, 'Brown'),
  ('Fiat', 1962, 'Black'),
  ('Renault', 1997, 'Brown'),
  ('Renault', 1967, 'Maroon'),
  ('Renault', 1986, 'Yellow'),
  ('BMW', 1970, 'Maroon'),
  ('Fiat', 1990, 'Silver'),
  ('Renault', 1972, 'Black');
```

#  Create the PrimeFaces DataTable Component

We create a <var>cars.xhtml</var> page under <var>src/main/resources/META-INF/resources</var>. It contains a <var>&lt;dataTable&gt;</var> element on which we specify the list of objects that need to be displayed using the <var>value</var> attribute. In this example we use the `cars` field on the `carsView` Managed Bean that we will create further below.

The <var>var</var> attribute specifies the name of the variable created by the data table that represents the current item in the value. We use this name in order to specify what field from the object needs to be displayed in each column.

For example in the first column we specify <var>#{car.id}</var> so that the `id` of the current `Car` object is shown.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"
  xmlns:h="http://java.sun.com/jsf/html"
  xmlns:p="http://primefaces.org/ui">

<h:head>
  <title>PrimeFaces DataTable Example</title>
</h:head>

<h:body>

  <p:dataTable var="car" value="#{carsView.cars}">
    <p:column headerText="Id">
      <h:outputText value="#{car.id}" />
    </p:column>

    <p:column headerText="Year">
      <h:outputText value="#{car.year}" />
    </p:column>

    <p:column headerText="Brand">
      <h:outputText value="#{car.brand}" />
    </p:column>

    <p:column headerText="Color">
      <h:outputText value="#{car.color}" />
    </p:column>
  </p:dataTable>

</h:body>
</html>
```

The only thing left to do is to create a Bean that can be used by the above JSF page in order to access to the `JpaRepository` so that data can be fetched from the database.

We create a `CarsView` class in which we auto-wire the `CarRepository`. The `cars` field is then initialized using the `findAll()` method on the repository which retrieves all available cars.

> Note that we use `@Named` instead of `@ManagedBean`. The reason for this is that [it is preferred to choose CDI to manage our beans](https://stackoverflow.com/a/4347707/4201470){:target="_blank"}. 

``` java
package com.codenotfound.primefaces.view;

import java.io.Serializable;
import java.util.List;

import javax.annotation.PostConstruct;
import javax.faces.view.ViewScoped;
import javax.inject.Inject;
import javax.inject.Named;

import com.codenotfound.primefaces.model.Car;
import com.codenotfound.primefaces.repository.CarRepository;

@Named
@ViewScoped
public class CarsView implements Serializable {

  private static final long serialVersionUID = 1L;

  @Inject
  private CarRepository carRepository;

  private List<Car> cars;

  @PostConstruct
  public void init() {
    cars = carRepository.findAll();
  }

  public List<Car> getCars() {
    return cars;
  }
}
```
# Testing the PrimeFaces DataTable Example

Let's test our PrimeFaces DataTable example by running following Maven command:

``` plaintext
mvn spring-boot:run
```

``` plaintext
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.0.1.RELEASE)

2018-04-30 21:16:35.699  INFO 2896 --- [           main] c.c.p.SpringPrimeFacesApplication        : Starting SpringPrimeFacesApplication on cnf-pc with PID 2896 (c:\blogs\codenotfound\code\jsf-primefaces\jsf-primefaces-datatable\target\classes started by CodeNotFound in c:\blogs\codenotfound\code\jsf-primefaces\jsf-primefaces-datatable)
2018-04-30 21:16:35.704  INFO 2896 --- [           main] c.c.p.SpringPrimeFacesApplication        : No active profile set, falling back to default profiles: default
2018-04-30 21:16:35.762  INFO 2896 --- [           main] ConfigServletWebServerApplicationContext : Refreshing org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@50b686dc: startup date [Mon Apr 30 21:16:35 CEST 2018]; root of context hierarchy
2018-04-30 21:16:37.178  INFO 2896 --- [           main] f.a.AutowiredAnnotationBeanPostProcessor : JSR-330 'javax.inject.Inject' annotation found and supported for autowiring
2018-04-30 21:16:37.314  INFO 2896 --- [           main] trationDelegate$BeanPostProcessorChecker : Bean 'org.springframework.transaction.annotation.ProxyTransactionManagementConfiguration' of type [org.springframework.transaction.annotation.ProxyTransactionManagementConfiguration$$EnhancerBySpringCGLIB$$2ed8ac49] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
2018-04-30 21:16:37.769  INFO 2896 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2018-04-30 21:16:37.797  INFO 2896 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2018-04-30 21:16:37.797  INFO 2896 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet Engine: Apache Tomcat/8.5.29
2018-04-30 21:16:37.809  INFO 2896 --- [ost-startStop-1] o.a.catalina.core.AprLifecycleListener   : The APR based Apache Tomcat Native library which allows optimal performance in production environments was not found on the java.library.path: [C:\Program Files\Java\jdk1.8.0_172\bin;C:\Windows\Sun\Java\bin;C:\Windows\system32;C:\Windows;C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShell\v1.0\;C:\Program Files\Java\jdk1.8.0_172\bin;C:\code\tools\apache-maven-3.5.3\bin;.]
2018-04-30 21:16:38.132  INFO 2896 --- [ost-startStop-1] org.apache.jasper.servlet.TldScanner     : At least one JAR was scanned for TLDs yet contained no TLDs. Enable debug logging for this logger for a complete list of JARs that were scanned but no TLDs were found in them. Skipping unneeded JARs during scanning can improve startup time and JSP compilation time.
2018-04-30 21:16:38.147  INFO 2896 --- [ost-startStop-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2018-04-30 21:16:38.147  INFO 2896 --- [ost-startStop-1] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 2388 ms
2018-04-30 21:16:38.411  INFO 2896 --- [ost-startStop-1] o.s.b.w.servlet.ServletRegistrationBean  : Servlet FacesServlet mapped to [/faces/*, *.jsf, *.faces, *.xhtml]
2018-04-30 21:16:38.413  INFO 2896 --- [ost-startStop-1] o.s.b.w.servlet.ServletRegistrationBean  : Servlet dispatcherServlet mapped to [/]
2018-04-30 21:16:38.419  INFO 2896 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'characterEncodingFilter' to: [/*]
2018-04-30 21:16:38.420  INFO 2896 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'hiddenHttpMethodFilter' to: [/*]
2018-04-30 21:16:38.420  INFO 2896 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'httpPutFormContentFilter' to: [/*]
2018-04-30 21:16:38.421  INFO 2896 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'requestContextFilter' to: [/*]
2018-04-30 21:16:38.421  INFO 2896 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'openEntityManagerInViewFilter' to: [/*]
2018-04-30 21:16:38.952  INFO 2896 --- [ost-startStop-1] org.reflections.Reflections              : Reflections took 455 ms to scan 6 urls, producing 755 keys and 4134 values
2018-04-30 21:16:39.263  INFO 2896 --- [ost-startStop-1] j.e.resource.webcontainer.jsf.config     : Initializing Mojarra 2.3.4 ( 20180405-0442 7a05fbdf38c7d725ccac9237e44de6dbc77bee7d) for context ''
2018-04-30 21:16:39.469  INFO 2896 --- [ost-startStop-1] j.e.r.webcontainer.jsf.application       : JSF1048: PostConstruct/PreDestroy annotations present.  ManagedBeans methods marked with these annotations will have said annotations processed.
2018-04-30 21:16:40.347  INFO 2896 --- [ost-startStop-1] .w.PostConstructApplicationEventListener : Running on PrimeFaces 6.2
2018-04-30 21:16:40.348  INFO 2896 --- [ost-startStop-1] .a.PostConstructApplicationEventListener : Running on PrimeFaces Extensions 6.2.4
2018-04-30 21:16:40.349  INFO 2896 --- [ost-startStop-1] o.omnifaces.VersionLoggerEventListener   : Using OmniFaces version 1.14.1
2018-04-30 21:16:40.504  INFO 2896 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
2018-04-30 21:16:40.683  INFO 2896 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
2018-04-30 21:16:40.745  INFO 2896 --- [           main] j.LocalContainerEntityManagerFactoryBean : Building JPA container EntityManagerFactory for persistence unit 'default'
2018-04-30 21:16:40.771  INFO 2896 --- [           main] o.hibernate.jpa.internal.util.LogHelper  : HHH000204: Processing PersistenceUnitInfo [        name: default        ...]
2018-04-30 21:16:40.881  INFO 2896 --- [           main] org.hibernate.Version                    : HHH000412: Hibernate Core {5.2.16.Final}
2018-04-30 21:16:40.883  INFO 2896 --- [           main] org.hibernate.cfg.Environment            : HHH000206: hibernate.properties not found
2018-04-30 21:16:40.934  INFO 2896 --- [           main] o.hibernate.annotations.common.Version   : HCANN000001: Hibernate Commons Annotations {5.0.1.Final}
2018-04-30 21:16:41.066  INFO 2896 --- [           main] org.hibernate.dialect.Dialect            : HHH000400: Using dialect: org.hibernate.dialect.H2Dialect
2018-04-30 21:16:41.654  INFO 2896 --- [           main] o.h.t.schema.internal.SchemaCreatorImpl  : HHH000476: Executing import script 'org.hibernate.tool.schema.internal.exec.ScriptSourceInputNonExistentImpl@76d67cbf'
2018-04-30 21:16:41.658  INFO 2896 --- [           main] j.LocalContainerEntityManagerFactoryBean : Initialized JPA EntityManagerFactory for persistence unit 'default'
2018-04-30 21:16:41.790  INFO 2896 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**/favicon.ico] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2018-04-30 21:16:42.118  INFO 2896 --- [           main] o.s.jdbc.datasource.init.ScriptUtils     : Executing SQL script from URL [file:/c:/blogs/codenotfound/code/jsf-primefaces/jsf-primefaces-datatable/target/classes/data.sql]
2018-04-30 21:16:42.124  INFO 2896 --- [           main] o.s.jdbc.datasource.init.ScriptUtils     : Executed SQL script from URL [file:/c:/blogs/codenotfound/code/jsf-primefaces/jsf-primefaces-datatable/target/classes/data.sql] in 5 ms.
2018-04-30 21:16:42.431  INFO 2896 --- [           main] s.w.s.m.m.a.RequestMappingHandlerAdapter : Looking for @ControllerAdvice: org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@50b686dc: startup date [Mon Apr 30 21:16:35 CEST 2018]; root of context hierarchy
2018-04-30 21:16:42.510  INFO 2896 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/error]}" onto public org.springframework.http.ResponseEntity<java.util.Map<java.lang.String, java.lang.Object>> org.springframework.boot.autoconfigure.web.servlet.error.BasicErrorController.error(javax.servlet.http.HttpServletRequest)
2018-04-30 21:16:42.512  INFO 2896 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/error],produces=[text/html]}" onto public org.springframework.web.servlet.ModelAndView org.springframework.boot.autoconfigure.web.servlet.error.BasicErrorController.errorHtml(javax.servlet.http.HttpServletRequest,javax.servlet.http.HttpServletResponse)
2018-04-30 21:16:42.547  INFO 2896 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/webjars/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2018-04-30 21:16:42.547  INFO 2896 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2018-04-30 21:16:43.001  INFO 2896 --- [           main] o.s.j.e.a.AnnotationMBeanExporter        : Registering beans for JMX exposure on startup
2018-04-30 21:16:43.004  INFO 2896 --- [           main] o.s.j.e.a.AnnotationMBeanExporter        : Bean with name 'dataSource' has been autodetected for JMX exposure
2018-04-30 21:16:43.010  INFO 2896 --- [           main] o.s.j.e.a.AnnotationMBeanExporter        : Located MBean 'dataSource': registering with JMX server as MBean [com.zaxxer.hikari:name=dataSource,type=HikariDataSource]
2018-04-30 21:16:43.057  INFO 2896 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2018-04-30 21:16:43.060  INFO 2896 --- [           main] c.c.p.SpringPrimeFacesApplication        : Started SpringPrimeFacesApplication in 7.689 seconds (JVM running for 11.282)
```

Once Spring Boot has started, open a web browser and enter the following URL: [http://localhost:8080/cars.xhtml](http://localhost:8080/cars.xhtml){:target="_blank"}.

A list of different cars should be rendered as shown below.

<figure>
  <img src="{{ site.url }}/assets/images/posts/jsf-primefaces/jsf-primefaces-datatable-example.png" alt="jsf primefaces datatable example">
</figure>

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/jsf-primefaces/tree/master/jsf-primefaces-datatable){:target="_blank"}.
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

Displaying the content of a database table using the PrimeFaces DataTable component can be quickly achieved using Spring Data an Spring Boot.

Hope you enjoyed this post. Drop a line in case you have some questions or would like to see another example.
