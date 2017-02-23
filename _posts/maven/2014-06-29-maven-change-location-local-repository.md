---
title: Maven - Change Local Repository Location
permalink: /2014/06/maven-change-location-local-repository.html
excerpt: A short tutorial on how to change the location of the Maven local repository.
date: 2014-06-29 21:00
categories: [Apache Maven]
tags: [Apache Maven, Change, Configuration, Local, Location, Maven, Repository, Setup]
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/maven-logo.png" alt="maven logo">
</figure>

A repository in Maven is used to hold build artifacts and dependencies of varying types. There are strictly only two types of repositories: local and remote. The local repository refers to a copy on your own machine that is a cache of the remote downloads and also contains the temporary build artifacts that have not yet been released.

When installing Maven, the local repository is located under a default location. The following tutorial shows how you can change the location of this local repository on Windows. 

Maven is configured based on a <var>settings.xml</var> file that can be specified at two levels:








