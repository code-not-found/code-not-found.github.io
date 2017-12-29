---
title: "Java - Download & Install JDK 1.6 on Windows"
permalink: /java-download-install-jdk-6-windows.html
excerpt: "A detailed step-by-step tutorial on how to download and install jdk1.6.0_45 on Windows."
date: 2014-06-28
last_modified_at: 2014-06-28
header:
  teaser: "assets/images/header/java-teaser.png"
categories: [Java]
tags: [Download, Install, Java, JDK, jdk1.6.0_45, Tutorial, Windows]
redirect_from:
  - /2014/06/java-install-jdk-windows.html
  - /2014/06/java-download-install-jdk
  - /2014/06/java-install-jdk.html
  - /2014/06/java-download-install-jdk-6-windows.html
published: true
---

<figure>
    <img src="{{ site.url }}/assets/images/logo/java-logo.png" alt="java logo" class="logo">
</figure>

[Java](https://www.java.com/en/){:target="_blank"} is a computer programming language that is concurrent, class-based and object-oriented. It was originally developed by James Gosling at Sun Microsystems. Java applications are compiled to bytecode (class file) that can run on any Java virtual machine (JVM) regardless of computer architecture.

Java is currently owned by the Oracle Corporation which acquired Sun Microsystems in 2010. Following tutorial will show you how to setup and configure Java 1.7 on Windows so you can develop and run Java code.

Check following post if you are looking to download and install [JDK 1.5]({{ site.url }}/java-download-install-jdk-5-windows.html) or [JDK 1.7]({{ site.url }}/java-download-install-jdk-7-windows.html).
{: .notice--primary}

# JDK Download & Install

Java can be obtained from the Oracle Java download page. There are a number of [different Java packages available](https://docs.oracle.com/javaee/6/firstcup/doc/gkhoy.html){:target="_blank"}, for this tutorial we will be installing Java Standard Edition (SE) on Windows.

In order to be able to compile Java code, we need the Java Development Kit (JDK) package that comes with a Java compiler. The JDK package also comes with a Java runtime environment (JRE) that is needed to run compiled Java code.

As we are installing an older Java version, you need to scroll all the way down to the bottom of the [Oracle Java download page](http://www.oracle.com/technetwork/java/javase/downloads/index.html){:target="_blank"} and click on the <var>Download</var> button in the <var>Java Archive</var> section. Then look for the <var>Java SE 6</var> link and after clicking on it, select the correct operating system under <var>Java SE Development Kit 6u45</var>.

> Here is the direct link to [download the jdk1.6.0_45 installer for Windows 32 or 64 bit](http://www.oracle.com/technetwork/java/javase/downloads/java-archive-downloads-javase6-419409.html){:target="_blank"}.

Accept the License Agreement and pick the correct download for your operating system. In this example, we will use the Windows 64 bit version.

<figure>
    <img src="{{ site.url }}/assets/images/posts/java/java-6-download-jdk.png" alt="java 6 download jdk">
</figure>

Sign in using your Oracle account (or create a new one) and the download should start. Once the download is complete, locate the <var>jdk-6u45-windows-x64.exe</var> file and double-click to run the installer.

<figure>
    <img src="{{ site.url }}/assets/images/posts/java/java-6-installer-start.png" alt="java 6 installer start">
</figure>

Click <var>Next</var> and on the following screen optionally change the installation location by clicking on the <var>Change...</var> button. In the example below the install location was changed to <var>'C:\Java\jdk1.6.0_45'</var>. From now on we will refer to this directory as: <var>[java_install_dir]</var>. 

<figure>
    <img src="{{ site.url }}/assets/images/posts/java/java-6-jdk-location.png" alt="java 6 jdk location">
</figure>

Next, the installer will present the installation location of the [public JRE](https://docs.oracle.com/javase/8/docs/technotes/guides/install/windows_jdk_install.html#CHDJCCEG){:target="_blank"}. We will skip this part of the installer as the JDK installed in the previous step comes with a private JRE that can run developed code. Just press <var>Cancel</var> and confirm by clicking <var>Yes</var> in the popup window.

<figure>
    <img src="{{ site.url }}/assets/images/posts/java/java-6-public-jre-location.png" alt="java 6 public jre location">
</figure>

Click <var>Next</var> and then <var>Close</var> to finish installing Java.

<figure>
    <img src="{{ site.url }}/assets/images/posts/java/java-6-installer-finish.png" alt="java 6 installer finish">
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

Click on the <var>New</var> button and enter "<kbd>JAVA_HOME</kbd>" as variable name and the <var>[java_install_dir]</var> as variable value. In this tutorial the installation directory is <var>'C:\Java\jdk1.6.0_45'</var>. Click <var>OK</var> to to save.

<figure>
    <img src="{{ site.url }}/assets/images/posts/java/java-6-set-home.png" alt="java 6 set home">
</figure>

Click on the <var>New</var> button and enter "<kbd>PATH</kbd>" as variable name and "<kbd>%JAVA_HOME%\bin</kbd>" as variable value. Click <var>OK</var> to save.

> Note that in case a <var>'PATH'</var> variable is already present you can add "<kbd>;%JAVA_HOME%\bin</kbd>" at the end of the variable value.

<figure>
    <img src="{{ site.url }}/assets/images/posts/java/java-set-path.png" alt="java set path">
</figure>

The result should be as shown below. Click <var>OK</var> to close the environment variables panel.

<figure>
    <img src="{{ site.url }}/assets/images/posts/java/java-6-environment-variables.png" alt="java 6 environment variables">
</figure>

In order to test the above configuration, open a command prompt by clicking on the Windows Start button and typing "<kbd>cmd</kbd>" followed by pressing <var>ENTER</var>. A new command prompt should open in which the following command can be entered to verify the installed Java version:

``` plaintext
java -version
```

The result should be as shown below.

<figure>
    <img src="{{ site.url }}/assets/images/posts/java/java-6-version.png" alt="java 6 version">
</figure>

---

This concludes the setting up and configuring JDK 1.6 on Windows. If you found this post helpful or have any questions or remarks, please leave a comment.
