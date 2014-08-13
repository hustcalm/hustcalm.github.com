---
layout: post
title: "KGTP中增加对GDB命令“set trace-buffer-size”的支持 - Week 4"
date: 2014-08-10 19:33:09 +0900
comments: true
categories: KGTP Linux GDB Tracing
---

### CSDN开源夏令营 - 第四周工作总结

本周主要是在回顾“set trace-buffer-size”完成通信的基础上，对于指定参数size的处理，比如指定-1或者unlimited的时候，在KGTP端该如何做解析，以及该如何处理此时的buffer size。同时，根据“开题报告"的进度，中期检查的任务已经完成。对Trace Buffer的实现，做了初步的调研。

#### 1. GDB和KGTP通信实现的回顾

利用GDBRSP，即GDB Remote Serial Protocol，GDB远程串行通信协议。简单来说，用于GDB远程调试发包，但是也支持File I/O和Console，而KGTP正是利用了GDBRSP对于File I/O的支持，完成GDB和KGTP的通信，在接受到GDB发送的Packet后，KGTP进行解析。而通信的介质则是通过debugfs或者其他kernel space和user space进行数据交换的接口进行的。
具体实现，可以参考上一篇博文[1]。

<!--more-->

#### 2. GDB的Packet格式

GDB向target发送的Packet必须遵守提前设定的约定，才能被target按照相应的规则解析处理。关于GDB的Packet格式，可以参考[2]，这里详细列出了所有的预定义的指令和它们相应的响应数据。如果想进一步了解GDBRSP，可以阅读[3]。由于KGTP对Trace Point最感兴趣，因此我们也最关注Tracepoint Packets，只要有两种，分别以Q和q开头，其中Q表示设置命令，即set，而q表示查询命令，即query，具体看[4]。关于如何开始和停止Trace Experiments，看[5]。GDB对于用户输入的commands，会转换后封装成Packet，通过GDBRSP发给target，因此需要结合着[4]和[5]才能把对某一个命令如何进行包解析搞清楚，比如用户输入”tstart“，则会发送一个”QTStart“的包。

#### 3. KGTP对GDB的Packet的处理实现

有了GDB和KGTP的通信的基础设施，比如DebugFS和GDBRSP，正常的文件读写即可完成GDB和KGTP的数据交换。而为了完成相应的任务，KGTP必须对接收到的GDB的Packet进行解析，而我们又知道了GDB得Packet格式，因此只需要对接收到的数据按照格式做相应的解析即可。

