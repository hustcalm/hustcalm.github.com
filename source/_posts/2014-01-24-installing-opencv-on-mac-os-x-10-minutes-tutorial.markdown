---
layout: post
title: "Installing OpenCV on Mac OS X - 10 Minutes Tutorial"
date: 2014-01-24 00:03:00 +0800
comments: true
categories: OpenCV Mac 
---

A simple tutorial to get OpenCV set up on Mac OS X. I get this done on my Macbook Pro with Mavericks OS. Luckily enough, you will success following the instructions blow.

------------------------------
### Prerequisites

#### Source Code

You can download the source code from [OpenCV Official Site](http://opencv.org/), I prefer to pull the source using `git` however.
    
    git clone https://github.com/Itseez/opencv.git

<!--more-->

#### Build Tools

I use [Homebrew](http://brew.sh/) as my package manager, to install `Homebrew`, use:

    ruby -e "$(curl -fsSL https://raw.github.com/Homebrew/homebrew/go/install)"

After brew installed, get `CMake` by:

    brew install cmake

------------------------------
### Build and Install OpenCV

Now the recipes are ready, let's cook:

    cd opencv
    mkdir build
    cd build
    cmake -G "Unix Makefiles" ..
    make -j8
    sudo make install

------------------------------
### Verify Installation

Let's rock by building a simple sample:

    cd opencv/samples/cpp
    g++ edge.cpp -o edge `pkg-config --cflags --libs opencv`
    ./edge

You should see a window pop up and demo the `Canny edge detection`.

------------------------------
Happy playing with OpenCV, drop me a line if you got problems.
