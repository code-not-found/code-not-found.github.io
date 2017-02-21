---
title: Blogger - SyntaxHighlighter Setup
permalink: /2014/04/blogger-syntaxhighlighter-setup.html
excerpt: A detailed tutorial on how to setup code syntax highlighting on your Blogger blog.
date: 2016-04-26 21:00
categories: [PrimeFaces]
tags: [Example, Hello World, JavaServer Faces, Jetty, JSF, Maven, PrimeFaces, Project, Tutorial]
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/syntaxhighlighter-logo.png" alt="syntaxhighlighter logo">
</figure>

Blogger, the blog-publishing service from Google, does not provide a syntax highlighter out of the box. This means you need to resort to third party solutions in order to have syntax highlighting in your posts. One of those solutions is the syntax highlighter created by Alex Gorbatchev which is called [SyntaxHighlighter](http://alexgorbatchev.com/SyntaxHighlighter/).

One of the advantages of this highlighter is that it is purely written in JavaScript and as a result it does not need a server side component. SyntaxHighlighter can be easily integrated into your Blogger blog as we will detail in this tutorial.

# SyntaxHighlighter Install

First thing to do is to open the dashboard of your Blogger blog as shown below. On the left hand side find the menu that says <file>Template</file> and click on it.

<figure>
    <img src="{{ site.url }}/assets/images/blogger/blogger-overview.png" alt="blogger overview">
</figure>

This will open the template page of your blog. Click on the button that says <file>Edit HTML</file> in order to open up the HTML editor as shown below.

<figure>
    <img src="{{ site.url }}/assets/images/blogger/blogger-template.png" alt="blogger template">
</figure>

Click anywhere inside the editor window and press <kbd>CTRL+F</kbd>. A search box should now appear in the upper right hand corner as shown below. In the search box enter "<kbd>&lt;/head&gt;<kbd>" (without quotes) and press <kbd>ENTER</kbd>.

<figure>
    <img src="{{ site.url }}/assets/images/blogger/blogger-edit-html-head.png" alt="blogger edit html head">
</figure>

The editor window will now jump the end of the HTML header tag where we will add the needed style sheets and JavaScript files. The SyntaxHighlighter configuration consists out of four parts:
1. The core files
2. The SyntaxHighlighter theme
3. The specific brush(es) needed for the blog
4. The configuration script

## 1) The core files

The core files consist out of the following JavaScript file and style sheet:

``` xml
    <script src="http://alexgorbatchev.com/pub/sh/current/scripts/shCore.js" type="text/javascript"></script>
    <link href="http://alexgorbatchev.com/pub/sh/current/styles/shCore.css" rel="stylesheet" type="text/css"></link>
```

## 2) The SyntaxHighlighter theme

There are a number of themes available for SyntaxHighlighter, for the complete list please check following [link](http://alexgorbatchev.com/SyntaxHighlighter/manual/themes/). The style sheet below is the default theme. 

``` xml
    <link href="http://alexgorbatchev.com/pub/sh/current/styles/shThemeDefault.css" rel="stylesheet" type="text/css" />
```

## 3) The specific brush(es) needed for the blog
Depending on the structured language that needs to be highlighted, the corresponding brush needs to be imported. For a complete list of all available brushes please check following [link](http://alexgorbatchev.com/SyntaxHighlighter/manual/brushes/). In this example we will add the brushes for '<var>Java</var>' and '<var>XML</var>'.

``` xml
    <script src="http://alexgorbatchev.com/pub/sh/current/scripts/shBrushJava.js" type="text/javascript"></scrip>
    <script src="http://alexgorbatchev.com/pub/sh/current/scripts/shBrushXml.js" type="text/javascript"></script>
```

> Only add the needed brushes as for each page the brushes are retrieved from alexgorbatchev.com (the SyntaxHighlighter site) and this increases your blog page load times!

## 4) The configuration script

After all needed dependencies have been added we need to enable a specific mode for Blogger and instruct SyntaxHighlighter to highlight all code blocks found on the web page. This is done by adding a JavaScript snippet as shown below. 