在`gtp.c`的`gtp_write`函数，针对接收到的GDB包做了具体的解析，具体代码如下：

    static ssize_t
    gtp_write(struct file *file, const char __user *buf, size_t size,
          loff_t *ppos)
    {
        char        *rsppkg = NULL;
        int     i, ret;
        unsigned char   csum;
        int     is_reverse;

        if (down_interruptible(>p_rw_lock))
            return -EINTR;

        if (size == 0) {
    #ifdef GTP_DEBUG
            printk(GTP_DEBUG "gtp_write: try write 0 size.\n");
    #endif
            goto error_out;
        }

        size = min_t(size_t, size, GTP_RW_MAX);
        if (copy_from_user(gtp_rw_buf, buf, size)) {
            size = -EFAULT;
            goto error_out;
        }

        if (gtp_rw_buf[0] == '+' || gtp_rw_buf[0] == '-'
            || gtp_rw_buf[0] == '\3' || gtp_rw_buf[0] == '\n') {
            if (gtp_rw_buf[0] == '+')
                gtp_rw_size = 0;
            size = 1;
            goto out;
        }

        if (size < 4) {
            size = -EINVAL;
            goto error_out;
        }
        /* Check format and get the rsppkg.  */
        for (i = 0; i < size - 2; i++) {
            if (gtp_rw_buf[i] == '$')
                rsppkg = gtp_rw_buf + i + 1;
            else if (gtp_rw_buf[i] == '#')
                break;
        }
        if (rsppkg && gtp_rw_buf[i] == '#') {
            /* Format is OK.  Check crc.  */
            if (gtp_noack_mode < 1)
                gtp_read_ack = 1;
            size = i + 3;
            gtp_rw_buf[i] = '\0';
        } else {
            printk(KERN_WARNING "gtp_write: format error\n");
            size = -EINVAL;
            goto error_out;
        }

        wake_up_interruptible_nr(>p_rw_wq, 1);

        up(>p_rw_lock);
        if (down_interruptible(>p_rw_lock))
            return -EINTR;

    #ifdef GTP_DEBUG
        printk(GTP_DEBUG "gtp_write: %s\n", rsppkg);
    #endif

        /* Handle rsppkg and put return to gtp_rw_buf.  */
        gtp_rw_buf[0] = '$';
        gtp_rw_bufp = gtp_rw_buf + 1;
        gtp_rw_size = 0;
        ret = 1;
        is_reverse = 0;
        switch (rsppkg[0]) {
        case '?':
            if (gtp_current_pid == 0)
                snprintf(gtp_rw_bufp, GTP_RW_BUFP_MAX, "S05");
            else
                snprintf(gtp_rw_bufp, GTP_RW_BUFP_MAX, "T05;thread:p%x.%x;",
                     gtp_current_pid, gtp_current_pid);
            gtp_rw_size += strlen(gtp_rw_bufp);
            gtp_rw_bufp += strlen(gtp_rw_bufp);
            break;
        case 'g':
            ret = gtp_gdbrsp_g();
            break;
        case 'm':
            ret = gtp_gdbrsp_m(rsppkg + 1);
            break;
        case 'Q':
    #ifdef GTP_RB
            /* This check for "tfind -1" and let GDB into step replay.
               XXX: just test on X86_64.  */
            if (gtp_replay_step_id) {
                if (strcmp("QTFrame:ffffffff", rsppkg) == 0) {
                    ret = 0;
                    goto switch_done;
                } else
                    gtp_replay_reset();
            }
    #endif
            if (rsppkg[1] == 'T')
                ret = gtp_gdbrsp_QT(rsppkg + 2);
            else if (strncmp("QStartNoAckMode", rsppkg, 15) == 0) {
                ret = 0;
                gtp_noack_mode = -1;
            }
            break;
        case 'q':
            if (rsppkg[1] == 'T')
                ret = gtp_gdbrsp_qT(rsppkg + 2);
            else if (rsppkg[1] == 'C') {
                snprintf(gtp_rw_bufp, GTP_RW_BUFP_MAX, "QC%x",
                     gtp_current_pid);
                gtp_rw_size += strlen(gtp_rw_bufp);
                gtp_rw_bufp += strlen(gtp_rw_bufp);
                ret = 1;
            } else if (strncmp("qSupported", rsppkg, 10) == 0) {
    #ifdef GTP_RB
                snprintf(gtp_rw_bufp, GTP_RW_BUFP_MAX,
                     "QStartNoAckMode+;ConditionalTracepoints+;"
                     "TracepointSource+;DisconnectedTracing+;"
                     "ReverseContinue+;ReverseStep+;"
    #if (LINUX_VERSION_CODE >= KERNEL_VERSION(2,6,30))
                     "EnableDisableTracepoints+;"
    #endif
                     "qXfer:traceframe-info:read+;");
    #endif
    #if defined(GTP_FRAME_SIMPLE) || defined(GTP_FTRACE_RING_BUFFER)
                snprintf(gtp_rw_bufp, GTP_RW_BUFP_MAX,
                     "QStartNoAckMode+;ConditionalTracepoints+;"
    #if (LINUX_VERSION_CODE >= KERNEL_VERSION(2,6,30))
                     "EnableDisableTracepoints+;"
    #endif
                     "TracepointSource+;DisconnectedTracing+;");
    #endif
                gtp_rw_size += strlen(gtp_rw_bufp);
                gtp_rw_bufp += strlen(gtp_rw_bufp);
                ret = 1;
            }
    #ifdef GTP_RB
            else if (strncmp("qXfer:traceframe-info:read::",
                       rsppkg, 28) == 0)
                ret = gtp_gdbrsp_qxfer_traceframe_info_read(rsppkg
                                        + 28);
    #endif
            else if (strncmp("qRcmd,", rsppkg, 6) == 0)
                ret = gtp_gdbrsp_qRcmd(rsppkg + 6);
            else if (strncmp("qAttached", rsppkg, 9) == 0) {
                snprintf(gtp_rw_bufp, GTP_RW_BUFP_MAX, "1");
                gtp_rw_size += 1;
                gtp_rw_bufp += 1;
            }
            break;
        case 'S':
        case 'C':
            ret = -1;
            break;
        case 'b':
            rsppkg[0] = rsppkg[1];
            is_reverse = 1;
        case 's':
        case 'c':
            ret = gtp_gdbrsp_resume (rsppkg[0] == 's', is_reverse);
            break;
        case 'v':
            if (strncmp("vAttach;", rsppkg, 8) == 0) {
    #ifdef GTP_RB
                if (gtp_replay_step_id)
                    gtp_replay_reset();
    #endif
                ret = gtp_gdbrsp_vAttach(rsppkg + 8);
            } else if (strncmp("vKill;", rsppkg, 7) == 0) {
    #ifdef GTP_RB
                if (gtp_replay_step_id)
                    gtp_replay_reset();
    #endif
                /* XXX:  When we add more code to support trace
                   user space program.  We need add more release
                   code to this part.
                   Release tracepoint for this tracepoint.  */
                ret = 0;
            }
            break;
        case 'D':
    #ifdef GTP_RB
            if (gtp_replay_step_id)
                gtp_replay_reset();
    #endif
            gtp_gdbrsp_D(rsppkg + 1);
            ret = 0;
            break;
        case 'H':
            ret = gtp_gdbrsp_H(rsppkg + 1);
            break;
        case 'Z':
        case 'z':
            if (rsppkg[1] == '0')
                ret = gtp_gdbrsp_breakpoint(rsppkg + 3,
                                (rsppkg[0] == 'Z'));
            break;
        }
    switch_done:
        if (ret == 0) {
            snprintf(gtp_rw_bufp, GTP_RW_BUFP_MAX, "OK");
            gtp_rw_bufp += 2;
            gtp_rw_size += 2;
        } else if (ret < 0) {
            snprintf(gtp_rw_bufp, GTP_RW_BUFP_MAX, "E%02x", -ret);
            gtp_rw_bufp += 3;
            gtp_rw_size += 3;
        }

        gtp_rw_bufp[0] = '#';
        csum = 0;
        for (i = 1; i < gtp_rw_size + 1; i++)
            csum += gtp_rw_buf[i];
        gtp_rw_bufp[1] = INT2CHAR(csum >> 4);
        gtp_rw_bufp[2] = INT2CHAR(csum & 0x0f);
        gtp_rw_bufp = gtp_rw_buf;
        gtp_rw_size += 4;

    out:
        wake_up_interruptible_nr(>p_rw_wq, 1);
    error_out:
        up(>p_rw_lock);
        return size;
    }

