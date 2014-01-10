---
layout: post
title: "Hello Octopress From Ubuntu 12.04"
date: 2012-11-24 01:00
comments: true
categories: Octopress
---
今天又把Octopress博客安了一个新家，Windows XP下虚拟出来的Ubuntu 12.04.想想从Cygwin上开始建立博客，后来到Ubuntu 11.04上维护，现在又搭一个新窝，自己都佩服自己的折腾能力了:)

遇到的主要问题，除了`posix-spawn`，竟然还有`openssl`，随之而来的依赖缺失，mkmf，编译的时候又输给了路径问题，Ubuntu本身带着1.8.7的`ruby`，而现在我的Octopress使用`1.9.3-p194`，并且用了`rbenv`和`ruby-build`进行版本管理，这给编译`openssl`的时候带来一个困扰，直接在`ext/openssl`下运行`ruby
extconf.rb`的话生成的Makefile是依赖系统的1.8.7的，因为直接执行`ruby`，由于路径设置问题，运行的是系统的Ruby，而不是我们自己安装的，所以这里需要小心，通过`~/.rbenv/versions/1.9.3-p194/bin/ruby extconf.rb`解决问题。

参考的链接有以下，感谢他们：

<!--more-->

*   [为已经存在的github Octopress配置本地环境](http://www.360doc.com/content/12/0216/16/1016783_187128091.shtml)
*   [在Cygwin中搭建Octopress环境](http://madeye.me/2011/12/17/setup-octopress-on-windows)
*   [Octopress:no such file to load:zlib,openssl](http://ns2.beta4better.org/2012/01/octopress-no-such-file-to-load-zlib-or-openssl.html)
*   [ubuntu下安装ruby后openssl找不到的问题](http://blog.csdn.net/dqatsh/article/details/2125089)
*   [in 'require':no such file to load -openssl (LoadError)](http://www.myexception.cn/ruby-rails/665543.html)
*   [安装ruby-debug-base是mkmf(LoadError)问题的解决办法](http://www.linuxdiyf.com/viewarticle.php?id=68545)

暂时就这么多了，发现有问题才比较有意思。

如果你在设置Octopress的时候有任何疑问，欢迎和我讨论:)

