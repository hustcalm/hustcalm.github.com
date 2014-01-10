---
layout: post
title: "由安装Windows 8.1谈起，Linux和Windows多系统的安装以及分区和启动项的管理"
date: 2013-07-01 10:54
comments: true
categories: OS Windows Linux
---
想想自己的Windows XP从09年寒假买了联想的小Y之后就一直在用，期间换过不少次系统，短时间用过Windows 7,也曾经在自己硬盘上安装了Windows + 4个Linux，还包括一个不成功的尝试，在小Y上尝鲜Mac OS X。大四做机器人比赛，由于开发的需要一直不敢折腾自己的电脑（很多开发工具只在Windows XP上表现良好，尤其是一些驱动），读了研究生后，启动越来越慢的XP成了一块心病，无奈懒于折腾，也习惯了XP下的一些工具和操作，当然更不能舍弃Windows这个领好的PC游戏平台！极品和仙剑，到了Linux只能止步了。

不过5分钟的开机启动实在是忍无可忍了，恰巧最近几天，微软放出了Windows 8.1的预览版本（叫做Windows Blue，我猜想，微软内部发布Windows的相关人员可能也喜欢看海贼王，里面的All Blue是一片神奇的海域啊，看看Windows 8.1安装过程中的鱼，更让人浮想联翩了），在把自己的Ubuntu 11.04果断替换为Linux Mint 15 RC 之后，这次再把Windows XP替换为Windows 8.1预览版！

<!--more-->

借着这个机会，再次总结一下关于多系统共存的一些基本问题，更多是给读者一些核心概念，关于某些具体操作，Google会给出更好的答案！Let's Go！

## 那些年我们用过的Windows XP
说真的，XP绝对是一代经典，目前仍然有很大的装机量（我猜测的），即Windows 2000之后，她俨然创造了微软发布的操作系统的一个辉煌，在当时的视觉效果，广泛的软件支持，征服了很多Windows用户。09年寒假，我入手小Y的时候，预装的是Vista，拿到机器的第一件事，就是安装XP！那个时候，番茄花园，深度，雨林木风，ghost之类的在国内大火啊！

现在想想，之所以一直用XP，一是因为兼容性好，绝大多数的驱动支持，丰富的第三方软件，二是因为系统较轻量级，系统资源消耗少，比较适合我的小Y（2009年的Y430,奔腾双核T4200 + G9300的独显），下面简单说说我是怎么用XP的。

### 桌面用fences
最喜欢双击后，整个世界清净了。

### 资源管理器用Total Commander
无需多言，此等神器，应该推广到每一个平台（转到Linux Mint后，发现Linux稍微能用的是muCommander，KDE下的Krusader比较强大，不过在gnome下要安装一堆Qt的库）。Twin-Panel的操作，内置的压缩解压缩支持，FTP支持，文件查看，强大的文件复制，重命名，读者如果在Windows平台下，用上这个软件会爱不释手的说！

### 快捷键方案用AutoHotkey + Win+R + Path
#### AutoHotkey
用AutoHotkey定义一些应用程序的全局启动快捷键，可以用很短的时间进入到自己的“工作空间”，我的一些常用设置，Win+T打开Total Commander，Win+N打开Evernote，Win+V打开Vim，Win+F打开everything，Win+K打开酷我音乐盒，Win+Q打开QQ，读者可以自行发挥想象，用较小的学习成本提高自己的操作效率。

#### Win+R
神器！简约而不简单，常用的操作完全可以在这里搞定了。我常用control进入控制面板，devmgmt.msc进入设备管理器，diskmgmt.msc进入磁盘管理，msconfig进入自启动程序的管理，main.cpl快速调整鼠标的左右手习惯（对于左手鼠标的我这个操作很常用，尤其是偶尔让别人用自己的电脑的时候），%Temp%进入系统临时文件的文件夹，X:\快速进入某个分区（X是分区代号，比如C，D，E等等），command执行自定义的一些可执行程序（一些常用的绿色软件，可以添加到Path中，在Win+R中输入程序名称可以直接调用，比如我把Foxit Reader的可执行文件命名为foxit.exe）。关于Win+R的妙用，大家还可以去看看“善用佳软”上的一些文章，更加专业具体！

#### Path
这个主要是结合Win+R使用，把自己常用的一些绿色软件统一放到一个目录下，比如在非系统盘建一个system32的文件夹，然后添加到Path中，推荐大家把Sysinternals Suit里面的实用工具添加到自己的“工具箱”中，另外的一些贴心小软件也可以放进去，比如procexp，who locks me，depency walker等等。

