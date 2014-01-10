---
layout: post
title: "PTAM Compilation On Linux-HowTo"
date: 2013-09-27 19:50
comments: true
categories: CV PTAM AR
---
The README.txt file shipped with the newest PTAM source code is really old (the cvs repositorys are gone for god's sake) ! I struggled to get it done on my Linux Mint 15 with totally failure:-(

However I made it with just pulling the bleeding edge libs(I mean TooN,libCVD and Gvars3) from github, add a header file `<unistd.h>` to `Tracker.cc`, edit the `Makefile` about the linker flags(I will tell the details below), and I absolutely think that 'share my experience to you' is my duty.

I succeeded on **Linux Mint 15** and **Ubuntu 12.04**, other distros won't be hard either:-)

<!--more-->

Now, let's start the journey!

## Installation the required dependencies
**Note the versions of the installed packages may differ from distros and versions of the same distro, just choose the ones suitable for you!**

### Build Tools
    sudo apt-get update
    sudo apt-get install build-essential cmake pkg-config

### Boost for C++
    sudo apt-get install libboost-dev libboost-doc

### Low level libraries for Linear Algebra
    sudo apt-get install liblapack-dev libblas-dev

### Image I/O && Camera Driver
    sudo apt-get install libjpeg-dev libpng-dev libtiff-dev libdc1394-22-dev libv4l-dev 

### Video I/O && Codec && Display
    sudo apt-get install libavcodec-dev libavformat-dev libavutil-dev libpostproc-dev libswscale-dev libavdevice-dev libsdl-dev
    sudo apt-get install libgtk2.0-dev libgstreamer0.10-dev libgstreamer-plugins-base0.10-dev 

### OpenGL
    sudo apt-get install mesa-common-dev libgl1-mesa-dev libglu1-mesa-dev freeglut3-dev

## Installation of OpenCV

    cd thePathYouWant
    wget http://downloads.sourceforge.net/project/opencvlibrary/opencv-unix/2.4.6.1/opencv-2.4.6.1.tar.gz
    tar zxvf opencv-2.4.6.1.tar.gz
    cd opencv-2.4.6.1/
    mkdir release 
    cd release
    cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local ..
    make
    sudo make install

You may want to alert the system about the new installed OpenCV libs,
    vi /etc/ld.so.conf
    add `include /usr/local/lib`
    sudo ldconfig

Do this if you mannualy install other libraries when running into problems(as libcvd and Gvars3 discussed below).

After this, **OpenCV dependencies can now be linked with simply: `pkg-config opencv --cflags --libs`**

Really cool, huh?

## Installation of TooN && libCVD && Gvars3

You may want to get the source code by just pulling from github like this:
    #!/bin/bash
    echo " Now pulling TooN..."
    git clone git://github.com/edrosten/TooN.git
    echo " TooN done for good!"
        
    echo " Now pulling libcvd..."
    git clone git://github.com/edrosten/libcvd.git
    echo " libcvd done for good!"
            
    echo " Now pulling gvars3..."
    git clone git://github.com/edrosten/gvars.git
    echo " gvars3 done for good!"
                
    echo " All done."

If this confuses you, just download the package like this:
    cd thePathYouWant
    wget http://www.edwardrosten.com/cvd/TooN-2.1.tar.gz
    wget http://www.edwardrosten.com/cvd/libcvd-20121025.tar.gz
    wget http://www.edwardrosten.com/cvd/gvars-3.0.tar.gz

**Note the versions may differ as time goes by, use git if you can** Or just look at this [link](http://www.edwardrosten.com/cvd/toon.html) for more details.

Also remember **do refer to the README files when you try to install the libraries**, cause they really do help in some manner:-)

### TooN
Just header files, easy to install huh?
    ./configure && make && sudo make install

### libCVD
Remember that libCVD really needs TooN, or you will get stuck when trying to compile PTAM:-(

The steps I used is like this:
    ./configure --without-ffmpeg --without-v4l1buffer --without-dc1394v1 --without-dc1394v2
    make
    sudo make install

If you run into problems, check the output of `./configure` carefully, they will give you some hints:-)

### Gvars3
This is also easy, like this:
    ./configure --disable-widgets
    make
    make install

### Make the libs work
    sudo ldconfig

## Compilation of PTAM
Now comes the real part of our journey, refer to the README.txt file for details.

### Copy the appropriate platform build files
    cd PTAM
    cp Build/Linux/* .

### Edit the Makefile to reference custom include or linker paths
    LINKFLAGS = -L /usr/local/lib -lGvars3 -lcvd -lGLU -lGL -llapack

`/usr/local/lib` is my custom linker path. 
    
### Make the Video Source right
*   VideoSource_Linux_DV.cc
*   VideoSource_Linux_V4L.cc
*   VideoSource_Linux_Gstreamer_File.cc
*   VideoSource_Linux_OpenCV.cc

The first two are for camera capturing and the last twos are shipped with [BeLioN-github/PTAM](https://github.com/BeLioN-github/PTAM) and can be used to serve images from video files.

### Make
Probably an error will popup, like "error, `usleep` was not declared in this scope", easily add 
    #include <unistd.h>
to `Tracker.cc` and you are done:-)

Just
    make
and enjoy!

## Acknowledgement
Definitely special thanks will go to [Georg Klein](http://www.robots.ox.ac.uk/~gk/), the author and PTAM! Good job, Georg!

I also refered to several posts while struggling,

*   [ubuntu11.10下安装PTAMM](http://hhfighting.blog.163.com/blog/static/55700323201242524213235/)
*   [TooN: Tom's Object-oriented numerics library](http://www.edwardrosten.com/cvd/toon.html)
*   [qt-opencv-multithreaded](http://code.google.com/p/qt-opencv-multithreaded/wiki/Documentation)
*   [OpenCV Tutorials-Installation in Linux](http://docs.opencv.org/trunk/doc/tutorials/introduction/linux_install/linux_install.html)

Hope this post will do some help and if you find any mistake in my blog, please do **leave your comment or drop me a line:-)**
