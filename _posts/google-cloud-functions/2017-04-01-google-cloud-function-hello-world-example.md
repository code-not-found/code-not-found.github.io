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
* Google Cloud Functions - beta
* npm 4.4

As Cloud Functions are written in JavaScript and executed on Node.js, we will be using [npm](https://www.npmjs.com/) which is the default package manager for the JavaScript runtime environment [Node.js](https://nodejs.org/). If NPM isn't installed on your system, first thing you need to do is to [install Node.js and update npm](https://docs.npmjs.com/getting-started/installing-node). Once installed, executed following command on the Node.js command prompt to check if everything is setup correctly, at the moment of writing the NPM version was _4.4.4_.

``` plaintext
npm -v
```

# Writing a Hello World HTTP function

Dependencies in Node.js are managed with npm and expressed in a [metadata file called package.json](https://docs.npmjs.com/files/package.json). Cloud Functions allow you to manage your dependencies by either pre-packaging them or by declaring them in a <var>package.json</var> file. Preference is given to the latter as in that case Cloud Functions will automatically download the dependencies for you when you deploy. As in this example we will not be needing any dependencies, we will include a basic <var>package.json</var> as shown below.

``` json
{
  "name": "google-cloud-functions-helloworld",
  "version": "0.0.1",
  "description": "Google Cloud Functions - Hello World Example",
  "homepage" : "https://www.codenotfound.com/2017/04/google-cloud-functions-hello-world-example.html",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "codenotfound",
  "license": "MIT"
}
```

Google Cloud Functions come in two distinct variants:
1. foreground functions
2. background functions 

You use use foreground functions when you want to directly invoke your function via HTTP(S). Background functions on the other hand are use when you want to have your Cloud Function invoked indirectly via a message on a [Google Cloud Pub/Sub topic](https://cloud.google.com/pubsub/docs/overview), or a change in a [Google Cloud Storage bucket](https://cloud.google.com/storage/docs/key-terms#buckets).

In this example we will create an [HTTP foreground function](https://cloud.google.com/functions/docs/writing/http) called `helloWorld`. The signature of an HTTP function takes two arguments: `request` and `response`. The `request` parameter represents the HTTP request that was sent to the function, and the `response` parameter represents the HTTP response that will be returned to the caller.

The body of the HTTP request is automatically parsed and populated based on the content-type. We can then access the different parts from the request like for example: the body, HTTP headers, HTTP method and HTTP query parameters. Inside our `helloWorld` function we will extract two query parameters called firstName and lastName and use them to construct a `greeting` JSON object.


``` javascript
'use strict';

exports.helloWorld = function helloWorld(request, response) {
    const firstName = request.query.firstName
    console.log('firstName=' + firstName);
    const lastName = request.query.lastName
    console.log('lastName=' + lastName);

    const greeting = { greeting: 'Hello ' + firstName + ' ' + lastName + '!' };
    console.log('greeting=' + JSON.stringify(greeting));
    
    response.status(200).send(greeting);
};
```

> Note: You should always call a termination method such as `send()`, `json()`, or `end()` when your function has completed. Otherwise your function may continue to run and be forcibly terminated by the system.





# Deploying the Hello World HTTP function





---

This wraps up our basic example on how to deploy a hello world Cloud Function on the Google Cloud Platform. If you enjoyed reading or have a question, please drop a line below.