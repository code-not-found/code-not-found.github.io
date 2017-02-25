---
title: AMQP - Install RabbitMQ on Windows 
permalink: /2014/11/amqp-install-rabbitmq-windows.html
excerpt: A step-by-step tutorial on how to install RabbitMQ on Windows.
date: 2014-11-18 21:00
categories: [AMQP]
tags: [nstall, Java Message Service, JMS, RabbitMQ, Setup, Tutorial]
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











