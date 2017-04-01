---
title: Google Cloud Functions - Hello World Example
permalink: /2017/04/google-cloud-functions-hello-world-example.html
excerpt: A step-by-step tutorial on how to create and deploy a Hello World Google Cloud Function.
date: 2017-04-01 21:00
categories: [Google Cloud Functions]
tags: [Example, Google Cloud Functions, Google Cloud Platform, Hello World, Tutorial]
published: false
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/google-cloud-functions-logo.png" alt="google cloud functions logo">
</figure>

[Google Cloud Functions](https://cloud.google.com/functions/) is a serverless execution environment for building and connecting services running in the cloud. Code executes in a fully managed environment, in other words there is no need to provision any infrastructure or worry about managing any servers. This is a category of cloud computing services also known as **function as a service** (FaaS).

Cloud Functions are written in JavaScript and execute in a Node.js environment on the [Google Cloud Platform](https://cloud.google.com/). Goal of this tutorial is to provide a quick introduction on how to get your first function up and running. We will do this by building and then running a Hello World HTTP Cloud Function using the Google Cloud Platform and NPM.

> Please note that at the moment of writing  Cloud Functions are still in beta phase.

Tools used:
* Google Cloud Functions
* NPM 4.4

As Cloud Functions are written in JavaScript and executed on Node.js, we will be using [NPM](https://www.npmjs.com/) which is the default package manager for the JavaScript runtime environment [Node.js](https://nodejs.org/). If NPM isn't installed on your system, first thing you need to do is to [install Node.js and update npm](https://docs.npmjs.com/getting-started/installing-node). Once installed, executed following command on the Node.js command prompt to check if everything is setup correctly, at the moment of writing the NPM version was _4.4.4_.

``` plaintext
npm -v
```

# Writing a Hello World HTTP function


---

This wraps up our basic example on how to deploy a hello world Cloud Function on the Google Cloud Platform. If you enjoyed reading or have a question, please drop a line below.