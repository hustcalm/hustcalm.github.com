---
layout: post
title: "Playing With Kinect For Windows Using OpenNI and SensorKinect Under Ubuntu 12.04 LTS"
date: 2014-12-31 16:19:56 +0800
comments: true
categories: Linux OpenSource
---
Just a quick recap to make myself feel better after such a struggle for 1 day and hopes it does some help to those who tries to bring **Kinect For Windows** work under Linux, here Ubuntu especially.

### Hardware Device
Kinect for Windows, not Kinect for XBox, so this post is kind of limited to some senarios. And I'm currently using Kinect For Windows Version 1, not the fasion V2.

<!--more-->

### Some Concepts
*   gspca_kinect    

A simple Kinect kernel driver which has been integrated in the driver, and works by default on my Ubuntu. Think **V4L** or **UCV**, this driver allows you to access the RGB and depth frames from Kinect. See [Getting up and running with the Kinect in Ubuntu 12.04](http://blog.jozilla.net/2012/03/29/getting-up-and-running-with-the-kinect-in-ubuntu-12-04/). Note however the author is using a Kinct for XBox.

*   OpenNI  

While, open source Kinect SDK supported by PrimeSense?

### Verify & Install
Follow [this](http://choorucode.com/2013/07/23/how-to-get-started-with-kinect-for-windows-on-ubuntu-using-openni/), if got compilition error, maybe you are lucky with [this patch](https://github.com/avin2/SensorKinect/pull/5).

### Try Something Cool
Why not play with [RGBDSLAM](https://github.com/felixendres/rgbdslam_v2)?

### Links That Matter
*   [How to get started with Kinect for Windows on Ubuntu using OpenNI](http://choorucode.com/2013/07/23/how-to-get-started-with-kinect-for-windows-on-ubuntu-using-openni/)
*   [Getting up and running with the Kinect in Ubuntu 12.04](http://blog.jozilla.net/2012/03/29/getting-up-and-running-with-the-kinect-in-ubuntu-12-04/)
*   [The OpenNI Grabber Framework in PCL](http://pointclouds.org/documentation/tutorials/openni_grabber.php)
*   [Ubuntu + Kinect + OpenNI + PrimeSense](http://mitchtech.net/ubuntu-kinect-openni-primesense/)
