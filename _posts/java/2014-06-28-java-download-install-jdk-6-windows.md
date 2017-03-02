---
title: Java - Download & Install JDK 1.6 on Windows 
permalink: /2014/06/java-download-install-jdk-6-windows.html
excerpt: A detailed tutorial on how to download and install jdk1.6.0_45 on Windows.
date: 2014-06-28 21:00
categories: [Java]
tags: [Download, Install, Java, JDK, jdk1.6.0_45, Tutorial, Windows]
redirect_from: "/2014/06/java-install-jdk-windows.html"
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/java-logo.png" alt="java logo">
</figure>

[Java](https://www.java.com/en/) is a computer programming language that is concurrent, class-based and object-oriented. It was originally developed by James Gosling at Sun Microsystems. Java applications are typically compiled to bytecode (class file) that can run on any Java virtual machine (JVM) regardless of computer architecture.

Java is currently owned by the Oracle Corporation which acquired Sun Microsystems in 2010. Following tutorial will show you how to setup and configure Java 1.6 on your Windows computer so you can develop and run Java code.

# JDK Download & Install

Java can be downloaded from the [Oracle Java download page](http://www.oracle.com/technetwork/java/javase/downloads/index.html). There are a number of different download packages available, for this tutorial we will be installing Java Standard Edition (SE) on Windows. In order to be able to compile Java code we need the Java Development Kit (JDK) package that comes with a Java compiler. The JDK package also comes with a Java runtime environment (JRE) that is needed to run compiled Java code.

For this tutorial we will use an older Java version which can be obtained by scrolling all the way down to the bottom of the page and clicking on the [Previous Releases - Java Archive link](http://www.oracle.com/technetwork/java/javase/archive-139210.html). Look for the <var>Java SE 6</var> link and after clicking on it select <var>Java SE Development Kit 6u45</var>.

> You can find here the direct link to [download the jdk1.6.0_45 installer for windows 32 or 64 bit](http://www.oracle.com/technetwork/java/javase/downloads/java-archive-downloads-javase6-419409.html#jdk-6u45-oth-JPR).

Accept the License Agreement and pick the correct download for your operating system. In this example we will use the Windows 64 bit version.

<figure>
    <img src="{{ site.url }}/assets/images/java/java-download-jdk.png" alt="java download jdk">
</figure>

Sign in using your Oracle account (or create a new one) and the download should start. Once the download is complete, locate the <var>jdk-6u45-windows-x64.exe</var> file and double click to run the installer.

<figure>
    <img src="{{ site.url }}/assets/images/java/java-installer-start.png" alt="java installer start">
</figure>

Click <var>Next</var> and on the following screen optionally change the installation location by clicking on the <var>Change...</var> button. In the example below the install location was change to <var>'C:\Java\jdk1.6.0_45'</var>. From now on we will refer to this directory as: <var>[java_install_dir]</var>. 

<figure>
    <img src="{{ site.url }}/assets/images/java/java-installer-location.png" alt="java installer location">
</figure>

Click <var>Next</var> and then <var>Close</var> after the installer successfully finished installing Java.

<figure>
    <img src="{{ site.url }}/assets/images/java/java-installer-finish.png" alt="java installer finish">
</figure>

# JDK Configuration

In order for Java applications to be able to run we need to setup a <var>'JAVA_HOME'</var> environment variable that will point to the Java installation directory. In addition if we want to run Java commands from a command prompt we need to setup the <var>'PATH'</var> environment variable to contain the Java bin directory.

When using Windows the above parameters can be configured on the Environment Variables panel. Click on the <var>Windows Start</var> button and enter "<kbd>env</kbd>" without quotes as shown below.

<figure>
    <img src="{{ site.url }}/assets/images/java/edit-environment-variables.png" alt="edit environment variables">
</figure>

Environment variables can be set at account level or at system level. For this example click on <var>Edit environment variables for your account</var> and following panel should appear.

<figure>
    <img src="{{ site.url }}/assets/images/java/environment-variables.png" alt="environment variables">
</figure>

Click on the <var>New</var> button and enter "<kbd>JAVA_HOME</kbd>" as variable name and the <var>[java_install_dir]</var> as variable value. In this tutorial the installation directory is <var>'C:\Java\jdk1.6.0_45'</var>. Click OK to to save.

<figure>
    <img src="{{ site.url }}/assets/images/java/set-java-home.png" alt="set java home">
</figure>

Click on the <var>New</var> button and enter "<kbd>PATH</kbd>" as variable name and "<kbd>%JAVA_HOME%\bin</kbd>" as variable value. Click <var>OK</var> to save.

> Note that in case a <var>'PATH'</var> variable is already present you can add "<kbd>;%JAVA_HOME%\bin</kbd>" at the end of the variable value.

<figure>
    <img src="{{ site.url }}/assets/images/java/set-java-home-path.png" alt="set java home path">
</figure>

The result should be as shown below. Click <var>OK</var> to close the environment variables panel.

<figure>
    <img src="{{ site.url }}/assets/images/java/environment-variables-java.png" alt="environment variables java">
</figure>

In order to test the above configuration, open a command prompt by clicking on the Windows Start button and typing "cmd" followed by pressing ENTER. A new command prompt should open in which the following command can be entered to verify the installed Java version:

``` plaintext
java -version
```

The result should be as shown below.

<figure>
    <img src="{{ site.url }}/assets/images/java/java-version.png" alt="java version">
</figure>

---

This concludes the setting up and configuring JDK 1.6 on Windows. If you found this post helpful or have any questions or remarks, please leave a comment.