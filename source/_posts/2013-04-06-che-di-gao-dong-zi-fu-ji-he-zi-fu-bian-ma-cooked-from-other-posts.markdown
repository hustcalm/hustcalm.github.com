---
layout: post
title: "彻底搞懂字符集和字符编码(Cooked from other posts)"
date: 2013-04-06 12:08
comments: true
categories: Geek
---
ASCII,ANSI,Unicode,UTF-8,BOM,UCS,GB2312,GBK,GB18030,BIG5,Code Page... Blabla...

对于以上概念，你是否都理解了呢？下面，带着大家彻底搞懂字符集和字符编码的相关知识。我是读了别人的posts，然后cook出来，整理思路，所以需要大家边读边仔细阅读相应的post哦～

## 入门篇
首先，简单了解一下。

*   [字符编解码的故事（ASCII，ANSI，Unicode，Utf-8区别）](http://www.imkevinyang.com/2009/02/%E5%AD%97%E7%AC%A6%E7%BC%96%E8%A7%A3%E7%A0%81%E7%9A%84%E6%95%85%E4%BA%8B%EF%BC%88ascii%EF%BC%8Cansi%EF%BC%8Cunicode%EF%BC%8Cutf-8%E5%8C%BA%E5%88%AB%EF%BC%89.html)

这篇文章，以类似《明朝那些事儿》的文笔，向大家介绍了”编码“的简史。

*   [字符编码笔记：ASCII，Unicode和UTF-8](http://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html)

阮一峰的博客，重点描述了ASCII和Unicode，并强调“**UTF-8是Unicode的实现方式之一**”，Little endian和Big endian的典故也说到了。跟上篇文章一样，适合入门。

## 初级篇
入门之后，需要搞清楚究竟“字符集”和“字符编码”是什么东东。

<!--more-->

*   [深入了解字符集和编码](http://www.iteye.com/topic/97803)

这篇文章，给出了“字符集”和“字符编码”的“**定义**”，并对ANSI编码有了比较合理的解释（使用2个字节来代表一个字符的各种汉字延伸编码方式，称为ANSI编码。 在简体中文系统下，ANSI编码代表GB2312编码，在日文操作系统下，ANSI编码代表JIS编码。 ）这里也提到了BOM的概念，在“**编程语言与编码**”那里描述了编程语言对于内部字符串是如何进行编码的。

## 中级篇
在基本了解了“字符集”和“字符编码”是什么东东后，让我们再仔细看看主流的“字符集和编码”。

*   [各种字符集和编码详解](http://space.itpub.net/55022/viewspace-713849)

本文也是按照“**字符编码的历程**”来展开，ASCII，ISO8859-1，GB2312，GBK，GB18030，BIG5，UCS，Unicode都说到了。而在编码方面，重点说了UTF-8，UTF-16，UTF-32，UTF-7。此外，提到了MIME（多用途网际邮件扩充协议），是现在邮件编码方式的主流，其中定义了两种编码方法Base64和QP（Quote-Pritable）。

## 高级篇
下面，更加细致地了解下“编码”是如何存储的，以及分析下优缺点。

*   [字符集和字符编码（Charset & Encoding）](http://www.cnblogs.com/skynet/archive/2011/05/03/2035105.html)

本文知识点比较系统，在看了以上的文章后，可以“**系统**”地整理下思路了。本文在描述了“ASCII字符集&编码","GBXXXX字符集&编码”以及“BIG5字符集&编码”后，提出了“**伟大的创想Unicode**”。之后，又说到了“互联网传输”的编码问题，Accept-Charset/Accept-Encoding/Accept-Language/Content-Type/Content-Encoding/Content-Language。

## 拾遗篇

了解“**字符集**”和“**字符编码**”的主线了，可能还对一些常见的概念疑惑。下面的链接就一些概念进行了简明的解释。

*   [中文字符编码标准+Unicode+Code Page](http://bbs.chinaunix.net/thread-3610023-1-1.html)

这里是想重点了解“**Code Page**”的，跟操作系统的实现有关。

*   [Code page](http://en.wikipedia.org/wiki/Code_page)

Code page is another term for character encoding. It consists of a table of values that describes the character set for a particular language. The term code page originated from IBM's EBCDIC-based mainframe systems,[1] but many vendors use this term including Microsoft, SAP,[2] and Oracle Corporation.[3] Vendors often allocate their own code page number to a character encoding, even if it is better known by another name (for example UTF-8 character encoding has code page numbers 1208 at
IBM, 65001 at Microsoft, 4110 at SAP).

*   [区位码和内码，外码，国标码](http://hi.baidu.com/kumosheng/item/0864400782d3df7abfe97e87)

我国“技术专家”的智慧结晶！看看汉字是怎么被编码的！～

*   [输入法中全角/半角的区别以及切换方法](http://www.htmer.com/article/232.htm)

刚学输入法的时候，就一直被“全角”和“半角”折磨着，现在终于搞明白了！

## 总结篇
输入，编码，显示，存储，读取，解码，输出。

看来，不管什么东西，还是要从“操作系统”的层面去认识！

