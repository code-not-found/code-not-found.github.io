---
title: "Java - Download & Install JDK 1.5 on Windows"
permalink: /java-download-install-jdk-5-windows.html
excerpt: "A detailed tutorial on how to download and install jdk 1.5.0_22 on Windows."
date: 2017-11-19
last_modified_at: 2017-11-19
header:
  teaser: "assets/images/teaser/java-teaser.png"
categories: [Java]
tags: [Download, Install, Java, JDK, jdk 1.5.0_22, Tutorial, Windows]
redirect_to:
  - https://downlinko.com/download-install-jdk-5-windows.html
published: true
---

<figure>
    <img src="{{ site.url }}/assets/images/logo/java-logo.png" alt="java logo" class="logo">
</figure>

[Java](https://www.java.com/en/){:target="_blank"} is a computer programming language that is concurrent, class-based and object-oriented. It was originally developed by James Gosling at Sun Microsystems. Java applications are compiled to bytecode (class file) that can run on any Java virtual machine (JVM) regardless of computer architecture.

Java is currently owned by the Oracle Corporation which acquired Sun Microsystems in 2010. Following tutorial will show you how to setup and configure Java 1.5 on Windows so you can develop and run Java code.

Check following posts if you are looking to download and install [JDK 1.6]({{ site.url }}/java-download-install-jdk-6-windows.html), [JDK 1.7]({{ site.url }}/java-download-install-jdk-7-windows.html), [JDK 1.8]({{ site.url }}/java-download-install-jdk-8-windows.html), [JDK 1.9]({{ site.url }}/java-download-install-jdk-9-windows.html) or [JDK 1.10]({{ site.url }}/java-download-install-jdk-10-windows.html).
{: .notice--primary}

# JDK Download & Install

Java can be obtained from the Oracle Java download page. There are a number of [different Java packages available](https://docs.oracle.com/javaee/6/firstcup/doc/gkhoy.html){:target="_blank"}, for this tutorial we will be installing Java Standard Edition (SE) on Windows.

In order to be able to compile Java code, we need the Java Development Kit (JDK) package that comes with a Java compiler. The JDK package also comes with a Java runtime environment (JRE) that is needed to run compiled Java code.

As we are installing an older Java version, you need to scroll all the way down to the bottom of the [Oracle Java download page](http://www.oracle.com/technetwork/java/javase/downloads/index.html){:target="_blank"} and click on the <var>Download</var> button in the <var>Java Archive</var> section. Then look for the <var>Java SE 5</var> link and after clicking on it, select the correct operating system under <var>Java SE Development Kit 5.0u22</var>.

> Here is the direct link to [download the jdk 1.5.0_22 installer for Windows](http://www.oracle.com/technetwork/java/javasebusiness/downloads/java-archive-downloads-javase5-419410.html){:target="_blank"}.

Accept the License Agreement and pick the correct download for your operating system. In this example, we will use the Windows 64 bit version.

<figure>
    <img src="{{ site.url }}/assets/images/posts/java/java-5-download-jdk.png" alt="java 5 download jdk">
</figure>

Sign in using your Oracle account (or create a new one) and the download should start. Once the download is complete, locate the <var>jdk-1_5_0_22-windows-i586-p.exe</var> file and double-click to run the installer.

<figure>
    <img src="{{ site.url }}/assets/images/posts/java/java-5-license-agreement.png" alt="java 5 license agreement">
</figure>

Click on the <var>I accept the terms in the license agreement</var> radio button and then click on <var>Next</var>. On the following screen optionally change the installation location by clicking on the <var>Change...</var> button. In this example the install location was changed to <var>'C:\Java\jdk1.5.0_22'</var>. From now on we will refer to this directory as: <var>[java_install_dir]</var>.

<figure>
    <img src="{{ site.url }}/assets/images/posts/java/java-5-jdk-location.png" alt="java 5 jdk location">
</figure>

Next, the installer will present the installation of some optional features. We will skip this part of the installer as the JDK installed in the previous step comes with everything to run and develope code. Just press <var>Cancel</var> and confirm by clicking <var>Yes</var> in the popup window.

<figure>
    <img src="{{ site.url }}/assets/images/posts/java/java-5-optional-features.png" alt="java 5 optional features">
</figure>

Click <var>Next</var> and then <var>Close</var> to finish installing Java.

<figure>
    <img src="{{ site.url }}/assets/images/posts/java/java-5-installer-finish.png" alt="java 5 installer finish">
</figure>

# JDK Configuration

In order for Java applications to be able to run we need to setup a <var>'JAVA_HOME'</var> environment variable that will point to the Java installation directory. In addition, if we want to run Java commands from a command prompt we need to setup the <var>'PATH'</var> environment variable to contain the Java bin directory.

When using Windows the above parameters can be configured on the Environment Variables panel. Click on the <var>Windows Start</var> button and enter "<kbd>env</kbd>" without quotes as shown below.

<figure>
    <img src="{{ site.url }}/assets/images/posts/java/edit-environment-variables.png" alt="edit environment variables">
</figure>

Environment variables can be set at account level or at system level. For this example click on <var>Edit environment variables for your account</var> and following panel should appear.

<figure>
    <img src="{{ site.url }}/assets/images/posts/java/environment-variables.png" alt="environment variables">
</figure>

Click on the <var>New</var> button and enter "<kbd>JAVA_HOME</kbd>" as variable name and the <var>[java_install_dir]</var> as variable value. In this tutorial the installation directory is <var>'C:\Java\jdk1.5.0_22'</var>. Click <var>OK</var> to to save.

<figure>
    <img src="{{ site.url }}/assets/images/posts/java/java-5-set-home.png" alt="java 5 set home">
</figure>

Click on the <var>New</var> button and enter "<kbd>PATH</kbd>" as variable name and "<kbd>%JAVA_HOME%\bin</kbd>" as variable value. Click <var>OK</var> to save.

> Note that in case a <var>'PATH'</var> variable is already present you can add "<kbd>;%JAVA_HOME%\bin</kbd>" at the end of the variable value.

<figure>
    <img src="{{ site.url }}/assets/images/posts/java/java-set-path.png" alt="java set path">
</figure>

The result should be as shown below. Click <var>OK</var> to close the environment variables panel.

<figure>
    <img src="{{ site.url }}/assets/images/posts/java/java-5-environment-variables.png" alt="java 5 environment variables">
</figure>

In order to test the above configuration, open a command prompt by clicking on the Windows Start button and typing "<kbd>cmd</kbd>" followed by pressing <var>ENTER</var>. A new command prompt should open in which the following command can be entered to verify the installed Java version:

``` plaintext
java -version
```

The result should be as shown below.

<figure>
    <img src="{{ site.url }}/assets/images/posts/java/java-5-version.png" alt="java 5 version">
</figure>

---

This concludes the setting up and configuring JDK 1.5 on Windows.

If you found this post helpful or have any questions or remarks, please leave a comment.
