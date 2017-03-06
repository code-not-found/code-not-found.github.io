---
title: Maven - Download & Install Maven 3.2 on Windows
permalink: /2014/06/maven-download-install-apache-maven-3-2-windows.html
excerpt: A detailed tutorial on how to download and install Maven 3.2.2 on Windows.
date: 2014-06-28 21:00
categories: [Apache Maven]
tags: [Apache Maven, Apache Maven 3.2.2, Download, Install, Maven, Tutorial, Windows]
redirect_from:
  - /2014/06/maven-install-windows.html
  - /2014/06/maven-install-maven.html
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

Extract the binaries archive downloaded in the previous step. The extracted root directory should contain a number of files and subdirectories as shown below. From now on we will refer to this directory as: <var>[maven_install_dir]</var>.

<figure>
    <img src="{{ site.url }}/assets/images/maven/maven-install-location.png" alt="maven install location">
</figure>

# Maven Configuration

Next we need to setup a <var>'M2_HOME'</var> environment variable that will point to the installed Maven runtime. In addition if we want to run Maven from a command prompt we need to setup the <var>'PATH'</var> environment variable to contain the Maven bin directory.

When using Windows the above parameters can be configured on the Environment Variables panel. Click on the <kbd>Windows Start</kbd> button and enter "<kbd>env</kbd>" without quotes as shown below.

<figure>
    <img src="{{ site.url }}/assets/images/maven/edit-environment-variables.png" alt="edit environment variables">
</figure>

Environment variables can be set at account level or at system level. For this example click on <kbd>Edit environment variables for your account</kbd> and following panel should appear.

<figure>
    <img src="{{ site.url }}/assets/images/maven/environment-variables.png" alt="environment variables">
</figure>

Click on the <kbd>New</kbd> button and enter "<kbd>M2_HOME</kbd>" as variable name and the "<kbd>[maven_install_dir]</kbd>" as variable value. In this tutorial the installation directory is <var>'C:\source4code\tools\apache-maven-3.2.2'</var>. Click <kbd>OK</kbd> to to save.

> Note that "M2_HOME" is used for Maven 2 and later. "MAVEN_HOME" is for Maven 1.

<figure>
    <img src="{{ site.url }}/assets/images/maven/maven-home-user-variable.png" alt="maven home user variable">
</figure>

Select the <var>'PATH'</var> entry and click on the <kbd>Edit</kbd> button. Add "<kbd>;%M2_HOME%\bin</kbd>" at the end of the variable value and click <kbd>OK</kbd> to save.

> Note that in case a <var>'PATH'</var> variable does not exist you can create it and use "<kbd>%M2_HOME%\bin</kbd>" as the variable value.

<figure>
    <img src="{{ site.url }}/assets/images/maven/maven-path-user-variable.png" alt="maven path user variable">
</figure>

The result should be as shown below. Click OK to close the Environment Variables panel.

<figure>
    <img src="{{ site.url }}/assets/images/maven/environment-variables-maven-configuration.png" alt="environment variables maven configuration">
</figure>

In order to test the above configuration, open a command prompt by clicking on the <kbd>Windows Start</kbd> button and typing "<kbd>cmd</kbd>" followed by pressing <kbd>ENTER</kbd>. A new command prompt should open in which the following command can be entered to verify the installed Maven version:

``` plaintext
mvn -version
```
The result should be that the Maven version is printed as shown below.

<figure>
    <img src="{{ site.url }}/assets/images/maven/maven-version-command.png" alt="maven version command">
</figure>

# Maven Usage

Let's finish the tutorial by creating a basic Maven HelloWorld project. Open a command prompt and navigate to the directory in which you want to create the project. In the example below we will use <var>C:\source4code\code</var>. Next enter following Maven command and press <kbd>ENTER</kbd>.

``` plaintext
mvn archetype:generate -DgroupId=info.source4code -DartifactId=hello-world -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

Maven will start looking for the needed dependencies in order to create the project which is based on the <var>'maven-archetype-quickstart'</var> archetype (= a Maven project templating toolkit). If needed, dependencies are downloaded to the local repository. Once all dependencies are resolved, the project is created and a <var>'BUILD SUCCESS'</var> statement is shown.

Feel free to browse the created <var>hello-world</var> directory. At the root you should be able to find the <var>pom.xml</var>, which is the XML representation of the Maven project.

<figure>
    <img src="{{ site.url }}/assets/images/maven/maven-create-new-project.png" alt="maven create new project">
</figure>

---

This concludes the setting up and configuring Maven. If you found this post helpful or have any questions or remarks, please leave a comment. 