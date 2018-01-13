---
title: "Java - Download & Install JDK 1.9 on Windows"
permalink: /java-download-install-jdk-9-windows.html
excerpt: "A detailed step-by-step tutorial on how to download and install jdk 9.0.1 on Windows."
date: 2018-01-13
last_modified_at: 2018-01-13
header:
  teaser: "assets/images/teaser/java-teaser.png"
categories: [Java]
tags: [Download, Install, Java, JDK, jdk 9, Tutorial, Windows]
published: true
---

<figure>
    <img src="{{ site.url }}/assets/images/logo/java-logo.png" alt="java logo" class="logo">
</figure>

[Java](https://www.java.com/en/){:target="_blank"} is a computer programming language that is concurrent, class-based and object-oriented. It was originally developed by James Gosling at Sun Microsystems. Java applications are compiled to bytecode (class file) that can run on any Java virtual machine (JVM) regardless of computer architecture.

Java is currently owned by the Oracle Corporation which acquired Sun Microsystems in 2010. Following tutorial will show you how to setup and configure Java 1.9 on Windows so you can develop and run Java code.

Check following posts if you are looking to download and install [JDK 1.5]({{ site.url }}/java-download-install-jdk-5-windows.html), [JDK 1.6]({{ site.url }}/java-download-install-jdk-6-windows.html), [JDK 1.7]({{ site.url }}/java-download-install-jdk-7-windows.html) or [JDK 1.8]({{ site.url }}/java-download-install-jdk-8-windows.html).
{: .notice--primary}

# JDK Download & Install

Java can be obtained from the Oracle Java download page. There are a number of [different Java packages available](https://docs.oracle.com/javaee/6/firstcup/doc/gkhoy.html){:target="_blank"}, for this tutorial we will be installing Java Standard Edition (SE) on Windows.

In order to be able to compile Java code, we need the Java Development Kit (JDK) package that comes with a Java compiler. The JDK package also comes with a Java runtime environment (JRE) that is needed to run compiled Java code.

As we are installing the latest Java version, you can find it at the top of the [Oracle Java download page](http://www.oracle.com/technetwork/java/javase/downloads/index.html){:target="_blank"}. Click on the <var>Download</var> button right below the <var>JDK</var>.

> Here is the direct link to [download the jdk 9.0.1 installer for Windows 64 bit](http://www.oracle.com/technetwork/java/javase/downloads/jdk9-downloads-3848520.html){:target="_blank"}.

Accept the License Agreement and pick the correct download for your operating system. In this example, we will use the Windows 64 bit version.

<figure>
    <img src="{{ site.url }}/assets/images/posts/java/java-9-download-jdk.png" alt="java 9 download jdk">
</figure>

Sign in using your Oracle account (or create a new one) and the download should start. Once the download is complete, locate the <var>jdk-9.0.1_windows-x64_bin.exe</var> file and double-click to run the installer.

<figure>
    <img src="{{ site.url }}/assets/images/posts/java/java-9-installer-start.png" alt="java 9 installer start">
</figure>

Click <var>Next</var> and on the following screen optionally change the installation location by clicking on the <var>Change...</var> button. In this example the install location was changed to <var>'C:\Java\jdk-9.0.1'</var>. From now on we will refer to this directory as: <var>[java_install_dir]</var>.

<figure>
    <img src="{{ site.url }}/assets/images/posts/java/java-9-jdk-location.png" alt="java 9 jdk location">
</figure>

We will not install the public JRE as the JDK Development tools include a private JRE that can run developed code. Select the <var>Public JRE</var> dropdown and click on <var>This feature will not be available.</var> as shown below.

<figure>
    <img src="{{ site.url }}/assets/images/posts/java/java-9-public-jre-location.png" alt="java 9 public jre location">
</figure>

Click <var>Next</var> and then <var>Close</var> to finish installing Java.

<figure>
    <img src="{{ site.url }}/assets/images/posts/java/java-9-installer-finish.png" alt="java 9 installer finish">
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

Click on the <var>New</var> button and enter "<kbd>JAVA_HOME</kbd>" as variable name and the <var>[java_install_dir]</var> as variable value. In this tutorial the installation directory is <var>'C:\Java\jdk-9.0.1'</var>. Click <var>OK</var> to to save.

<figure>
    <img src="{{ site.url }}/assets/images/posts/java/java-9-set-home.png" alt="java 9 set home">
</figure>

Click on the <var>New</var> button and enter "<kbd>PATH</kbd>" as variable name and "<kbd>%JAVA_HOME%\bin</kbd>" as variable value. Click <var>OK</var> to save.

> Note that in case a <var>'PATH'</var> variable is already present you can add "<kbd>;%JAVA_HOME%\bin</kbd>" at the end of the variable value.

<figure>
    <img src="{{ site.url }}/assets/images/posts/java/java-set-path.png" alt="java set path">
</figure>

The result should be as shown below. Click <var>OK</var> to close the environment variables panel.

<figure>
    <img src="{{ site.url }}/assets/images/posts/java/java-9-environment-variables.png" alt="java 9 environment variables">
</figure>

In order to test the above configuration, open a command prompt by clicking on the Windows Start button and typing "<kbd>cmd</kbd>" followed by pressing <var>ENTER</var>. A new command prompt should open in which the following command can be entered to verify the installed Java version:

``` plaintext
java -version
```

The result should be as shown below.

<figure>
    <img src="{{ site.url }}/assets/images/posts/java/java-9-version.png" alt="java 9 version">
</figure>

---

This concludes the setting up and configuring JDK 1.9 on Windows.

If you found this post helpful or have any questions or remarks, please leave a comment.