### 文件搜索用Everything
全硬盘文件索引，支持通配符和正则查找，用用就知道！

### Terminal用Cygwin的MinTTY
Cygwin和MinGW应该算两个东西，前者旨在提供一个Linux的模拟环境，提供大多数下Linux的实用工具（当然也包括编译器的支持了，gcc），而后者更多的是提供一套工具链，而提供的shell是其次（我个人的理解，不喜勿喷）。两者的最大区别是，Cygwin下用gcc编译的程序不是native可以在Windows下运行的，需要一些dll的支持（尤其是cygwin1.dll，这是整个API模拟层的实现吧，当然可以通过改变编译选项脱离对这些dll的依赖，而其本质是把需要用到的接口函数链接到自己的程序内部，从而增大了程序的体积），而MinGW本质上是cross compiling，编译出来的程序可以native地运行！把MinTTY全屏，很难区分出来你是在什么平台下！

### ssh客户端用Putty
当然可以在Cygwin里面直接跑一个ssh（可以结合screen使用），不过Putty更符合Windows用户的flavor，不多说了，开个虚拟机，通过Putty ssh过去，就是一个实实在在的Terminal了，不过相比于Cygwin，资源消耗多些（虚拟机的开销）。

### 虚拟机用VMware
不推荐用VB（VirtualBox），不是黑这个产品，而是相比而言，VMware的体验确实更好（如果不喜欢盗版，那还是用VB吧）。

