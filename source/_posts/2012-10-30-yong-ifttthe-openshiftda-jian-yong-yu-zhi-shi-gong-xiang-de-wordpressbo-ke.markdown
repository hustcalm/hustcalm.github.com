---
layout: post
title: "用ifttt和Openshift搭建用于知识共享的Wordpress博客"
date: 2012-10-30 10:18
comments: true
categories: WordPress
---
笔者一直通过RSS订阅了解一些行业信息并且看一些比较好玩的东西，比如[异次元软件世界](http://www.iplaysoft.com/)什么的:)用了很长一段时间的QQ邮箱的阅读空间，整理了很多的收藏和分享以及自己的Tag Cloud，无奈，QQ邮箱的阅读空间除了对[Weibo](http://weibo.com/u/1236738912)比较友好以外，跟其它产品的交互比较弱，给QQ团队发建议希望他们能够提供导出Tag的功能，没人搭理:(

Anyway，现在转到Google Reader下了，把QQ邮箱的阅读空间的RSS导出到OPML文件，之后导入到GReader。

分享是学习的一大乐趣，之前一直是把好文分享到微博或者Delicious，现在萌生了把自己的订阅再挑选一些值得玩味的文章自动更新到blog的想法。互联网的神奇使得我的这个想法很快就实现了，主要用到的服务如下：

<!--more-->

*   [ifttt](http://www.ifttt.com)
实现GReader星标文章自动post到WordPress站点。

*   [Openshift](http://openshift.redhat.com)
提供Wordpress站点的托管。

*   [WordPress](http://wordpress.org)
不解释。用到了一些插件：

**Akismet** 用于处理垃圾评论。

**Auto Describe Taxonomies** 自动生成文章的tag和category。

**Efficient Related Posts** 显示相关文章。

**WP Limit Posts Automatically** 显示文章摘要而不是全文。

**WTI Like Post** 实现文章的“thumbs up“和”thumbs down“功能。

效果请看[笔者的知识分享站点](http://wordpress-hustcalm.rhcloud.com).

Enjoy auto blogging and knowledge sharring；）
