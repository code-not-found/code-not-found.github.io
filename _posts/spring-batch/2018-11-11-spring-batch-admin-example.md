---
title: "Spring Batch Admin Example"
permalink: /spring-batch-admin-example.html
excerpt: "A detailed step-by-step tutorial on how to use a Spring Boot admin UI to manage Spring Batch jobs."
date: 2018-11-11
last_modified_at: 2018-11-11
header:
  teaser: "assets/images/spring-batch/spring-batch-admin.png"
categories: [Spring Batch]
tags: [Admin, Example, Maven, Spring Batch, Spring Boot, Tutorial, UI]
published: true
---

<img src="{{ site.url }}/assets/images/spring-batch/spring-batch-admin.png" alt="spring batch admin" class="align-right title-image">

Looking for a [Spring Batch Admin](https://docs.spring.io/spring-batch-admin/trunk/){:target="_blank"} UI tutorial?

You might be surprised to learn it is no longer supported.

But don't worry as in this post **I'll show you the recommended replacement**. And how to set it up.

Let's dive right in.

## What is Spring Batch Admin?

Spring Batch Admin provides a web-based user interface (UI) that allows you to manage [Spring Batch](https://spring.io/projects/spring-batch){:target="_blank"} jobs. The project, however, is **end-of-life since December 31, 2017**.

[Spring Cloud Data Flow](https://cloud.spring.io/spring-cloud-dataflow/){:target="_blank"} is now the [recommended replacement](https://github.com/spring-projects/spring-batch-admin#note-this-project-is-being-moved-to-the-spring-attic-and-is-not-recommended-for-new-projects--spring-cloud-data-flow-is-the-recommended-replacement-for-managing-and-monitoring-spring-batch-jobs-going-forward--you-can-read-more-about-migrating-to-spring-cloud-data-flow-here){:target="_blank"} for managing and monitoring Spring Batch jobs. It is one of the [main Spring Projects](https://spring.io/projects){:target="_blank"}.

Let's show how you can configure Spring Cloud Data Flow to run a batch job.

We re-use the [Spring Batch capitalize names](https://github.com/code-not-found/spring-batch/tree/master/spring-batch-capitalize-names){:target="_blank"} project. It contains a batch job that converts person names from lower case into upper case.

We then start a Spring Cloud Data Flow server and configure the batch job. Using the web-based UI we launch the job and check the status.

## General Project Setup

We will use the following tools/frameworks:
* _Spring Batch 3.0_
* _Spring Boot 1.5_
* _Maven 3.5_

> [Spring Cloud Data Flow](https://github.com/spring-cloud/spring-cloud-dataflow){:target="_blank"} does not yet support Spring Boot 2.0. That is why this example uses Spring Boot 1.5 and Spring Batch 3.0.

We will create two Maven projects (one for the batch job and one for Spring Cloud Data Flow) that have the following structure:

<img src="{{ site.url }}/assets/images/spring-batch/spring-batch-admin-maven-projects.png" alt="spring batch admin maven projects">

## Creating a Spring Batch Task

Spring Cloud Data Flow is **a toolkit for building data processing pipelines**. The pipelines consist of Spring Boot applications. This means we can run a Spring Boot batch job using a Data Flow server.

All we need to do is annotate our existing `SpringBatchApplication` with `@EnableTask` as shown below.

This class-level annotation tells Spring Cloud Task to bootstrap its functionality. It enables a `TaskConfigurer` that registers the application in a `TaskRepository`.

Spring Cloud Task will also [associate the execution of a batch job with a task's execution](https://docs.spring.io/spring-cloud-task/docs/1.3.0.RELEASE/reference/htmlsingle/#batch-association){:target="_blank"} so that one can be traced back to the other. This association is by default in any context that has both a Spring Batch Job configured and the `spring-cloud-task-batch` JAR available within the classpath.

We can now define a [Spring Cloud Task](https://spring.io/projects/spring-cloud-task){:target="_blank"} as we will see in the next section.

{% highlight xml %}
package com.codenotfound.batch;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.task.configuration.EnableTask;

@EnableTask
@SpringBootApplication
public class SpringBatchApplication {

  public static void main(String[] args) {
    SpringApplication.run(SpringBatchApplication.class, args);
  }
}
{% endhighlight %}

Spring Cloud Task uses a datasource for storing the results of task executions. When running on Spring Cloud Data Flow we need to make sure that [common datasource settings are shared among both](http://docs.spring.io/spring-cloud-dataflow/docs/1.7.0.RELEASE/reference/htmlsingle/#_task_database_configuration){:target="_blank"}.

By default Spring Cloud Data Flow uses an in-memory instance of [H2](http://www.h2database.com/html/main.html){:target="_blank"} with the following URL: <var>jdbc:h2:tcp://localhost:19092/mem:dataflow</var>.

We set the same datasource on our Spring Batch Task using the <var>application.yml</var> properties file located in <var>src/main/resources</var>.

{% highlight yml %}
spring:
  datasource:
    url: jdbc:h2:tcp://localhost:19092/mem:dataflow
    driverClassName: org.h2.Driver
{% endhighlight %}

To enable the above configuration changes we need to add extra dependencies in the Maven <var>pom.xml</var> file.

The `spring-cloud-starter-task` starter includes the dependencies for testing Spring Boot applications. It imports libraries that include [JUnit](http://junit.org/junit4/){:target="_blank"}, [Hamcrest](http://hamcrest.org/JavaHamcrest/){:target="_blank"} and [Mockito](https://site.mockito.org/){:target="_blank"}.

We also declare a dependency on `h2`. Spring Boot will take care of the [auto-configuration of the datasource](https://docs.spring.io/spring-boot/docs/1.5.7.RELEASE/reference/html/using-boot-auto-configuration.html){:target="_blank"} when it finds the H2 library on the classpath.

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>spring-batch-task</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>spring-batch-task</name>
  <description>Spring Batch Task Example</description>
  <url>https://www.codenotfound.com/spring-batch-admin-example.html</url>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.17.RELEASE</version>
    <relativePath /> <!-- lookup parent from repository -->
  </parent>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>1.8</java.version>
    <spring-cloud-starter-task.version>1.3.0.RELEASE</spring-cloud-starter-task.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-batch</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-task</artifactId>
      <version>${spring-cloud-starter-task.version}</version>
    </dependency>

    <dependency>
      <groupId>com.h2database</groupId>
      <artifactId>h2</artifactId>
      <scope>runtime</scope>
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

The last thing left to do is to package our Spring Batch Task application as a Spring Boot JAR.

Open a command prompt and navigate to the <var>spring-batch-task</var> project. Execute following Maven command:

{% highlight plaintext %}
mvn install
{% endhighlight %}

The result is a <var>spring-batch-task-0.0.1-SNAPSHOT.jar</var> JAR file in the <var>spring-batch-task/target</var> directory:

<img src="{{ site.url }}/assets/images/spring-batch/spring-batch-task-jar.png" alt="spring batch task jar">

## Running a Spring Cloud Data Flow Server

In this example, we will run Spring Cloud Data Flow on a local server.

Create a new <var>spring-cloud-data-flow-server</var> Maven project.

The `spring-boot-starter` starter in the <var>pom.xml</var> will import the needed Spring Boot dependencies.

`spring-cloud-dataflow-server-local` takes care of the Spring Cloud Data Flow dependencies.

> We also specify the <var>hibernate.version</var> as otherwise a `java.lang.reflect.InvocationTargetException is` thrown.

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.codenotfound</groupId>
  <artifactId>spring-cloud-data-flow-server</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>spring-cloud-data-flow-server</name>
  <description>Spring Cloud Data Server on Spring Boot</description>
  <url>https://www.codenotfound.com/spring-batch-admin-example.html</url>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.17.RELEASE</version>
    <relativePath /> <!-- lookup parent from repository -->
  </parent>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>1.8</java.version>

    <spring-cloud-dataflow.version>1.7.1.RELEASE</spring-cloud-dataflow.version>
    <hibernate.version>5.3.7.FINAL</hibernate.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-dataflow-server-local</artifactId>
      <version>${spring-cloud-dataflow.version}</version>
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

Add the `@EnableDataFlowServer` annotation to the Spring Boot main class. This activates a Spring Cloud Data Flow Server implementation.

{% highlight xml %}
package com.codenotfound.batch;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.dataflow.server.EnableDataFlowServer;

@EnableDataFlowServer
@SpringBootApplication
public class SpringDataFlowServerApplication {

  public static void main(String[] args) {
    SpringApplication.run(SpringDataFlowServerApplication.class, args);
  }
}
{% endhighlight %}

That's it. We can now start our Local Data Flow Server.

Fire up a command prompt in the <var>spring-cloud-data-flow-server</var> project directory. Execute following Maven command:

{% highlight plaintext %}
mvn spring-boot:run
{% endhighlight %}

The application will boot up as shown below.

{% highlight plaintext %}
____                              ____ _                __
/ ___| _ __  _ __(_)_ __   __ _   / ___| | ___  _   _  __| |
\___ \| '_ \| '__| | '_ \ / _` | | |   | |/ _ \| | | |/ _` |
___) | |_) | |  | | | | | (_| | | |___| | (_) | |_| | (_| |
|____/| .__/|_|  |_|_| |_|\__, |  \____|_|\___/ \__,_|\__,_|
____ |_|    _          __|___/                 __________
|  _ \  __ _| |_ __ _  |  ___| | _____      __  \ \ \ \ \ \
| | | |/ _` | __/ _` | | |_  | |/ _ \ \ /\ / /   \ \ \ \ \ \
| |_| | (_| | || (_| | |  _| | | (_) \ V  V /    / / / / / /
|____/ \__,_|\__\__,_| |_|   |_|\___/ \_/\_/    /_/_/_/_/_/



2018-11-11 11:38:49.831  INFO 7516 --- [           main] c.c.c.ConfigServicePropertySourceLocator : Fetching config from server at: http://localhost:8888
2018-11-11 11:38:50.975  WARN 7516 --- [           main] c.c.c.ConfigServicePropertySourceLocator : Could not locate PropertySource: I/O error on GET request for "http://localhost:8888/spring-cloud-dataflow-server-local/default": Connection refused: connect; nested exception is java.net.ConnectException: Connection refused: connect
2018-11-11 11:38:50.975  INFO 7516 --- [           main] c.c.b.SpringDataFlowServerApplication    : No active profile set, falling back to default profiles: default
2018-11-11 11:38:52.224  INFO 7516 --- [           main] .s.d.r.c.RepositoryConfigurationDelegate : Multiple Spring Data modules found, entering strict repository configuration mode!
2018-11-11 11:38:52.251  INFO 7516 --- [           main] .s.d.r.c.RepositoryConfigurationDelegate : Multiple Spring Data modules found, entering strict repository configuration mode!
2018-11-11 11:38:52.382  INFO 7516 --- [           main] .s.d.r.c.RepositoryConfigurationDelegate : Multiple Spring Data modules found, entering strict repository configuration mode!
2018-11-11 11:38:52.853  INFO 7516 --- [           main] o.s.cloud.context.scope.GenericScope     : BeanFactory id=4515019a-2b81-3169-9bfb-ef940568dc6f
2018-11-11 11:38:53.970  INFO 7516 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat initialized with port(s): 9393 (http)
2018-11-11 11:38:54.001  INFO 7516 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2018-11-11 11:38:54.001  INFO 7516 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet Engine: Apache Tomcat/8.5.34
2018-11-11 11:38:54.131  INFO 7516 --- [ost-startStop-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2018-11-11 11:38:54.584  INFO 7516 --- [ost-startStop-1] o.s.c.d.s.config.web.WebConfiguration    : Start Embedded H2
2018-11-11 11:38:54.584  INFO 7516 --- [ost-startStop-1] o.s.c.d.s.config.web.WebConfiguration    : Starting H2 Server with URL: jdbc:h2:tcp://localhost:19092/mem:dataflow
2018-11-11 11:38:55.081  INFO 7516 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Started.
2018-11-11 11:38:55.184  INFO 7516 --- [           main] j.LocalContainerEntityManagerFactoryBean : Building JPA container EntityManagerFactory for persistence unit 'default'
2018-11-11 11:38:55.213  INFO 7516 --- [           main] o.hibernate.jpa.internal.util.LogHelper  : HHH000204: Processing PersistenceUnitInfo [
      name: default
      ...]
2018-11-11 11:38:55.692  INFO 7516 --- [           main] org.hibernate.Version                    : HHH000412: Hibernate Core {5.3.7.Final}
2018-11-11 11:38:55.694  INFO 7516 --- [           main] org.hibernate.cfg.Environment            : HHH000206: hibernate.properties not found
2018-11-11 11:38:55.923  INFO 7516 --- [           main] o.hibernate.annotations.common.Version   : HCANN000001: Hibernate Commons Annotations {5.0.4.Final}
2018-11-11 11:38:56.136  INFO 7516 --- [           main] org.hibernate.dialect.Dialect            : HHH000400: Using dialect: org.hibernate.dialect.H2Dialect
2018-11-11 11:38:57.011  INFO 7516 --- [           main] j.LocalContainerEntityManagerFactoryBean : Initialized JPA EntityManagerFactory for persistence unit 'default'
2018-11-11 11:38:57.664  INFO 7516 --- [           main] o.s.b.c.r.s.JobRepositoryFactoryBean     : No database type set, using meta data indicating: H2
2018-11-11 11:38:57.841  INFO 7516 --- [           main] o.s.jdbc.datasource.init.ScriptUtils     : Executing SQL script from class path resource [org/springframework/batch/core/schema-h2.sql]
2018-11-11 11:38:57.876  INFO 7516 --- [           main] o.s.jdbc.datasource.init.ScriptUtils     : Executed SQL script from class path resource [org/springframework/batch/core/schema-h2.sql] in 33 ms.
2018-11-11 11:38:57.883  INFO 7516 --- [           main] o.s.jdbc.datasource.init.ScriptUtils     : Executing SQL script from class path resource [org/springframework/cloud/task/schema-h2.sql]
2018-11-11 11:38:57.891  INFO 7516 --- [           main] o.s.jdbc.datasource.init.ScriptUtils     : Executed SQL script from class path resource [org/springframework/cloud/task/schema-h2.sql] in 8 ms.
2018-11-11 11:38:58.610  INFO 7516 --- [           main] o.s.b.a.s.SimpleJobServiceFactoryBean    : No database type set, using meta data indicating: H2
2018-11-11 11:38:59.151  INFO 7516 --- [           main] b.a.s.AuthenticationManagerConfiguration :

Using default security password: c99c3621-2859-4028-9173-d93b6697ea5a

2018-11-11 11:38:59.365  INFO 7516 --- [           main] o.s.c.d.s.r.s.DataflowRdbmsInitializer   : Adding dataflow schema classpath:schemas/h2/common.sql for h2 database
2018-11-11 11:38:59.365  INFO 7516 --- [           main] o.s.c.d.s.r.s.DataflowRdbmsInitializer   : Adding dataflow schema classpath:schemas/h2/streams.sql for h2 database
2018-11-11 11:38:59.365  INFO 7516 --- [           main] o.s.c.d.s.r.s.DataflowRdbmsInitializer   : Adding dataflow schema classpath:schemas/h2/tasks.sql for h2 database
2018-11-11 11:38:59.365  INFO 7516 --- [           main] o.s.c.d.s.r.s.DataflowRdbmsInitializer   : Adding dataflow schema classpath:schemas/h2/deployment.sql for h2 database
2018-11-11 11:38:59.365  INFO 7516 --- [           main] o.s.c.d.s.r.s.DataflowRdbmsInitializer   : Adding dataflow schema classpath:schemas/h2/jpa.sql for h2 database
2018-11-11 11:38:59.371  INFO 7516 --- [           main] o.s.jdbc.datasource.init.ScriptUtils     : Executing SQL script from class path resource [schemas/h2/common.sql]
2018-11-11 11:38:59.371  INFO 7516 --- [           main] o.s.jdbc.datasource.init.ScriptUtils     : Executed SQL script from class path resource [schemas/h2/common.sql] in 0 ms.
2018-11-11 11:38:59.371  INFO 7516 --- [           main] o.s.jdbc.datasource.init.ScriptUtils     : Executing SQL script from class path resource [schemas/h2/streams.sql]
2018-11-11 11:38:59.371  INFO 7516 --- [           main] o.s.jdbc.datasource.init.ScriptUtils     : Executed SQL script from class path resource [schemas/h2/streams.sql] in 0 ms.
2018-11-11 11:38:59.371  INFO 7516 --- [           main] o.s.jdbc.datasource.init.ScriptUtils     : Executing SQL script from class path resource [schemas/h2/tasks.sql]
2018-11-11 11:38:59.381  INFO 7516 --- [           main] o.s.jdbc.datasource.init.ScriptUtils     : Executed SQL script from class path resource [schemas/h2/tasks.sql] in 10 ms.
2018-11-11 11:38:59.386  INFO 7516 --- [           main] o.s.jdbc.datasource.init.ScriptUtils     : Executing SQL script from class path resource [schemas/h2/deployment.sql]
2018-11-11 11:38:59.386  INFO 7516 --- [           main] o.s.jdbc.datasource.init.ScriptUtils     : Executed SQL script from class path resource [schemas/h2/deployment.sql] in 0 ms.
2018-11-11 11:38:59.386  INFO 7516 --- [           main] o.s.jdbc.datasource.init.ScriptUtils     : Executing SQL script from class path resource [schemas/h2/jpa.sql]
2018-11-11 11:38:59.391  INFO 7516 --- [           main] o.s.jdbc.datasource.init.ScriptUtils     : Executed SQL script from class path resource [schemas/h2/jpa.sql] in 5 ms.
2018-11-11 11:38:59.592  WARN 7516 --- [           main] c.n.c.sources.URLConfigurationSource     : No URLs will be polled as dynamic configuration sources.
2018-11-11 11:38:59.592  INFO 7516 --- [           main] c.n.c.sources.URLConfigurationSource     : To enable URLs as dynamic configuration sources, define System property archaius.configurationSource.additionalUrls or make config.properties available on classpath.
2018-11-11 11:38:59.602  WARN 7516 --- [           main] c.n.c.sources.URLConfigurationSource     : No URLs will be polled as dynamic configuration sources.
2018-11-11 11:38:59.602  INFO 7516 --- [           main] c.n.c.sources.URLConfigurationSource     : To enable URLs as dynamic configuration sources, define System property archaius.configurationSource.additionalUrls or make config.properties available on classpath.
2018-11-11 11:39:00.423  INFO 7516 --- [           main] o.s.l.c.support.AbstractContextSource    : Property 'userDn' not set - anonymous context will be used for read-write operations
2018-11-11 11:39:00.683  INFO 7516 --- [           main] o.s.s.c.ThreadPoolTaskScheduler          : Initializing ExecutorService 'taskScheduler'
2018-11-11 11:39:01.702  INFO 7516 --- [           main] o.s.c.d.s.c.support.MetricStore          : Metrics Collector URI = []
2018-11-11 11:39:02.332  INFO 7516 --- [           main] ration$HystrixMetricsPollerConfiguration : Starting poller
2018-11-11 11:39:02.402  INFO 7516 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat started on port(s): 9393 (http)
2018-11-11 11:39:02.412  INFO 7516 --- [           main] c.c.b.SpringDataFlowServerApplication    : Started SpringDataFlowServerApplication in 14.471 seconds (JVM running for 19.197)
{% endhighlight %}

Open a web browser and navigate to [http://localhost:9393/dashboard](http://localhost:9393/dashboard){:target="_blank"}.

This will open the Spring Cloud Data Flow dashboard as shown below.

Now it's time to add our Spring Batch Task. Click on the <var>Add Application(s)</var> button.

<img src="{{ site.url }}/assets/images/spring-batch/spring-cloud-data-flow-dashboard-add-application.png" alt="spring cloud data flow dashboard add application">

Click on <var>Register one or more applications</var>.

<img src="{{ site.url }}/assets/images/spring-batch/spring-cloud-data-flow-dashboard-add-application-register.png" alt="spring cloud data flow dashboard add application register">

Enter <kbd>capitalize-names-app</kbd> as <var>name</var> of our application and select <kbd>Task</kbd> as <var>type</var>.

For the <var>URI</var> we enter the location of our Spring Boot Task JAR: <kbd>file://C:/Users/Codenotfound/repos/spring-batch/spring-batch-admin/spring-batch-task/target/spring-batch-task-0.0.1-SNAPSHOT.jar</kbd>

Once done click on <var>Register the application(s)</var>.

<img src="{{ site.url }}/assets/images/spring-batch/spring-cloud-data-flow-register-application.png" alt="spring cloud data flow register application">

Our application is now registered.

Now click on the <var>Task</var> menu to create a new task that we can execute.

<img src="{{ site.url }}/assets/images/spring-batch/spring-cloud-data-flow-applications-tasks.png" alt="spring cloud data flow applications tasks">

Click on the <var>Create task(s)</var> button.

<img src="{{ site.url }}/assets/images/spring-batch/spring-cloud-data-flow-tasks-create-task.png" alt="spring cloud data flow tasks create task">

Drag the <var>Capitalize-Names-App</var> task on the canvas and connect the <var>START</var> and <var>END</var> nodes as shown below.

Click on <var>Create Task</var>.

<img src="{{ site.url }}/assets/images/spring-batch/spring-cloud-data-flow-task-visual-editor.png" alt="spring cloud data flow task visual editor">

Enter <kbd>capitalize-names-task</kbd> as task name and click on <var>Create the task</var>.

<img src="{{ site.url }}/assets/images/spring-batch/spring-cloud-data-flow-task-creation-confirmation.png" alt="spring cloud data flow task creation confirmation">

Our <var>capitalize-names-task</var> is now ready to be used. Click on the play icon to start an instance.

<img src="{{ site.url }}/assets/images/spring-batch/spring-cloud-data-flow-tasks-definitions-launch.png" alt="spring cloud data flow tasks definitions launch">

Click on the <var>Launch the task</var> button.

<img src="{{ site.url }}/assets/images/spring-batch/spring-cloud-data-flow-tasks-launch-task.png" alt="spring cloud data flow tasks launch task">

A new <var>Executions</var> tab now appears under the <var>Task</var> section. Click on it to consult the status of the task that we started.

<img src="{{ site.url }}/assets/images/spring-batch/spring-cloud-data-flow-tasks-executions.png" alt="spring cloud data flow tasks executions">

We can see that there is one execution instance for our <var>capitalize-names-task</var>. Click on the information icon to see the details.

<img src="{{ site.url }}/assets/images/spring-batch/spring-cloud-data-flow-launch-tasks-executions-show-details.png" alt="spring cloud data flow tasks executions show details">

To see information on the batch job that was run click on the <var>Job Execution Ids</var> identifier.

<img src="{{ site.url }}/assets/images/spring-batch/spring-cloud-data-flow-launch-tasks-executions-details.png" alt="spring cloud data flow tasks executions details">

A new page opens that shows us the details on the Spring Batch <var>capitalizeNamesJob</var> job. We can even see the status and information on the step that was executed.

<img src="{{ site.url }}/assets/images/spring-batch/spring-cloud-data-flow-launch-tasks-executions-batch-job-details.png" alt="spring cloud data flow tasks executions batch job details">

So, in other words, we successfully used a Spring Batch Admin UI to launch a Spring Batch job!

> Note that you can also consult the log files of the executed batch job. Check the console output of the Spring Cloud Server for the location of the log files.

<img src="{{ site.url }}/assets/images/spring-batch/spring-cloud-data-flow-console-log-location.png" alt="spring cloud data flow console log location">

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-batch/tree/master/spring-batch-admin){:target="_blank"}.
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

In this tutorial, we illustrated an end-to-end scenario in which we used an admin UI to launch and monitor a Spring Batch job.

Hope you enjoyed this post.

Leave a comment if you did.

Thanks!
