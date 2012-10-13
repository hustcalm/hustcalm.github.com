---
layout: post
title: "Ubuntu下检出Cygwin搭建的Octopress遇到的问题"
date: 2012-10-13 19:37
comments: true
categories: Octopress Cygwin Ubuntu
tags: Cygwin
---
原则很简单，只要记住“your_local_octopress_directory”对应的是remote source branch，而“_deploy”对应的是remote_master branch即可。		
而参考上一篇文章中的链接“为已经存在的github Octopress配置本地环境“，分别检出了source和master到本地，然后就出现了以上的对应关系。		
可问题是，Cygwin搭建的Octopress会出现问题，具体请参考链接[在Cygwin中搭建Octopress环境](madeye.me/2011/12/17/setup-octopress-on-windows)，主要是posix-spawn的问题，修改Gemfile.lock以及gems下面的albino.rb即可，可以Ubuntu不存在这种问题，因此需要隔离这一修改，最直观的是想把Gemfile放到.gitignore里面去，不管它了。另外，如果更新Octopress的话，Ruby的版本也会有所变化，我的策略是直接修改.rbenv-version，不理会版本的更新，当然也可以用.gitignore实现。		
而现在写文章先不管Octopress了，只管_deploy。		
唉，郁闷:(。
