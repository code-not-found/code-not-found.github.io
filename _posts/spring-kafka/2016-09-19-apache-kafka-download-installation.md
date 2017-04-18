---
title: "Apache Kafka - Download &amp; Installation"
permalink: /2016/09/apache-kafka-download-installation.html
excerpt: "A step-by-step tutorial on how to install and run Apache Kafka on Windows."
date: 2016-09-19
modified: 2017-04-15
categories: [Spring Kafka]
tags: [Apache Kafka, Apache ZooKeeper, Download, Installation, Kafka, Windows, ZooKeeper]
published: true
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/apache-kafka-logo.jpg" alt="spring logo" class="logo">
</figure>

[Apache Kafka](http://kafka.apache.org/) is an open-source message broker project developed by the Apache Software Foundation written in Scala. The project aims to provide a high-throughput, low-latency platform capable of handling hundreds of megabytes of reads and writes per second from thousands of clients. Following tutorial shows how to download and install Apache Kafka on Windows and perform a start/stop of the installed instance.


It is important to note that Kafka will not work without [Apache ZooKeeper](https://zookeeper.apache.org/), which is essentially a distributed hierarchical key-value store. Like Kafka, ZooKeeper is a software project of the Apache Software Foundation. [Kafka uses ZooKeeper for: electing a controller, cluster membership, topic configuration, quotas and ACLs](https://www.quora.com/What-is-the-actual-role-of-ZooKeeper-in-Kafka).

Note that for running Kafka and ZooKeeper, a [Java Runtime Environment needs to be installed and configured](http://www.oracle.com/technetwork/java/javase/downloads/index.html) (with JAVA_HOME correctly set). If you are not sure if Java is installed on your machine, open a console and executed following command:

``` plaintext
java -version
```

# Download and Setup ZooKeeper

<figure>
    <img src="{{ site.url }}/assets/images/logos/apache-zookeeper-logo.png" alt="apache zookeeper logo">
</figure>

Head over to the [Apache ZooKeeper download](https://zookeeper.apache.org/releases.html) page and and click on the download link in the <var>Download</var> section. This will redirect to a mirror site, click on the suggested mirror link and from the <var>index</var> select the <var>stable</var> directory as shown below. Download the gzipped TAR file, at the moment of writing this tutorial the latest stable release was [zookeeper-3.4.9](http://www-us.apache.org/dist/zookeeper/stable/).

<figure>
    <img src="{{ site.url }}/assets/images/spring-kafka/apache-zookeeper-stable-releases.png" alt="apache zookeeper stable releases">
</figure>

Extract the archive that was downloaded in the previous step. The extracted root directory should contain a number of files and subdirectories as shown below. From now on we will refer to this directory as: <var>[zookeeper_install_dir]</var>.

<figure>
    <img src="{{ site.url }}/assets/images/spring-kafka/apache-zookeeper-install-directory.png" alt="apache zookeeper install directory">
</figure>

Follow the below steps in order to setup a minimal working ZooKeeper configuration:
1. Navigate to the ZooKeeper configuration directory located under <var>[zookeeper_install_dir]/conf</var>.
2. Copy the file <var>zoo_sample.cfg</var> and rename to <var>zoo.cfg</var>.
3. Open the newly created <var>zoo.cfg</var> in a text editor.
4. Find the "<kbd>dataDir=/tmp/zookeeper</kbd>" entry and change it to "<kbd>dataDir=C:/temp/zookeeper</kbd>". Make sure to use forward slashes in the path name!
5. Next, set the <var>'ZOOKEEPER_HOME'</var> and corresponding <var>'PATH'</var> environment variables. Click the Windows Start button and then type "<kbd>env</kbd>" without quotes in the search box. Select the <var>Edit environment variables for your account</var> entry, this will open the environment variables window. 
    * Add a new variable using "<kbd>ZOOKEEPER_HOME</kbd>" as name and "<kbd>[zookeeper_install_dir]</kbd>" as value. Click "<kbd>OK</kbd>" to to save.
    * Edit (or add if it doesn't exist) the variable with name "<kbd>PATH</kbd>" and add "<kbd>;%ZOOKEEPER_HOME%\bin</kbd>" to the end of the value. Click "<kbd>OK</kbd>" to save.

<figure>
    <img src="{{ site.url }}/assets/images/spring-kafka/set-apache-zookeeper-environment-variables.png" alt="set apache zookeeper environment variables">
</figure>

Now that ZooKeeper is configured, let's go ahead and start it. Open a command prompt by clicking on the Windows Start button and typing "<kbd>cmd</kbd>" followed by pressing "<kbd>ENTER</kbd>". Use following command to startup ZooKeeper:

``` plaintext
zkserver
```

By default ZooKeeper will generate a number of log statements at start-up as shown below. One of the log entries will mention <var>'binding to port 0.0.0.0/0.0.0.0:2181'</var>. This indicates that ZooKeeper was successfully started.

<figure>
    <img src="{{ site.url }}/assets/images/spring-kafka/apache-zookeeper-startup-trace.png" alt="apache zookeeper startup trace">
</figure>

# Download and Setup Kafka

Open the [Kafka releases page](http://kafka.apache.org/downloads.html) which contains the latest binary downloads. Kafka is written in [Scala](https://www.scala-lang.org/), which is programming language that has full support for functional programming. Scala source code is intended to be compiled to Java bytecode, so that the resulting executable code runs on a Java virtual machine.

You'll notice that the release page contains multiple versions of Scala for a specific Kafka release. This only matters if you are using Scala yourself. If not the case, go ahead and choose the highest supported version. At the moment of writing the latest stable release was [kafka_2.11-0.10.0.1.tgz](https://www.apache.org/dyn/closer.cgi?path=/kafka/0.10.0.1/kafka_2.11-0.10.0.1.tgz).

Extract the gzipped TAR file, downloaded in the previous step. The extracted root directory should contain a number of files and subdirectories as shown below. From now on we will refer to this directory as: <var>[kafka_install_dir]</var>.

> Make sure to extract to a directory path that does not contain spaces.

<figure>
    <img src="{{ site.url }}/assets/images/spring-kafka/apache-kafka-install-directory.png" alt="apache kafka install directory">
</figure>

Follow the below steps in order to setup a minimal working Kakfa configuration: 
1. Navigate to the Kafka configuration directory located under <var>[kafka_install_dir]/config</var>.
2. Edit the file <var>server.properties</var> in a text editor.
3. Find the "<kbd>log.dirs=/tmp/kafka-logs</kbd>" entry and change it to "<kbd>log.dirs=C:/temp/kafka-logs</kbd>". Make sure to use forward slashes in the path name!

> Make sure Zookeeper is up and running before starting Kafka.

In order to start Kafka, open a command prompt by clicking on the Windows Start button and typing "<kbd>cmd</kbd>" followed by pressing "<kbd>ENTER</kbd>". Navigate to the <var>[kafka_install_dir]</var>. Use following command to startup Kafka:

``` plaintext
.\bin\windows\kafka-server-start.bat .\config\server.properties
```

Kafka will generate a number of log statements at start-up as shown below. The last log entries will mention <var>'[Kafka Server 01], started'</var>. This means that a Kafka instance is up and running.

<figure>
    <img src="{{ site.url }}/assets/images/spring-kafka/apache-kafka-startup-trace.png" alt="apache kafka startup trace">
</figure>

---

This concludes installing ZooKeeper and Kafka on Windows. If you found this post helpful or have any questions or remarks, please leave a comment.