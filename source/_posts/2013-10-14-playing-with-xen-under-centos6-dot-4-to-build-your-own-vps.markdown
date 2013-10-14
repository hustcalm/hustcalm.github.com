---
layout: post
title: "Playing with Xen under Centos6.4 to build your own VPS"
date: 2013-10-14 10:28
comments: true
categories: Xen Virtualization Centos Linux VPS
---

Believe it or not, we are under an age called Cloud Computing.

I used to be curious about VPS(Virtual Private Server), how they work, how to build them, etc. As far as I know, AWS, GCE or whatever else solutions depend heavily on Virtualization Technologies and AWS is definitly playing with Xen currently. 

I would like very much to have a trial for building my own VPS and blow are my steps doing it. You may need to read some basic wikis about Xen, [Xen Overview](http://wiki.xen.org/wiki/Xen_Overview) and [Xen Beginners Guide](http://wiki.xen.org/wiki/Xen_Beginners_Guide) should do some help:-)

<!--more-->

## Getting a working Dom0 on Centos6.4
### Install Centos 
Use whatever way you like, remember a **minimal install** is recommended.

### Install Xen
Thanks to **Xen4Centos** Project, we can get Xen installed in several simple commands(log in as root).

    yum -y update
    yum install centos-release-xen
    yum install xen
    /usr/bin/grub-bootxen.sh
    reboot

After your machine comes to alive, simply type:
    uname -r
    xl list
to verify that Xen Dom0 is running.

### Config network
I use DHCP to get a valid IP for my box and there is a trick if you can't ping each other when two machinea are in the same LAN(probably the routing table is not working for you and just delete the record from your routing table). 

As me, for example, my laptop got a IP, say 192.168.0.10 and my VPS got 192.168.0.11, but they can't talk to each:-(

First, check your routing table,
    route
Then, delete the record that is evil,
    route del -net 192.168.0.0 netmask 255.255.255.0
**Please substitute the parameters to suite your own networking**.

After that, ping each other,
    ping 192.168.0.11
to verify the networking is good to go.

In order to give our DomUs valid IPs, we may use a network bridge, do as follows:
    yum -y install bridge-utils
    cp /etc/sysconfig/network-scripts/ifcfg-eth0 /etc/sysconfig/network-scripts/ifcfg-xenbr0

My modified and working scripts look like this, for ifcfg-xenbr0:
    DEVICE=xenbr0
    #HWADDR=00:24:E8:46:CC:C9
    TYPE=bridge
    #UUID=ff0ab497-0303-4dc3-b6f3-4a6cbd90466b
    ONBOOT=yes
    NM_CONTROLLED=no
    BOOTPROTO=dhcp

And for ifcfg-eth0:
    DEVICE=eth0
    HWADDR=00:24:E8:46:CC:C9
    TYPE=Ethernet
    UUID=ff0ab497-0303-4dc3-b6f3-4a6cbd90466b
    ONBOOT=yes
    NM_CONTROLLED=no
    #BOOTPROTO=dhcp
    BRIDGE=xenbr0

Then, make the configuration work:
    service network restart
If you got errors such as `error:check cable`, do this manualy:
    ifup xenbr0
Verify it works by issuing:
    ifconfig xenbr0

### Diable SELinux
Honestly speaking, I don't know WTF the real reason to do this, but people say `SELinux can really interfere with Xen`.
    sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
    setenforce 0

You are good to go with a damn working Dom0 and congratulations!

## Bringing Your DomUs Up
It's up to you to use PV or HVM, personally, I prefer to PV due to higher performance. But if you want a Windows(and other OS whose kernel can't be modified), HVM is your only choice.

As for me, I want 3 DomUs, namely Debian, Fedora and Arch Linux.

### Xen4Cli or Xen4Libvirt
Xen4Cli(xm and xl, currently xm deprecated) are intended for advanced users looking to hand setup the network, the backing filestore and the xen environment. While Xen4Libvirt are for newbies looking for the easiest install path.

I prefer `xl` since I want to know what exactly is happening, and `xl` works for Debian and Arch Linux. However for Fedora, `Libvert` seems to be the right way(using `virt-install`).

To install Libvirt,
    yum install libvirt python-virtinst libvirt-daemon-xen
After the install, restart your Dom0 machine.

### File or LVM based backend
Since Centos6.4 is shipped perfectly with LVM and also the default partition schema using LVM during OS Installation, I prefer `LVM`.

As for me, I do the following:
    lvresize -L -200G vg_centos6/lv_home -r
    lvcreate -L 30G -n fedora vg_centos6
    lvcreate -L 30G -n debian vg_centos6
    lvcreate -L 30G -n archlinux vg_centos6

Then verify the Logical Volumes using:
    lvdisplay
Or
    lvs


If you are not familiar with LVM, search Google for a good tutorial first.

### Bridge, Routing or NAT
Three networking modes are provided and choose one according to your own networking environment.

I got bridge working for me as described in above section.

### Let DomUs fly
#### Debian
I choose debian7.1 netinst.iso due to my poor bandwidth,
    wget -c http://mirrors.sohu.com/debian-cd/7.1.0/amd64/iso-cd/debian-7.1.0-amd64-netinst.iso
    mount -o loop /path/to/debian-7.1.0-amd64-netinst.iso /mnt/debian
    cp /mnt/debian/install.amd/xen/debian.cfg /etc/xen/
    cd /etc/xen/
    vi debian.cfg  ## Make modifications according to the comments
    xl create -c debian.cfg
