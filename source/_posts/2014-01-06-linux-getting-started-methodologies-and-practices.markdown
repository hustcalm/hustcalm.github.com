---
layout: post
title: "Linux Getting Started - Methodologies and Practices"
date: 2014-01-06 22:22
comments: true
categories: Linux
---

This is basically a topic that I gave to [Open-CAS](http://blog-ossclub.rhcloud.com/) group members as a technique share. Since I have known Linux for over 4 years and heavily used it for 2 years more or less, I find myself realy attracted by it. 

Also I want others to have the chance to live with a better OS(Unix and Linux, also Mac OS of course), so it's a good time to take the good points out of my mind and let more people know.

It's good to learn from each other especially when there are problems to solve. So I put my mind at a `github` repo called [linux-getting-started](https://github.com/hustcalm/linux-getting-started) and hope that others would join me to make this repo more useful.

<!--more-->

### What's in this repo
Currently, I'm addressing two aspects for `Linux Getting Started`, namely `Methodologies` and `Practices`. I know these are really big words, but they just fit here since I want to tell people `how to learn` not just `what to learn`.

#### Methodologies
Know what you are gonna do before you step out. Basically, for Linux beginners, the first step that I recommend is `install a Linux distro that you like`. Then just `use it` before you want to know everything about it(seems that no one will never ever). It's an OS after all, you use it for your daily job and solve your problems, not a magic software that just plays magic.

A good book would give you a real quick start, I have divided the books into three levels:

*   [Beginner]      

Newbies who just get to know Linux - Learn how to live with shell 

*   [Developer]     

Experienced programmers especially for C/C++ guys - get used to Linux Platform

*   [Next]          

Ones who will dive deeper into the most complicated but fun part - Kernel and Driver

**In one word, use it, learn it, dive into it!**

#### Practices
Using and Reading will give a quite well understanding for Linux, but not enough for mastering it. To become `real Linux users or developers`, remember `never leave your eyes off the screen and your fingers off your keyboard`, practice will just make perfect things and a great Linuxer, believe me:-)

I address two technical parts currently in this repo, `Shell Scripting` and `Makefile`. Other parts will definitely added as long as I maintain this repo.

##### Shell Scripting
Before beginning with `Shell Scripting`, I suppose you have already been familiar with varies of commands or utilities in shell.

To be straight, `shell` itself is a program which will interpret user's commands or instructions, convert them to `system calls` or other things that OS can understand. So basically speaking, `shell` is the interface between `user` and `OS Kernel`.

As a script language, it has its own variables, functions and flow control statements, also it features many other stuff which will ease much pain for system administration but maybe worse if not used properly. When we are writing shell scripts, we are actually dealing with processes and interprocess communication, each process would do its job and we can use `pipe`, `socket`, `named-pipes` or `file` to let them talk to each other.

You can definitely play some magic with shell scripting by choosing your favorite shell, like `Bash`, `Ksh`, `Zsh`, etc.

##### Makefile
As you get to know `shell`, you are ready to know something about `makefile`. Before that, I will talk about `make`, `autotools` first.

When we are trying to build a big project, no one want to invoke every single compiling commands by hand which is annoying but also error prone. Here comes `make` to figure out this problem. So when you download a package with source code, to build it, simply type:

    make
    sudo make install

Then you are done!

Is this that easy? Reality will really disappoint us, because the user's OS or environment always will not be consistent with the developer's. So `make` does not easy the pain for buiding a package or software from scratch, but making it easier to manage a big project.

So what's the solution? Here comes the `Autotools`, which will generate `makefiles` automatically according to the configuration of your system. Like you got a third party library A, and one package needs the library for one feature, `Autotools` will take care of whether you got the library installed, if yes, then build the package with the feature enabled, or just disable it otherwise.

Cool, isn't it? Yes for the users, but maybe no for the developers, I mean all the pain will be transfered to the developers since they got to write `Autotools` scripts to make things right. Just another wrapper to give a good, cross-platform and generic solution? Some people would quite agree with this opinion.

Thanks to the `Autotools`, people are getting used to:

    ./configure
    make
    sudo make install

There are many others excellent build systems or tools out there, like `CMake`, `QMake`, `Scons`, `Ninja` ,etc. Go and try them out!

------------------------------

### Where is the slide
Honestly speaking, the slide is just a chain which helps me to make a clear topic. So you will not find it big, just about 15 pages, however to spread all the sparking mind, you will find it's really a `big one` again.

If you want, download the slide at [here](https://github.com/hustcalm/linux-getting-started/blob/master/linux-getting-started-%40hustcalm.pdf).


------------------------------

### What if I want to see something else
Cool, buddy, I do hope that we can maintain this repo together and help more people to get started with Linux. So you are welcome to add interesting topics or useful links or good references or books to this repo.

You may either `fork` this [repo](https://github.com/hustcalm/linux-getting-started), add something interesting, then `pull request` to me.

Or you can just add what you want to know to the `wishList` file after `fork`, maybe shortly I will add stuff about the topic that you care about.


Help others, help yourself:-)
