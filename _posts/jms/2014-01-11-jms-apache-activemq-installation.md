---
title: JMS - Apache ActiveMQ Installation
permalink: /2014/01/jms-apache-activemq-installation.html
excerpt: A step-by-step tutorial on how to install and run Apache ActiveMQ on Windows or Unix.
date: 2014-01-11 21:00
categories: [JMS]
tags: [ActiveMQ, Apache ActiveMQ, Installation, Java, JMS, Unix, Windows]
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/apache-activemq-logo.png" alt="apache activemq logo">
</figure>

[Apache ActiveMQ](http://activemq.apache.org/) is an open source message broker written in Java that offers JMS, REST and WebSocket interfaces. It supports protocols like AMQP, MQTT, OpenWire and STOMP that can be used by applications in different languages. Following tutorial details how to install ActiveMQ on Windows or Unix and perform a start/stop of the installed instance.


 First thing to do is to download the ActiveMQ binaries. Go the the [ActiveMQ download page](http://activemq.apache.org/download.html) and click on the latest stable release link in the <file>Latest Releases</file> section. Then in the <file>Getting the Binary Distributions</file> section click on the download link for your operating system. At the time of writing the latest stable release was apache-activemq-5.13.3.

<figure>
    <img src="{{ site.url }}/assets/images/jms/apache-activemq-download.png" alt="apache activemq download">
</figure>

# Install ActiveMQ on Windows

Extract the Windows binaries archive that was downloaded in the previous step. The extracted root directory should contain a number of files and subdirectories as shown below. From now on we will refer to this directory as: '<var>[activemq_win_install_dir]</var>'.

<figure>
    <img src="{{ site.url }}/assets/images/jms/windows-apache-activemq-files.png" alt="windows apache activemq files">
</figure>

Open a command prompt and navigate to <file>[activemq_win_install_dir]</file>. Change to the <file>bin</file> subdirectory and execute the following command to start ActiveMQ: 

``` plaintext
activemq start
```

By default ActiveMQ will generate a number of log statements at start-up as shown below:

<figure>
    <img src="{{ site.url }}/assets/images/jms/windows-apache-activemq-startup-trace.png" alt="windows apache activemq startup trace">
</figure>

One of the latest log entries will mention '<var>ActiveMQ WebConsole available at http://0.0.0.0:8161/</var>'. This indicates that ActiveMQ was successfully started.

# Install ActiveMQ on Unix

Extract the Unix binaries archive downloaded in the first step. The extracted root directory should contain a number of files and subdirectories as shown below. From now on we will refer to this directory as: '<var>[activemq_unix_install_dir]</var>'.

<figure>
    <img src="{{ site.url }}/assets/images/jms/unix-apache-activemq-files.png" alt="unix apache activemq files">
</figure>

Open a terminal and navigate to <file>[activemq_unix_install_dir]</file>. Change to the <file>bin</file> subdirectory and execute the following command to start ActiveMQ as a foreground process:

``` plaintext
./activemq console
```

The ActiveMQ broker can also be started as a background process (note that the corresponding process identifier is stored in the <file>[activemq_unix_install_dir]/data</file> directory for future reference). In order to achieve this, execute the following command instead of the above (hit "<kbd>CTRL+C</kbd>" first if you already started using a foreground process).

``` plaintext
./activemq start
```

<figure>
    <img src="{{ site.url }}/assets/images/jms/unix-apache-activemq-start.png" alt="unix apache activemq start">
</figure>

Once started as a background process there are a number of additional commands we can run to manage the running broker instance. For example, execute following to see if ActiveMQ is still running:

``` plaintext
./activemq status
```

<figure>
    <img src="{{ site.url }}/assets/images/jms/unix-apache-activemq-status.png" alt="unix apache activemq status">
</figure>

The ActiveMQ web site contains a [complete overview of the available Unix commands](http://activemq.apache.org/unix-shell-script.html#UnixShellScript-Functionaloverview). In order to stop the background process, use the below command, but for now leave the broker running as we will first explorer the web console.

``` plaintext
./activemq stop
```

<figure>
    <img src="{{ site.url }}/assets/images/jms/unix-apache-activemq-stop.png" alt="unix apache activemq stop">
</figure>

Note that if you would like ActiveMQ to start automatically at system startup you would need to [run the ActiveMQ broker as a Unix deamon process](http://activemq.apache.org/unix-shell-script.html#UnixShellScript-Runningactivemqasaunixdaemon).

# ActiveMQ WebConsole

The ActiveMQ Web Console is a web based administration tool for working with ActiveMQ. The console can be accessed by entering following URL in a web browser: [http://localhost:8161/](http://localhost:8161/). If ActiveMQ is up and running and installed successfully, following welcome page should be displayed:

<figure>
    <img src="{{ site.url }}/assets/images/jms/apache-activemq-console.png" alt="apache activemq console">
</figure>

Click on the <file>Manage ActiveMQ broker</file> link and enter following default credentials: User name="<kbd>admin</kbd>" and Password="<kbd>admin</kbd>". A home page will be displayed that shows some basic statistics on the ActiveMQ broker. In addition it contains a number of menus that allow you to explore the different configuration items (queues, topics, connections, ...) of the broker.

<figure>
    <img src="{{ site.url }}/assets/images/jms/apache-activemq-console-welcome.png" alt="apache activemq console welcome">
</figure>

Let's finish this tutorial by stopping the running ActiveMQ instance. Switch back to the console in which ActiveMQ was started and press "<kbd>CTRL+C</kbd>" (note that if ActiveMQ was started as a background process, the stop command needs to be run instead). If needed type "<kbd>Y</kbd>" when prompted to '<var>Terminate batch job</var>' followed by "<kbd>ENTER</kbd>". The console will return to the prompt as shown below and ActiveMQ is stopped. 

<figure>
    <img src="{{ site.url }}/assets/images/jms/apache-activemq-stop.png" alt="apache activemq stop">
</figure>

This concludes setting up and configuring Apache ActiveMQ. If you found this post helpful or have any questions or remarks, please leave a comment below.