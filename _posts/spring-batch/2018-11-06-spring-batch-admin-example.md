---
title: "Spring Batch Admin Example"
permalink: /spring-batch-admin-example.html
excerpt: "A detailed step-by-step tutorial on how to use a Spring Boot admin UI to manage Spring Batch jobs."
date: 2018-11-08
last_modified_at: 2018-11-08
header:
  teaser: "assets/images/spring-batch/spring-batch-admin.png"
categories: [Spring Batch]
tags: [Admin, Example, Maven, Spring Batch, Spring Boot, Tutorial, UI]
published: false
---

<img src="{{ site.url }}/assets/images/spring-batch/spring-batch-admin.png" alt="spring batch admin" class="align-right title-image">

Looking for a [Spring Batch Admin](https://docs.spring.io/spring-batch-admin/trunk/){:target="_blank"} UI tutorial?

You might be surprised to learn it is no longer supported.

Don't worry as in this post **I'll show you the recommended replacement**. And how to set it up.

Let’s dive right in.

## What is Spring Batch Admin?

Spring Batch Admin provides a user interface that allows you to manage batch jobs. The project is **end of life since December 31, 2017**.

[Spring Cloud Data Flow](https://cloud.spring.io/spring-cloud-dataflow/){:target="_blank"} is now the recommended replacement for managing and monitoring [Spring Batch](https://spring.io/projects/spring-batch){:target="_blank"} jobs.

Let's demonstrate how Spring Cloud Data Flow works in combination with Spring Batch.

We start from a basic [Spring Batch capitalize names](https://github.com/code-not-found/spring-batch/tree/master/spring-batch-capitalize-names){:target="_blank"} project that converts person names from lower case into upper case.

We then start a Spring Cloud Data Flow server and configure the batch job. Using the web-based UI we launch the job and monitor the status.

## General Project Setup

> [Spring Cloud Data Flow](https://github.com/spring-cloud/spring-cloud-dataflow){:target="_blank"} does not yet support Spring Boot 2.0. That is why this example uses Spring Boot 1.5 and Spring Batch 3.0.

We will use the following tools/frameworks:
* _Spring Batch 3.0_
* _Spring Boot 1.5_
* _Maven 3.5_

We will create two Maven projects have the following structure:

<img src="{{ site.url }}/assets/images/spring-batch/spring-batch-admin-maven-projects.png" alt="spring batch admin maven projects">

## Creating a Spring Batch Task






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
