---
layout: post
title: "POSIX threads-多线程编程初步"
date: 2013-09-06 10:24
comments: true
categories: POSIX pthread
---

学习，总是要深入进去才是好的。

由于多摄像头视频处理的需要，这两天“正式学习”了多线程编程，切入点是POSIX的pthread库。不管承认与否，在进程之后的线程，对软件开发有非常深远的影响。本质上的算法+数据，我个人的理解是，线程是在算法上做的改进（也就是对于整个作业，如何合理地切分，符合CPU和操作系统的工作习惯，也就是在比较小的开销下完成任务），相比于进程而言，由于多个线程都在同一个memory space，因此线程间通信以及创建线程的overhead都比进程小了不少，业内也把thread称为LWP（Light Wight Process）。

<!--more-->

多线程好处多多，容易让人头痛的便是线程的“同步”问题，也就是大家常说的互斥访问“临界区资源”，常用的方法就是，mutex，条件变量，信号量等。

学习的过程中，主要就线程的创建，终结，join，detach，参数传递，mutex的使用，条件变量和信号量的使用进行了实际的编码练习，剩下的关于调度策略，优先级，线程本地变量等还需要进一步学习。参考了一些精华文章，自己把写的demo放到了github上了。这不是一篇tutorial，所以就不具体说技术细节了，大家去看下面给出的链接和demo吧。

### 学习资源链接
*   [POSIX Threads Programming Tutorial From Lawrence Livermore National Laboratory](https://computing.llnl.gov/tutorials/pthreads/)
*   [Multithreaded Programming (POSIX pthreads Tutorial)](http://randu.org/tutorials/threads/)
*   [Creating multi-threaded C++ code](http://codebase.eu/tutorial/posix-threads-c/index.php)
*   [POSIX thread (pthread) libraries](http://www.yolinux.com/TUTORIALS/LinuxTutorialPosixThreads.html)
*   [POSIX threads explained-A simple and nimble tool for memory sharing](http://www.ibm.com/developerworks/library/l-posix1/index.html)
*   [Operating System - Multi-Threading](http://www.tutorialspoint.com/operating_system/os_multi_threading.htm)
*   [<pthread.h>](http://pubs.opengroup.org/onlinepubs/7908799/xsh/pthread.h.html)

### Demo
*   [POSIX-Threads-Beginner-s-Guide](https://github.com/hustcalm/POSIX-Threads-Beginner-s-Guide)

之后还是要系统地学习一下OS，MIT的6.828是一个不错的起点！
