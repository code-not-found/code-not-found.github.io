---
title: Maven - Download & Install Maven 3.2 on Windows
permalink: /2014/06/maven-download-install-apache-maven-3-2-windows.html
excerpt: A detailed tutorial on how to download and install Maven 3.2.2 on Windows.
date: 2014-06-28 21:00
categories: [Apache Maven]
tags: [Apache Maven, Apache Maven 3.2.2, Download, Install, Maven, Tutorial, Windows]
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/maven-logo.png" alt="maven logo">
</figure>

[Maven](https://maven.apache.org/) is a build automation tool used primarily for Java projects. Maven addresses two aspects of building software:
1. It describes how software is built.
2. It describes its dependencies.

Maven uses conventions for the build procedure, and only exceptions need to be written down. Maven is built using a plugin-based architecture that allows it to make use of any application controllable through standard input. The following tutorial will show you how to download, install and configure Apache Maven 3.2.2 on Windows.

# Maven Install

Before starting this tutorial, make sure that [a Java runtime environment (JRE) is installed and configured on your system]({{ site.url }}/2014/06/java-install-jdk-windows.html). A quick way to check this is by opening a command prompt and executing following statement which prints the installed JRE, if correctly installed.

``` plaintext
java -version
```

Head over to the [Maven download page](https://maven.apache.org/download.cgi) and locate <var>Files</var> section. Download the binary ZIP file, at the time of writing it was <var>apache-maven-3.2.2-bin.zip</var>. This version is still available in [the archives section of the Maven site](https://archive.apache.org/dist/maven/maven-3/). In this tutorial we will install on Windows.

<figure>
    <img src="{{ site.url }}/assets/images/maven/maven-download-page.png" alt="maven download page">
</figure>




