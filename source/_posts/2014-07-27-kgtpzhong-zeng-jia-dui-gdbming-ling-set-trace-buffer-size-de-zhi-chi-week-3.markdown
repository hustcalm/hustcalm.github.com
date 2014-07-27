---
layout: post
title: "KGTP中增加对GDB命令“set trace-buffer-size”的支持 - Week 3"
date: 2014-07-27 19:34:52 +0800
comments: true
categories: KGTP Linux GDB Tracing
---

### CSDN开源夏令营 - 第三周工作总结

这一周主要实现了“set trace-buffer-size”实现的数据通信部分，即完成了GDB和KGTP的数据交互部分，下面简单分析一下实现。在说代码实现之前，还是简单地回顾一下，如何摸清KGTP的脉络。

#### 1. 如何跟踪KGTP

我的平台是Fedora，步骤如下：

    make D=1
    sudo insmod gtp.ko
    sudo gdb /usr/lib/debug/lib/modules/3.14.8-200.fc20.x86_64/vmlinux -ex 'target remote /sys/kernel/debug/gtp'

<!--more-->

此时，进入了gdb的console，可以通过gdb的commands和KGTP通信了。观察KGTP的一个重要的手段则是看printk的信息，由于编译时使用了“make D=1”，因此对应KGTP的代码，比如：

     #ifdef GTP_DEBUG                                                                                            
         printk(GTP_DEBUG "gtp_gdbrsp_qtbuffer\n");                                                              
     #endif    

查看这些的方式则是使用“journalctl -k”，方便起见可以只查看末尾的部分信息，比如“journalctl -k -n 100”，在我的本地输出的部分信息如下：

    Jul 27 06:07:29 localhost.localdomain kernel: gtp_write: qAttached
    Jul 27 06:07:29 localhost.localdomain kernel: gtp_read
    Jul 27 06:07:29 localhost.localdomain kernel: gtp_write: qOffsets
    Jul 27 06:07:29 localhost.localdomain kernel: gtp_read
    Jul 27 06:07:29 localhost.localdomain kernel: gtp_write: g
    Jul 27 06:07:29 localhost.localdomain kernel: gtp_read
    Jul 27 06:07:29 localhost.localdomain kernel: gtp_write: m0,1
    Jul 27 06:07:29 localhost.localdomain kernel: gtp_gdbrsp_m: addr = 0x0 len = 1
    Jul 27 06:07:29 localhost.localdomain kernel: gtp_read
    Jul 27 06:07:29 localhost.localdomain kernel: gtp_write: m0,1
    Jul 27 06:07:29 localhost.localdomain kernel: gtp_gdbrsp_m: addr = 0x0 len = 1
    Jul 27 06:07:29 localhost.localdomain kernel: gtp_read
    Jul 27 06:07:29 localhost.localdomain kernel: gtp_write: qSymbol::
    Jul 27 06:07:29 localhost.localdomain kernel: gtp_read
    Jul 27 06:07:29 localhost.localdomain kernel: gtp_write: qTStatus
    Jul 27 06:07:29 localhost.localdomain kernel: gtp_gdbrsp_qT: Status
    Jul 27 06:07:29 localhost.localdomain kernel: gtp_read
    Jul 27 06:07:29 localhost.localdomain kernel: gtp_write: qTfP
    Jul 27 06:07:29 localhost.localdomain kernel: gtp_gdbrsp_qT: fP
    Jul 27 06:07:29 localhost.localdomain kernel: gtp_read

根据以上信息，可以顺藤摸瓜找到相应被调用的函数，从而屡清楚程序的执行逻辑，这对于理解KGTP的工作原理是非常有帮助的。关于journalctl[1]的使用，请自行“man journalctl”。在非systemd[2]的系统下，可能还需要通过“less /var/log/”这样的方式查看内核的输出，根据你使用的Linux随机应变即可。

#### 2. GDB和KGTP的通信实现

利用GDBRSP[3]，即GDB Remote Serial Protocol，GDB远程串行通信协议。关于Remote Serial Protocol，可以参考[4]。简单来说，用于GDB远程调试发包，但是也支持File I/O和Console[5]，而KGTP正是利用了GDBRSP对于File I/O的支持，完成GDB和KGTP的通信，在接受到GDB发送的Packet后，KGTP进行解析。而通信的介质则是通过debugfs或者其他kernel space和user space进行数据交换的接口进行的。

