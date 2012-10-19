---
layout: post
title: "Cross Complile VLC for Windows On Ubuntu How To and Trouble Shooting "
date: 2012-10-19 11:53
comments: true
categories: VLC 
---
## Obtain the toolchain ##
I just compile VLC2.0.3 for windows with Ubuntu 11.04 and want to share the compiling process and some trouble shooting.I mainly rely on official guide on [Win32Compile-VideoLANWiki](http://www.wiki.videolan.org/Win32Compile).		To get the toolchain, I just run `apt-get install gcc-mingw32 mingw32-binutils`.And also the development tools,run `apt-get install lua automake autoconf autopoint libtool make gettext pkg-config git subversion cmake cvs zip p7zip nsis bzip2`.		
After that I got my Host triplet,`i586-mingw32msvc` for Windows 32-bits, using the Mingw32 toolchain.
## Get the source code ##
If you want to use an official release, as I did, just run:
	wget ftp://ftp.videolan.org/pub/videolan/vlc/2.0.3/vlc-2.0.3.tar.xz
	tar zxJf vlc-2.0.3.tar.xz
	cd vlc-2.0.3
If you are using the Git development version, you must do a bootstrap to generate the files for configuration and make:
	git clone git://git.videolan.org/vlc.git
	cd vlc
	./bootstrap
## Prepare 3rd party libraries ##
I got the prebuilt libraries by running:
	mkdir -p contrib/win32
	cd contrib/win32
	../bootstrap --host=i586-ming232msvc
	make prebuilt
the codes above will download the latest prebuilt libs on your disk and then install them automatically.	
If you got a lot time to burn and enjoy compiling the contribs yourself,you may do this:
	apt-get install subversion yasm cvs cmake
	makdir -p contrib/win32
	cd contrib/win32
	../bootstrap --host=i586-mingw32msvc
	make fetch
	make
## Go Back ##
	cd -
## Bootstrap ##
	./bootstrap
## Configure ##
	mkdir win32 && cd win32
	../extras/package/win32/configure.sh --host=i586-mingw32msvc
you can get a lot of information with `../configure -- help | less`.
## Building VLC ##
	make
## Packaging VLC ##
I just run
	make package-win32
and got an auto-installer generated with NSIS and also a zip package and 7zip package.		
If everything goes well,congratulations,you have got your own compiled VLC for windows,but always there will be some minor problems.The list below is what I got:	
	
*	liblibbluray_plugin.la No such file or directory

*	dialos/preferences.hpp expected unqualified-id before 'char'

*	error:implicit declaration of av_free,av_malloc,av_realloc

*	can't find some language files *.nsh while making nsis installer


## Trouble Shooting ##
### liblibbluray_plugin.la ###
To be honest, I just avoid this by losing my VLC playing bluray discs with --disable-bluray passing to configure.
### qt compiling about expected unqualified-id before 'char' ###
	In file included from dialogs_provider.cpp:42:0:
	dialogs/preferences.hpp: At global scope:
	dialogs/preferences.hpp:72:19: error: expected unqualified-id before 'char'
	dialogs/preferences.hpp:72:18: error: expected ';' at end of member declaration
	dialogs/preferences.hpp:72:24: error: expected unqualified-id before ',' token
	I solved this problem by referencing this [link](http://forum.videolan.org/view>topic.php?f=14&t=102257).

### error: implicit declaration of av_free,av_malloc,av_realloc ###
I simply browse avcodec.c and add declarations to avcodec.h by referencing this [link](http://ffmpeg.org/doxygen/trunk/mem_8c.html).

### no such language file *.nsh ###
Just read extras/package/win32/vlc.win32.nsi and delete the lines about languages,then you can successfully make the nsis work but if you want your VLC playing with locales perfectly, you'd better add the languages mannually.	
**The End.Feel free to contact me if you got any problem about compiling:)**

