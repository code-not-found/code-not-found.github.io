---
title: "Spring Batch Tasklet Example"
permalink: /spring-batch-tasklet-example.html
excerpt: "A detailed step-by-step tutorial on how to implement a Spring Batch Tasklet using Spring Boot and Maven."
date: 2018-10-21
last_modified_at: 2018-11-06
header:
  teaser: "assets/images/spring-batch/spring-batch-tasklet-example.png"
categories: [Spring Batch]
tags: [Example, Maven, Spring Batch, Spring Boot, Tasklet, Tutorial ]
published: true
---

<img src="{{ site.url }}/assets/images/spring-batch/spring-batch-tasklet-example.png" alt="spring batch tasklet example" class="align-right title-image">

Want to know what a _Spring Batch Tasklet_ is?

And more important: **when to use it?**

Then you’re in the right place.

Because today I’m going to show you a detailed example.

Let’s do this!

## What is a Spring Batch Tasklet?

In Spring batch, a `Tasklet` is an interface that performs a single task within a `Step`. A typical use case for implementing a `Tasklet` is the setup up or cleaning of resources before or after the execution of a `Step`.

In fact, Spring Batch offers two different ways for **implementing a step of a batch job**:
1. using [chunks](https://docs.spring.io/spring-batch/4.0.x/reference/html/step.html#chunkOrientedProcessing){:target="_blank"}
2. using a [tasklet](https://docs.spring.io/spring-batch/4.0.x/reference/html/step.html#taskletStep){:target="_blank"}

In the [Spring Batch Job example]({{ site.url }}/spring-batch-hello-world-example.html) we saw that a batch job consists out of one or more `Step`s. And a `Tasklet` represents the work that is done in a `Step`.

A [Tasklet](https://docs.spring.io/spring-batch/trunk/apidocs/org/springframework/batch/core/step/tasklet/Tasklet.html){:target="_blank"} is in fact a simple interface that has one method: `execute()`. A `Step` calls this method repeatedly until it either finishes or throws an exception.

Spring Batch contains a number of implementations of the `Tasklet` interface. One of them is a "chunk oriented processing" `Tasklet`. If you look at the [ChunkOrientedTasklet](https://docs.spring.io/spring-batch/trunk/apidocs/org/springframework/batch/core/step/item/ChunkOrientedTasklet.html){:target="_blank"} you can see it implements the `Tasklet` interface.

So let's recap the above:

| Question                            | Answer                                                                                           |
|:------------------------------------|:-------------------------------------------------------------------------------------------------|
| When do I use a Tasklet?            | When you need to execute a single granular task.                                                 |
| How does a Tasklet work?            | Everything happens within a single transaction boundary that either finishes or throws an error. |
| Is a Tasklet often used?            | It is typically not used very often. In most cases, you will use chunks to handle large volumes. |
| What is a typical Tasklet use case? | Usually used to setup up or clean resources before or after the main processing.                 |

To show how Spring Batch `Tasklet` works let's build an example.

We start from a basic [Spring Batch capitalize names example](https://github.com/code-not-found/spring-batch/tree/master/spring-batch-capitalize-names){:target="_blank"} that converts person names into upper case.

We then change the example so that it reads two CSV files. When the batch `Job` is finishes we delete the input CSV files using a `Tasklet`.

## General Project Setup

We will use the following tools/frameworks:
* _Spring Batch 4.1_
* _Spring Boot 2.1_
* _Maven 3.5_

Our Maven project has the following structure:

<img src="{{ site.url }}/assets/images/spring-batch/spring-batch-tasklet-maven-project.png" alt="spring batch tasklet maven project">

The Maven and Spring Boot setup are identical to a previous [Spring Batch example]({{ site.url }}/spring-batch-hello-world-example.html) so we will not cover them in this post.

## Creating a Spring Batch Tasklet

To create a Spring Batch Tasklet you need to implement the `Tasklet` interface.

Let's start by creating a `FileDeletingTasklet` that will delete all files in a directory. Add the `execute()` method that loops over the available files and tries to delete them. When all files are deleted we return the <var>FINISHED</var> status so that the `Step` finishes.

We add a `setDirectory()` method that allows configuring the directory that needs to be cleaned.

The [afterPropertiesSet()](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/InitializingBean.html){:target="_blank"} allows to check if a directory was set when `FileDeletingTasklet` is initialized as a Spring Bean.

{% highlight java %}
package com.codenotfound.batch.job;

import java.io.File;
import java.io.IOException;
import org.springframework.batch.core.StepContribution;
import org.springframework.batch.core.UnexpectedJobExecutionException;
import org.springframework.batch.core.scope.context.ChunkContext;
import org.springframework.batch.core.step.tasklet.Tasklet;
import org.springframework.batch.repeat.RepeatStatus;
import org.springframework.core.io.Resource;
import org.springframework.util.Assert;

public class FileDeletingTasklet implements Tasklet {

  private Resource directory;

  @Override
  public RepeatStatus execute(StepContribution stepContribution, ChunkContext chunkContext)
      throws IOException {

    File dir = directory.getFile();
    Assert.state(dir.isDirectory(), "Not a directory");

    for (File file : dir.listFiles()) {
      boolean deleted = file.delete();
      if (!deleted) {
        throw new UnexpectedJobExecutionException("Could not delete file: " + file.getPath());
      }
    }
    return RepeatStatus.FINISHED;
  }

  public void setDirectory(Resource directory) {
    this.directory = directory;
  }

  public void afterPropertiesSet() throws Exception {
    Assert.notNull(directory, "Directory must be set");
  }
}
{% endhighlight %}

Now that our Spring Batch Tasklet is created let's adapt the `CapitalizeNamesJobConfig` to include it.

{% highlight java %}
package com.codenotfound.batch.job;

import java.io.IOException;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.batch.core.Job;
import org.springframework.batch.core.Step;
import org.springframework.batch.core.configuration.annotation.EnableBatchProcessing;
import org.springframework.batch.core.configuration.annotation.JobBuilderFactory;
import org.springframework.batch.core.configuration.annotation.StepBuilderFactory;
import org.springframework.batch.item.file.FlatFileItemReader;
import org.springframework.batch.item.file.FlatFileItemWriter;
import org.springframework.batch.item.file.MultiResourceItemReader;
import org.springframework.batch.item.file.builder.FlatFileItemReaderBuilder;
import org.springframework.batch.item.file.builder.FlatFileItemWriterBuilder;
import org.springframework.batch.item.file.builder.MultiResourceItemReaderBuilder;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.FileSystemResource;
import org.springframework.core.io.Resource;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import org.springframework.core.io.support.ResourcePatternResolver;
import com.codenotfound.model.Person;

@Configuration
@EnableBatchProcessing
public class CapitalizeNamesJobConfig {

  private static final Logger LOGGER = LoggerFactory.getLogger(CapitalizeNamesJobConfig.class);

  @Autowired
  public JobBuilderFactory jobBuilders;

  @Autowired
  public StepBuilderFactory stepBuilders;

  @Bean
  public Job convertNamesJob() {
    return jobBuilders.get("capitalizeNamesJob").start(convertNamesStep()).next(deleteFilesStep())
        .build();
  }

  @Bean
  public Step convertNamesStep() {
    return stepBuilders.get("capitalizeNamesStep").<Person, Person>chunk(10)
        .reader(multiItemReader()).processor(itemProcessor()).writer(itemWriter()).build();
  }

  @Bean
  public Step deleteFilesStep() {
    return stepBuilders.get("deleteFilesStep").tasklet(fileDeletingTasklet()).build();
  }

  @Bean
  public MultiResourceItemReader<Person> multiItemReader() {
    ResourcePatternResolver patternResolver = new PathMatchingResourcePatternResolver();
    Resource[] resources = null;
    try {
      resources = patternResolver.getResources("file:target/test-inputs/*.csv");
    } catch (IOException e) {
      LOGGER.error("error reading files", e);
    }

    return new MultiResourceItemReaderBuilder<Person>().name("multiPersonItemReader")
        .delegate(itemReader()).resources(resources).setStrict(true).build();
  }

  @Bean
  public FlatFileItemReader<Person> itemReader() {
    return new FlatFileItemReaderBuilder<Person>().name("personItemReader").delimited()
        .names(new String[] {"firstName", "lastName"}).targetType(Person.class).build();
  }

  @Bean
  public PersonItemProcessor itemProcessor() {
    return new PersonItemProcessor();
  }

  @Bean
  public FlatFileItemWriter<Person> itemWriter() {
    return new FlatFileItemWriterBuilder<Person>().name("personItemWriter")
        .resource(new FileSystemResource("target/test-outputs/persons.txt")).delimited()
        .delimiter(", ").names(new String[] {"firstName", "lastName"}).build();
  }

  @Bean
  public FileDeletingTasklet fileDeletingTasklet() {
    FileDeletingTasklet tasklet = new FileDeletingTasklet();
    tasklet.setDirectory(new FileSystemResource("target/test-inputs"));

    return tasklet;
  }
}
{% endhighlight %}

## Unit Test the Spring Batch Tasklet



---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-batch/tree/master/spring-batch-tasklet){:target="_blank"}.
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

In this tutorial, you learned the difference between a `Tasklet` and a `Chunk`. We also created a Tasklet example using Spring Batch.

If you have any questions or if you enjoyed this post.

Drop a line below.

Thanks!
