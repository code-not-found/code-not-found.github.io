---
title: GitHub - Add Remote Using Git Gui on Windows
permalink: /2014/04/github-add-remote-git-gui-windows.html
excerpt: A detailed tutorial on how to add a GitHub remote repository using Git Gui on Windows.
date: 2016-04-27 21:00
categories: [Git]
tags: [Add, Git, Git GUI, GitHub, Remote, Repository, Setup, Tutorial, upload code, Windows]
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/git-logo.png" alt="git logo">
</figure>

[GitHub](https://github.com/) is a web-based hosting service for software development projects that use the Git revision control system. It is used to share code with other people and a GitHub account is free for open source projects. Following tutorial will show you how to setup and configure Git Gui on your Windows computer so you can upload code towards a remote GitHub repository.

# Git GUI Install

First let's start by going to the [Git downloads page](http://git-scm.com/downloads) and download the Git installer for your operating system. In this tutorial we will use Windows.

<figure>
    <img src="{{ site.url }}/assets/images/git/git-download-page.png" alt="git download page">
</figure>

Double click to run the downloaded <var>'.exe'</var> file and click <kbd>Next<kbd> keeping the default settings on the different installer steps. At the end click <kbd>Finish</kbd> and Git should be successfully installed.

<figure>
    <img src="{{ site.url }}/assets/images/git/git-installer.png" alt="git installer">
</figure>


Start the Git command processor by clicking on the <var>Git Bash<var> link inside the Git program group (<var>Start>All Programs>Git<var>). A bash window should appear as shown below.

<figure>
    <img src="{{ site.url }}/assets/images/git/git-bash.png" alt="git bash">
</figure>

 ## Configure Git

First thing that needs to be done is to setup some basic Git parameters like user name and email address. In order to do so enter following commands and replace the value between the quotes with your own values.

``` plaintext
git config --global user.name "<user_name>"
git config --global user.email "<user_email>"
```

Shown below is the execution of the two commands.

<figure>
    <img src="{{ site.url }}/assets/images/git/git-bash-config.png" alt="git bash config">
</figure>

In order to verify if the values are set correctly enter following command:

``` plaintext
git config --list
```
The result is a list of configuration parameters as shown below.

<figure>
    <img src="{{ site.url }}/assets/images/git/git-bash-config-list.png" alt="git bash config list">
</figure>





