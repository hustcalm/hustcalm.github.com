---
layout: post
title: "Notes about IEEE 1394 AVT Cameras - FirePackage and OpenCV"
date: 2013-12-10 21:10
comments: true
categories: OpenCV 1394 AVT
---

I'm trying to make AVT 1394 Cameras work with OpenCV using the standard `VideoCapture` recently but got no luck. Briefly speaking, OpenCV's support for camera input are `Operating System` and `Device Driver` aware. 

In this post, I will give some basic related concepts related to Firewire(aka IEEE 1394) and AVT software packages, also OpenCV's `VideoCapture` workflow.

<!--more-->

--------------------
### FireWire
Aka [IEEE 1394](http://en.wikipedia.org/wiki/IEEE_1394), it's a serial bus interface standard for high-speed communications and isochronous real-time data transfer, developed by `Apple` in early 1990s and called `FireWire`. The main competitor nowadays is `USB`.

For the official introduction part, you can refer to the wikipedia link above, I want to address two points here.

#### Standards and versions
The most used must be `1394a` and `1394b`, and to make things work, do pay some attention to your `1394 Host Controller`.

Unlike USB and USB 2.0, the physical interface between 1394a and 1394b is significantly different. As such, users will need to acquire cables with a 1394a (4-pin or 6-pin) connector on one end and a 1394b (9-pin) connector on the other. And a 1394a device would work connecting to a 1394b bus whereas one would fail vice versa.

#### Pins of connector
As for `1394a` and `1394b`, there are three kinds of connectors used: 4-pin, 6-pin and 9-pin. Hence there are kinds of cables connecting different styles of connectors list blow:

![FireWire Plug Connector at the cable](http://connector.pinouts.ru/diagram/firewire_cable.gif).

Luck is you may get whatever cable to fullfil your need as long as you are connecting your device to a right bus.

For more details about the interface pinout, read this article [FireWire (IEEE1394) bus interface pinout](http://pinouts.ru/Slots/ieee1394_pinout.shtml).

--------------------
### AVT Cameras
[AVT](http://www.alliedvisiontec.com) does manufacture so many cameras and provide great technical support. It's kind of a choice for you to choose between the software packages,seriously speaking.

#### AVT Software Selector
Now they are focusing on [VIMBA](http://www.alliedvisiontec.com/apac/products/software/vimba-sdk.html) and maintain it actively. 

However for the legacy ones, like `FirePackage`, `Universal Package`, etc, look into this link [Legacy Software (SDKs, Apps, Adapters, and Interfaces)](http://www.alliedvisiontec.com/de/produkte/legacy.html).

To select the software that suits you, refer to these links:

*   [AVT Software Support](http://www.alliedvisiontec.com/apac/support/downloads/software.html)
*   [AVT Software Downloads for AVT Cameras](http://1stvision.com/avt_downloads.htm)
*   [AVT Software Selector Guide](http://www.alliedvisiontec.com/fileadmin/content/PDF/Software/AVT_software/AVT_software_stuff/AVTSoftwareSelectorGuide_v3.3.0.pdf)

#### Inside FirePackage
Although called legacy SDKs, FirePackage is widely used currently. For the main idea, you can give quick glimps for the documentation after installation and get a basic knowledge of `FireClass`, `FireCtrl`, `FireGrab` and `FireStack`.

For the package architecture,see this:
{% img /images/blog_images/architecture_of_AVT_FirePackage.png %}

#### AVT and OpenCV
Bad luck that we can't use AVT cameras just with `VideoCapture` because that the cameras might use a device driver that OpenCV can't talk to.

However if your camera compliants with [CMU 1394 Digital Camera Driver](http://www.cs.cmu.edu/~iwan/1394/), probably you will be happy again:-)

--------------------
### OpenCV and Cameras
Quote from this post [Getting Started with OpenCV capturing](https://pixhawk.ethz.ch/tutorials/camera/getting_started).
>   Currently two camera interfaces can be used on Windows: Video for Windows (VFW) and Matrox Imaging Library (MIL) and two on Linux: Video for Linux(V4L) and IEEE1394. For the latter there exists two implemented interfaces (CvCaptureCAM_DC1394_CPP and CvCapture_DC1394V2).

Generaly speaking, your camera would probably work if it is `VFW` or `MIL` compliant under Windows or it suits to standard `V4L` or `IEEE1394` driver model.

But if not, you can even sub-class the `VideoCapture` class, and implement your camera driver to make it work seamlessly with OpenCV.
