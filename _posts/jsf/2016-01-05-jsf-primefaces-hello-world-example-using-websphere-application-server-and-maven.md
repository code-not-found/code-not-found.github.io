---
title: JSF - PrimeFaces Hello World Example using WebSphere Application Server and Maven 
permalink: /2016/01/jsf-primefaces-hello-world-example-using-websphere-application-server-and-maven.html
excerpt: A PrimeFaces 'Hello World' example using WebSphere Application Server and Maven.
date: 2016-01-05 21:00
categories: [PrimeFaces]
tags: [Example, IBM, JavaServer Faces, Jetty, JSF, Maven, PrimeFaces, WAS, WebSphere Application Server]
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/primefaces-logo.png" alt="primefaces logo">
</figure>

[IBM WebSphere Application Server](http://www-03.ibm.com/software/products/en/appserv-was) (WAS) is a software framework and middleware developed by International Business Machines Corporation. It hosts Java based web applications and is built using Open standards such as Java EE, XML, and Web Services. WebSphere Application Server is released under a commercial license.

The following post illustrates a basic example in which we will configure, build and run a Hello World PrimeFaces example using WebSphere Application Server and Maven.

Tools used:
* JSF 2.2
* PrimeFaces 5.3
* WebSphere Application Server 8.5
* Maven 3

# WebSphere Application Server Setup
The example assumes you have an instance of WAS installed on the machine from which you will be executing Maven. If this is the case you can go ahead and [jump directly to the configuration of the was-maven-plugin]({{ site.url }}/2016/01/jsf-primefaces-hello-world-example-using-websphere-application-server-and-maven.html#was-maven-plugin).

If you don't have an instance of WebSphere Application Server installed on your machine, you can [download the developers edition](http://www-03.ibm.com/software/products/en/appserv-wasfordev) that can be used for development and testing purposes at no charge. Click on the <var>'Downloads'</var> tab and then choose <var>'Try trial'</var>. Login with your IBM ID (register one if needed) and the below downloads page should show.

<figure>
    <img src="{{ site.url }}/assets/images/jsf/websphere-application-server-for-developers.png" alt="websphere application server for developers">
</figure>

Download and extract the installer ZIP archive for you operating system. Launch the IBM Installation Manager and click through the different setup steps and start the installation. When the installation is complete select the option to launch the <var>'Profile Management Tool to create an application server profile for a development environment'</var>.

Once the Profile Management Tool is launched, enable administrative security as shown below. In this example we will use following values: User name="<kbd>admin</kbd>" and Password="<kbd>admin</kbd>".

<figure>
    <img src="{{ site.url }}/assets/images/jsf/websphere-application-server-profile-management-tool.png" alt="websphere application server profile management tool">
</figure>

On the last step leave the <var>'Launch the First steps console'</var> selected and finish. A new window will pop-up from which you can select the <var>'Installation verification'</var> link which will launch a first steps output console that prints a number of details on your newly created development server profile.

















