具体的实现，参考gtp.c的函数 `gtp_init`，部分代码如下：

     gtp_dir = debugfs_create_file("gtp", S_IRUSR | S_IWUSR, NULL,
                       NULL, &gtp_operations);
     if (gtp_dir == NULL || gtp_dir == ERR_PTR(-ENODEV)) {
         gtp_dir = NULL;
         goto out;
     }
     gtpframe_dir = debugfs_create_file("gtpframe", S_IRUSR, NULL,
                        NULL, &gtpframe_operations);
     if (gtpframe_dir == NULL || gtpframe_dir == ERR_PTR(-ENODEV)) {
         gtpframe_dir = NULL;
         goto out;
     }

可以看到，使用DebugFS，建立了相应的文件节点，比如gtp，gtpframe，并且注册了相应的file operations，比如`gtp_oprations`，`gtpframe_operations`。

拿前者为例，声明的代码如下：

    static const struct file_operations gtp_operations = {
         .owner      = THIS_MODULE,
         .open       = gtp_open,
         .release    = gtp_release,
     #if (LINUX_VERSION_CODE < KERNEL_VERSION(2,6,35))
         .ioctl      = gtp_ioctl,
     #else
         .unlocked_ioctl = gtp_ioctl,
         .compat_ioctl   = gtp_ioctl,
     #endif
         .read       = gtp_read,
         .write      = gtp_write,
         .poll       = gtp_poll,
     };

而这其中最核心的则是`gtp_read`和`gtp_write`，其中`gtp_read`，用于GDB从KGTP读取数据（**copy_to_user**），gtp_write，则是KGTP从GDB接收数据（**copy_from_user**）。具体的代码，这里就不贴了，大家可以自行分析。

GDB和KGTP通过GDBRSP关联，通过执行`target remote /sys/kernel/debug/gtp`实现。

#### 3. 添加“set-trace-buffer-size”的通信支持

有了以上的理论分析和准备，我们就可以着手实现“set trace-buffer-size”命令的解析了。首先，我们需要知道这个命令对应的GDBRSP的Query Packet以及KGTP利用哪个函数对其进行解析。通过[6]和[7]，我们得知如下信息：

    set circular-trace-buffer on
    set circular-trace-buffer off
    Choose whether a tracing run should use a linear or circular buffer for trace data. A linear buffer will not lose any trace data, but may fill up prematurely, while a circular buffer will discard old trace data, but it will have always room for the latest tracepoint hits.

    show circular-trace-buffer
    Show the current choice for the trace buffer. Note that this may not match the agent’s current buffer handling, nor is it guaranteed to match the setting that might have been in effect during a past run, for instance if you are looking at frames from a trace file.

    set trace-buffer-size n
    set trace-buffer-size unlimited
    Request that the target use a trace buffer of n bytes. Not all targets will honor the request; they may have a compiled-in size for the trace buffer, or some other limitation. Set to a value of unlimited or -1 to let the target use whatever size it likes. This is also the default.

    show trace-buffer-size
    Show the current requested size for the trace buffer. Note that this will only match the actual size if the target supports size-setting, and was able to handle the requested size. For instance, if the target can only change buffer size between runs, this variable will not reflect the change until the next run starts. Use tstatus to get a report of the actual buffer size.


    ‘QTBuffer:circular:value’
    This packet directs the target to use a circular trace buffer if value is 1, or a linear buffer if the value is 0.

    ‘QTBuffer:size:size’
    This packet directs the target to make the trace buffer be of size size if possible. A value of -1 tells the target to use whatever size it prefers.

嗯，有线索了，GDB执行相应地指令，则会通过GDBRSP向KGTP发送相应的Query Packet，注意到“circular-trace-buffer”和“trace-buffer-size”的Packet的格式相同。关于GDB的General Query Packet，参考[8]。**Packets starting with ‘q’ are general query packets; packets starting with ‘Q’ are general set packets. General query and set packets are a semi-unified form for retrieving and sending information to and from the stub.**

因此，参考“circular-trace-buffer”的实现是一个很好的突破口，所幸的是，KGTP已经实现了对其的支持。

#### 4. 添加“set trace-buffer-size”的具体实现

