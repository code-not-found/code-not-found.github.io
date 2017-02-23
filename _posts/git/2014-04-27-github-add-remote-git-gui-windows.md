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

Double click to run the downloaded <var>'.exe'</var> file and click <kbd>Next</kbd> keeping the default settings on the different installer steps. At the end click <kbd>Finish</kbd> and Git should be successfully installed.

<figure>
    <img src="{{ site.url }}/assets/images/git/git-installer.png" alt="git installer">
</figure>


Start the Git command processor by clicking on the <var>Git Bash</var> link inside the Git program group (<var>Start&gt;All Programs&gt;Git</var>). A bash window should appear as shown below.

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

## Generating SSH keys

It is strongly recommend to use an SSH connection when interacting with GitHub. SSH keys are a way to identify trusted computers, without involving passwords. To generate a new SSH key pair, copy and paste the command below, making sure to substitute the email value between the quotes with your own.

``` plaintext
ssh-keygen -t rsa -C "<user_email>"
```

When asked to <var>'Enter file in which to save the key'</var> just press <kbd>ENTER</kbd> to continue. Then a passphrase is requested which acts as a password you need to enter each time you want to use your key with SSH. Enter your password twice and the result should be as shown below.

<figure>
    <img src="{{ site.url }}/assets/images/git/git-generate-ssh-keys.png" alt="git generate ssh keys">
</figure>

Locate the generated keys by going to the location as shown in the console output. In the above example the location is: <var>C:\Users\source4code\.ssh</var>.

<figure>
    <img src="{{ site.url }}/assets/images/git/git-generated-ssh-keys-location.png" alt="git generated ssh keys location">
</figure>

The <var>id_rsa</var> file contains your private key and the <var>id_rsa.pub</var> file contains your public key.

## Configure GitHub

Create an account at [GitHub](https://github.com/) and sign in. Add a new remote repository by clicking the <var>+ New repository</var> button. Enter a repository name and check the <var>'Initialize this repository with a README'</var> checkbox so a <var>README.md</var> is automatically added as shown below.

<figure>
    <img src="{{ site.url }}/assets/images/git/github-create-repo.png" alt="github create repo">
</figure>

Next step is to add the public SSH key to your GitHub account. To do so access the GitHub account settings by clicking on the wrench/screwdriver icon in the top right hand corner. Then on the left hand side menu click on the [SSH keys](https://github.com/settings/ssh) link.

Click on the <var>Add SSH key</var> button in the top right hand corner. In the <var>'Title'</var> text field enter a name for the public key that we will add (in the example below the name "test-repo" is used). Then open the 'id_rsa.pub' file that was generated in the previous section and copy paste the contents in the 'Key' text field as shown below. Save the SSH key by clicking the Add key button.






