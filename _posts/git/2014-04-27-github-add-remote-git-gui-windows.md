---
title: "GitHub - Add Remote Using Git Gui on Windows"
permalink: /github-add-remote-git-gui-windows.html
excerpt: "A detailed tutorial on how to add a GitHub remote repository using Git Gui on Windows."
date: 2014-04-27
last_modified_at: 2014-04-27
header:
  teaser: "assets/images/teaser/git-teaser.png"
categories: [Git]
tags: [Add, Git, Git GUI, GitHub, Remote, Repository, Setup, Tutorial, Upload Code, Windows]
redirect_from:
  - /2014/04/git-upload-source-code-github-git-gui.html
  - /2014/04/github-add-remote-git-gui-windows.html
published: true
---

<figure>
    <img src="{{ site.url }}/assets/images/logo/git-logo.png" alt="git logo" class="logo">
</figure>

[GitHub](https://github.com/){:target="_blank"} is a web-based hosting service for software development projects that use the [Git revision control system](https://git-scm.com/){:target="_blank"}. It is used to share code with other people and a GitHub account is free for open source projects.

Following tutorial will show you how to setup and configure Git Gui on your Windows computer so you can upload code towards a remote GitHub repository.

> Note that we have switched to [Sourcetree](https://www.sourcetreeapp.com/){:target="_blank"} which is another free Git GUI alternative for Mac and Windows. We highly recommend you check it out as it is simple to use and very powerful in terms of features.

# Git GUI Install

First, let's start by going to the [Git downloads page](http://git-scm.com/downloads){:target="_blank"} and download the Git installer for your operating system. In this tutorial, we will use Windows.

<figure>
    <img src="{{ site.url }}/assets/images/posts/git/git-download-page.png" alt="git download page">
</figure>

Double click to run the downloaded <var>'.exe'</var> file and click <var>Next</var> keeping the default settings on the different installer steps. At the end click <var>Finish</var> and Git should be successfully installed.

<figure>
    <img src="{{ site.url }}/assets/images/posts/git/git-installer.png" alt="git installer">
</figure>


Start the Git command processor by clicking on the <var>Git Bash</var> link inside the Git program group <var>Start&gt;All Programs&gt;Git</var>. A bash window should appear as shown below.

<figure>
    <img src="{{ site.url }}/assets/images/posts/git/git-bash.png" alt="git bash">
</figure>

## Configure Git
 
The first thing that needs to be done is to setup some basic Git parameters like user name and email address. In order to do so enter following commands and replace the value between the quotes with your own values.

``` plaintext
git config --global user.name "<user_name>"
git config --global user.email "<user_email>"
```

Shown below is the execution of the two commands.

<figure>
    <img src="{{ site.url }}/assets/images/posts/git/git-bash-config.png" alt="git bash config">
</figure>

In order to verify if the values are set correctly enter the following command:

``` plaintext
git config --list
```
The result is a list of configuration parameters as shown below.

<figure>
    <img src="{{ site.url }}/assets/images/posts/git/git-bash-config-list.png" alt="git bash config list">
</figure>

## Generating SSH keys

It is strongly recommended to use an SSH connection when interacting with GitHub. SSH keys are a way to identify trusted computers, without involving passwords. To generate a new SSH key pair, copy and paste the command below, making sure to substitute the email value between the quotes with your own.

``` plaintext
ssh-keygen -t rsa -C "<user_email>"
```

When asked to <var>'Enter file in which to save the key'</var> just press <var>ENTER</var> to continue. Then a passphrase is requested which acts as a password you need to enter each time you want to use your key with SSH. Enter your password twice and the result should be as shown below.

<figure>
    <img src="{{ site.url }}/assets/images/posts/git/git-generate-ssh-keys.png" alt="git generate ssh keys">
</figure>

Locate the generated keys by going to the location as shown in the console output. In the above example the location is: <var>C:\Users\source4code\.ssh</var>.

<figure>
    <img src="{{ site.url }}/assets/images/posts/git/git-generated-ssh-keys-location.png" alt="git generated ssh keys location">
</figure>

The <var>id_rsa</var> file contains your private key and the <var>id_rsa.pub</var> file contains your public key.

## Configure GitHub

Create an account at [GitHub](https://github.com/){:target="_blank"} and sign in. Add a new remote repository by clicking the <var>+ New repository</var> button. Enter a repository name and check the <var>'Initialize this repository with a README'</var> checkbox so a <var>README.md</var> is automatically added as shown below.

<figure>
    <img src="{{ site.url }}/assets/images/posts/git/github-create-repo.png" alt="github create repo">
</figure>

Next step is to add the public SSH key to your GitHub account. To do so access the GitHub account settings by clicking on the wrench/screwdriver icon in the top right-hand corner. Then on the left-hand side menu click on the [SSH keys](https://github.com/settings/ssh){:target="_blank"} link.

Click on the <var>Add SSH key</var> button in the top right hand corner. In the <var>'Title'</var> text field enter a name for the public key that we will add (in the example below the name "<kbd>test-repo</kbd>" is used). Then open the <var>id_rsa.pub</var> file that was generated in the previous section and copy paste the contents in the <var>'Key'</var> text field as shown below. Save the SSH key by clicking the <var>Add key</var> button.

<figure>
    <img src="{{ site.url }}/assets/images/posts/git/github-ssh-keys.png" alt="github ssh keys">
</figure>

The newly added key is part of the SSH keys that are associated with your account as shown below.

> Note that the key fingerprint shown should be the same as the one that was printed during SSH keys creation in the previous section: 41:d7:ed:23:51:e0:ac:54:b4:52:6a:cf:b4:52:02

<figure>
    <img src="{{ site.url }}/assets/images/posts/git/github-ssh-key-added.png" alt="github ssh key added">
</figure>

## Configure Git GUI

Start Git GUI by clicking on the <var>Git GUI</var> link inside the Git program group. Following window should appear.

<figure>
    <img src="{{ site.url }}/assets/images/posts/git/git-gui-startup.png" alt="git gui startup">
</figure>

The first thing to do is to create a new local Git repository. Click on the <var>Create New Repository</var> link and select a folder in which you would like to create a new local repository. In the example below the local repository is created at <var>C:/source4code/code/test-repo</var>. Click the <var>Create</var> button to complete the repository creation.

<figure>
    <img src="{{ site.url }}/assets/images/posts/git/git-gui-create-new-repository.png" alt="git gui create new repository">
</figure>

A new window will open which shows the newly created Git repository.

<figure>
    <img src="{{ site.url }}/assets/images/posts/git/git-gui-repo-window.png" alt="git gui repo window">
</figure>

Next step is to add the remote Git repository at GitHub. Click on the <var>Remote</var> menu and select <var>Add</var>. A new window will pop-up in which a name for the remote repository and the location need to be added. In this example we will enter "<kbd>test-repo</kbd>" as name and "<kbd>git@github.com:source4code/test-repo.git</kbd>" as location as shown below.

<figure>
    <img src="{{ site.url }}/assets/images/posts/git/git-gui-add-new-remote.png" alt="git gui add new remote">
</figure>

> The location for the remote GitHub repository can be found by logging into GitHub, selecting the repository to be added and then clicking on the SSH link at the bottom right-hand side of the screen.

<figure>
    <img src="{{ site.url }}/assets/images/posts/git/github-ssh-url.png" alt="github ssh url">
</figure>

If the <var>'Fetch Immediately'</var> action in the popup window was left selected, the first thing Git GUI will do is fetch the content of the remote GitHub repository. Enter the passphrase of the SSH keys generated in the previous section and press <var>OK</var>. The result should be a success status as shown below. Click the <var>Close</var> button to finish.

<figure>
    <img src="{{ site.url }}/assets/images/posts/git/github-fetch-succes.png" alt="github fetch succes">
</figure>

# Git GUI Usage

Now that the remote repository was fetched we need to merge it with the local repository. Before this can be done, a local baseline is needed to merge with. Open the local repository location, in this example it was <var>C:/source4code/code/test-repo</var> and add a file <var>test1.txt</var>.

<figure>
    <img src="{{ site.url }}/assets/images/posts/git/git-add-new-file.png" alt="git add new file">
</figure>

Switch back to Git GUI and press the <var>Rescan</var> button. The added file should now appear under the <var>'Unstaged Changes'</var> section. Then click on the <var>Stage Changed</var> button to tag the added file to be part of the files on which we will create a baseline and click <var>Yes</var> to confirm the untracked file. The file should now appear in the <var>'Staged Changes'</var> section as shown below.

<figure>
    <img src="{{ site.url }}/assets/images/posts/git/git-gui-staged-changes.png" alt="git gui staged changes">
</figure>

> You can move files between the <var>'Unstaged Changes'</var> and <var>'Staged Changes'</var> section by using the <var>Commit</var> menu.

Add an initial commit message in the corresponding text box as shown below.

<figure>
    <img src="{{ site.url }}/assets/images/posts/git/git-gui-initial-commit-message.png" alt="git gui initial commit message">
</figure>

Press the <var>Commit</var> button. The file will disappear from the <var>'Staged Changes'</var> section and at the bottom of the screen a <var>"Created commit"</var> message should appear.

<figure>
    <img src="{{ site.url }}/assets/images/posts/git/git-gui-commit.png" alt="git gui commit">
</figure>

Now the local and remote baseline need to be merged. Click on the <var>Merge</var> menu and select <var>Local Merge</var>. A new window will pop-up that shows the possible baselines that can be merged with the local master baseline.

> If the remote <var>'test-repo/master'</var> does not show as below, make sure to restart Git GUI!

<figure>
    <img src="{{ site.url }}/assets/images/posts/git/git-gui-local-merge.png" alt="git gui local merge">
</figure>

Click on the <var>Merge</var> button and a success status should be appear which shows that the <var>README.md</var> file from the remote repository was successfully added.

<figure>
    <img src="{{ site.url }}/assets/images/posts/git/git-gui-local-merge-success.png" alt="git gui local merge success">
</figure>

The last thing left to do is to push the merged local baseline to GitHub. In order to do so select the <var>Remote</var> menu and click on the <var>Push</var> menu item. A new window is shown which shows the local master that will be pushed to the remote <var>'test-repo'</var>.

<figure>
    <img src="{{ site.url }}/assets/images/posts/git/git-gui-push-to-remote.png" alt="git gui push to remote">
</figure>

Click the <var>Push</var> button and enter the passphrase of the SSH keys. A success message should be shown. Close the message by clicking on the <var>Close</var> button and open your GitHub account. Select your repository and the <var>test1.txt</var> file should be present as shown below.

<figure>
    <img src="{{ site.url }}/assets/images/posts/git/github-file-added.png" alt="github file added">
</figure>

From now on you can add/change files on your local repository that can then be pushed to GitHub once committed.

---

This concludes our tutorial on adding a remote repository to Git GUI so that code files can be uploaded to GitHub. If you found this post helpful or have any questions or remarks, please leave a comment.
