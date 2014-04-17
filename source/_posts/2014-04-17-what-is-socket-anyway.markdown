---
layout: post
title: "What is socket anyway?"
date: 2014-04-17 21:05:06 +0800
comments: true
categories: Unix Socket
---

I bet you've heard of `socket` for a while, but do you really know what is a `socket`?

This post is a quick getting-started for figuring out the basic concepts of socket and hopefully enpowers you to begin writing simple network programming snippets. It aims to bootstrap your understanding to socket and network programming and serves as a good starting point to truth. From definition to practice and deep into research, let's start the venture!

<!--more-->

## The definition

*   [Berkeley sockets](http://en.wikipedia.org/wiki/Berkeley_sockets)

Let history and standards tell everything.

## Unix or Internet

*   [unix domain sockets vs. internet sockets](http://lists.freebsd.org/pipermail/freebsd-performance/2005-February/001143.html)

See what the freebsd gurus say.

## Unix socket deeper

*   [Demystifying Unix Domain Sockets](http://www.thomasstover.com/uds.html)

Code tells the truth.

## Network Programming Go

*   [Beej's Guide to Network Programming Using Internet Sockets](http://beej.us/guide/bgnet/output/html/singlepage/bgnet.html)

Practices make best.

## What's Next

*   [How to Build a simple HTTP server in C](http://stackoverflow.com/questions/176409/how-to-build-a-simple-http-server-in-c)

Wonder how to implement a HTTP server, like `Apache` or `Nginx`, you'd better implement a simple, stupid but your own one.

*   [The C10K problem](http://www.kegel.com/c10k.html)

Well, this is really where it hurts. Catch up with the paper, conquer it or give up early!
