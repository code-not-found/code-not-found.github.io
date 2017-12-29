---
title: "JMS - HermesJMS Download and Installation"
permalink: /jms-hermesjms-download-installation.html
excerpt: "A step-by-step tutorial on how to download and install HermesJMS on Windows or Unix."
date: 2014-01-11
last_modified_at: 2014-01-11
header:
  teaser: "assets/images/teaser/hermesjms-teaser.png"
categories: [JMS]
tags: [Configuration, Download, Hermes JMS, HermesJMS, Installation, Java, JMS, Tutorial, Unix, Windows]
redirect_from:
  - /2014/01/jms-install-hermesjms-windows.html
  - /2014/01/jms-installing-hermesjms.html
  - /2014/01/jms-hermesjms-download-installation.html
published: true
---

<figure>
    <img src="{{ site.url }}/assets/images/logo/hermesjms-logo.png" alt="hermesjms logo" class="logo">
</figure>

[HermesJMS](http://hermesjms.com/){:target="_blank"} is a graphical user interface that helps you interact with many of the popular JMS providers. It allows to publish and edit messages, browse or search queues and topics, copy messages around and delete them. Hermes JMS is an open source project hosted by [Sourceforge](https://sourceforge.net/projects/hermesjms/).

Following tutorial shows how to install Hermes JMS on Windows or Unix and perform a start/stop of the application.

In this guide we will be covering following steps:
* [Prerequisite & Downloading the Installer]({{ site.url }}/jms-hermesjms-download-installation.html#prerequisites--downloading-the-installer)
* [Installation Procedure for Windows & Unix]({{ site.url }}/jms-hermesjms-download-installation.html#installation-procedure-for-windows--unix)
* [Startup & Exit on Windows]({{ site.url }}/jms-hermesjms-download-installation.html#startup--exit-on-windows)
* [Startup & Exit on Unix]({{ site.url }}/jms-hermesjms-download-installation.html#startup--exit-on-unix)
* [Changing the Default Configuration Location on Windows]({{ site.url }}/jms-hermesjms-download-installation.html#changing-the-default-configuration-location-on-windows)
* [Changing the Default Configuration File on Unix]({{ site.url }}/jms-hermesjms-download-installation.html#changing-the-default-configuration-file-on-unix)

# Prerequisites & Downloading the Installer

Hermes JMS is written in Java. It needs [a Java runtime environment (JRE) of version 1.6 or higher to be installed and configured](http://www.oracle.com/technetwork/java/javase/downloads/index.html){:target="_blank"} (with JAVA_HOME correctly set). If you are not sure what Java version is installed on your machine, open a console and execute following command:

``` plaintext
java -version
```

Let's get started by downloading the HermesJMS installer Java archive (JAR). Go the the [HermesJMS download page](http://www.hermesjms.com/confluence/display/HJMS/Home){:target="_blank"} and click on the Sourceforge link in the <var>Downloading and Webstarting</var> section. This will redirect to a Sourceforge page, click on the <var>hermes-installer-X.XX.jar</var> link and download the JAR file.

Seeing as Hermes JMS is written in Java, the installer will run on both 32-bit and 64-bit systems.

> Update: it looks like the HermesJMS home page is currently not available. You can however still [download the latest stable release: 'hermes-installer-1.14.jar' from SourceForge](http://sourceforge.net/projects/hermesjms/files/hermesjms/1.14/hermes-installer-1.14.jar/download){:target="_blank"}.

# Installation Procedure for Windows & Unix

Open a console window and navigate to the location of the downloaded <var>hermes-installer-1.14.jar</var>. Execute the following command to start the Hermes JMS installer:

``` plaintext
java -jar hermes-installer-1.14.jar
```
<figure>
    <img src="{{ site.url }}/assets/images/posts/jms/hermesjms-install.png" alt="hermesjms install">
</figure>

A HermesJMS installation window will open as shown below:

<figure>
    <img src="{{ site.url }}/assets/images/posts/jms/hermesjms-welcome.png" alt="hermesjms welcome">
</figure>

Clicking <var>Next</var> will show some best practices on how to use HermesJMS configuration files (which we will cover later in this tutorial). Click <var>Next</var> again and review the license agreement. Select the <var>I accept the terms of this license agreement</var> radio button in order to be able to continue the installation.

<figure>
    <img src="{{ site.url }}/assets/images/posts/jms/hermesjms-license.png" alt="hermesjms license">
</figure>

Click <var>Next</var> and change the default installation path (if needed). If prompted to create the target directory, click <var>OK</var>. From now on we will refer to this directory as: <var>[hermesjms_install_dir]</var>. 

> Do **not** install Hermes with a **path with spaces** in, some JNDI implementations bundled with JMS providers have problems with this.

<figure>
    <img src="{{ site.url }}/assets/images/posts/jms/hermesjms-path.png" alt="hermesjms path">
</figure>

Click <var>Next</var> twice and review the installation settings (there should be a single HermesJMS installation pack that is selected by default). If the settings are correct click <var>Next</var> to start the installation. The progress of the installation will be indicated by a progress bar as shown below.

<figure>
    <img src="{{ site.url }}/assets/images/posts/jms/hermesjms-installation-progress.png" alt="hermesjms installation progress">
</figure>

Once the <var>'Pack installation progress'</var> bar mentions mentions <var>'[Finished]'</var>, click on <var>Quit</var> to exit the installer. Alternatively click <var>Next</var> twice to also create application shortcuts for starting and uninstalling HermesJMS.

> When installing Hermes JMS on a 64-bit Windows machine, clicking next after the pack installation progress has finished will result in a "_Can't load IA 32-bit .dll on a 64-bit platform_" error being thrown. This is linked to the next step that creates the shortcuts but does not impact the correct installation of HermesJMS. Simply click on "Quit" to exit the installer.

An alternative way of creating a shortcut on Windows is to navigate to the <var>[hermesjms_install_dir]\bin</var> directory using the Windows explorer. Right click on the <var>hermes.bat</var> file and select <var>Send to > Desktop(create shortcut)</var> as shown below.

<figure>
    <img src="{{ site.url }}/assets/images/posts/jms/hermesjms-create-shortcut-windows.png" alt="hermesjms create shortcut windows">
</figure>

# Startup & Exit on Windows

Open a console window and navigate to the <var>[hermesjms_install_dir]</var>. Change to the <var>bin</var> subdirectory and execute the following command to start Hermes JMS:

``` plaintext
hermes.bat
```

<figure>
    <img src="{{ site.url }}/assets/images/posts/jms/hermesjms-start-windows.png" alt="hermesjms start windows">
</figure>

The Hermes JMS application should startup and open as shown below:

<figure>
    <img src="{{ site.url }}/assets/images/posts/jms/hermesjms-home-screen-windows.png" alt="hermesjms home screen windows">
</figure>

> On first startup, HermesJMS will create a <var>.hermes</var> directory that contains the default configuration XML, log files and a messages directory. If for some reason the application does not start as shown above, search for this directory (it will probably be located in your user's home directory or in the root where Windows is installed, typically C\:) and have a look at the <var>hermes.log</var> file for more information on what went wrong.

In order to exit HermesJMS simply select <var>File > Exit</var> from the console top menu as shown below.

<figure>
    <img src="{{ site.url }}/assets/images/posts/jms/hermesjms-quit-windows.png" alt="hermesjms quit windows">
</figure>

# Startup & Exit on Unix

Open a terminal window and navigate to the <var>[hermesjms_install_dir]</var>. Change to the <var>bin</var> subdirectory and execute the following command to start Hermes JMS:

``` plaintext
./hermes.sh
```

<figure>
    <img src="{{ site.url }}/assets/images/posts/jms/hermesjms-start-unix.png" alt="hermesjms start unix">
</figure>

The Hermes JMS application should startup and open as shown below: 

<figure>
    <img src="{{ site.url }}/assets/images/posts/jms/hermesjms-home-screen-unix.png" alt="hermesjms home screen unix">
</figure>

> If HermesJMS does not start correctly, check the 'hermes.log' file located in the <var>[hermesjms_install_dir]/bin</var> directory for more information on what went wrong.

In order to exit HermesJMS simply select <var>File > Exit</var> from the console top menu (same as on Windows).

# Changing the Default Configuration Location on Windows

It is recommended to keep your <var>hermes-config.xml</var> (by default located in a <var>.hermes</var> directory) separate from the installation. This way you can easily share or replicate your configuration on different machines. In addition, it will allow for easier upgrades without affecting your existing configuration.

Changing the default location of the configuration on Windows can be done by setting the <var>'HERMES_CONFIG'</var> environment variable. There are [a number of ways to edit the environment variables of a standard user](http://superuser.com/a/25038). One of them is to open a command prompt and enter the following command:

``` plaintext
rundll32 sysdm.cpl,EditEnvironmentVariables
```

This will open the <var>Environment Variables panel</var>. Click on <var>New</var> and enter "<kbd>HERMES_CONFIG</kbd>" as variable name and a location for the configuration directory as variable value. In the below screenshot the variable value is "<var>C:\Users\CodeNotFound\hermes_config</var>". Click <var>OK</var> to to save.

<figure>
    <img src="{{ site.url }}/assets/images/posts/jms/hermesjms-configuration-location-windows.png" alt="hermesjms configuration location windows">
</figure>

Last thing to do is to copy the <var>hermes-config.xml</var> from <var>.hermes</var> to the newly created configuration directory. Go ahead and start HermesJMS and this time the log files will appear under the configuration directory indicating the configuration was successful. You can also see the new location in the title of the HermesJMS application window.

> Make sure the <var>hermes-config.xml</var> is copied otherwise a <var>"_NoConfigurationException_"</var> will be thrown in the logs and the application will not start.

# Changing the Default Configuration File on Unix

You can change the default configuration file used by the <var>hermes.sh</var> shell script by by setting the <var>'HERMES_CFG'</var> environment variable.

Go to your user's home directory and create (or edit if already exists) a <var>.bashrc</var> file. Add the below line and change the value between brackets to the path of the configuration file you want to use.

``` plaintext
export HERMES_CFG="[hermes_config_file_path]"
```

In the example below the variable value is <var>'/home/codenotfound/hermes_config/hermes-config.xml'</var>.

<figure>
    <img src="{{ site.url }}/assets/images/posts/jms/hermesjms-configuration-location-unix.png" alt="hermesjms configuration location unix">
</figure>

Save the file and perform a user log off in order for the variable to get loaded when logging on again. Start HermesJMS and this time you should see in the title of the application window that the custom configuration file was loaded instead of the default one.

---

This concludes the installation and setup of the Hermes JMS. If you found this post helpful or have any questions or remarks, please leave a comment.
