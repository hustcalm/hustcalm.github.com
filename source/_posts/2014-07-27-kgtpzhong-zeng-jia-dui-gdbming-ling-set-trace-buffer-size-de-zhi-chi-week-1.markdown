---
layout: post
title: "KGTP中增加对GDB命令“set trace-buffer-size”的支持 - Week 1"
date: 2014-07-27 19:34:45 +0800
comments: true
categories: KGTP Linux GDB Tracing
---

### CSDN开源夏令营 - 第一周工作总结

这一周主要对KGTP的实现进行了简单的熟悉和分析，主要参考的资料在：[1]。

#### 1.什么是KGTP

T大把KGTP称为Linux内核中的GDB快刀，其全称是Linux kernel GDB tracepoint module，是一个灵活轻量级实时的Linux调试器和跟踪器。

Linux的tracer infrastructure五花八门，包括Ftrace，Utrace等，建议大家参考一下这篇paper，【Ptrace, Utrace, Uprobes: Lightweight, Dynamic Tracing of User Apps】，下载链接在：[2]。Tracer在收集信息的时候不停止Linux内核，但是不能被GDB控制。

<!--more-->

而在调试Linux内核的时候，我们也有很多选择，比如KDB，KGDB等，Debugger将停止Linux内核，但是可以被GDB控制。

那么KGTP做了什么事情呢？Yes，作为Tracer和Debugger的一个桥梁，从而做到不停止Linux内核，且可以被本地或者远程的GDB控制。

目前，KGTP支持X86-32，X86-64，MIPS和ARM，支持Linux内核2.6.18到upstream，直接Android（因为Android底层仍然是Linux Kernel）。

#### 2. KGTP的实现

KGTP的大部分trace功能基于Kprobe，而用户态应用程序的trace功能则依赖Uprobe，对于使用Kprobe的情况，使用Kprobes-optimization还可提高Kprobe的速度。

作为Debugger和Tracer之间的桥梁，KGTP通过GDB的RSP协议与GDB通信，完成数据的交换和用户命令的解析，具体的信息可以看GDB的文档，比如：[GDB Remote-Protocol](https://www.sourceware.org/gdb/onlinedocs/gdb/Remote-Protocol.html)。KGTP内核态和用户态的数据交换，通过DebugFS或者ProcFS暴露给用户态的GDB。在接收到相关的GDB命令数据包后，完成相关的解析工作，设置对应的Tracepoint，在用户开始trace后（tstart），开始在Ring Buffer采集数据，结束后（tstop）可以供用户查看（tfind）。

KGTP作为一个内核module存在，只需要编译KGTP后insmod，而不需要重新编译内核，因此非常灵活，当不需要KGTP的时候，直接rmmod即可。

KGTP的数据分析主要使用GDB，因此代码中不需要很多数据分析的部分，核心源代码文件gtp.c，仅有13000行左右。

总结起来，KGTP是一个灵活且轻量级的实现，可以实时地对Linux进行跟踪和调试，这对于线上服务器的问题处理是非常有用的。

#### 3. KGTP的hack需要具备哪些知识

C语言功底+一定的Linux内核开发基础，具体的点总结如下：

*    （1）Linux内核的同步机制（锁，信号量等）
*    （2）字符驱动程序的实现原理，主要是GDB和KGTP的通信需要用到
*    （3）Linux内核module的编写，因此KGTP是以一个module的形式存在的
*    （4）Linux内核的tasklet和workqueue，KGTP的后台进程是一个守护进程gtpd
*    （5）Ring buffer的实现，Linux内核trace的RB的实现以及KGTP自身RB的实现
*    （6）Linux tracer的实现原理，因为KGTP是基于Kprobe和Uprobe实现的
*    （7）GDB的基本原理，尤其是GDBRSP

#### 4. 如何增加对GDB命令“set trace-buffer-size”的支持

*    （1）实现命令包的解析，参考[3][4]
*    （2）实现Ring Buffer数据的拷贝和其他处理（主要针对新分配缓冲区小于原有缓冲区的情况）

#### 5. 如何部署KGTP

强烈建议采用“一键安装”的方式，KGTP提供了部署脚本[kgtp.py]，十分方便。

Kernel需要相应地debug info，因此如果是自己编译内核，则需要：

    General setup —>
    [*] Kprobes
    [*] Enable loadable module support —>
    Kernel hacking —>
    [*] Debug Filesystem
    [*] Compile the kernel with debug info

如果是Distro，需要安装Linux内核调试镜像和Linux内核源码包和开发包。

只有这样，Kernel才能被GDB加载调试。

以Fedora为例，当完成了以上步骤后，直接

    sudo gdb /usr/lib/debug/lib/modules/3.14.8-200.fc20.x86_64/vmlinux -ex 'target remote /sys/kernel/debug/gtp'

根据使用内核版本的不同，加载的kernel image路径会有稍许区别，比如上面的`3.14.8-200.fc20.x86_64`。

至此，部署完毕。

#### 6. 如何使用KGTP

请参考KGTP的使用手册，具体链接见：[5]和[6]。

#### 7. 参考链接

*    [1] http://teawater.github.io/kgtp/index.html
*    [2] http://kernel.org/doc/ols/2007/ols2007v1-pages-215-224.pdf
*    [3] https://sourceware.org/gdb/current/onlinedocs/gdb/Starting-and-Stopping-Trace-Experiments.html
*    [4] https://sourceware.org/gdb/current/onlinedocs/gdb/Tracepoint-Packets.html
*    [5] http://teawater.github.io/kgtp/kgtp.html
*    [6] http://teawater.github.io/kgtp/kgtpcn.html
