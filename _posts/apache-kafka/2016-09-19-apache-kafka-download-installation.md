---
title: Apache Kafka - Download & Installation 
permalink: /2016/09/apache-kafka-download-installation.html
excerpt: A step-by-step tutorial on how to install and run Apache Kafka on Windows.
date: 2016-09-19 21:00
categories: [Apache Kafka]
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

<figure>
    <img src="{{ site.url }}/assets/images/logos/apache-zookeeper-logo.png" alt="apache zookeeper logo">
</figure>

Head over to the [Apache Zookeeper download](https://zookeeper.apache.org/releases.html) page and and click on the download link in the <ins>Download</ins> section. This will redirect to a mirror site, click on the suggested mirror link and from the <ins>index</ins> select the <ins>stable</ins> directory as shown below. Download the gzipped TAR file, at the moment of writing this tutorial the latest stable release was zookeeper-3.4.9.

<figure>
    <img src="{{ site.url }}/assets/images/apache-kafka/apache-zookeeper-stable-releases.png" alt="apache zookeeper stable releases">
</figure>

Extract the archive that was downloaded in the previous step. The extracted root directory should contain a number of files and subdirectories as shown below. From now on we will refer to this directory as: '<var>[zookeeper_install_dir]</var>'.

<figure>
    <img src="{{ site.url }}/assets/images/apache-kafka/apache-zookeeper-install-directory.png" alt="apache zookeeper install directory">
</figure>

Follow the below steps in order to setup a minimal working ZooKeeper configuration:
1. Navigate to the ZooKeeper configuration directory located under <ins>[zookeeper_install_dir]/conf</ins>.
2. Copy the file <ins>zoo_sample.cfg</ins> and rename to <ins>zoo.cfg</ins>.
3. Open the newly created <ins>zoo.cfg<ins> in a text editor.
4. Find the "<kbd>dataDir=/tmp/zookeeper</kbd>" entry and change it to "<kbd>dataDir=C:/temp/zookeeper</kbd>". Make sure to use forward slashes in the path name!
5. Next set the '<var>ZOOKEEPER_HOME</var>' and corresponding '<var>PATH</var>' environment variables. Click the Windows Start button and then type "<kbd>env</kbd>" without quotes in the search box. Select the <ins>Edit environment variables for your account</ins> entry, this will open the environment variables window. 
    * Add a new variable using "<kbd>ZOOKEEPER_HOME</kbd>" as name and "<kbd>[zookeeper_install_dir]</kbd>" as value. Click <ins>OK<ins> to to save.
    * Edit (or add if it doesn't exist) the variable with name "<kbd>PATH</kbd>" and add "<kbd>;%ZOOKEEPER_HOME%\bin</kbd>" to the end of the value. Click <ins>OK</ins> to save.

<figure>
    <img src="{{ site.url }}/assets/images/apache-kafka/set-zookeeper-environment-variables.png" alt="set zookeeper environment variables">
</figure>

Now that ZooKeeper is configured, let's go ahead and start it. Open a command prompt by clicking on the Windows Start button and typing "<kbd>cmd</kbd>" followed by pressing <ins>ENTER<ins>. Use following command to startup ZooKeeper:

``` plaintext
zkserver
```

By default ZooKeeper will generate a number of log statements at start-up as shown below. One of the log entries will mention '<var>binding to port 0.0.0.0/0.0.0.0:2181</var>'. This indicates that ZooKeeper was successfully started.

<figure>
    <img src="{{ site.url }}/assets/images/apache-kafka/zookeeper-startup-trace.png" alt="zookeeper startup trace">
</figure>

# Download and Setup Kafka


















