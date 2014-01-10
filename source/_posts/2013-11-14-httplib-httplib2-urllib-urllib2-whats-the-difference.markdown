---
layout: post
title: "httplib httplib2 urllib urllib2-what's the difference"
date: 2013-11-14 11:24
comments: true
categories: Python Network
---
Lots of people may got confused by the Python modules- `httplib, httplib2, urllib, urllib2`. Judge from their names, we may prefer the x2 module because they may be version 2 of the module and should work better. Maybe you are right for some degree(as for the httplib). 

However, things are a little bit complicated which needs a brief explanation indeed.

<!--more-->

## What's the claim
First, let's see what a specified module's aim. I will quote from the official site:-)

### httplib
[Here](http://docs.python.org/2/library/httplib.html) is a link from `The Python Standard Library` for `Python 2.7.6`.

    This module defines classes which implement the client side of the HTTP and HTTPS protocols. 
    It is normally not used directly — the module urllib uses it to handle URLs that use HTTP and HTTPS.

### httplib2
[This](https://code.google.com/p/httplib2/) is the official website hosted on `Google Code`. It claims:

    A comprehensive HTTP client library that supports many features left out of other HTTP libraries.
and the `Project Goal`:
    To become a worthy addition to the standard Python library.

It features `HTTP and HTTPS`, `Keep-Alive`, `Authentication`(support *Digest*, *Basic*, *WSSE*, *HMAC Digest* and *Google Account Authentication*), `Caching`, `All Methods`, `Redirects`, `Compression`, `Lost update support` and `Unit Tested`.

### urllib
Quote from [here](http://docs.python.org/2/library/urllib.html):
    
    This module provides a high-level interface for fetching data across the World Wide Web. 
    In particular, the urlopen() function is similar to the built-in function open(), but accepts Universal Resource Locators (URLs) instead of filenames.  
    Some restrictions apply — it can only open URLs for reading, and no seek operations are available.

### urllib2
[The Standard Python Library](http://docs.python.org/2/library/urllib2.html) stats that:

    The urllib2 module defines functions and classes which help in opening URLs (mostly HTTP) in a complex world — basic and digest authentication, redirections, cookies and more.
    It's an extensible library for opening URLs.

## What's the difference
### httplibx and urllibx
`urllib/urllib2` is built on top of `httplib`. It offers more features than writing to `httplib` directly.
However, `httplib` gives you finer control over the underlying connections.

If you're dealing solely with HTTP/HTTPs and need access to HTTP specific stuff, use httplib.
For all other cases, use urllibx(note that `urllib and urllib2` have different capabilities, thus are always used together).

### httplib and httplib2
Basically, httplib2 is `Google's python httplib implementation` but much more powerful.

They actually do the same things utilizing HTTP/HTTPs and other network protocals like FTP, typically we can call them `HTTP client library`.

If `httplib` can't fulfill your need(as if you need the `Redirects` feature), consider switching to `httplib2` then. 

To make things simpler, I'd rather using `httplib2` as my default `HTTP client library` for Python.

### urllib and urllib2
Quote from this post [Python: difference between urllib and urllib2](http://www.hacksparrow.com/python-difference-between-urllib-and-urllib2.html):

`urllib` and `urllib2` are both Python modules that do URL request related stuff but offer different functionalities. Their two most significant differences are listed below:

*   `urllib2` can accept a Request object to set the headers for a URL request, urllib accepts only a URL. That means, you cannot masquerade your User Agent string etc.
*   `urllib` provides the urlencode method which is used for the generation of GET query strings, urllib2 doesn't have such a function. This is one of the reasons why urllib is often used along with urllib2.

**Note**:

The urllib module has been split into parts and renamed in Python 3 to `urllib.request`, `urllib.parse`, and `urllib.error`. The 2to3 tool will automatically adapt imports when converting your sources to Python 3. Also note that the urllib.urlopen() function has been removed in Python 3 in favor of urllib2.urlopen().

## Usage
*   [HOWTO Fetch Internet Resources Using urllib2](http://docs.python.org/2.7/howto/urllib2.html#urlerror)
*   [How to use urllib2 in Python](http://www.pythonforbeginners.com/python-on-the-web/how-to-use-urllib2-in-python/)
*   [python 模块：httplib、urllib和urllib2的简单用法](http://adchoices.sinaapp.com/topic/47/python-%E6%A8%A1%E5%9D%97-httplib-urllib%E5%92%8Curllib2%E7%9A%84%E7%AE%80%E5%8D%95%E7%94%A8%E6%B3%95)
*   [Python 标准库 urllib2 的使用细节](http://zhuoqiang.me/python-urllib2-usage.html)
*   [httplib2 v0.4 documentation](http://httplib2.googlecode.com/hg/doc/html/libhttplib2.html)

## Reference
*   [Python: difference between urllib and urllib2](http://www.hacksparrow.com/python-difference-between-urllib-and-urllib2.html)
*   [Resolving HTTP Redirects in Python](http://www.zacwitte.com/resolving-http-redirects-in-python)