我们可以看到根据接收到的Packet，分别调用了不同的handler来做处理，这里我们重点关注Tracepoint Packets，关于以QT开头的包，处理代码如下：

    static int
    gtp_gdbrsp_QT(char *pkg)
    {
        int ret = 1;

    #ifdef GTP_DEBUG
        printk(GTP_DEBUG "gtp_gdbrsp_QT: %s\n", pkg);
    #endif

        if (strcmp("init", pkg) == 0)
            ret = gtp_gdbrsp_qtinit();
        else if (strcmp("Stop", pkg) == 0)
            ret = gtp_gdbrsp_qtstop();
        else if (strcmp("Start", pkg) == 0)
            ret = gtp_gdbrsp_qtstart();
        else if (strncmp("DP:", pkg, 3) == 0)
            ret = gtp_gdbrsp_qtdp(pkg + 3);
        else if (strncmp("DPsrc:", pkg, 6) == 0)
            ret = gtp_gdbrsp_qtdpsrc(pkg + 6);
        else if (strncmp("Disconnected:", pkg, 13) == 0)
            ret = gtp_gdbrsp_qtdisconnected(pkg + 13);
        else if (strncmp("Buffer:", pkg, 7) == 0)
            ret = gtp_gdbrsp_qtbuffer(pkg + 7);
        else if (strncmp("Frame:", pkg, 6) == 0)
            ret = gtp_gdbrsp_qtframe(pkg + 6);
        else if (strncmp("ro:", pkg, 3) == 0)
            ret = gtp_gdbrsp_qtro(pkg + 3);
        else if (strncmp("DV:", pkg, 3) == 0)
            ret = gtp_gdbrsp_qtdv(pkg + 3);
    #if (LINUX_VERSION_CODE >= KERNEL_VERSION(2,6,30))
        else if (strncmp("Enable:", pkg, 7) == 0)
            ret = gtp_gdbrsp_qtenable_qtdisable(pkg + 7, 1);
        else if (strncmp("Disable:", pkg, 8) == 0)
            ret = gtp_gdbrsp_qtenable_qtdisable(pkg + 8, 0);
    #endif

    #ifdef GTP_DEBUG
        printk(GTP_DEBUG "gtp_gdbrsp_QT: return %d\n", ret);
    #endif

        return ret;
    }

