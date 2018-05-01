---
title: "Maven - Download & Install Apache Maven 3.5 on Windows"
permalink: /maven-download-install-apache-maven-3-5-windows.html
excerpt: "A detailed tutorial on how to download and install Maven 3.5.3 on Windows."
date: 2018-05-01
last_modified_at: 2018-05-01
header:
  teaser: "assets/images/teaser/maven-teaser.png"
categories: [Apache Maven]
tags: [Apache Maven, Apache Maven 3.5, Download, Install, Maven, Tutorial, Windows]
published: true
---

<figure>
    <img src="{{ site.url }}/assets/images/logo/maven-logo.png" alt="maven logo" class="logo">
</figure>

[Maven](https://maven.apache.org/){:target="_blank"} is a build automation tool used primarily for Java projects. Maven addresses two aspects of building software:
1. It describes **how software is built**.
2. It describes **its dependencies**.

The following tutorial shows how to download, install and configure Apache Maven 3.5.3 on Windows.

# Maven Prerequisites

As Maven needs [Java](https://java.com/en/download/){:target="_blank"} in order to work, make sure that [a Java runtime environment (JRE) is installed and configured on your system]({{ site.url }}/java-download-install-jdk-8-windows.html).

Maven 3.3+ requires JDK 1.7 or above to execute.

In order to check if Java is available on your system, open a command prompt and execute the following statement.

``` plaintext
java -version
```

If a Java Runtime Environment (JRE) is correctly installed and configured the version will be printed as shown below.

<figure>
    <img src="{{ site.url }}/assets/images/posts/maven/installed-java-version.png" alt="installed java version">
</figure>

# Maven Download & Install

Head over to the [Maven download page](https://maven.apache.org/download.cgi){:target="_blank"} and locate <var>Files</var> section. Download the binary ZIP file, at the time of writing it was <var>apache-maven-3.5.3-bin.zip</var>.

> Here is the direct link to [download the apache-maven-3.5.3 binary ZIP file for Windows 32 or 64 bit](http://www-us.apache.org/dist/maven/maven-3/3.5.3/binaries/apache-maven-3.5.3-bin.zip){:target="_blank"}.

<figure>
    <img src="{{ site.url }}/assets/images/posts/maven/maven-3-5-download-page.png" alt="maven 3.5 download page">
</figure>

Extract the binary archive downloaded in the previous step. The extracted root directory should contain a number of files and subdirectories as shown below.

In this example the install location is <var>'C:\tools\apache-maven-3.5.3'</var>. From now on we will refer to this directory as: <var>[maven_install_dir]</var>.

<figure>
    <img src="{{ site.url }}/assets/images/posts/maven/maven-3-5-install-location.png" alt="maven 3.5 install location">
</figure>

# Maven Configuration

Next we need to setup a <var>'M2_HOME'</var> environment variable that will point to the installed Maven runtime. In addition, if we want to run Maven from a command prompt, we need to configure the <var>'PATH'</var> environment variable to contain the Maven bin directory.

When using Windows the above parameters can be configured on the Environment Variables panel. Click on the <var>Windows Start</var> button and enter "<kbd>env</kbd>" without quotes as shown below.

<figure>
    <img src="{{ site.url }}/assets/images/posts/maven/edit-environment-variables.png" alt="edit environment variables">
</figure>

Environment variables can be set at account level or at system level. For this example click on <var>Edit environment variables for your account</var> and following panel should appear.

<figure>
    <img src="{{ site.url }}/assets/images/posts/maven/environment-variables.png" alt="environment variables">
</figure>

Click on the <var>New</var> button and enter "<kbd>M2_HOME</kbd>" as variable name and the "<kbd>[maven_install_dir]</kbd>" as variable value. In this tutorial the installation directory is <var>'C:\tools\apache-maven-3.5.3'</var>. Click <var>OK</var> to to save.

> Note that "M2_HOME" is used for Maven 2 and later. "MAVEN_HOME" is for Maven 1.

<figure>
    <img src="{{ site.url }}/assets/images/posts/maven/maven-3-5-set-home.png" alt="maven 3.5 set home">
</figure>

Select the <var>'PATH'</var> entry and click on the <var>Edit</var> button. Add "<kbd>;%M2_HOME%\bin;</kbd>" at the end of the variable value and click <var>OK</var> to save.

> Note that in case a <var>'PATH'</var> variable does not exist you can create it and use "<kbd>%M2_HOME%\bin;</kbd>" as the variable value.

<figure>
    <img src="{{ site.url }}/assets/images/posts/maven/maven-set-path.png" alt="maven set path">
</figure>

The result should be as shown below. Click <var>OK</var> to close the Environment Variables panel.

<figure>
    <img src="{{ site.url }}/assets/images/posts/maven/maven-3-5-environment-variables.png" alt="maven 3.5 environment variables">
</figure>

In order to test the above configuration, open a command prompt by clicking on the <var>Windows Start</var> button and typing "<kbd>cmd</kbd>" followed by pressing <var>ENTER</var>. A new command prompt should open in which the following command can be entered to verify the installed Maven version:

``` plaintext
mvn -version
```
The result should be that the Maven version is printed as shown below.

<figure>
    <img src="{{ site.url }}/assets/images/posts/maven/maven-3-5-version.png" alt="maven 3.5 version">
</figure>

# Maven Usage

Let's finish the tutorial by creating a basic Maven HelloWorld project.

Open a command prompt and navigate to the directory in which you want to create the project. In the example below we will use <var>C:\Users\codenotfound</var>.

Next enter following Maven command and press <var>ENTER</var>.

``` plaintext
mvn archetype:generate -DgroupId=com.codenotfound -DartifactId=hello-world -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

Maven will start looking for the needed dependencies in order to create the project which is based on the <var>'maven-archetype-quickstart'</var> archetype (= a Maven project templating toolkit). If needed, dependencies are downloaded to the local repository. Once all dependencies are resolved, the project is created and a <var>'BUILD SUCCESS'</var> statement is shown.

Feel free to browse the created <var>hello-world</var> directory. At the root you should be able to find the <var>pom.xml</var>, which is the XML representation of the Maven project.

<figure>
    <img src="{{ site.url }}/assets/images/posts/maven/maven-3-5-create-new-project.png" alt="maven 3.5 create new project">
</figure>

---

This concludes the setting up and configuring Maven 3.5.

If you found this post helpful or have any questions or remarks, please leave a comment.
