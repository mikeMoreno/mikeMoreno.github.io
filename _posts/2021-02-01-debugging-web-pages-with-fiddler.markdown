---
layout: post
title:  "Debugging Web Pages with Fiddler"
date:   2021-02-1
categories: fiddler debugging
---

Fiddler is a web debugging proxy server that was originally written by Eric Lawrence of Microsoft 
and has since been acquired by Telerik. It’s commonly used to view HTTP/S traffic between client 
and server software. A lesser-known feature is Fiddler’s AutoResponder. The AutoResponder lets you supply 
your HTML and JavaScript files in place of files that the server would send you.


How it works is you provide a matching rule for a URL, and if a URL matches, Fiddler responds with a file you specify.


Let’s try creating a simple rule for example.com. Our rule will match on the full [example.com](http://example.com/) 
address, and it will respond with our HTML file. Our file will be a copy of example.com’s HTML, but the background color will be changed.


Our rule will look like this (make sure to check all the required boxes):


![our rule](https://raw.githubusercontent.com/mikeMoreno/mikeMoreno.github.io/master/images/examples/debugging-web-pages-with-fiddler/our-rule.png "Our Rule")


After the rule is enabled and we visit example.com, we should see that our HTML file has been supplied,
instead of the real example.com file:


The AutoResponder can also be used to supply JavaScript files. It’s a great tool in any web developer’s toolkit.