而以qT打头的包，处理的接口如下：

    static int
    gtp_gdbrsp_qT(char *pkg)
    {
        int ret = 1;

    #ifdef GTP_DEBUG
        printk(GTP_DEBUG "gtp_gdbrsp_qT: %s\n", pkg);
    #endif

        if (strcmp("Status", pkg) == 0)
            ret = gtp_gdbrsp_qtstatus();
        else if (strcmp("fP", pkg) == 0)
            ret = gtp_gdbrsp_qtfp();
        else if (strcmp("sP", pkg) == 0)
            ret = gtp_gdbrsp_qtsp();
        else if (strcmp("fV", pkg) == 0)
            ret = gtp_gdbrsp_qtfsv(1);
        else if (strcmp("sV", pkg) == 0)
            ret = gtp_gdbrsp_qtfsv(0);
        else if (strncmp("V:", pkg, 2) == 0)
            ret = gtp_gdbrsp_qtv(pkg + 2);

        return ret;

从以上接口可以看到，是一个dispatch的过程，KGTP首先判断接收到的GDB包属于哪一类包，之后分发，而对于一个类型的包，又细分很多子包，最终分发给具体的接口实现，拿上面为例，有如下一个流程：
**gtp_write -> gtp_gdbrsp_QT -> gtp_gdbrsp_qtbuffer**。

通过以上分析，我们已经在Packet层面对GDB和KGTP的通信有了深入的理解。

#### 4. KGTP中set-buffer-size的处理逻辑

有了以上分析，在用户输入了以下指令：

    set remote trace buffer-size on
    set trace-buffer-size xxx

之后会发生什么呢？

让我们分别测试一下。

（1）set trace-buffer-size 100

    Aug 03 11:00:08 localhost.localdomain kernel: gtp_write: QTBuffer:size:64
    Aug 03 11:00:08 localhost.localdomain kernel: gtp_gdbrsp_QT: Buffer:size:64
    Aug 03 11:00:08 localhost.localdomain kernel: gtp_gdbrsp_qtbuffer
    Aug 03 11:00:08 localhost.localdomain kernel: gtp_gdbrsp_qtbuffer:setting buffer size to 100
    Aug 03 11:00:08 localhost.localdomain kernel: gtp_gdbrsp_QT: return 0
    Aug 03 11:00:08 localhost.localdomain kernel: gtp_read

（2）set trace-buffer-size 1000000

    Aug 03 11:00:33 localhost.localdomain kernel: gtp_write: QTBuffer:size:f4240
    Aug 03 11:00:33 localhost.localdomain kernel: gtp_gdbrsp_QT: Buffer:size:f4240
    Aug 03 11:00:33 localhost.localdomain kernel: gtp_gdbrsp_qtbuffer
    Aug 03 11:00:33 localhost.localdomain kernel: gtp_gdbrsp_qtbuffer:setting buffer size to 1000000
    Aug 03 11:00:33 localhost.localdomain kernel: gtp_gdbrsp_QT: return 0
    Aug 03 11:00:33 localhost.localdomain kernel: gtp_read

（3）set trace-buffer-size -1

    Aug 03 11:00:18 localhost.localdomain kernel: gtp_write: QTBuffer:size:-1
    Aug 03 11:00:18 localhost.localdomain kernel: gtp_gdbrsp_QT: Buffer:size:-1
    Aug 03 11:00:18 localhost.localdomain kernel: gtp_gdbrsp_qtbuffer
    Aug 03 11:00:18 localhost.localdomain kernel: gtp_gdbrsp_qtbuffer:setting buffer size to 0
    Aug 03 11:00:18 localhost.localdomain kernel: gtp_gdbrsp_QT: return 0
    Aug 03 11:00:18 localhost.localdomain kernel: gtp_read

（4）set trace-buffer-size unlimited

    Aug 03 11:00:44 localhost.localdomain kernel: gtp_write: QTBuffer:size:-1
    Aug 03 11:00:44 localhost.localdomain kernel: gtp_gdbrsp_QT: Buffer:size:-1
    Aug 03 11:00:44 localhost.localdomain kernel: gtp_gdbrsp_qtbuffer
    Aug 03 11:00:44 localhost.localdomain kernel: gtp_gdbrsp_qtbuffer:setting buffer size to 0
    Aug 03 11:00:44 localhost.localdomain kernel: gtp_gdbrsp_QT: return 0
    Aug 03 11:00:44 localhost.localdomain kernel: gtp_read

可以看到，`This packet directs the target to make the trace buffer be of size size if possible. A value of -1 tells the target to use whatever size it prefers`.

也就是说，-1和unlimited等价，GDB告诉target可以使用自己认为合适的size，而给一个合理范围的正整数n，则会要求target使用的buffer size为n，单位是byte。

因此，当我们接到GDB发来的size为-1的包时，可以直接忽略掉，而收到normal size的包时，需要跟当前使用的buffer size作对比，然后做相应处理。

#### 5. KGTP中set-buffer-size的具体实现

目前KGTP的set-buffer-size分支，已经实现了对size处理的逻辑，参考[6]，具体代码如下：

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
            int unlimited_or_not;

            pkg += 5;

            if (pkg[0] == '\0')
                return -EINVAL;

            // The size may equal to -1, so compare the pkg with "-1"
            if((strncmp("-1", pkg, 2) == 0) && (*(pkg + 2) == '\0')) {
                unlimited_or_not = 1;
            }

            if(unlimited_or_not == 1) {
                // The user wants KGTP to choose the right size
                // So we just ignore the request

    #ifdef GTP_DEBUG
        printk(GTP_DEBUG "gtp_gdbrsp_qtbuffer:keep buffer size as the user tells KGTP to use unlimited size");
    #endif

                return 0;
            }

            // User wants to change the buffer size
            hex2ulongest(pkg, &size);

    #ifdef GTP_DEBUG
        printk(GTP_DEBUG "gtp_gdbrsp_qtbuffer:setting buffer size to %ld\n", size);
    #endif

            // Handle the new ringbuffer size blow
            // gtp_set_trace_buffer_size(size);

            return 0;
        }

        return 1;
    }

