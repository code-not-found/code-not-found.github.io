---
title: "Spring Batch Scheduler Example"
permalink: /spring-batch-scheduler-example.html
excerpt: "A detailed step-by-step tutorial on how to use a scheduler to run Spring Batch jobs using Spring Boot and Maven."
date: 2019-06-01
last_modified_at: 2019-06-01
header:
  teaser: "assets/images/spring-batch/spring-batch-scheduler.png"
categories: [Spring Batch]
tags: [Example, Maven, Scheduler, Spring Batch, Spring Boot, Tutorial]
published: false
---

<img src="{{ site.url }}/assets/images/spring-batch/spring-batch-scheduler.png" alt="spring batch scheduler" class="align-right title-image">

In this post I'm going to show you how to run Spring Batch at a schedule.

You'll also see how you can start/stop a scheduled job.

Let's get started.

## How Do You Schedule a Job with Spring Batch?

There are several ways to schedule a job with Spring Batch:
* You can use a scheduling tool like for example [Quartz](http://www.quartz-scheduler.org/){:target="_blank"}.
* You can use the [scheduling capabilities of Spring](https://docs.spring.io/spring/docs/5.1.0.RELEASE/spring-framework-reference/integration.html#scheduling){:target="_blank"}.
* You can use a software utility that comes with the OS like for example [Cron](https://en.wikipedia.org/wiki/Cron){:target="_blank"}.