### 浏览器用Firefox + Vimperator
Life Change的插件Vimperator无疑把Firefox用户牢牢抓住（如果使用Vimperator的话），直接set gui=none，你告诉别人你正在使用Chrome Book也未尝不可，一个简单的b键可以把当前打开的标签一览无余，更多关于Vimperator的东东，可以参考我之前写的[Firefox利器Vimperator简明攻略](http://hustcalm.me/blog/2013/06/09/firefoxli-qi-vimperatorjian-ming-gong-lue/)。

更加Geek一些的可以考虑用Pentadactyl（和Vimperator本是同源，后来核心开发者撤出后重启炉灶，打造了它，速度更快，可定制性更强）。

用Chrome的用户，稍微委屈下，用Vimium吧！

### 邮件客户端用Foxmail或者Thunderbird
感觉Foxmail比较耗内存，占CPU，但是使用起来比Thunderbird舒服（个人观点）。如果使用Thunderbird，Linux，Mac OS和Windows下都有，减少了熟悉成本。

### IRC客户端用Firefox的插件ChatZilla
大赞这个插件，打开后的窗口犹如一个本地客户端，命令式的操作效率很高，支持绝大多数主流的IRC Server。

### 看电影用KMPlayer
绿色版的，Win+R输入kmplayer即可！

### 听音乐用douban.fm
这年头，各种青年都玩豆瓣吧。

### 小结
全屏的Firefox + 全屏的MinTTY + TotalCommander + Everything，XP下的效率很高的说！实在离不开Linux，那就开个虚拟机，只要配置跟的上，全屏后犹如本地系统！电脑配置一般的，关掉虚拟机中Linux的GUI，节省一些资源，如果嫌虚拟机窗口和真机之间切换太过麻烦，那就用Putty ssh过去！

简单点说，Windows XP也可以很好用，经过适当打造，也不误Linux的体验！

## 关于多系统安装的核心词汇
以下的关键词，应该是安装Linux和Windows多系统的时候出现频率比较高的，根据个人的理解，罗列如下：

分区管理，MBR，引导分区，启动项管理，Grub，Grub Legacy，Grub2，Grub4Dos，ntloader，BootManager，BCD，EasyBCD，Windows PE，SliTaz，Puppy Linux，系统维护盘，U 盘系统，光盘启动，ISO，刻录...

## 系统安装实例
多系统安装最麻烦的就是启动项的问题，主要是由于系统安装的先后次序造成的，根据我的经验，一般比较稳妥的安装方法是，低版本的Windows到高版本的Windows，再到Linux，举个例子，你可以先安装Windows XP，然后安装Windows 7,之后安装Ubuntu。另外一个需要注意的问题是分区问题，Windows下分区限制最多4个主分区，而引导分区必须是主分区，其它的可以是逻辑分区（这里不是特别确定了，如果有误，请留言指正）。

下面用两个实例，讲解一下多系统的安装，顺序分别是Windows->Linux，Linux->Windows。我的硬盘的分区情况如下：

一个主分区(Windows所在的分区） + 一个扩展分区(Windows下分了4个逻辑分区）+ 一个Linux根分区 + 一个Swap分区。

更具体点，
C盘 + D盘 + E盘 + F盘 + G盘 + Linux分区 + Swap分区。

### 在存在Windows的情况下安装Linux（光盘，U盘，硬盘）
核心在于引导，这里又分为两种情况：
*   已经存在一个Linux系统，开机即可进入Grub
*   系统只有Windows

有Linux的情况下，直接用Grub引导，如果只有Windows，用Grub4Dos。下载需要安装的Linux的ISO镜像文件，放到Windows一个分区下（推荐放到根目录，方便后续操作），之后在Grub中挂在loop设备，设定Linux kernel和initrd后即可boot到Live CD环境，之后卸载loop设备按照常规流程安装即可！

#### 只有Windows需要安装Ubuntu
在Windows给Ubuntu腾出分区，之后可以选择用光盘安装（最方便的，直接光驱启动即可），U盘安装（和光盘类似，只是需要制作U盘启动盘，有专门的制作工具），硬盘安装（用Grub4Dos，具体方法可以Google下）。

#### 在Windows XP和Ubuntu并存的情况下安装Linux Mint
有了Ubuntu，就相当于有了Grub，接下来只需要正确地挂在Linux Mint的ISO镜像文件，指定kernel和initrd即可，示例如下：
    insmod loopback
    loopback loop (hd0,x)/linuxmint-15-cinnamon-dvd-32bi-rc.iso
    linux (loop)/casper/vmlinuz boot=casper isoscan/filename=/linuxmint-15-cinnamon-dvd-32bit-rc.iso
    initrd (loop)/casper/initrd.lz
    boot
上面的（hd0,x）是下载的ISO文件所在的分区，可以通过磁盘管理查看自行推算出，并且把ISO文件放在了分区的根目录。

### 在存在Linux的情况下安装Windows（光盘，U盘，硬盘）
Grub是Linux的专利，理论上chainloader+1引导Windows完全可行，不过前提是有ntloader（如果有错，欢迎指正）。

#### 只有Ubuntu需要安装Windows
操作类似，在Ubuntu下为Windows腾出分区，之后用光盘安装或者U盘（启动到Windows PE，虚拟光驱挂载ISO，然后setup.exe），理论上将Windows的ISO文件解压后把相应位置后也可以实现硬盘引导，我对此没有太多研究，不予讨论。

#### 在Windows XP和Linux Mint共存的情况下安装Windows 8.1
这里比较棘手的问题，就是安装Windows后，MBR被重写，无法找到Linux的启动项了，这时需要修复Grub！

U盘启动到Windows PE（推荐大家使用天意系统维护盘），虚拟光驱加载Windows 8.1的ISO文件，然后双击setup.exe开始安装，format掉Windows XP所在的分区后一路Next。

Windows 8.1安装成功后，开始修复Grub！由于我用的Linux Mint使用Grub2，因此需要找到一个包含Grub 2的Live CD，我用Ubuntu 12.04进入到Live CD环境下，执行以下操作：
    sudo -i
    fdisk -l
    mkdir /media/rescuegrub
    mount /dev/sdax /media/rescuegrub
    grub-install --root-directory=/media/rescuegrub /dev/sda
    reboot
其中/dev/sdax是Linux Mint所在的根分区，如果你单独分了boot分区，还需要挂载那个分区：
    mount /dev/sday /media/rescuegrub/boot
重启后进入到Linux Mint，执行：
    sudo update-grub2
    sudo reboot
此时再重启，可能会出现error no file found的情况，但是可以正常进入系统，这是由于我使用的Ubuntu 12.04的Grub版本较低，再次进入到Linux Mint，执行：
    sudo grub-install /dev/sda
    sudo reboot
问题解决！

除了修复Grub，也可以使用Windows的引导管理器来引导Linux，选择Grub4Dos或者EasyBCD均可！

## 总结
多系统的安装，重点是理解了多系统并存的基本概念，也就是分区和启动项的管理，可以折腾的东西有很多，大家可以根据个人喜好选择自己的方案！需要提醒大家的是，对分区要谨慎操作，数据无价！

全文完，写这篇文章更多是自己的一个小总结，如果对大家有所帮助就再好不过了，有问题随时可以讨论:-)