让我们测试一下：

（1）set trace-buffer-size -1

    Aug 03 12:37:28 localhost.localdomain kernel: gtp_write: QTBuffer:size:-1
    Aug 03 12:37:28 localhost.localdomain kernel: gtp_gdbrsp_QT: Buffer:size:-1
    Aug 03 12:37:28 localhost.localdomain kernel: gtp_gdbrsp_qtbuffer
    Aug 03 12:37:28 localhost.localdomain kernel: gtp_gdbrsp_qtbuffer:keep buffer size as the user tells KGTP to use unlimited size
    Aug 03 12:37:28 localhost.localdomain kernel: gtp_gdbrsp_QT: return 0
    Aug 03 12:37:28 localhost.localdomain kernel: gtp_read

（2）set trace-buffer-size unlimited

    Aug 03 12:37:57 localhost.localdomain kernel: gtp_write: QTBuffer:size:-1
    Aug 03 12:37:57 localhost.localdomain kernel: gtp_gdbrsp_QT: Buffer:size:-1
    Aug 03 12:37:57 localhost.localdomain kernel: gtp_gdbrsp_qtbuffer
    Aug 03 12:37:57 localhost.localdomain kernel: gtp_gdbrsp_qtbuffer:keep buffer size as the user tells KGTP to use unlimited size
    Aug 03 12:37:57 localhost.localdomain kernel: gtp_gdbrsp_QT: return 0
    Aug 03 12:37:57 localhost.localdomain kernel: gtp_read

Bingo！目前已经正确地实现了对size的解析处理，接下来的工作是继续调研KGTP trace buffer的实现，并根据size大小做出调整。

#### 6. 参考链接

*   [1] http://blog.csdn.net/calmdownba/article/details/38174759
*   [2] https://sourceware.org/gdb/current/onlinedocs/gdb/Packets.html#Packets
*   [3] https://sourceware.org/gdb/current/onlinedocs/gdb/Remote-Protocol.html#Remote-Protocol
*   [4] https://sourceware.org/gdb/current/onlinedocs/gdb/Tracepoint-Packets.html
*   [5] https://sourceware.org/gdb/current/onlinedocs/gdb/Starting-and-Stopping-Trace-Experiments.html
*   [6] https://code.csdn.net/Calmdownba/kgtp/tree/set-buffer-size
