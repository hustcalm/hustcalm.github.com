---
layout: post
title: "KGTP中增加对GDB命令“set trace-buffer-size”的支持 - Week 2"
date: 2014-07-27 19:34:48 +0800
comments: true
categories: KGTP Linux GDB Tracing
---
### CSDN开源夏令营 - 第二周工作总结

这一周在上一周对整体框架的认识和理解之上，开始全面地阅读KGTP的源代码，理解了梗概，很多细节需要再仔细研读，主要参考是其代码仓库：[1]。

#### 1. KGTP的核心代码文件

毫无疑问，KGTP最精髓的代码都在gtp.c文件中，约13500行代码，这其中包括了与GDB的通信，具体Query Packets的处理，Trace Points的注册和删除，事件的触发，硬件断点的设置和处理，等几乎所有的KGTP核心事务。在短短1万多行代码中，体现了几乎所有Linux Kernel开发过程中会用到的features，SMP的支持，Linux的list实现，锁和同步，工作队列，tasklet，硬件和软件中断，等等。

<!--more-->

其它辅助文件则有：

*    `gtp_rb.c`      KGTP自己实现的一套Ring Buffer（默认使用）
*    `perf_event.c`  Perf的核心实现，里面也实现Ring Buffer
*    `ring_buffer.c` Linux内核为Trace的infrastructure提供的统一的Ring Buffer实现（比如Ftrace）

#### 2. KGTP核心功能实现的源码分析

按照数据通信流程，**GDB ——>   GDBRSP ——> SYSFS ——> KGTP ——> Kernel**，简单分析如下：

`GDBRSP`是Remote Serial Protocol，用于GDB远程调试发包，但是也支持File I/O和Console，而KGTP正是利用了GDBRSP对于File I/O的支持，完成GDB和KGTP的通信，在接受到GDB发送的Packet后，KGTP进行解析。而通信的介质则是通过debugfs或者其他kernel space和user space进行数据交换的接口进行（参考[2]）。

KGTP接收到了GDB的指令后，基于Kprobes和Uprobes进行内核和用户应用程序的trace，step和watch可以基于hardware-breakpoints实现。

KGTP自己维护了一个ring buffer的，用于Trace Frame的存储，查询及dump。

##### （1）KGTP和GDB的通信

核心函数：

*    `gtp_init`    建立对应的ProcFS或者DebugFS文件结点

以下函数利用GDBRSP完成GDB和KGTP基于packet的通信：

*    `gtp_open`
*    `gtp_release`
*    `gtp_ioctl`
*    `gtp_write`
*    `gtp_read`
*    `gtp_poll`

##### （2）KGTP对GDB数据包的处理

`gtp_gdbrsp_*` 系列函数完成了对GDB数据包的解析和处理。

比如：

*    `gtp_gdbrsp_QT`        处理QT的packet
*    `gtp_gdbrsp_qtstart`   Start Trace Experiments，注册kprobe，uprobe以及watchpoints，hardware breakpoints等，并分配存储空间
*    `gtp_qdbrsp_qtstop`    Stop Trace Experiments，flush work queue，tasklet_kill，以及unregister在qtstart注册的所有probe points和一些回调函数

参考的话，就是GDB的官方手册了，比如：

