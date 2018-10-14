---
title: "Golang REST API Tutorial - Hello World"
permalink: /golang-rest-api-hello-world.html
excerpt: "A detailed step-by-step tutorial on how to implement a Hello World REST API using Go."
date: 2018-09-22
last_modified_at: 2018-09-22
header:
  teaser: "assets/images/post/golang/golang-rest-api-definitive-guide.png"
categories: [Golang]
tags: [API, Example, Go, Golang, Hello World, Tutorial]
published: false
---

<img src="{{ site.url }}/assets/images/posts/golang/golang-rest-api-definitive-guide.png" alt="golang rest api definitive guide" class="align-right title-image">

Want to learn **how to build a REST API using Go**?

Then you’re in the right place.

Because in this tutorial I’m going to show you a basic Golang REST API example.

The best part?

I'll detail every step so you can follow along.

Let’s get started!

## Introducing golang

Before we get into the code let's have a quick look at [Go](https://golang.org/){:target="_blank"} (often referred to as Golang).

Go is an open source programming language. It comes with built-in concurrency and is statically typed. Golang is maintained and developed by a team at Google and contributors from the open source community.

If you are looking for a hands-on introduction to Go then be sure to check out [Go by Example](https://gobyexample.com/){:target="_blank"}.

## Installing Golang

To use Go, you need to have it installed on your computer. If you are working with Windows, go ahead and [download and install Golang](https://downlinko.com/download-install-golang-windows.html){:target="_blank"} now.

For other operating systems check the <var>Stable versions</var> section on the [Go downloads page](https://golang.org/dl/){:target="_blank"}.

This guide was created using Go version <var>1.11</var>. You can use following command to know the version installed on your system:

``` plaintext
go version
```

Open a command line, execute the command and the result should be as shown below.

<img src="{{ site.url }}/assets/images/posts/golang/golang-version-command.png" alt="golang version command">

## Installing an IDE

You can use a plain text editor to write Go code. However, in this tutorial we will use an integrated development environment or IDE.

The Golang wiki contains an list of [different IDEs that allow you to work with Go](https://github.com/golang/go/wiki/IDEsAndTextEditorPlugins){:target="_blank"}.

For this guide we will use <var>Visual Studio Code</var>. This is a free and open source IDE created by Microsoft. A nice feature is that it supports Go syntax highlighting out of the box.

So on Windows, [download and install Visual Studio Code](https://downlinko.com/download-install-visual-studio-code-windows.html){:target="_blank"} to be able to continue with the next steps. For other operating systems head over to the [Visual Studio Code downloads page](https://code.visualstudio.com/#alt-downloads){:target="_blank"}.

Once the installation is complete, open Visual Studio Code. Now click on the <var>Extension</var> button on the left hand side. In the search box enter <kbd>go</kbd>. Click on the green <var>Install</var> button to setup the [Go language support extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode.Go){:target="_blank"}.

<img src="{{ site.url }}/assets/images/posts/golang/golang-go-extension.png" alt="golang version command">

Once the installation is done click on the blue <var>Reload</var> button. Rich language support for Go is now enabled.

## Create Your Go Workspace

Go programmers typically keep all their Go code in a single workspace. The <var>GOPATH</var> environment variable specifies the location of this workspace.

If you want to know your current <var>GOPATH</var> workspace directory execute following command:

``` plaintext
go env GOPATH
```

The [default Go workspace location](https://golang.org/doc/code.html#GOPATH){:target="_blank"} is a directory named <var>go</var> inside your user home directory.

<img src="{{ site.url }}/assets/images/posts/golang/golang-gopath-location.png" alt="golang version command output">

> If you would like to work in a different location, you need to [set the GOPATH environment variable](https://github.com/golang/go/wiki/SettingGOPATH){:target="_blank"} to the path of that directory.

To compile and run our program, we need to setup a package path. Inside your <var>GOPATH</var> workspace directory create the following directory structure: <var>/src/golang-api/helloworld</var>

<img src="{{ site.url }}/assets/images/posts/golang/rest-api/golang-rest-api-create-helloworld-package.png" alt="golang rest api create helloworld package">

Switch back to Visual Studio Code and add your Golang package to the workspace. Click on <var>File > Add Folder to Workspace</var>.

<img src="{{ site.url }}/assets/images/posts/golang/rest-api/visual-studio-code-add-folder-to-workspace.png" alt="visual studio code add folder to workspace">

Select the <var>helloworld</var> directory and click on <var>Add</var>.

<img src="{{ site.url }}/assets/images/posts/golang/rest-api/visual-studio-code-helloworld-workspace.png" alt="visual studio code helloworld workspace">

We are now ready to start coding.

## Create a Hello World REST API in Go

Right-click on the <var>helloworld</var> folder and select <var>New File</var> from the pop-up menu.

<img src="{{ site.url }}/assets/images/posts/golang/rest-api/visual-studio-code-helloworld-new-file.png" alt="visual studio code helloworld new file">

Fill in <kbd>main.go</kbd> as file name and press <var>ENTER</var>.

> If you get a pop-up asking to install additional tools click on <var>Install</var>. Note that Git needs to be configured on your system in order for this to work.

<img src="{{ site.url }}/assets/images/posts/golang/rest-api/visual-studio-code-helloworld-main.png" alt="visual studio code helloworld-main">
