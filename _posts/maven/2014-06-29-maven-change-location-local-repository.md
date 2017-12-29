---
title: "Maven - Change Local Repository Location"
permalink: /maven-change-location-local-repository.html
excerpt: "A short tutorial on how to change the location of the Maven local repository."
date: 2014-06-29
last_modified_at: 2014-06-29
header:
  teaser: "assets/images/teaser/maven-teaser.png"
categories: [Apache Maven]
tags: [Apache Maven, Change, Configuration, Local, Location, Maven, Repository, Setup]
redirect_from:
  - /2014/06/maven-change-location-of-local.html
  - /2014/06/maven-change-location-local-repository.html
published: true
---

<figure>
    <img src="{{ site.url }}/assets/images/logo/maven-logo.png" alt="maven logo" class="logo">
</figure>

A repository in [Maven](https://maven.apache.org/){:target="_blank"} is used to hold build artifacts and dependencies of varying types. There are strictly only two types of repositories: local and remote.

The **local repository** refers to a copy on your own machine that is a cache of the remote downloads and also contains the temporary build artifacts that have not yet been released.

When [installing Maven]({{ site.url }}/maven-download-install-apache-maven-3-2-windows.html), the local repository is located under a default location. The following tutorial shows how you can change the location of this local repository on Windows. 

# Maven Local Repository

Maven is configured based on a <var>settings.xml</var> file that can be specified at two levels:

1. **User Level**: provides configuration for a single user and is typically provided in <var>${user.home}/.m2/settings.xml</var>.
2. **Global level**: provides configuration for all Maven users on a machine (assuming they're all using the same Maven installation) and it's typically provided in <var>${maven.home}/conf/settings.xml</var>.

In this example we will change the local repository location by creating/editing a <var>settings.xml</var> file at user level.

Navigate to <var>[maven_install_dir]/conf</var> and if not already present copy the <var>setting.xml</var> file to the <var>.m2</var> directory located in the user home directory (in this example the user is <var>'source4code'</var>) as shown below.

<figure>
    <img src="{{ site.url }}/assets/images/posts/maven/maven-copy-settings-xml.png" alt="maven copy settings xml">
</figure>

Open the copied <var>settings.xml</var> file and add/update the <var>'&lt;localRepository&gt;'</var> element to point to the new location of the local repository. In this example the location is set to <var>C:\source4code\local-repo</var>.

``` xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0" 
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
  <!-- localRepository
   | The path to the local repository maven will use to store artifacts.
   |
   | Default: ${user.home}/.m2/repository
  <localRepository>/path/to/local/repo</localRepository>
  -->
  <localRepository>C:\source4code\local-repo</localRepository>

  ...
  
</settings>
```

Next time Maven tries to resolve and download dependencies to the local repository, they will be stored in the newly defined location.

The below image shows the result after creating a quickstart Maven project. The artifacts are now downloaded to the <var>C:\source4code\local-repo</var> directory that was configured in the above <var>settings.xml</var>.

<figure>
    <img src="{{ site.url }}/assets/images/posts/maven/maven-new-local-repository.png" alt="maven new local repository">
</figure>

---

This concludes changing the location of the Maven local repository. If you found this post helpful or have any questions or remarks, please leave a comment below.
