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










