---
title: JMS - Apache ActiveMQ Installation
permalink: /2014/01/jms-apache-activemq-installation.html
excerpt: A step-by-step tutorial on how to install and run Apache ActiveMQ on Windows or Unix.
date: 2014-01-11 21:00
categories: [JMS]
tags: [ActiveMQ, Apache ActiveMQ, Installation, Java, JMS, Unix, Windows  ]
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/apache-activemq-logo.png" alt="apache activemq logo">
</figure>

[Apache ActiveMQ](http://activemq.apache.org/) is an open source message broker written in Java that offers JMS, REST and WebSocket interfaces. It supports protocols like AMQP, MQTT, OpenWire and STOMP that can be used by applications in different languages. Following tutorial details how to install ActiveMQ on Windows or Unix and perform a start/stop of the installed instance. 


 First thing to do is to download the ActiveMQ binaries. Go the the [ActiveMQ download page](http://activemq.apache.org/download.html) and click on the latest stable release link in the <ins>Latest Releases</ins> section. Then in the <ins>Getting the Binary Distributions</ins> section click on the download link for your operating system. At the time of writing the latest stable release was apache-activemq-5.13.3. 

<figure>
    <img src="{{ site.url }}/assets/images/jms/apache-activemq-download.png" alt="apache activemq download">
</figure>

# Install ActiveMQ on Windows

Extract the Windows binaries archive that was downloaded in the previous step. The extracted root directory should contain a number of files and subdirectories as shown below. From now on we will refer to this directory as: '<var>[activemq_win_install_dir]</var>'. 

<figure>
    <img src="{{ site.url }}/assets/images/jms/windows-apache-activemq-files.png" alt="windows apache activemq files">
</figure>














