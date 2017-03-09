---
title: AMQP - Install RabbitMQ on Windows 
permalink: /2014/11/amqp-install-rabbitmq-windows.html
excerpt: A step-by-step tutorial on how to install RabbitMQ on Windows.
date: 2014-11-18 21:00
categories: [AMQP]
tags: [Install, Java Message Service, JMS, RabbitMQ, Setup, Tutorial]
redirect_from:
  - /2014/11/jms-install-rabbitmq-windows.html
  - /2014/11/jms-install-rabbitmq.html
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/rabbitmq-logo.png" alt="rabbitmq logo">
</figure>

RabbitMQ is an open source message broker software that implements the Advanced Message Queuing Protocol (AMQP). The RabbitMQ server is written in the Erlang programming language and client libraries to interface with the broker are available for all major programming languages. Following tutorial shows how to install RabbitMQ and perform a start/stop of the installed instance on Windows.

# Install Erlang

Erlang is a general-purpose concurrent, garbage-collected programming language and runtime system. It was designed by Ericsson to support distributed, fault-tolerant applications. It was originally a proprietary language within Ericsson, but was released as open source in 1998. OTP (Open Telecom Platform) is the open source distribution of Erlang.

First thing to do is to download the OTP binaries. Go the the [Erlang download page](http://www.erlang.org/download.html) and click on the Windows binary link for your system (32-bit or 64-bit). At the time of writing the latest stable release was <var>otp_win64_17.3.exe</var>. Note that there are also pre-built packages for platforms such as: [Raspbian, Ubuntu, Fedora, OS X, and more](https://www.erlang-solutions.com/downloads/download-erlang-otp).

Double click to run the downloaded <var>'.exe'</var> file and click <var>Next</var> keeping the default settings on the first installer step.

<figure>
    <img src="{{ site.url }}/assets/images/amqp/erlang-installer-choose-components.png" alt="erlang installer choose components">
</figure>

Optionally change the default destination folder and click <var>Next</var> and then <var>Install</var>. In the example below the install location was change to <var>'D:\source4code\tools\erl6.2'</var>. From now on we will refer to this directory as: <var>[erlang_install_dir]</var>.

<figure>
    <img src="{{ site.url }}/assets/images/amqp/erlang-install-location.png" alt="erlang install location">
</figure>

If Microsoft Visual C++ is not already setup on your system, a second installer window will pop-up. Click the <var>'I have read and accept the license terms'</var> check-box and click <var>Install</var>.

<figure>
    <img src="{{ site.url }}/assets/images/amqp/erlang-msvisualc-installer.png" alt="erlang msvisualc installer">
</figure>

Click <var>Finish</var> when the Microsoft Visual C++ setup is complete and then click <var>Close</var> to finish the OTP installation. 

<figure>
    <img src="{{ site.url }}/assets/images/amqp/erlang-installer-finish.png" alt="erlang installer finish">
</figure>

In order for Erlang applications to be able to run we need to setup an <var>'ERLANG_HOME'</var> environment variable that will point to the Erlang installation directory. When using Windows the above parameters can be configured on the Environment Variables panel. Click on the <var>Windows Start</var> button and enter "<kbd>env</kbd>" without quotes as shown below.

<figure>
    <img src="{{ site.url }}/assets/images/amqp/environment-variables.png" alt="environment variables">
</figure>

Environment variables can be set at account level or at system level. For this example click on <var>Edit environment variables for your account</var> and following panel should appear.

<figure>
    <img src="{{ site.url }}/assets/images/amqp/edit-environment-variables.png" alt="edit environment variables">
</figure>

Click on the <var>New</var> button and enter "<kbd>ERLANG_HOME</kbd>" as variable name and the <var>[erlang_install_dir]</var> as variable value. In this tutorial the installation directory is <var>'D:\source4code\tools\erl6.2'</var>. Click OK to to save.

<figure>
    <img src="{{ site.url }}/assets/images/amqp/environment-variable-erlang-home.png" alt="environment variable erlang home">
</figure>

# Install RabbitMQ

RabbitMQ can be downloaded from the [RabbitMQ download page](https://www.rabbitmq.com/download.html). There are a number of different download packages available, for this tutorial we will be installing the [manual install package](https://www.rabbitmq.com/install-windows-manual.html) on Windows.At the time of writing the latest stable release was <var>'rabbitmq-server-windows-3.4.1.zip'</var>.

Extract the binaries archive downloaded in the previous step. The extracted root directory should contain a number of files and subdirectories as shown below. From now on we will refer to this directory as: <var>[rabbitmq_install_dir]</var>.

In order to start RabbitMQ, open a command prompt by clicking on the Windows Start button and typing "<kbd>cmd</kbd>" followed by pressing <var>ENTER</var>. A new command prompt window should open. Navigate to the <var>[rabbitmq_install_dir]/sbin</var> and enter following command:

``` plaintext
rabbitmq-server
```

<figure>
    <img src="{{ site.url }}/assets/images/amqp/rabbitmq-start.png" alt="rabbitmq start">
</figure>

In order to stop RabbitMQ, open another command prompt at the [rabbitmq_install_dir]/sbin and enter following command:

``` plaintext
rabbitmqctl stop
```

<figure>
    <img src="{{ site.url }}/assets/images/amqp/rabbitmq-stop.png" alt="rabbitmq stop">
</figure>

# Setup RabbitMQ

The <var>'rabbitmq-management'</var> plugin provides a browser-based UI for management and monitoring of the RabbitMQ server . In order to enable the UI, make sure RabbitMQ is running and open a new command prompt at <var>[rabbitmq_install_dir]/sbin</var> in which you enter following:

``` plaintext
rabbitmq-plugins enable rabbitmq_management
```

<figure>
    <img src="{{ site.url }}/assets/images/amqp/rabbitmq-enable-management-console.png" alt="rabbitmq enable management console">
</figure>

Open the RabbitMQ web console in a browser using: [http://localhost:15672](http://localhost:15672) and following page should be displayed:

<figure>
    <img src="{{ site.url }}/assets/images/amqp/rabbitmq-management-console-login.png" alt="rabbitmq management console login">
</figure>

Enter following default credentials: Username="<kbd>guest</kbd>" and Password="<kbd>guest</kbd>" and click on <var>Login</var>. The overview page will be displayed that shows some basic information on the RabbitMQ server:

<figure>
    <img src="{{ site.url }}/assets/images/amqp/rabbitmq-management-console.png" alt="rabbitmq management console">
</figure>

---

This concludes setting up and configuring RabbitMQ. If you found this post helpful or have any questions or remarks, please leave a comment. 