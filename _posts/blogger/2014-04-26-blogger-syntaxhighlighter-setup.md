---
title: Blogger - SyntaxHighlighter HTML Configuration
permalink: /blogger-syntaxhighlighter-html-configuration.html
excerpt: "A detailed step-by-step tutorial on how to setup code syntax highlighting on your Blogger blog."
date: 2014-04-26
last_modified_at: 2014-04-26
header:
  teaser: "assets/images/teaser/blogger-teaser.png"
categories: [Blogger]
tags: [Blogger, BlogSpot, Brushes, Configuration, HTML, SyntaxHighlighter, Tutorial]
redirect_from:
  - /2014/04/how-to-setup-syntaxhighlighter-on-blogger.html
  - /2014/04/blogger-add-syntaxhighlighter.html
  - /2014/04/blogger-setup-syntaxhighlighter.html
  - /2014/04/blogger-syntaxhighlighter-setup.html
published: true
---

<figure>
    <img src="{{ site.url }}/assets/images/logo/blogger-logo.png" alt="blogger logo" class="logo">
</figure>

[Blogger](https://www.blogger.com/){:target="_blank"}, the blog-publishing service from Google, does not provide a syntax highlighter out of the box. This means you need to resort to third-party solutions in order to have syntax highlighting in your posts. One of those solutions is the syntax highlighter created by Alex Gorbatchev which is called [SyntaxHighlighter](http://alexgorbatchev.com/SyntaxHighlighter/){:target="_blank"}.

One of the advantages of this highlighter is that it is purely written in JavaScript and as a result, it does not need a server-side component. SyntaxHighlighter can be easily integrated into your Blogger blog as we will detail in this tutorial.

# SyntaxHighlighter Install

The first thing to do is to open the dashboard of your Blogger blog as shown below. On the left-hand side find the menu that says <var>Template</var> and click on it.

<figure>
    <img src="{{ site.url }}/assets/images/posts/blogger/blogger-overview.png" alt="blogger overview">
</figure>

This will open the template page of your blog. Click on the button that says <var>Edit HTML</var> in order to open up the HTML editor as shown below.

<figure>
    <img src="{{ site.url }}/assets/images/posts/blogger/blogger-template.png" alt="blogger template">
</figure>

Click anywhere inside the editor window and press <var>CTRL+F</var>. A search box should now appear in the upper right hand corner as shown below. In the search box enter "<kbd>&lt;/head&gt;</kbd>" (without quotes) and press <var>ENTER</var>.

<figure>
    <img src="{{ site.url }}/assets/images/posts/blogger/blogger-edit-html-head.png" alt="blogger edit html head">
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

There are [a number of themes available for SyntaxHighlighter](http://alexgorbatchev.com/SyntaxHighlighter/manual/themes/){:target="_blank"}, the style sheet below is the default theme.

``` xml
<link href="http://alexgorbatchev.com/pub/sh/current/styles/shThemeDefault.css" rel="stylesheet" type="text/css" />
```

## 3) The specific brush(es) needed for the blog
Depending on the structured language that needs to be highlighted, the corresponding brush needs to be imported. The SyntaxHighlighter site contains [a complete list of all available brushes](http://alexgorbatchev.com/SyntaxHighlighter/manual/brushes/){:target="_blank"}. In this example we will add the brushes for <var>'Java'</var> and <var>'XML'</var>.

``` xml
<script src="http://alexgorbatchev.com/pub/sh/current/scripts/shBrushJava.js" type="text/javascript"></scrip>
<script src="http://alexgorbatchev.com/pub/sh/current/scripts/shBrushXml.js" type="text/javascript"></script>
```

> Only add the needed brushes as for each page the brushes are retrieved from alexgorbatchev.com (the SyntaxHighlighter site) and this increases your blog page load times!

## 4) The configuration script

After all needed dependencies have been added we need to enable a specific mode for Blogger and instruct SyntaxHighlighter to highlight all code blocks found on the web page. This is done by adding a JavaScript snippet as shown below.

``` xml
<script language="javascript" type="text/javascript">
    SyntaxHighlighter.config.bloggerMode = true;
    SyntaxHighlighter.all();
</script>
```

The complete script to be inserted in the Blogger template is shown below. Copy and paste right before the <var>'&lt;/head&gt;'</var> tag as shown on the screenshot.

``` xml
<!-- 'SyntaxHighlighter' additions START -->
<script src="http://alexgorbatchev.com/pub/sh/current/scripts/shCore.js" type="text/javascript"></script>
<link href="http://alexgorbatchev.com/pub/sh/current/styles/shCore.css" rel="stylesheet" type="text/css" />
<link href="http://alexgorbatchev.com/pub/sh/current/styles/shThemeDefault.css" rel="stylesheet" type="text/css" />
<script src="http://alexgorbatchev.com/pub/sh/current/scripts/shBrushJava.js" type="text/javascript"></script>
<script src="http://alexgorbatchev.com/pub/sh/current/scripts/shBrushXml.js" type="text/javascript"></script>

<script language="javascript" type="text/javascript">
    SyntaxHighlighter.config.bloggerMode = true;
    SyntaxHighlighter.all();
</script>
<!-- 'SyntaxHighlighter' additions END -->
```

Click the <var>Save template</var> button to save the changes made to your Blogger template. This concludes the setup, in the next section, will see how to use SyntaxHighlighter.

<figure>
    <img src="{{ site.url }}/assets/images/posts/blogger/blogger-edit-html-setup.png" alt="blogger edit html setup">
</figure>


## SyntaxHighlighter Usage

In order to use SyntaxHighlighter we need to wrap the section to be highlighted with an XML tag called <var>&lt;pre&gt;</var>. This tag has one required parameter called <var>'brush'</var> which is the same brush that was added in section 3 of the above setup.

For this example we will add a `HelloWorld` Java class to a <var>&lt;pre&gt;</var> tag with a <var>'Java'</var> brush and a Hello World XML file to a <var>&lt;pre&gt;</var> tag with a <var>'XML'</var> brush. Copy the below code and paste it inside a Blogger post as shown.

> Make sure all right angle brackets within the <pre> tags are HTML escaped, in other words, all < (less than character) must be replaced with "&amp;lt;" (without quotes, as shown below)!

``` xml
<pre class="brush: java">
public class HelloWorld {

    public static void main(String[] args) {
        System.out.println("Hello World!");
    }
}
</pre>

<pre class="brush: xml">
&lt;?xml version="1.0" encoding="UTF-8" ?>
&lt;text>Hello World!&lt;/text>
</pre>
```

<figure>
    <img src="{{ site.url }}/assets/images/posts/blogger/blogger-edit-post.png" alt="blogger edit post">
</figure>

Save and publish the page and the result should look like:

<figure>
    <img src="{{ site.url }}/assets/images/posts/blogger/syntaxhighlighter-example.png" alt="syntaxhighlighter example">
</figure>

# SyntaxHighlighter Options

In addition to the mandatory <var>'brush'</var> parameter, the <var>&lt;pre&gt;</var> tag has a number of optional parameters. For example it is possible to highlight one or more lines to focus the reader's attention by adding the <var>'highlight'</var> parameter as shown below. Checkout the [full list of additional parameters that can be configured](http://alexgorbatchev.com/SyntaxHighlighter/manual/configuration/){:target="_blank"}.

``` xml
<pre class="brush: java; highlight: [3,4,5]">
public class HelloWorld {

    public static void main(String[] args) {
        System.out.println("Hello World!");
    }
}
</pre>
```

The result of above snippet:

<figure>
    <img src="{{ site.url }}/assets/images/posts/blogger/syntaxhighlighter-highlight-example.png" alt="syntaxhighlighter highlight example">
</figure>

---

This concludes setting up SyntaxHighlighter on Blogger. If you found this post helpful or have any questions or remarks, please leave a comment.
