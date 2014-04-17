---
layout: post
title: "Build PTAM on Fedora 20 the easy way - 10 Minutes Tutorial"
date: 2014-04-17 19:21:41 +0800
comments: true
categories: AR CV PTAM Linux Fedora
---
It's been a while since the last post [PTAM Compilation on Linux-HowTo](http://hustcalm.me/blog/2013/09/27/ptam-compilation-on-linux-howto/) about playing with PTAM. And now Isis Innovation has released the source code under GPLv3 and put it on [Github](https://github.com/Oxford-PTAM/PTAM-GPL).

Last time, we demostrate detailed instructions on how to get PTAM running on Debian derived systems(Linux Mint 15 and Ubuntu 12.04), now I want talk about building the source code on Fedora 20. It won't be long, since I just want to talk about the key components, thus the mandatory dependencies(**TooN, libCVD, Gvars3, OpenGL, libjpeg, libpng, libtiff**, etc).

<!--more-->

As the PTAM source code use the GNU autoconf tools, so the typical way to install is `./configure && make && sudo make install`, however, to make sure that certain third libs are correctly configured and can found when running `configure`, you may want to have a look at the output after `configure` and install any key dependency which is missed.

Whenever you are not sure, see the **Official Website** for sure!!!

## Install Dependencies

### TooN

[TooN](http://www.edwardrosten.com/cvd/toon.html) is a numerics library used by libCVD. Since they are just a bundle of header files, installation is trivial.

    sudo yum install liblapack-devel
    sudo yum install libblas-devel

    ./configure && make && sudo make install

### libCVD

[libCVD](http://www.edwardrosten.com/cvd/) is a very portable and high performance C++ library for computer vision, image, and video processing. 

This is the key component for building PTAM, thus should be careful. Make sure you installed OpenGL correctly, also for the video source, if you are using a UVC webcam, check if `v4l2` is working. Or if you are using a IEEE1394 camera, get libxx1394 series libs installed which can drive your camera. For Image I/O, always have libjpeg, libpng and libtiff.

See the output after running `configure` and ensure all the options related are OK.

    sudo yum install freeglut-devel

    ./configure

    make

    sudo make install

### GVars3

[GVars3](http://www.edwardrosten.com/cvd/gvars3.html) is a configuration library which integrates well with TooN. Not much to worry when buiding.

    ./configure && make && sudo make install

## Build PTAM 

### Run ldconfig

The libs built above may reside in `/usr/local/lib` when using the default directorys in their Makefiles, to make the system aware of the newly installed libs.

    sudo vi /etc/ld.so.conf
    add /usr/local/lib to the file
    sudo ldconfig

### Prepare for Makefile

    cd the_PTAM_Directory
    cp Build/Linux/* .
    vi Makefile
    add -lGLU -lGL -llapack to the linker commands

### Fix usleep declaration

    vi Tracker.cc
    add #include <unistd.h> to the first line

### Build

    make

If nothing wrong, you are done!

## Run PTAM

Simply invoke `CameraCalibrator` to calibrate your camera. Then invoke `PTAM` for real fun.

If you use UVC webcam using `v4l2`, you may want to install:

    sudo yum install v4l-utils

Use `v4l2-ctl` to play with your camera.

## Trouble Shooting

As the orinigal [README.txt](http://www.robots.ox.ac.uk/~gk/PTAM/README.txt) says, PTAM works well with Nvidia display card, I got `Segmentation Fault as soon as ... got video source` as I'm using a display card shipped with an Intel motherboard.

However, I think the root cause is that the display card driver does not know how to handle the color space coming from the webcam, so I'm planning to deep into the mechanism and try to find out why. [George Klein](http://ewokrampage.wordpress.com/troubleshooting-faq/) says that a nvidia display card and driver combo works fine after all.
