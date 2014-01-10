---
layout: post
title: "Getting Started with Qt and Qt Creator"
date: 2014-01-02 15:42
comments: true
categories: Qt C/C++ Programming
---

Recently I'm doing a project for the Lab and choose C++ as the programming language, but for the Framework, I considered MFC and Qt at first but choose Qt without hesitation as soon as I recalled the pains when I tried to play with MFC. In short, MFC is just showing too much to the developer and looks a little annoying and more complicated(C++ itself is complicated enough).

To be clear, I did not try Qt before either and totally new to it. But after two months, I wrote a running application which can deal with Cameras, IMU and Inclinometer, also a 6DOF controller. Besides, we have multithreading in it and multiply timers are running simultaneously. To summarize what I have learned and also share my experience to you is the desire to write this post.

<!--more-->

I also find that Qt Creator is really cool with the plugin `FakeVim` enabled(It is installed by default ,at least in my Qt Creator 2.8.1 Windows version), a light wight IDE with powerful abilities! So in this post, after talking about how to get started with Qt, I will give some appreciations to Qt Creator:-)

------------------------------

### Beginning with Qt
#### Get to know Qt
It's essentially a GUI toolkit when fist designed, but after years of development, it has evolved into a framework which is cross platform and popular.

For official introduction, visit [Qt Project](http://qt-project.org/).

#### Choosing the way that you use
I will not talk about the versions ,like 5.x or 4.x. Let's say something about the `prebuilt-binaries` and `source code`.

If you choose to build from scratch, it would take a bit of time to do that since Qt is big enough. If using the built binaries, do look at the related compilor, for Windows, you got options for VS 2008, VS 2010, VS 2012 and MinGW stuff. If you are using VS 2008(in my case), choosing this file [Qt libraries 4.8.5 for Windows (VS 2008, 235 MB)](http://download.qt-project.org/official_releases/qt/4.8/4.8.5/qt-win-opensource-4.8.5-vs2008.exe), just for an example,then probably you are good to go when you build your project with VS 2008 as the backend.

For Qt Creator, I prefer to use the perbuilt one too, but if you want to install some specified plugins, build from source code may be needed.

#### A good book would help a lot
I read the book `C++ GUI Programming with Qt 4(2nd Edition)` not because it's the best book for Qt, just because that we got one Chinese version copy of the book in our Lab. I went through the first 3 chapters and the appendix(mainly to know more about `qmake`), then referenced other chapters while I got problems when coding.

For the comment that I wrote for the book, you can find it [here](http://book.douban.com/review/6489896/).

However, this book is recommended as the official Qt book. You may want to try it out yourself and make your own judge, if you find another excellent book, do let me know:-)

#### See what the official says
For the official tutorials, you can refer to their website, like:

*   [Getting Started with Qt](http://qt-project.org/doc/qt-5/gettingstarted.html)
*   [How to Learn Qt](http://qt-project.org/doc/qt-4.8/how-to-learn-qt.html)
*   [How to Learn Qt](http://qt-project.org/doc/qt-4.7/how-to-learn-qt.html)

You may notice that each Qt version got their own getting started web page, however, don't bother, checkout them all and pick the brilliant part that suits yourself.

As time goes by, there may be new links and the listed links would be out of date. Go to the official website and find the real links then.

------------------------------

### Qt Creator as your Swiss Army Knife
Yes, it is just an IDE, compared with Eclipse, it's dedicated to Qt and C++(Not exactly, it supports Android and even BlackBerry development), so much light-wighted. However, it's flexible and with lots of plugins, see [Qt Creator Plugin Gallery](http://qt-project.org/wiki/Qt_Creator_Plug-in_Gallery).

As I'm not expert to Qt Creator either, I will give several examples here.

#### FakeVim
Default shipped with Qt Creator, I'm really excited when I find it. Support external `.vimrc` and give you almost the native vim experience.

I used the default `FakeVim` key bindings, they are just as same as the official vim, besides that, you may use other short cuts provided by Qt Creator, among them, `F2` for following the symbol under cursor and `F4` for switching header and souce file are the most frequently used. For generating source code tags, it's well cared by a background process, however when got large source files, I suffered for temporary `Not Responding`.

For quick view of the options:
{% img /images/blog_images/qt_creator-fakevim.png %}

#### Doxygen
This plugin uses the standard [Doxygen](http://www.doxygen.org) as the backend and give useful utilities for documenting the whole project or a file or a class, function. variable, whatever.

The following links really help a lot:o

*   [Using doxygen with Qt Creator](https://facwiki.cs.byu.edu/HCMI/index.php/Using_doxygen_with_Qt_Creator)
*   [QtCreator-Doxygen](http://dev.kofee.org/projects/qtcreator-doxygen/wiki)

#### Version Control
Qt Creator makes it really easy for integrating with version control tools, an image will tell everything:
{% img /images/blog_images/qt_creator-versioncontrol.png %}


------------------------------

Hope this post do a little help for your getting started with Qt Programming:-)
