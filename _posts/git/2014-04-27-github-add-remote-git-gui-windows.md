---
title: GitHub - Add Remote Using Git Gui on Windows
permalink: /2014/04/github-add-remote-git-gui-windows.html
excerpt: A detailed tutorial on how to add a GitHub remote repository using Git Gui on Windows.
date: 2014-04-27 21:00
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


Start the Git command processor by clicking on the <kbd>Git Bash</kbd> link inside the Git program group (<kbd>Start&gt;All Programs&gt;Git</kbd>). A bash window should appear as shown below.

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

Create an account at [GitHub](https://github.com/) and sign in. Add a new remote repository by clicking the <kbd>+ New repository</kbd> button. Enter a repository name and check the <var>'Initialize this repository with a README'</var> checkbox so a <var>README.md</var> is automatically added as shown below.

<figure>
    <img src="{{ site.url }}/assets/images/git/github-create-repo.png" alt="github create repo">
</figure>

Next step is to add the public SSH key to your GitHub account. To do so access the GitHub account settings by clicking on the wrench/screwdriver icon in the top right hand corner. Then on the left hand side menu click on the [SSH keys](https://github.com/settings/ssh) link.

Click on the <kbd>Add SSH key</kbd> button in the top right hand corner. In the <var>'Title'</var> text field enter a name for the public key that we will add (in the example below the name "<kbd>test-repo</kbd>" is used). Then open the <var>id_rsa.pub</var> file that was generated in the previous section and copy paste the contents in the <var>'Key'</var> text field as shown below. Save the SSH key by clicking the <kbd>Add key</kbd> button.

<figure>
    <img src="{{ site.url }}/assets/images/git/github-ssh-keys.png" alt="github ssh keys">
</figure>

The newly added key is part of the SSH keys that are associated with your account as shown below.

> Note that the key fingerprint shown should be the same as the one that was printed during SSH keys creation in the previous section: 41:d7:ed:23:51:e0:ac:54:b4:52:6a:cf:b4:52:02

<figure>
    <img src="{{ site.url }}/assets/images/git/github-ssh-key-added.png" alt="github ssh key added">
</figure>

## Configure Git GUI

Start Git GUI by clicking on the <kbd>Git GUI</kbd> link inside the Git program group. Following window should appear.

<figure>
    <img src="{{ site.url }}/assets/images/git/git-gui-startup.png" alt="git gui startup">
</figure>

First thing to do is to create a new local Git repository. Click on the <kbd>Create New Repository</kbd> link and select a folder in which you would like to create a new local repository. In the example below the local repository is created at <var>C:/source4code/code/test-repo</var>. Click the <kbd>Create</kbd> button to complete the repository creation.

<figure>
    <img src="{{ site.url }}/assets/images/git/git-gui-create-new-repository.png" alt="git gui create new repository">
</figure>

A new window will open which shows the newly created Git repository.

<figure>
    <img src="{{ site.url }}/assets/images/git/git-gui-repo-window.png" alt="git gui repo window">
</figure>

Next step is to add the remote Git repository at GitHub. Click on the <kbd>Remote</kbd> menu and select <kbd>Add</kbd>. A new window will pop-up in which a name for the remote repository and the location need to be added. In this example we will enter "<kbd>test-repo</kbd>" as name and "<kbd>git@github.com:source4code/test-repo.git</kbd>" as location as shown below.

<figure>
    <img src="{{ site.url }}/assets/images/git/git-gui-add-new-remote.png" alt="git gui add new remote">
</figure>

> The location for the remote GitHub repository can be found by logging into GitHub, selecting the repository to be added and then clicking on the SSH link at the bottom right-hand side of the screen.

<figure>
    <img src="{{ site.url }}/assets/images/git/github-ssh-url.png" alt="github ssh url">
</figure>

If the <var>'Fetch Immediately'</var> action in the popup window was left selected, the first thing Git GUI will do is fetch the content of the remote GitHub repository. Enter the passphrase of the SSH keys generated in the previous section and press <kbd>OK</kbd>. The result should be a success status as shown below. Click the <kbd>Close</kbd> button to finish.

<figure>
    <img src="{{ site.url }}/assets/images/git/github-fetch-succes.png" alt="github fetch succes">
</figure>

# Git GUI Usage

Now that the remote repository was fetched we need to merge it with the local repository. Before this can be done, a local baseline is needed to merge with. Open the local repository location, in this example it was <var>C:/source4code/code/test-repo</var> and add a file <var>test1.txt</var>.

<figure>
    <img src="{{ site.url }}/assets/images/git/git-add-new-file.png" alt="git add new file">
</figure>

Switch back to Git GUI and press the <kbd>Rescan</kbd> button. The added file should now appear under the <var>Unstaged Changes</var> section. Then click on the <kbd>Stage Changed</kbd> button to tag the added file to be part of the files on which we will create a baseline and click <kbd>Yes</kbd> to confirm the untracked file. The file should now appear in the <var>Staged Changes</var> section as shown below.

<figure>
    <img src="{{ site.url }}/assets/images/git/git-gui-staged-changes.png" alt="git gui staged changes">
</figure>

> You can move files between the <var>Unstaged Changes</var> and <var>Staged Changes</var> section by using the <kbd>Commit</kbd> menu.

Add an initial commit message in the corresponding text box as shown below.

<figure>
    <img src="{{ site.url }}/assets/images/git/git-gui-initial-commit-message.png" alt="git gui initial commit message">
</figure>

Press the <kbd>Commit</kbd> button. The file will disappear from the <var>Staged Changes</var> section and at the bottom of the screen a <var>'Created commit'</var> message should appear.

<figure>
    <img src="{{ site.url }}/assets/images/git/git-gui-commit.png" alt="git gui commit">
</figure>

Now the local and remote baseline need to be merged. Click on the <kbd>Merge</kbd> menu and select <kbd>Local Merge</kbd>. A new window will pop-up that shows the possible baselines that can be merged with the local master baseline.

> If the remote <var>'test-repo/master'</var> does not show as below, make sure to restart Git GUI!

<figure>
    <img src="{{ site.url }}/assets/images/git/git-gui-local-merge.png" alt="git gui local merge">
</figure>

Click on the <kbd>Merge</kbd> button and a success status should be appear which shows that the <var>'README.md'</var> from the remote repository was successfully added.

<figure>
    <img src="{{ site.url }}/assets/images/git/git-gui-local-merge-success.png" alt="git gui local merge success">
</figure>

The last thing left to do is to push the merged local baseline to GitHub. In order to do so select the <kbd>Remote</kbd> menu and click on the <kbd>Push</kbd> menu item. A new window is shown which shows the local master that will be pushed to the remote <var>'test-repo'</var>.

<figure>
    <img src="{{ site.url }}/assets/images/git/git-gui-push-to-remote.png" alt="git gui push to remote">
</figure>

Click the <kbd>Push</kbd> button and enter the passphrase of the SSH keys. A success message should be shown. Close the message by clicking on the <kbd>Close</kbd> button and open your GitHub account. Select your repository and the <var>test1.txt</var> file should be present as shown below.

<figure>
    <img src="{{ site.url }}/assets/images/git/github-file-added.png" alt="github file added">
</figure>

From now on you can add/change files on your local repository that can then be pushed to GitHub once committed.

---

This concludes our tutorial on adding a remote repository to Git GUI so that code files can be uploaded to GitHub. If you found this post helpful or have any questions or remarks, please leave a comment.