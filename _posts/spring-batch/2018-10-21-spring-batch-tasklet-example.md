---
title: "Spring Batch Tasklet Example"
permalink: /spring-batch-tasklet-example.html
excerpt: "A detailed step-by-step tutorial on how to implement a Spring Batch Tasklet using Spring Boot."
date: 2018-10-21
last_modified_at: 2019-05-30
header:
  teaser: "assets/images/spring-batch/spring-batch-tasklet.png"
categories: [Spring Batch]
tags: [Example, Maven, Spring Batch, Spring Boot, Tasklet, Tutorial]
published: true
---

<img src="{{ site.url }}/assets/images/spring-batch/spring-batch-tasklet.png" alt="spring batch tasklet" class="align-right title-image">

Want to know what a [Spring Batch Tasklet](https://docs.spring.io/spring-batch/trunk/apidocs/org/springframework/batch/core/step/tasklet/Tasklet.html){:target="_blank"} is?

And more important: **when to use it?**

Then you're in the right place.

Because today I'm going to show you a detailed example.

Let's do this!

If you want to learn more about Spring Batch - head on over to the [Spring Batch tutorials page]({{ site.url }}/spring-batch-tutorials).
{: .notice--primary}

## 1. What is a Spring Batch Tasklet?

In Spring Batch, a `Tasklet` is an interface that performs a single task within a `Step`. A typical use case for implementing a `Tasklet` is the setup up or cleaning of resources before or after the execution of a `Step`.

In fact, Spring Batch offers two different ways for **implementing a step of a batch job**: using [Chunks](https://docs.spring.io/spring-batch/4.1.x/reference/html/step.html#chunkOrientedProcessing){:target="_blank"} or using a [Tasklet](https://docs.spring.io/spring-batch/4.1.x/reference/html/step.html#taskletStep){:target="_blank"}.

In the [Spring Batch Job example]({{ site.url }}/spring-batch-example.html), we saw that a batch job consists out of one or more `Step`s. And a `Tasklet` represents the work that is done in a `Step`.

The `Tasklet` interface has one method: [execute()](https://docs.spring.io/spring-batch/trunk/apidocs/org/springframework/batch/core/step/tasklet/Tasklet.html#execute-org.springframework.batch.core.StepContribution-org.springframework.batch.core.scope.context.ChunkContext-){:target="_blank"}. A `Step` calls this method repeatedly until it either finishes or throws an exception.

The Spring Batch framework contains some implementations of the `Tasklet` interface. One of them is a "chunk oriented processing" `Tasklet`. If you look at the [ChunkOrientedTasklet](https://docs.spring.io/spring-batch/trunk/apidocs/org/springframework/batch/core/step/item/ChunkOrientedTasklet.html){:target="_blank"} you can see it implements the `Tasklet` interface.

_So let's recap the above:_

| Question                            | Answer                                                                                           |
|:------------------------------------|:-------------------------------------------------------------------------------------------------|
| When do I use a Tasklet?            | When you need to execute a single granular task.                                                 |
| How does a Tasklet work?            | Everything happens within a single transaction boundary that either finishes or throws an error. |
| Is a Tasklet often used?            | It is not used very often. In most cases, you will use chunks to handle large volumes. |
| What is a typical Tasklet use case? | Usually used to setup up or clean resources before or after the main processing.                 |

To show you how a Spring Batch Tasklet works let's create a simple example.

We start from a basic [Spring Batch capitalize names](https://github.com/code-not-found/spring-batch/tree/master/spring-batch-capitalize-names){:target="_blank"} project that converts person names from lower into upper case.

We then change the batch job so that it reads multiple CSV files. When the `Job` finishes we clean up the input files using a `Tasklet`.

## 2. General Project Overview

We will use the following tools/frameworks:
* Spring Batch 4.1
* Spring Boot 2.1
* Maven 3.6

Our Maven project has the following structure:

<img src="{{ site.url }}/assets/images/spring-batch/spring-batch-tasklet-maven-project.png" alt="spring batch tasklet maven project">

The Maven and Spring Boot setup are identical to a previous [Spring Batch example]({{ site.url }}/spring-batch-example.html) so we will not cover them in this post.

## 3. Creating a Spring Batch Tasklet

To create a Spring Batch Tasklet you need to implement the `Tasklet` interface.

Let's start by creating a `FileDeletingTasklet` that will delete all files in a directory. Add the `execute()` method that walks over the available files and tries to delete them.

When all files are deleted we return the <var>FINISHED</var> status so that the `Step` that calls the `FileDeletingTasklet` can finish.

We also add a constructor that sets the directory that needs to be cleaned.

{% highlight java %}
package com.codenotfound.batch;

import java.io.File;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.stream.Stream;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.batch.core.StepContribution;
import org.springframework.batch.core.UnexpectedJobExecutionException;
import org.springframework.batch.core.scope.context.ChunkContext;
import org.springframework.batch.core.step.tasklet.Tasklet;
import org.springframework.batch.repeat.RepeatStatus;
import org.springframework.core.io.Resource;

public class FileDeletingTasklet implements Tasklet {

  private static final Logger LOGGER =
      LoggerFactory.getLogger(FileDeletingTasklet.class);

  private Resource directory;

  public FileDeletingTasklet(Resource directory) {
    this.directory = directory;
  }

  @Override
  public RepeatStatus execute(StepContribution stepContribution,
      ChunkContext chunkContext) {
    try (Stream<Path> walk =
        Files.walk(Paths.get(directory.getFile().getPath()))) {
      walk.filter(Files::isRegularFile).map(Path::toFile)
          .forEach(File::delete);
    } catch (IOException e) {
      LOGGER.error("error deleting files", e);
      throw new UnexpectedJobExecutionException(
          "unable to delete files");
    }

    return RepeatStatus.FINISHED;
  }
}
{% endhighlight %}

Now that our Spring Batch Tasklet is created let's change the `CapitalizeNamesJobConfig` to include it.

We add a `deleteFilesStep` Bean that uses the `FileDeletingTasklet`. We then adapt the `capitalizeNamesJob` Bean so that this new `Step` is executed at the end.

We also add a `multiItemReader` Bean that reads multiple input files.

We finish by creating the `fileDeletingTasklet` Bean on which we specify the directory that needs to be cleaned.

{% highlight java %}
package com.codenotfound.batch;

import java.io.IOException;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.batch.core.Job;
import org.springframework.batch.core.Step;
import org.springframework.batch.core.configuration.annotation.JobBuilderFactory;
import org.springframework.batch.core.configuration.annotation.StepBuilderFactory;
import org.springframework.batch.item.file.FlatFileItemReader;
import org.springframework.batch.item.file.FlatFileItemWriter;
import org.springframework.batch.item.file.MultiResourceItemReader;
import org.springframework.batch.item.file.builder.FlatFileItemReaderBuilder;
import org.springframework.batch.item.file.builder.FlatFileItemWriterBuilder;
import org.springframework.batch.item.file.builder.MultiResourceItemReaderBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.FileSystemResource;
import org.springframework.core.io.Resource;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import org.springframework.core.io.support.ResourcePatternResolver;
import com.codenotfound.model.Person;

@Configuration
public class CapitalizeNamesJobConfig {

  private static final Logger LOGGER =
      LoggerFactory.getLogger(CapitalizeNamesJobConfig.class);

  @Bean
  public Job capitalizeNamesJob(JobBuilderFactory jobBuilders,
      StepBuilderFactory stepBuilders) {
    return jobBuilders.get("capitalizeNamesJob")
        .start(capitalizeNamesStep(stepBuilders))
        .next(deleteFilesStep(stepBuilders)).build();
  }

  @Bean
  public Step capitalizeNamesStep(StepBuilderFactory stepBuilders) {
    return stepBuilders.get("capitalizeNamesStep")
        .<Person, Person>chunk(10).reader(multiItemReader())
        .processor(itemProcessor()).writer(itemWriter()).build();
  }

  @Bean
  public Step deleteFilesStep(StepBuilderFactory stepBuilders) {
    return stepBuilders.get("deleteFilesStep")
        .tasklet(fileDeletingTasklet()).build();
  }

  @Bean
  public MultiResourceItemReader<Person> multiItemReader() {
    ResourcePatternResolver patternResolver =
        new PathMatchingResourcePatternResolver();
    Resource[] resources = null;
    try {
      resources = patternResolver
          .getResources("file:target/test-inputs/*.csv");
    } catch (IOException e) {
      LOGGER.error("error reading files", e);
    }

    return new MultiResourceItemReaderBuilder<Person>()
        .name("multiPersonItemReader").delegate(itemReader())
        .resources(resources).setStrict(true).build();
  }

  @Bean
  public FlatFileItemReader<Person> itemReader() {
    return new FlatFileItemReaderBuilder<Person>()
        .name("personItemReader").delimited()
        .names(new String[] {"firstName", "lastName"})
        .targetType(Person.class).build();
  }

  @Bean
  public PersonItemProcessor itemProcessor() {
    return new PersonItemProcessor();
  }

  @Bean
  public FlatFileItemWriter<Person> itemWriter() {
    return new FlatFileItemWriterBuilder<Person>()
        .name("personItemWriter")
        .resource(new FileSystemResource(
            "target/test-outputs/persons.txt"))
        .delimited().delimiter(", ")
        .names(new String[] {"firstName", "lastName"}).build();
  }

  @Bean
  public FileDeletingTasklet fileDeletingTasklet() {
    return new FileDeletingTasklet(
        new FileSystemResource("target/test-inputs"));
  }
}
{% endhighlight %}

## 4. Unit Test the Spring Batch Tasklet

Let's update the existing unit test case so that we can check if our file deleting Tasklet works.

First, we copy the input files to the location from which our batch job will read. This is done in the `copyFiles()` method before the test case runs using the `@Before` annotation.

We then launch the batch job and check if it finishes successfully. We also check if all input files have been deleted.

{% highlight java %}
package com.codenotfound;

import static org.assertj.core.api.Assertions.assertThat;
import java.io.File;
import java.io.IOException;
import java.net.URISyntaxException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import org.junit.BeforeClass;
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
import org.springframework.core.io.ClassPathResource;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.util.FileSystemUtils;
import com.codenotfound.batch.job.BatchConfig;
import com.codenotfound.batch.job.CapitalizeNamesJobConfig;

@RunWith(SpringRunner.class)
@SpringBootTest(
    classes = {SpringBatchApplicationTests.BatchTestConfig.class})
public class SpringBatchApplicationTests {

  private static Path csvFilesPath, testInputsPath;

  @Autowired
  private JobLauncherTestUtils jobLauncherTestUtils;

  @BeforeClass
  public static void copyFiles()
      throws URISyntaxException, IOException {
    csvFilesPath = Paths.get(new ClassPathResource("csv").getURI());
    testInputsPath = Paths.get("target/test-inputs");
    try {
      Files.createDirectory(testInputsPath);
    } catch (Exception e) {
      // if directory exists do nothing
    }

    FileSystemUtils.copyRecursively(csvFilesPath, testInputsPath);
  }

  @Test
  public void testHelloWorldJob() throws Exception {
    JobExecution jobExecution = jobLauncherTestUtils.launchJob();
    assertThat(jobExecution.getExitStatus().getExitCode())
        .isEqualTo("COMPLETED");

    // check that all files are deleted
    File testInput = testInputsPath.toFile();
    assertThat(testInput.list().length).isEqualTo(0);
  }

  @Configuration
  @Import({BatchConfig.class, CapitalizeNamesJobConfig.class})
  static class BatchTestConfig {

    @Autowired
    private Job capitalizeNamesJob;

    @Bean
    JobLauncherTestUtils jobLauncherTestUtils()
        throws NoSuchJobException {
      JobLauncherTestUtils jobLauncherTestUtils =
          new JobLauncherTestUtils();
      jobLauncherTestUtils.setJob(capitalizeNamesJob);

      return jobLauncherTestUtils;
    }
  }
}
{% endhighlight %}

Now run above test case. Open a command prompt and execute following Maven command:

{% highlight plaintext %}
mvn test
{% endhighlight %}

Maven will download the needed dependencies, compile the code and run the unit test case. The result should be a successful build as shown below:

{% highlight plaintext %}
 .   ____          _            __ _ _
/\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
\\/  ___)| |_)| | | | | || (_| |  ) ) ) )
'  |____| .__|_| |_|_| |_\__, | / / / /
=========|_|==============|___/=/_/_/_/
:: Spring Boot ::        (v2.1.5.RELEASE)

