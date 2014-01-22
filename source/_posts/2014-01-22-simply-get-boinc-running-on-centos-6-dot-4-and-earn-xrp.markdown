---
layout: post
title: "Simply Get BOINC Running On CentOS 6.4 and Earn XRP"
date: 2014-01-22 08:56:56 +0800
comments: true
categories: BOINC WCG Distribute
---

Recently I got to know [BOINC](http://boinc.berkeley.edu) from a friend and find it funny to play with. `BOINC` is an open-source software platform for computing using volunteered resources, which means that people can donate their computing resources to varies of research projects through it. `BOINC` is an acronym for `Berkeley Open Infrastructure for Network Computing`. See [their website](http://boinc.berkeley.edu) for more details.

<!--more-->

However, `BOINC` has the client side and the server side, in order to let people know what projects they can contribute to, how about their statistics and rankings, etc, we have [World Community Grid](http://worldcommunitygrid.org), namely the server side. **World Community Grid(WCG)** is an effort to create the world's largest public computing grid to tackle scientific research projects that benefit humanity, Launched on November 16, 2004, it is funded and operated by IBM with client
software currently available for Windows, Linux, Mac OS X and FreeBSD operating systems. Here for the client software, it is the `BOINC` we refered above. For **WCG** itself, go to [their website](http://worldcommunitygrid.org) and also see [this article from IBM](http://www.ibm.com/smarterplanet/us/en/business_analytics/article/wcg.html).

Basically, people just donate their computing resources for free, but we can also donate for good. Here comes the [Computing for Good](https://www.computingforgood.org) brought by [rippleLabs](http://ripple.com). They are giving away XRP in exchange for donating computing power to scientific research via World Community Grid. Anyone with an Internet-connected computer or Android device can participate.

So we are going to earn XRP for donating our computing power, for me, on a `DELL Optiplex 960` running `CentOS 6.4 x86_64`. Here goes the steps.

------------------------------
### Register for World Community Grid
Go to [World Community Grid](http://worldcommunitygrid.org) and click `Join Today!`, or just click [here](http://www.worldcommunitygrid.org/reg/viewRegister.do) for direct portal. Complete the registration form and you are ready to go. For the software, you may download according to your operating system from here, or go to `BOINC`. I will give installation details in the next section.

**After your registration, login and go to settings, then join the team `Ripple Labs`.**

------------------------------
### Get BOINC Running
I will be detailed for my `CentOS 6.4 x86_64`, for others, refer to [BOINC User Mannual](http://boinc.berkeley.edu/wiki/User_manual).

#### Check your repos
    ls -al /etc/yum.repos.d/

#### Add EPEL repo
    wget http://download.fedoraproject.org/pub/epel/x86_64/epel-release-6-8.noarch.rpm
    sudo rpm -ivh epel-release-6-8.noarch.rpm
    sudo rmp --import /etc/pki/rpm-gpg/RMP-GPG-KEY-EPEL-6
    yum clean all
    yum list

#### Install BOINC using yum
Below is the version for Terminal guys, for more details you may want to refer to [Installing BOINC on Fedora](http://boinc.berkeley.edu/wiki/Installing_BOINC_on_Fedora).

The `boinc-manger` is your case if you prefer using GUI for interfacing, however if working under Terminal, `boinc-client` is enough. I will install both here:

    su -c 'yum install boinc-client boinc-manager'

Make it an auto-start service:

    chkconfig boinc-client on

Verify the installation:

    ps aux | grep boinc

See what has been installed:

    which boinc_client | xargs file
    which boinc        | xargs file
    which boinccmd     | xargs file
    which boinc_gui    | xargs file
    which boincmgr     | xargs file

For help:
    
    man boinc
    man boinccmd
    man boincmgr

Set up your accounts:
    
    sudo usrmod -G boinc -a your_username
    sudo chmod g+rw /var/lib/boinc
    sudo chmod g+rw /var/lib/boinc/*.*
    sudo ln -s /var/lib/boinc/gui_rpc_auth.cfg ~/gui_rpc_auth.cfg

Finally, **logout and login again** to renew your groupship and permission to /var/lib/boinc.

#### Attach WCG project to BOINC

    boinccmd --project_attach http://www.worldcommunitygrid.org/ YOUR_ACCOUNT_KEY
    boinccmd update

`YOUR_ACCOUNT_KEY` can be found at your `WCG Profile`.

#### Check the BOINC status

    boinccmd --get_state

------------------------------
### Register Ripple Wallet
Go to [ripple website](https://ripple.com) and register a wallet. Do remember the related keys for security.

------------------------------
### Connect your WCG and Ripple Wallet
Finally, go to [Computing for Good](https://www.computingforgood.org) and get your `WCG` and `Ripple Wallet` connected.

Just click `REGISTER`, the guide will get you done very soon, all you need to do is to provide the specified `wallet address` and `WCG account`.

------------------------------
Happy digging and contributing:-)
