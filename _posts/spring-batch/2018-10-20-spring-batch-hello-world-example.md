---
title: "Spring Batch Hello World Example"
permalink: /spring-batch-hello-world-example.html
excerpt: "A detailed step-by-step tutorial on how to implement a Hello World Spring Batch job using Spring Boot and Maven."
date: 2018-10-20
last_modified_at: 2018-11-05
header:
  teaser: "assets/images/spring-batch/spring-batch-hello-world-example.png"
categories: [Spring Batch]
tags: [Example, Maven, Spring Batch, Spring Boot, Tutorial]
published: true
---

<img src="{{ site.url }}/assets/images/spring-batch/spring-batch-hello-world-example.png" alt="spring batch hello world example" class="align-right title-image">

I’m going to show you exactly how to create a [Spring Batch](https://spring.io/projects/spring-batch){:target="_blank"} _Hello World_ example that uses [Maven](https://maven.apache.org/){:target="_blank"} and [Spring Boot](https://spring.io/projects/spring-boot){:target="_blank"}.

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

To show how Spring Batch works let's build a simple HelloWorld batch job.

In the example, we read a persons first and last name from a <var>person.csv</var> CSV file. From this data, a greeting is generated. This greeting is then written to a <var>greetings.txt</var> text file.

## General Project Setup

We will use the following tools/frameworks:
* _Spring Batch 4.1_
* _Spring Boot 2.1_
* _Maven 3.5_

Our project has the following directory structure:

<img src="{{ site.url }}/assets/images/spring-batch/spring-batch-hello-world-maven-project.png" alt="spring batch hello world maven project">

## Maven Setup

We build and run our example using **Maven**. If not already the case make sure to [download and install Apache Maven](https://downlinko.com/download-install-apache-maven-windows.html){:target="_blank"}.

Shown below is the XML representation of our Maven project. We define it in a <var>pom.xml</var> file. It contains the needed dependencies to compile and run the example.

To run the batch job, we will use **Spring Boot**. We also use [Spring Boot Starters](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters){:target="_blank"} so that we do not have to manage the different Spring dependencies.

The `spring-boot-starter-batch` starter imports the Spring Boot and Spring Batch dependencies.

The `spring-boot-starter-test` starter includes the dependencies for testing Spring Boot applications. It imports libraries that include [JUnit](http://junit.org/junit4/){:target="_blank"}, [Hamcrest](http://hamcrest.org/JavaHamcrest/){:target="_blank"} and [Mockito](https://site.mockito.org/){:target="_blank"}.

We also declare a dependency on `spring-batch-test`. This library contains some helper classes that will help test our batch job.

In the plugins section, we define the [Spring Boot Maven Plugin](https://docs.spring.io/spring-boot/docs/current/reference/html/build-tool-plugins-maven-plugin.html){:target="_blank"}: `spring-boot-maven-plugin`. This allows us to build a single, runnable "uber-jar". This is a convenient way to execute and transport our code. Also, the plugin allows us to start the example via a Maven command.

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
    <version>2.1.0.RELEASE</version>
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

Spring Batch by default uses a database to store metadata on the configured batch jobs.

In this example, we will **run Spring Batch without a database**. Instead, an in-memory `Map` based repository is used.

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

In our example, the input data is stored in a <var>src/test/resources/csv/persons.csv</var> file.

Each line in the file contains a comma-separated first and last name.

{% highlight plaintext %}
John, Doe
Jane, Doe
{% endhighlight %}

We will map this data to a `Person` object. This is a simple POJO that contains a first and last name.

{% highlight java %}
package com.codenotfound.model;

public class Person {
  private String firstName;
  private String lastName;

  public Person() {
    // default constructor
  }

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
    return firstName + " " + lastName;
  }
}
{% endhighlight %}

## Configuring the Spring Batch Job

Let's go ahead and configure our batch job.

We create a `HelloWorldJobConfig` class. The [@Configuration](https://docs.spring.io/spring/docs/5.1.0.RELEASE/spring-framework-reference/core.html#beans-java-basic-concepts){:target="_blank"} annotation at the top of the class indicates that Spring can use this class as a source of bean definitions.

We then add the [@EnableBatchProcessing](https://docs.spring.io/spring-batch/4.1.x/api/org/springframework/batch/core/configuration/annotation/EnableBatchProcessing.html){:target="_blank"} annotation which enables Spring Batch features. It also provides a base configuration for setting up batch jobs. As we disabled the auto-configuration of a `DataSource`, by default a `Map` based `JobRepository` is used.

> By adding this annotation a lot happens. Here is an overview of what `@EnableBatchProcessing` creates:

* a `JobRepository` (bean name "jobRepository")
* a `JobLauncher` (bean name "jobLauncher")
* a `JobRegistry` (bean name "jobRegistry")
* a `JobExplorer` (bean name "jobExplorer")
* a `PlatformTransactionManager` (bean name "transactionManager")
* a `JobBuilderFactory` (bean name "jobBuilders") as a convenience to prevent you from having to inject the job repository into every job
* a `StepBuilderFactory` (bean name "stepBuilders") as a convenience to prevent you from having to inject the job repository and transaction manager into every step

In the `helloWorlJob` Bean we use the `JobBuilderFactory` to create the job. We pass the name of the job and the step that needs to be run.

> Note that [Spring will automatically wire](https://docs.spring.io/spring/docs/5.1.0.RELEASE/spring-framework-reference/core.html#beans-autowired-annotation){:target="_blank"} the `jobBuilders` and `stepBuilders` Beans In the `helloWorlJob()` Bean.

The `helloWorldStep` Bean defines the different items our step executes. We use the `StepBuilderFactory` to create the step.

First, we pass the name of the step. Using `chunk()` we specify the number of items that are processed within each transaction. Chunk also specifies the input (`Person`) and output (`String`) type of the step. We then add the `ItemReader` (reader), `ItemProcessor` (processor), and `ItemWriter` (writer) to the step.

To read the person CSV file we use the [FlatFileItemReader](https://docs.spring.io/spring-batch/4.1.x/reference/html/readersAndWriters.html#flatFileItemReader){:target="_blank"}. This is a class that provides basic functionality to read and parse flat files.

There is a `FlatFileItemReaderBuilder` builder implementation that allows us to create a `FlatFileItemReader`. We start by specifying that the result of reading each line in the file is a `Person` object. We then add a name for the reader and the specify the resource (in this case the <var>persons.csv</var> file) that needs to be read.

In order for the `FlatFileItemReader` to process our file, we need to specify some extra information. First, we define that the data in the file is delimited (defaults to comma as its delimiter).

We also specify how each field on a line needs to be mapped to our `Person` object. This is done using `names()` that enables Spring Batch to map fields by matching a name with a setter on the object. So in our example, the first field of a line will be mapped using the <var>firstName</var> setter. For this to work, we also need to specify the target type which is a `Person` object.

The `PersonItemProcessor` handles the processing of the data. It converts a `Person` into a greeting `String`. We will define this in a separate class further below.

Once the data is processed we will write it to a text file. We use the [FlatFileItemWriter](https://docs.spring.io/spring-batch/4.1.x/reference/html/readersAndWriters.html#flatFileItemWriter){:target="_blank"} to help us with this task.

We use a `FlatFileItemWriterBuilder` builder implementation to create a `FlatFileItemWriter`. We add a name for the writer and specify the resource (in this case the <var>greeting.txt</var> file) to which data needs to be written.

The `FlatFileItemWriter` needs to know how to turn our generated output into a single string that can be written to a file. As in this example, our output is already a string we can use the [PassThroughLineAggregator](https://docs.spring.io/spring-batch/4.1.x/reference/html/readersAndWriters.html#PassThroughLineAggregator){:target="_blank"}. This is the most basic implementation, which assumes that the object is already a string.

{% highlight java %}
package com.codenotfound.batch.job;

import org.springframework.batch.core.Job;
import org.springframework.batch.core.Step;
import org.springframework.batch.core.configuration.annotation.EnableBatchProcessing;
import org.springframework.batch.core.configuration.annotation.JobBuilderFactory;
import org.springframework.batch.core.configuration.annotation.StepBuilderFactory;
import org.springframework.batch.item.file.FlatFileItemReader;
import org.springframework.batch.item.file.FlatFileItemWriter;
import org.springframework.batch.item.file.builder.FlatFileItemReaderBuilder;
import org.springframework.batch.item.file.builder.FlatFileItemWriterBuilder;
import org.springframework.batch.item.file.transform.PassThroughLineAggregator;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.ClassPathResource;
import org.springframework.core.io.FileSystemResource;
import com.codenotfound.model.Person;

@Configuration
@EnableBatchProcessing
public class HelloWorldJobConfig {

  @Bean
  public Job helloWorlJob(JobBuilderFactory jobBuilders, StepBuilderFactory stepBuilders) {
    return jobBuilders.get("helloWorldJob").start(helloWorldStep(stepBuilders)).build();
  }

  @Bean
  public Step helloWorldStep(StepBuilderFactory stepBuilders) {
    return stepBuilders.get("helloWorldStep").<Person, String>chunk(10).reader(reader())
        .processor(processor()).writer(writer()).build();
  }

  @Bean
  public FlatFileItemReader<Person> reader() {
    return new FlatFileItemReaderBuilder<Person>().name("personItemReader")
        .resource(new ClassPathResource("csv/persons.csv")).delimited()
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

## Processing the Data

In most cases, you will want to apply some data processing during a batch job. An [ItemProcessor](https://docs.spring.io/spring-batch/4.1.x/reference/html/readersAndWriters.html#itemProcessor){:target="_blank"} allows you to do just that.

In our example, we convert a `Person` object to a simple greeting `String`.

To do so, we create a `PersonItemProcessor` that implements the `ItemProcessor` interface. We implement the `process()` method which adds the first and last name of a person to a string.

For debugging purposes, we also log the result.

{% highlight java %}
package com.codenotfound.batch.job;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.batch.item.ItemProcessor;
import com.codenotfound.model.Person;

public class PersonItemProcessor implements ItemProcessor<Person, String> {

  private static final Logger LOGGER = LoggerFactory.getLogger(PersonItemProcessor.class);

  @Override
  public String process(Person person) throws Exception {
    String greeting = "Hello " + person.getFirstName() + " " + person.getLastName() + "!";

    LOGGER.info("converting '{}' into '{}'", person, greeting);
    return greeting;
  }
}
{% endhighlight %}

## Testing the Spring Batch Example

To wrap up our example we will create a basic unit test case. It will run our batch job and check if it finishes successfully.

We use the `@RunWith` and `@SpringBootTest` [testing annotations](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/reference/html/boot-features-testing.html#boot-features-testing-spring-boot-applications){:target="_blank"} to tell JUnit to run using Spring's testing support and bootstrap with Spring Boot's support.

Spring Batch ships with a `JobLauncherTestUtils` utility class for testing batch jobs.

We first create an inner `BatchTestConfig` class that adds our <var>helloWorld</var> job to a `JobLauncherTestUtils` bean. We then use the `launchJob()` method of this bean to run the batch job. If the job ran without any errors, the value of the exit code is <var>COMPLETED</var>.

{% highlight java %}
package com.codenotfound.batch;

import static org.assertj.core.api.Assertions.assertThat;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.batch.core.Job;
import org.springframework.batch.core.JobExecution;
import org.springframework.batch.core.launch.NoSuchJobException;
import org.springframework.batch.test.JobLauncherTestUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;
import org.springframework.test.context.junit4.SpringRunner;
import com.codenotfound.batch.job.HelloWorldJobConfig;

@RunWith(SpringRunner.class)
@SpringBootTest(classes = {SpringBatchApplicationTests.BatchTestConfig.class})
public class SpringBatchApplicationTests {

  @Autowired
  private JobLauncherTestUtils jobLauncherTestUtils;

  @Test
  public void testHelloWorldJob() throws Exception {
    JobExecution jobExecution = jobLauncherTestUtils.launchJob();
    assertThat(jobExecution.getExitStatus().getExitCode()).isEqualTo("COMPLETED");
  }

  @Configuration
  @Import(HelloWorldJobConfig.class)
  static class BatchTestConfig {

    @Autowired
    private Job helloWorlJob;

    @Bean
    JobLauncherTestUtils jobLauncherTestUtils() throws NoSuchJobException {
      JobLauncherTestUtils jobLauncherTestUtils = new JobLauncherTestUtils();
      jobLauncherTestUtils.setJob(helloWorlJob);

      return jobLauncherTestUtils;
    }
  }
}
{% endhighlight %}

To run above test case, open a command prompt in the projects root folder and execute following Maven command:

{% highlight plaintext %}
mvn test
{% endhighlight %}

The result is a successful build during which the batch job is executed.

{% highlight plaintext %}
.   ____          _            __ _ _
/\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
\\/  ___)| |_)| | | | | || (_| |  ) ) ) )
'  |____| .__|_| |_|_| |_\__, | / / / /
=========|_|==============|___/=/_/_/_/
:: Spring Boot ::        (v2.1.0.RELEASE)

2018-10-27 10:21:51.563  INFO 15792 --- [           main] c.c.batch.SpringBatchApplicationTests    : Starting SpringBatchApplicationTests on DESKTOP-2RB3C1U with PID 15792 (started by Codenotfound in C:\Users\Codenotfound\repos\spring-batch\spring-batch-hello-world)
2018-10-27 10:21:51.563  INFO 15792 --- [           main] c.c.batch.SpringBatchApplicationTests    : No active profile set, falling back to default profiles: default
2018-10-27 10:21:51.594  INFO 15792 --- [           main] s.c.a.AnnotationConfigApplicationContext : Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@4fb3ee4e: startup date [Sat Oct 27 10:21:51 CEST 2018]; root of context hierarchy
2018-10-27 10:21:52.188  INFO 15792 --- [           main] c.c.batch.SpringBatchApplicationTests    : Started SpringBatchApplicationTests in 0.937 seconds (JVM running for 1.87)
2018-10-27 10:21:52.297  WARN 15792 --- [           main] o.s.b.c.c.a.DefaultBatchConfigurer       : No datasource was provided...using a Map based JobRepository
2018-10-27 10:21:52.329  INFO 15792 --- [           main] o.s.b.c.l.support.SimpleJobLauncher      : No TaskExecutor has been set, defaulting to synchronous executor.
2018-10-27 10:21:52.376  INFO 15792 --- [           main] o.s.b.c.l.support.SimpleJobLauncher      : Job: [SimpleJob: [name=helloWorldJob]] launched with the following parameters: [{random=617281}]
2018-10-27 10:21:52.391  INFO 15792 --- [           main] o.s.batch.core.job.SimpleStepHandler     : Executing step: [helloWorldStep]
2018-10-27 10:21:52.438  INFO 15792 --- [           main] c.c.batch.job.PersonItemProcessor        : converting 'John Doe' into 'Hello John Doe!'
2018-10-27 10:21:52.438  INFO 15792 --- [           main] c.c.batch.job.PersonItemProcessor        : converting 'Jane Doe' into 'Hello Jane Doe!'
2018-10-27 10:21:52.454  INFO 15792 --- [           main] o.s.b.c.l.support.SimpleJobLauncher      : Job: [SimpleJob: [name=helloWorldJob]] completed with the following parameters: [{random=617281}] and the following status: [COMPLETED]
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 1.906 s - in com.codenotfound.batch.SpringBatchApplicationTests
2018-10-27 10:21:52.594  INFO 15792 --- [       Thread-1] s.c.a.AnnotationConfigApplicationContext : Closing org.springframework.context.annotation.AnnotationConfigApplicationContext@4fb3ee4e: startup date [Sat Oct 27 10:21:51 CEST 2018]; root of context hierarchy
[INFO]
[INFO] Results:
[INFO]
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 6.089 s
[INFO] Finished at: 2018-10-27T10:21:53+02:00
[INFO] ------------------------------------------------------------------------
{% endhighlight %}

You can find the result in the <var>target/test-outputs/greetings.txt</var> file:

{% highlight plaintext %}
Hello John Doe!
Hello Jane Doe!
{% endhighlight %}

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-batch/tree/master/spring-batch-hello-world){:target="_blank"}.
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

In this getting started tutorial you learned how to create a simple Spring Batch example with Spring Boot and Maven.

Let me know if you liked this post.

Leave a comment below.

Thanks!
