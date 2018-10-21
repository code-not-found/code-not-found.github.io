---
title: "Spring Batch Tasklet Example"
permalink: /spring-batch-tasklet-example.html
excerpt: "A detailed step-by-step tutorial on how to implement a Spring Batch Tasklet using Spring Boot and Maven."
date: 2018-10-21
last_modified_at: 2018-10-21
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

Spring Batch offers two different ways for **implementing a step of a batch job**:
* using [chunks](https://docs.spring.io/spring-batch/4.0.x/reference/html/step.html#chunkOrientedProcessing){:target="_blank"}
* using a [tasklet](https://docs.spring.io/spring-batch/4.0.x/reference/html/step.html#taskletStep){:target="_blank"}

In our [Spring Batch Job example]({{ site.url }}/spring-batch-hello-world-example.html) we saw that a batch job consists out of one or more `Step`s. A `Tasklet` represents the work that is done in a `Step`. It contains the strategy for processing in a `Step`.

A [Tasklet](https://docs.spring.io/spring-batch/trunk/apidocs/org/springframework/batch/core/step/tasklet/Tasklet.html){:target="_blank"} is in fact a simple interface that has one method: `execute()`. A `Step` calls this method repeatedly until it either finishes or throws an exception.

Spring Batch contains a number of implementations of the `Tasklet` interface. One of them is a chunk oriented processing `Tasklet`. If you look at the [ChunkOrientedTasklet](https://docs.spring.io/spring-batch/trunk/apidocs/org/springframework/batch/core/step/item/ChunkOrientedTasklet.html){:target="_blank"} you can see it implements the `Tasklet` interface.

> In Spring batch, a `Tasklet` is an interface that performs a single task within a `Step`. A typical use case for implementing a `Tasklet` is the setup up or cleaning of resources before or after the execution of a `Step`.

So let's recap:

| Question                            | Answer                                                                           |
|:------------------------------------|:---------------------------------------------------------------------------------|
| When do I use a Tasklet?            | When you need to execute a single granular task.                                 |
| How does a Tasklet work?            | Everything happens within a single transaction boundary.                         |
| Is a Tasklet often used?            | In most cases you will use chunks to handle large volumes.                       |
| What is a typical Tasklet use case? | Usually used to setup up or clean resources before or after the main processing. |

## General Project Setup

To show how Spring Batch `Tasklet` works let's build an example.

TODO

We will use the following tools/frameworks:
* _Spring Batch 4.0_
* _Spring Boot 2.0_
* _Maven 3.5_

Our Maven project has the following structure:

<img src="{{ site.url }}/assets/images/spring-batch/spring-batch-tasklet-maven-project.png" alt="spring batch tasklet maven project">

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/spring-batch/tree/master/spring-batch-tasklet){:target="_blank"}.
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

In this tutorial you also learned the difference between a `Tasklet` and a `Chunk`. We also created a Tasklet example using Spring Batch.

If you have any questions or if you enjoyed this post.

Drop a line below.

Thanks!