Then install Debian as a very normal one.

#### Arch Linux
I download archlinux-2013.10.01-dual.iso by issuing:
    wget http://mirrors.sohu.com/archlinux/iso/latest/archlinux-2013.10.01-dual.iso
Then create a arch.cfg file in `/etc/xen/` similar like:
    # Refer to https://wiki.archlinux.org/index.php/Xen#Configuring_a_paravirtualized_.28PV.29_Arch_domU

    name = "arch"
    kernel = "/mnt/arch/arch/boot/x86_64/vmlinuz"
    ramdisk = "/mnt/arch/arch/boot/x86_64/archiso.img"
    extra = "archisobasedir=arch archisolabel=ARCH_201310"
    memory = 512
    disk = [ "phy:/dev/vg_centos6/archlinux,sda1,w", "file:/root/isos/arch/archlinux-2013.10.01-dual.iso,sdb,r" ]
    vif = [ '' ]
After that, bring the DomU up by:
    mount /path/to/archlinux-2013.10.01-dual.iso /mnt/arch
    cd /etc/xen/
    xl create -c arch.cfg
Install your Arch Linux as you like since it's really a highly customized distro.

#### Fedora
Again I got Fedora-20-Alpha-x86_64-netinst.iso by issuing:
    wget http://download.fedoraproject.org/pub/fedora/linux/releases/test/20-Alpha/Fedora/x86_64/iso/Fedora-20-Alpha-x86_64-netinst.iso
However, nightmares begin since I want to install it using the Debian way. `xl` just fails to boot normally but `virt-install` does the job.

Using the `virt-install` method:
    virt-install --virt-type xen -n fedora19 -r 512 --vcpus=2 -f /dev/vg_centos6/fedora19 --location http://tel.mirrors.163.com/fedora/releases/19/Fedora/x86_64/os/ --os-type linux --accelerate --nographics
Or you can download the ISO file first, then issuing your own httpd locally, thus making it:
    mount -o loop /path/to/fedora.iso /mnt/fedora
    yum -y install httpd
    service httpd start
    ln -s /mnt/fedora /var/www/html
    service iptables stop
    virt-install --virt-type xen -n fedora19 -r 512 --vcpus=2 -f /dev/vg_centos6/fedora19 --location http://your.http.server.ip.address.here/ --os-type linux --accelerate --nographics

Remember this is for full Fedora installation, if you are using a netinst, you need to copy all the files to /var/www/html instead of creating a symbolic link and modify the .treeinfo file. No warrants here, since I did't try it myself.

## References
*   [Xen4 CentOS6 QuickStart](http://wiki.centos.org/HowTos/Xen/Xen4QuickStart)
*   [Xen4 Libvirt for CentOS 6](http://wiki.centos.org/HowTos/Xen/Xen4QuickStart/Xen4Libvirt)
*   [Install Xen 4 with Libvirt / XL on CentOS 6 (2013)](http://drewsymo.com/cloud-computing/install-xen-on-centos-and-create-a-fedora-debian-vm/)
*   [Virtualization With Xen On CentOS 6.2 (x86_64) (Paravirtualization & Hardware Virtualization)](http://www.howtoforge.com/virtualization-with-xen-on-centos-6.2-x86_64-paravirtualization-and-hardware-virtualization)
*   [Xen Configuration File Options](http://wiki.xen.org/wiki/Xen_Configuration_File_Options)
*   [XL Domain Configuration File Syntax](http://xenbits.xen.org/docs/unstable/man/xl.cfg.5.html)
*   [Network Configuration Examples (Xen 4.1+)](http://wiki.xen.org/wiki/Network_Configuration_Examples_(Xen_4.1%2B))
*   [Debian Guest Installation Using Debian Installer](http://wiki.xen.org/wiki/Debian_Guest_Installation_Using_Debian_Installer)
*   [Xen For Arch Linux Wiki page](https://wiki.archlinux.org/index.php/Xen)
*   [Arch Linux Installation Guide](https://wiki.archlinux.org/index.php/Installation_Guide)
*   [Install Fedora 10 PV DomU at Xen 3.3.1 CentOS 5.2 Dom0 & Xen 3.3.0 Intrepid Server Dom0 (Novell’s Xen-ified Kernel) via local Apache Mirror](http://bderzhavets.wordpress.com/2009/01/10/install-fedora-10-pv-domu-at-xen-331-centos-52-dom0-xen-330-intrepid-server-dom0-novells-xen-inified-kernel-via-local-apache-mirror/)
*   [Installing a domU without virt-install Fedora-xen](https://lists.fedoraproject.org/pipermail/xen/2012-November/005938.html)
*   [CentOS 6安装配置Xen](http://www.centos.bz/2012/03/centos-6-install-deploy-xen/)
*   [Archlinux 简明安装指南](http://www.cnblogs.com/hseagle/p/3299713.html)
*   [archlinux （2012.12.01-dual） i686 硬盘安装](http://blog.csdn.net/holdsky/article/details/8497764)
*   [Xen](http://en.wikipedia.org/wiki/Xen)

------
I do believe you will get lots of annoying problems while playing with Xen. Enjoy the problems and enjoy Xen:-)