根据以上分析，我们很快找到了两个关键函数，“gtp_gdbrsp_QT  ”和“gtp_gdbrsp_qtbuffer”。OK，添加对“set trace-buffer-size”的解析，代码如下：

    static int
    gtp_gdbrsp_qtbuffer(char *pkg)
    {
    #ifdef GTP_DEBUG
        printk(GTP_DEBUG "gtp_gdbrsp_qtbuffer\n");
    #endif

        // Handle QTBuffer:circular:value
        if (strncmp("circular:", pkg, 9) == 0) {
            ULONGEST setting;

            pkg += 9;
            if (pkg[0] == '\0')
                return -EINVAL;
            hex2ulongest(pkg, &setting);

    #ifdef GTP_FTRACE_RING_BUFFER
    #if (LINUX_VERSION_CODE > KERNEL_VERSION(2,6,38)) \
        || defined(GTP_SELF_RING_BUFFER)
            gtp_circular = (int)setting;
            if (gtp_frame)
                ring_buffer_change_overwrite(gtp_frame, (int)setting);
    #else
            if (gtp_circular != (int)setting)
                gtp_circular_is_changed = 1;
    #endif
    #endif
            gtp_circular = (int)setting;

            return 0;
        }
        // Handle QTBuffer:size:size 
        else if (strncmp("size:", pkg, 5) == 0) {

            ULONGEST size;

            pkg += 5;

            if (pkg[0] == '\0')
                return -EINVAL;
            hex2ulongest(pkg, &size);

            // Handle the new ringbuffer size blow

            return 0;
        }

        return 1;
    }

重新编译，安装模块，然后在GDB中输入“set trace-buffer-size 100”，注意这里的100是十进制的，看下“journalctl -k”的输出，啥也没有。怎么回事，上面分析的不是挺美好的吗？经过几次尝试未果，把问题定位在了GDB对于Remote Packet的处理，不会是没有enable吧？

查看文档[9]，果然，应该使用`set remote set-buffer-size on`先使能，否则应该是直接被gdbrsp丢掉了。OK，使用`set trace-buffer-size 100`，输出：

    Jul 27 07:18:45 localhost.localdomain kernel: gtp_write: QTBuffer:size:64
    Jul 27 07:18:45 localhost.localdomain kernel: gtp_gdbrsp_QT: Buffer:size:64
    Jul 27 07:18:45 localhost.localdomain kernel: gtp_gdbrsp_qtbuffer
    Jul 27 07:18:45 localhost.localdomain kernel: gtp_gdbrsp_QT: return 0
    Jul 27 07:18:45 localhost.localdomain kernel: gtp_read

而使用`set trace-buffer-size unlimited`，输出：

    Jul 27 07:19:53 localhost.localdomain kernel: gtp_write: QTBuffer:size:-1
    Jul 27 07:19:53 localhost.localdomain kernel: gtp_gdbrsp_QT: Buffer:size:-1
    Jul 27 07:19:53 localhost.localdomain kernel: gtp_gdbrsp_qtbuffer
    Jul 27 07:19:53 localhost.localdomain kernel: gtp_gdbrsp_QT: return 0
    Jul 27 07:19:53 localhost.localdomain kernel: gtp_read

OK，至此对于“set trace-buffer-size”的通信支持就完成了，接下来需要根据用户设置的size对trace buffer做出调整，接下来的文章会说到如何实现。

#### 5. 参考链接

*    [1] http://www.freedesktop.org/software/systemd/man/journalctl.html
*    [2] https://wiki.archlinux.org/index.php/systemd
*    [3] https://sourceware.org/gdb/onlinedocs/gdb/Remote-Protocol.html
*    [4] http://blog.csdn.net/hmsiwtv/article/details/8759129
*    [5] https://sourceware.org/gdb/onlinedocs/gdb/File_002dI_002fO-Remote-Protocol-Extension.html#File_002dI_002fO-Remote-Protocol-Extension
*    [6] https://sourceware.org/gdb/current/onlinedocs/gdb/Starting-and-Stopping-Trace-Experiments.html
*    [7] https://sourceware.org/gdb/current/onlinedocs/gdb/Tracepoint-Packets.html
*    [8] https://www.sourceware.org/gdb/onlinedocs/gdb/General-Query-Packets.html#General-Query-Packets
*    [9] https://sourceware.org/gdb/onlinedocs/gdb/Remote-Configuration.html
