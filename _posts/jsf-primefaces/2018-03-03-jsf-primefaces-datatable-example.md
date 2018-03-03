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
* Spring Data JPA 1.11
* Spring Boot 1.5
* PrimeFaces 6.1
* JoinFaces 2.4
* Maven 3.5

The example is based on a previous [Hello World Primefaces Tutorial]({{ site.url }}/jsf-primefaces-example-spring-boot-maven.html).

Instead of hard coding the data that will be displayed in the DataTable, we will fetch it from a [H2 database](http://www.h2database.com/html/main.html){:target="_blank"}. By adding the `h2` Maven dependency to the POM file, [Spring Boot will auto-configure an embedded H2 database](https://docs.spring.io/spring-boot/docs/1.5.7.RELEASE/reference/html/boot-features-sql.html#boot-features-embedded-database-support){:target="_blank"} that we can use.

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
    <artifactId>jsf-spring-boot-parent</artifactId>
    <version>2.4.1</version>
    <relativePath /> <!-- lookup parent from repository -->
  </parent>

  <properties>
    <java.version>1.8</java.version>
  </properties>

  <dependencies>
    <!-- spring-boot -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <!-- h2 -->
    <dependency>
      <groupId>com.h2database</groupId>
      <artifactId>h2</artifactId>
    </dependency>
    <!-- joinfaces -->
    <dependency>
      <groupId>org.joinfaces</groupId>
      <artifactId>jsf-spring-boot-starter</artifactId>
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

We will use the [Java Persistence API](http://www.oracle.com/technetwork/java/javaee/tech/persistence-jsp-140049.html){:target="_blank"} to define how the `Car` class maps to a H2 relational database table.

The `@Entity` annotation, which marks the class as an entity class. Typically, an entity represents a table in a relational database, and each entity instance corresponds to a row in that table. So for the `Car` class, a table named <var>CAR</var> will be created with the `id`, `brand`, `year` and `color` fields as columns.

The `id` filed is annotated with `@Id` to marks it as a primary key. The @GeneratedValue annotation specifies that the primary key will be automatically generated.

``` java
package com.codenotfound.primefaces.model;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

@Entity(name = "Car")
public class Car {

  @Id
  @GeneratedValue
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

Next we create a [Spring Data Repository](https://docs.spring.io/spring-data/jpa/docs/1.11.7.RELEASE/reference/html/#repositories.core-concepts){:target="_blank"} which will auto-generate the implementation for our `Car` domain object. Simply extend the `JpaRepository` and pass the domain class to manage as well as the id type of the domain class as type arguments.

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

In addition to setting up the H2 database, [Spring Boot will also initialize it with data](https://docs.spring.io/spring-boot/docs/1.5.7.RELEASE/reference/html/howto-database-initialization.html#howto-initialize-a-database-using-spring-jdbc){:target="_blank"} if the needed script is found on the classpath. Simply add an <var>data.sql</var> file under <var>src/main/resources/META-INF/resources</var> that contains the data to be inserted.

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

The <var>var</var> attribute specifies the name of the variable created by the data table that represents the current item in the value. We use this name in order to specify what field from the object needs to be displayed in each column. For example in the first column we specify <var>#{car.id}</var> so that the `id` of the current `Car` object is shown.

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

The only thing left to do is to create a Bean that can be accessed by the above JSF page in order to access to the `JpaRepository` so that data can be fetched from the database.

We create a `CarsView` class in which we auto-wire the `CarRepository`. The `cars` field is then initialized using the `findAll()` method on the repository which retrieves all available cars.

> Note that we use `@Named` instead of `@ManagedBean`. The reason for this is that [it is preferred to choose one framework to manage our beans](https://stackoverflow.com/a/18388289/4201470){:target="_blank"}. 

``` java
package com.codenotfound.primefaces.view;

import java.io.Serializable;
import java.util.List;

import javax.annotation.PostConstruct;
import javax.faces.bean.ViewScoped;
import javax.inject.Named;

import org.springframework.beans.factory.annotation.Autowired;

import com.codenotfound.primefaces.model.Car;
import com.codenotfound.primefaces.repository.CarRepository;

@Named
@ViewScoped
public class CarsView implements Serializable {

  private static final long serialVersionUID = 1L;

  @Autowired
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

Once Spring Boot has started, open a web browser and enter the following URL: [http://localhost:9090/codenotfound/cars.xhtml](http://localhost:9090/codenotfound/cars.xhtml){:target="_blank"}.

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
