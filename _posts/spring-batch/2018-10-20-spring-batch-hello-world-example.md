---
title: "Spring Batch Hello World Example"
permalink: /spring-batch-hello-world-example.html
excerpt: "A detailed step-by-step tutorial on how to implement a Hello World Spring Batch job using Spring Boot and Maven."
date: 2018-10-20
last_modified_at: 2018-10-20
header:
  teaser: "assets/images/spring-batch/spring-batch-hello-world-example.png"
categories: [Spring Batch]
tags: [Example, Maven, Spring Batch, Spring Boot, Tutorial ]
published: true
---

<img src="{{ site.url }}/assets/images/spring-batch/spring-batch-hello-world-example.png" alt="spring batch hello world example" class="align-right title-image">

I’m going to show you exactly how to create a [Spring Batch](https://spring.io/projects/spring-batch){:target="_blank"} Hello World example that uses [Maven](https://maven.apache.org/){:target="_blank"} and [Spring Boot](https://spring.io/projects/spring-boot){:target="_blank"}.

(Step-by-step)

So if you’re a Spring Batch beginner, **you’ll love this guide**.

Ready?

## How Does the Spring Batch Framework Work?

Before we dive into the code let's look at the Spring Batch framework. It contains following key building blocks:

<img src="{{ site.url }}/assets/images/spring-batch/spring-batch-framework.png" alt="spring batch framework">

A batch process consists out of a `Job`. This is an entity that encapsulates the entire batch process.

A `Job` can consist out of one or more `Step`s. In most cases, a step will read data (`ItemReader`), process it (`ItemProcessor`) and then write it (`ItemWriter`).

The `JobLauncher` handles launching a `Job`.

And finally the `JobRepository` stores metadata about configured and executed `Job`s.

## General Project Setup

To show how Spring Batch works let's build a simple _Hello World_ batch job.

In the example, we read person data from a CSV file. From this data, a greeting is generated. This greeting is then written to a text file.

We will use the following tools/frameworks:
* Spring Batch 4.0
* Spring Boot 2.0
* Maven 3.5

Our Maven project has the following structure:

<img src="{{ site.url }}/assets/images/spring-batch/spring-batch-hello-world-maven-project.png" alt="spring batch hello world maven project">

## Maven Setup

We will build and run our example using **Maven**. If not already the case make sure to [download and install Apache Maven](https://downlinko.com/download-install-apache-maven-windows.html){:target="_blank"}.

Shown below is the XML representation of our Maven project. It is stored in the <var>pom.xml</var> file. It contains the needed dependencies to compile and run the example.

To run the batch job, we will use **Spring Boot**. We also use [Spring Boot Starters](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters){:target="_blank"} so that we do not have to manage the different Spring dependencies.

The `spring-boot-starter-batch` starter imports the Spring Boot and Spring Batch dependencies.

The `spring-boot-starter-test` starter includes the dependencies for testing Spring Boot applications. It imports libraries that include [JUnit](http://junit.org/junit4/){:target="_blank"}, [Hamcrest](http://hamcrest.org/JavaHamcrest/){:target="_blank"} and [Mockito](https://site.mockito.org/){:target="_blank"}.

We also declare a dependency on `spring-batch-test`. This library contains some helper classes that will help test our batch job.

In the plugins section, we define the [Spring Boot Maven Plugin](https://docs.spring.io/spring-boot/docs/current/reference/html/build-tool-plugins-maven-plugin.html){:target="_blank"}: `spring-boot-maven-plugin`. This allows us to build a single, runnable "uber-jar" which is a convenient way to execute and transport our code. Also, the plugin allows us to start the example via a Maven command.

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>spring-batch-hello-world</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>spring-batch-hello-world</name>
  <description>Spring Batch Hello World Example</description>
  <url>https://www.codenotfound.com/spring-batch-hello-world-example.html</url>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.6.RELEASE</version>
    <relativePath /> <!-- lookup parent from repository -->
  </parent>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>1.8</java.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-batch</artifactId>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework.batch</groupId>
      <artifactId>spring-batch-test</artifactId>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>

</project>
{% endhighlight %}

## Spring Boot Setup

We use Spring Boot so that we have a Spring Batch application that you can "just run". Start by creating a `SpringBatchApplication` class. It contains the `main()` method that uses Spring Boot's `SpringApplication.run()` to launch the application.

> Note that `@SpringBootApplication` is a convenience annotation that adds: `@Configuration`, `@EnableAutoConfiguration`, and `@ComponentScan`.

For more information on Spring Boot, check the [Spring Boot getting started guide](https://spring.io/guides/gs/spring-boot/){:target="_blank"}.

Spring Batch by default uses a database to store metadata on the configured batch jobs. In this example, we will **run Spring Batch without a database**. Instead, an in-memory `Map` based repository is used.

> For this to work we need to specify <var>exclude = {DataSourceAutoConfiguration.class}</var>. This prevents Spring Boot from auto-configuring a `DataSource` connection to a database.

{% highlight java %}
package com.codenotfound.batch;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration;

@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
public class SpringBatchApplication {

  public static void main(String[] args) {
    SpringApplication.run(SpringBatchApplication.class, args);
  }
}
{% endhighlight %}

## Creating the Model

Before you process data it is generally expected that you map it to a domain object.

In this example we will map the comma separated data from the CSV file to a `Person` object. This is a simple POJO that contains a first and last name.

{% highlight java %}
package com.codenotfound.model;

public class Person {
  private String firstName;
  private String lastName;

  public Person() {}

  public String getFirstName() {
    return firstName;
  }

  public void setFirstName(String firstName) {
    this.firstName = firstName;
  }

  public String getLastName() {
    return lastName;
  }

  public void setLastName(String lastName) {
    this.lastName = lastName;
  }

  @Override
  public String toString() {
    return "person[firstName=" + firstName + " ,lastName=" + lastName + "]";
  }
}
{% endhighlight %}

## Configuring the Spring Batch Job

Let's go ahead and configure our batch job.

We create a `HelloWorldJobConfig` class. The `@Configuration` annotation at the top of the class [indicates](https://docs.spring.io/spring/docs/5.1.0.RELEASE/spring-framework-reference/core.html#beans-java-basic-concepts){:target="_blank"} that Spring can use this class as a source of bean definitions.

We then add the `@EnableBatchProcessing` annotation which [enables](https://docs.spring.io/spring-batch/4.0.x/api/org/springframework/batch/core/configuration/annotation/EnableBatchProcessing.html){:target="_blank"} Spring Batch features. It also provides a base configuration for setting up batch jobs. As we disabled the auto-configuration of a `DataSource`, by default a Map based `JobRepository` is used.

By adding this annotation a lot happens. Here is an overview of what `@EnableBatchProcessing` creates:
* a `JobRepository` (bean name "jobRepository")
* a `JobLauncher` (bean name "jobLauncher")
* a `JobRegistry` (bean name "jobRegistry")
* a `JobExplorer` (bean name "jobExplorer")
* a `PlatformTransactionManager` (bean name "transactionManager")
* a `JobBuilderFactory` (bean name "jobBuilders") as a convenience to prevent you from having to inject the job repository into every job
* a `StepBuilderFactory` (bean name "stepBuilders") as a convenience to prevent you from having to inject the job repository and transaction manager into every step

We use the last two factories to create our job and the steps it consists of. [Autowire](https://docs.spring.io/spring/docs/5.1.0.RELEASE/spring-framework-reference/core.html#beans-autowired-annotation){:target="_blank"} both `jobBuilders` and `stepBuilders`.

In the `helloWorlJob()` bean we use the `JobBuilderFactory` to create the job. We pass the name of the job and the step that needs to be executed.

The `helloWorldStep()` bean defines the different items our step executes. We use the `StepBuilderFactory` to define the step.

First we pass the name of the step. Using `chunk()` we specify the number of items that are processed within each transaction. Chunk also specifies the input (`Person`) and output (`String`) type of the step. We then add the `ItemReader` (reader), `ItemProcessor` (processor), and `ItemWriter` (writer) to the step.

To read the person CSV file we use the [FlatFileItemReader](https://docs.spring.io/spring-batch/4.0.x/reference/html/readersAndWriters.html#flatFileItemReader){:target="_blank"}. This is a class that provides basic functionality to read and parse flat files.

There is a `FlatFileItemReaderBuilder` builder implementation that allows us to quickly create a `FlatFileItemReader`. We start by specifying that the result of reading each line in the file is a `Person` object. We then add a name for the reader and the resource (in this case the <var>persons.csv</var> file) that needs to be read.

In order for the `FlatFileItemReader` to process our file we need to specify some extra information. First we define that the data in the file is delimited (defaults to comma as its delimiter).

We also specify how each field on a line needs to be mapped to our `Person` object. This is done using `names()` that enables Spring Batch tp automatically map fields by matching a name with a setter on the object. So in our example the first field of a line will be mapped using the <var>firstName</var> setter. For this to work we also need to specify the target type which is a `Person` object.

The `PersonItemProcessor` handles the processing of the data. It is defined further below.

Once the data is processed we will write it to a text file. We use the [FlatFileItemWriter](https://docs.spring.io/spring-batch/4.0.x/reference/html/readersAndWriters.html#flatFileItemWriter){:target="_blank"} to help us with this task.

{% highlight java %}
package com.codenotfound.batch.job;

import org.springframework.batch.core.Job;
import org.springframework.batch.core.Step;
import org.springframework.batch.core.configuration.annotation.DefaultBatchConfigurer;
import org.springframework.batch.core.configuration.annotation.EnableBatchProcessing;
import org.springframework.batch.core.configuration.annotation.JobBuilderFactory;
import org.springframework.batch.core.configuration.annotation.StepBuilderFactory;
import org.springframework.batch.item.file.FlatFileItemReader;
import org.springframework.batch.item.file.FlatFileItemWriter;
import org.springframework.batch.item.file.builder.FlatFileItemReaderBuilder;
import org.springframework.batch.item.file.builder.FlatFileItemWriterBuilder;
import org.springframework.batch.item.file.transform.PassThroughLineAggregator;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.ClassPathResource;
import org.springframework.core.io.FileSystemResource;
import com.codenotfound.model.Person;

@Configuration
@EnableBatchProcessing
public class HelloWorldJobConfig {

  @Autowired
  public JobBuilderFactory jobBuilders;

  @Autowired
  public StepBuilderFactory stepBuilders;

  @Bean
  public Job helloWorlJob() {
    return jobBuilders.get("helloWorldJob").start(helloWorldStep()).build();
  }

  @Bean
  public Step helloWorldStep() {
    return stepBuilders.get("helloWorldStep").<Person, String>chunk(10).reader(reader())
        .processor(processor()).writer(writer()).build();
  }

  @Bean
  public FlatFileItemReader<Person> reader() {
    return new FlatFileItemReaderBuilder<Person>().name("personItemReader")
        .resource(new ClassPathResource("persons.csv")).delimited()
        .names(new String[] {"firstName", "lastName"}).targetType(Person.class).build();
  }

  @Bean
  public PersonItemProcessor processor() {
    return new PersonItemProcessor();
  }

  @Bean
  public FlatFileItemWriter<String> writer() {
    return new FlatFileItemWriterBuilder<String>().name("greetingItemWriter")
        .resource(new FileSystemResource("target/test-outputs/greetings.txt"))
        .lineAggregator(new PassThroughLineAggregator<>()).build();
  }
}
{% endhighlight %}
