---
title: Apache Kafka - Download & Installation 
permalink: /2016/09/apache-kafka-download-installation.html
excerpt: A step-by-step tutorial on how to install and run Apache Kafka on Windows.
date: 2016-09-19 21:00
categories: [Spring Kafka]
tags: [Apache Kafka, Download, Installation, Kafka, Windows]
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/apache-kafka-logo.png" alt="apache kafka logo">
</figure>

[Apache Kafka](http://kafka.apache.org/) is an open-source message broker project developed by the Apache Software Foundation written in Scala. The project aims to provide a high-throughput, low-latency platform capable of handling hundreds of megabytes of reads and writes per second from thousands of clients. Following tutorial shows how to download and install Apache Kafka on Windows and perform a start/stop of the installed instance. 


It is important to note that Kafka will not work without [Apache ZooKeeper](https://zookeeper.apache.org/), which is essentially a distributed hierarchical key-value store. Like Kafka, ZooKeeper is a software project of the Apache Software Foundation. [Kafka uses ZooKeeper for: electing a controller, cluster membership, topic configuration, quotas and ACLs](https://www.quora.com/What-is-the-actual-role-of-ZooKeeper-in-Kafka).

Note that for running Kafka and ZooKeeper, a [Java Runtime Environment needs to be installed and configured](http://www.oracle.com/technetwork/java/javase/downloads/index.html) (with JAVA_HOME correctly set). If you are not sure if Java is installed on your machine, open a console and executed following command: 

``` plaintext
java -version
```

# Download and Setup Zookeeper








