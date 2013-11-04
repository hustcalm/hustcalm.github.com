---
layout: post
title: "Install CentOS 6.4 minial with Grub2 and a USB Stick - The Simple Way"
date: 2013-11-04 17:08
comments: true
categories: Linux Centos
---

What a shame that I failed over and over to just install CentOS 6.4 minimal to my Desktop PC!!!

I got a broken Ubuntu 12.04 on my hand with a working Grub2, cheers:-) Also, I luckily have a 4 GB USB stick(who esle dosen't,kidding).

## The Goal
Install CentOS 6.4 minial with Grub2 and a single USB stick, making no bootable USB stick, but do some tricks with the ISO file instead.

<!--more-->

## Preparation
In a nutshell, we need a working Grub(or other bootloaders, I use Grub2), a USB Stick with a vfat or ext2 partition and CentOS ISO file.
### Bootloader
As I have a broken Ubuntu 12.04 and a already working Grub2, I would just go on.

However, you have many choices due to your OS. Such as:

*   Grub Legacy
*   Grub4dos
*   EasyBCD

### ISO
Download it from a mirror that's close enough to you. Check out the [Mirror List](http://isoredirect.centos.org/centos/6/isos/x86_64/) and I choose [mirrors.163.com](http://mirrors.163.com/centos/6.4/isos/x86_64/).

Save the `CentOS-6.4-x86_64-minimal.iso` to your hard drive.

### USB Stick
According to the CentOS wiki, the installer will recognize ext2 or vfat. I choose vfat in my case.

My device on my Linuxmint shows /dev/sdb and the partition shows /dev/sdb1. Just ensure that you have a VFAT partition, other things don't matter too much really.

To play with device and partitions, you can use `fdisk` and `mkfs.*` or other tools that you favor.

### Extract Files
I put the ISO file under `~/Downloads`. And I follow the instructions below:
    mkdir /mnt/centos
    sudo mount -o loop -t iso9660 ~/Downloads/CentOS-6.4-x86_64-minimal.iso /mnt/centos
    mkdir /mnt/usbdisk
    mount /dev/sdb1 /mnt/usbdisk
    cp -r /mnt/centos/isolinux /mnt/centos/images /mnt/usbdisk
    sudo umount /mnt/centos
    cp ~/Downloads/CentOS-6.4-x86_64-minimal.iso /mnt/usbdisk
    sudo umount /mnt/usbdisk

After this, we have `CentOS-6.4-x86_64-minimal.iso`,`isolinux` and `images` on our USB Stick.

## Installation
Now use Grub2 to boot into CentOS installation environment and complete the installation. I have a SATA hard disk locally and do remember **CentOS installer will make your local hard disk to sdb or sdc, but not sda. The USB Stick will be sda during the installation.**

To enter Grub2 command line, press `SHIFT` when your PC boots, or you may miss the menu entry due to **HIDDEN_TIMEOUT** configruration. Then press `c` to get a command line.

Then follow the instructions below, you may adjust a little to suit your own PC.
    linux (hd1,msdos1)/isolinux/vmlinuz
    initrd (hd1,msdos1)/isolinux/initrd.img
    boot

When you prompt to select `Installation Method`, use `Hard Drive` and select `sda1`, follow the routine procedures and you are done:-)

**I tried `URL Method` but failed, either `mirrors.163.com` nor `mirrors.sohu.com` would work:-(**

## Summary
All the essentials reside in Grub2 and isolinux or you may say bootloader and ISO stuff. Check them out and deep it for deep is the best way to solve varies of problems related to boot, installation and rescue.

You may install by other methods, CD-ROW, bootable USB Stick or whatever you like.

Enjoy the struggling and enjoy the gain!