GDB的[Query Packet](https://www.sourceware.org/gdb/onlinedocs/gdb/General-Query-Packets.html).
Packets starting with ‘q’ are general query packets; packets starting with ‘Q’ are general set packets. General query and set packets are a semi-unified form for retrieving and sending information to and from the stub.

##### （3）Tracepoints和Breakpoints的注册和删除

*    `gtp_uprobe_register`
*    `gtp_register_hwb`
*    `gtp_unregister_hwb`

##### （4）TSV的处理

`gtp_var_*`

可以重点看一下`gtp_var_special_add_all`。

##### （5）Ring Buffer的处理

这里后续再详细分析，Ring Buffer的实现也是KGTP的核心之一。

#### 3. 阅读源码小技巧&问题总结


参考`LXR`[3]，直接使用`Identifier Search`[4]，遇到不明白的宏定义，函数定义，都可以直接到Linux源码中一探究竟。

以下是我阅读代码过程遇到的一些问题总结：

（1）EXPORT_SYMBOL

http://stackoverflow.com/questions/9836467/whats-meaning-of-export-symbol-in-linux-kernel-code

http://www.linux.com/learn/linux-training/31161-the-kernel-newbie-corner-kernel-symbols-whats-available-to-your-module-what-isnt

（2）container_of

http://lxr.free-electrons.com/source/include/linux/kernel.h#L833

    /**
    827  * container_of - cast a member of a structure out to the containing structure
    828  * @ptr:        the pointer to the member.
    829  * @type:       the type of the container struct this is embedded in.
    830  * @member:     the name of the member within the struct.
    831  *
    832  */
    833 #definecontainer_of(ptr,type, member) ({                      \
    834         const typeof( ((type *)0)->member ) *__mptr = (ptr);    \
    835         (type *)( (char *)__mptr - offsetof(type,member) );})

（3）kzalloc

http://lxr.free-electrons.com/source/include/linux/slab.h#L649

（4）kmalloc

http://lxr.free-electrons.com/source/include/linux/slab.h#L452

（5）list

http://lxr.free-electrons.com/source/include/linux/list.h

（6）INIT_WORK

http://lxr.free-electrons.com/source/include/linux/workqueue.h

（7）vfree

http://lxr.free-electrons.com/source/mm/vmalloc.c#L1490

（8）preempt_disable

http://lxr.free-electrons.com/source/include/linux/preempt.h#L38

（9）barrier

https://www.kernel.org/doc/Documentation/memory-barriers.txt

（10）local_irq_save

http://lxr.free-electrons.com/source/include/linux/irqflags.h#L93

（11）notifier_block

http://lxr.free-electrons.com/source/include/linux/notifier.h#L53

（12）flush_workqueue

http://lxr.free-electrons.com/source/kernel/workqueue.c#L2641

（13）tasklet_kill

http://lxr.free-electrons.com/source/kernel/softirq.c#L566

（14）unregister_wide_hw_breakpoint

http://lxr.free-electrons.com/source/kernel/events/hw_breakpoint.c#L536

（15）unregister_kprobe

http://lxr.free-electrons.com/source/kernel/kprobes.c#L1668

（16）wake_up_interruptible_nr

http://lxr.free-electrons.com/source/kernel/sched/wait.c#L88

（17）queue_work

http://lxr.free-electrons.com/source/kernel/sched/wait.c#L88

（18）rcu_read_lock

http://lxr.free-electrons.com/source/include/linux/rcupdate.h#L798

（19）tasklet_init

http://lxr.free-electrons.com/source/kernel/softirq.c#L555

（20）IPI

http://en.wikipedia.org/wiki/Inter-processor_interrupt

（21）register_die_notifier

http://lxr.free-electrons.com/source/kernel/notifier.c#L544

（22）EBUSY

http://lxr.free-electrons.com/source/include/uapi/asm-generic/errno-base.h#L19


#### 4. KGTP的调试&源码注释

编译KGTP的时候添加D=1，便以debug的方式编译，在Fedora下可以通过journalctl -k查看其输出，其中-k是指过滤Kernel信息。其它系统下可能是在/var/log之类的文件查看，总之KGTP通过内核接口printfk输出，视具体的系统不同，查看内核信息的方式也会有变化。

关于有注释的代码，请看[5]。

#### 5. 参考链接

*    [1] https://github.com/teawater/kgtp
*    [2] http://people.ee.ethz.ch/~arkeller/linux/multi/kernel_user_space_howto.html#toc1
*    [3] http://lxr.free-electrons.com
*    [4] http://lxr.free-electrons.com/ident
*    [5] https://code.csdn.net/Calmdownba/kgtp/tree/comments-from-scratch
