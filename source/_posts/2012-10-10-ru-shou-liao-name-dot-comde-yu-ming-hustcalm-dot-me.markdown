---
layout: post
title: "入手了name.com的域名hustcalm.me"
date: 2012-10-10 01:50
comments: true
categories: domain name.com
---
相比来说，name.com家的注册貌似比godaddy家的简单些，而且设置起来更适合于新手，具体请参考文章[Name.com和Godaddy的一些区别](http://hugege.com/2010/01/19/the-diference-between-name-com-and-godaddy/)。

今天发现一个比较郁闷的事情，本地rake preview没有问题，rake generate and rake deploy后却在hustcalm.github.com上看不到效果，后来神奇地绑定域名hustcalm.me后看到了效果。莫非是域名绑定惹的祸？可是在绑定前就存在此问题。

下面简单说说如何给Github Pages绑定Name.com的注册域名，以我注册的hustcalm.me域名为例。
<!--more-->
echo "hustcalm.me" >> ~/blog/octopress/source/CNAME

git add .

git commit -a -m 'add custom domain CNAME configuration'

git push origin source

接下来到Name.com的Account里面设置域名指向，添加两个A记录，分别为：

hustcalm.me 207.97.227.245

www.hustcalm.me 207.97.227.245

稍等一会，就可以访问hustcalm.me了。

感觉Name.com家的服务还是很快的。另外，在注册域名的时候默认附带一个收费的whois privacy，不知道是干啥的，但是买了下来。
官方文章请参考[Deploying to Github Pages](http://octopress.org/docs/deploying/github)