2019-05-30 21:20:22.937  INFO 8312 --- [           main] c.c.SpringBatchApplicationTests          : Starting SpringBatchApplicationTests on DESKTOP-2RB3C1U with PID 8312 (started by Codenotfound in C:\Users\Codenotfound\repos\spring-batch\spring-batch-tasklet)
2019-05-30 21:20:22.939  INFO 8312 --- [           main] c.c.SpringBatchApplicationTests          : No active profile set, falling back to default profiles: default
2019-05-30 21:20:23.473  WARN 8312 --- [           main] o.s.b.c.c.a.DefaultBatchConfigurer       : No datasource was provided...using a Map based JobRepository
2019-05-30 21:20:23.473  WARN 8312 --- [           main] o.s.b.c.c.a.DefaultBatchConfigurer       : No transaction manager was provided, using a ResourcelessTransactionManager
2019-05-30 21:20:23.496  INFO 8312 --- [           main] o.s.b.c.l.support.SimpleJobLauncher      : No TaskExecutor has been set, defaulting to synchronous executor.
2019-05-30 21:20:23.520  INFO 8312 --- [           main] c.c.SpringBatchApplicationTests          : Started SpringBatchApplicationTests in 0.91 seconds (JVM running for 1.753)
2019-05-30 21:20:23.903  INFO 8312 --- [           main] o.s.b.c.l.support.SimpleJobLauncher      : Job: [SimpleJob: [name=capitalizeNamesJob]] launched with the following parameters: [{random=838477}]
2019-05-30 21:20:23.927  INFO 8312 --- [           main] o.s.batch.core.job.SimpleStepHandler     : Executing step: [capitalizeNamesStep]
2019-05-30 21:20:23.975  INFO 8312 --- [           main] c.c.batch.PersonItemProcessor            : converting 'Jessica Doe' into 'JESSICA DOE'
2019-05-30 21:20:23.983  INFO 8312 --- [           main] c.c.batch.PersonItemProcessor            : converting 'Julia Doe' into 'JULIA DOE'
2019-05-30 21:20:23.983  INFO 8312 --- [           main] c.c.batch.PersonItemProcessor            : converting 'Jasmin Doe' into 'JASMIN DOE'
2019-05-30 21:20:23.984  INFO 8312 --- [           main] c.c.batch.PersonItemProcessor            : converting 'Jennifer Doe' into 'JENNIFER DOE'
2019-05-30 21:20:23.984  INFO 8312 --- [           main] c.c.batch.PersonItemProcessor            : converting 'Jack Doe' into 'JACK DOE'
2019-05-30 21:20:23.984  INFO 8312 --- [           main] c.c.batch.PersonItemProcessor            : converting 'Jake Doe' into 'JAKE DOE'
2019-05-30 21:20:23.984  INFO 8312 --- [           main] c.c.batch.PersonItemProcessor            : converting 'Josh Doe' into 'JOSH DOE'
2019-05-30 21:20:23.984  INFO 8312 --- [           main] c.c.batch.PersonItemProcessor            : converting 'Joe Doe' into 'JOE DOE'
2019-05-30 21:20:24.033  INFO 8312 --- [           main] o.s.batch.core.job.SimpleStepHandler     : Executing step: [deleteFilesStep]
2019-05-30 21:20:24.053  INFO 8312 --- [           main] o.s.b.c.l.support.SimpleJobLauncher      : Job: [SimpleJob: [name=capitalizeNamesJob]] completed with the following parameters: [{random=838477}] and the following status: [COMPLETED]
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 1.977 s - in com.codenotfound.SpringBatchApplicationTests
[INFO]
[INFO] Results:
[INFO]
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  7.278 s
[INFO] Finished at: 2019-05-30T21:20:24+02:00
[INFO] ------------------------------------------------------------------------
{% endhighlight %}

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-batch/tree/master/spring-batch-tasklet){:target="_blank"}.
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

In this tutorial, you learned what a Spring Batch `Tasklet` is. We also created a Tasklet example using Spring Batch, Spring Boot, and Maven.

Leave a comment if you have any questions. Or if you enjoyed this post.

Thanks!
