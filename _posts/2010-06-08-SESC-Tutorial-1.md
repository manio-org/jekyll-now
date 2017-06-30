---
layout: post
title:  "SESC Tutorial – 1. Install Whole SESC Step by Step"
date:   2010-06-08 16:16:01 -0600
categories: jekyll update
---


SESC is good. But after so many years, there are still few documents.? Researchers are often dragged by some tiny troubles. The research can really start, after they can run the hello world. This tutorial aims to reduce the time spending on setuping the evnironment. It lets the researchers pass the tedious evironment setuping process and get down to the exciting researches as soon as possible.

This tutorial is written by Jun He. His e-mail is jhe24(AT)iit.edu. His homepage is http://manio.org. There may be some mistakes in this tutorial. Comments are appreciated. Also, I’ll be very glad if someone wants to work on this tutorial with me.

In this tutorial, I also list the errors which might encounter, so the readers can find out what to do when they do something wrong.? The errors make the reader know more about SESC.

## Index

- What's SESC?
- My Working Environment
- How to Install Old GCC3.4 by deb package in Ubuntu 8.04
- How to build SESC utils
- How to Build SESC source code


## What’s SESC ?

Homepage: http://sesc.sourceforge.net/index.html

SESC is a cycle accurate architectural simulator. It models a very wide set of architectures: single processors, CMPs, PIMs, and thread level speculation.

SESC started as the pet project of Jose Renau while doing his PhD at Urbana-Champaign in the IACOMA group. Currently, he is a new faculty at University of California, Santa Cruz.

My Working Environment:

- Ubuntu 8.04 Desktop LTS (Mine is on VMWare 7.01) Accuatlly, you can run sesc in any Linux distributions which are not too old.
- SESC: checked out in 05/25/2010 by cvs.
- Internet Access. If you can access internet in Linux, the process of installing will be much easier, because you can use apt-get install.
- gcc 3.4.x. Usually, this version of gcc is not installed in your OS. The way of installing this version of gcc will be introduced immediately.


Please follow the following howtos one by one to setup SESC.




## My Working Environment:

- Ubuntu 8.04 Desktop LTS (Mine is on VMWare 7.01) Accuatlly, you can run sesc in any Linux distributions which are not too old.
- SESC: checked out in 05/25/2010 by cvs.
- Internet Access. If you can access internet in Linux, the process of installing will be much easier, because you can use apt-get install.
- gcc 3.4.x. Usually, this version of gcc is not installed in your OS. The way of installing this version of gcc will be introduced immediately.
- Please follow the following howtos one by one to setup SESC.



## How to Install Old GCC3.4 by deb package in Ubuntu 8.04

**Why we need a older version of gcc?**

Because the new version will cause errors, weird errors. The reason for this might be that some new features have been added to gcc 4.x, and they are not compatible with older codes. SESC were mainly developed around 2005, using gcc 3.4. If we use gcc 3.4, we’ll get the same SESC as Jose, the father of SESC.

1. Download the following 3 deb packages from http://archive.ubuntu.com/ubuntu/pool/main/g/gcc-3.4/

a)  cpp-3.4_3.4.4-6ubuntu8_i386.deb (1707096 bytes
b)  gcc-3.4_3.4.4-6ubuntu8_i386.deb (484408 bytes)
c)  gcc-3.4-base_3.4.4-6ubuntu8_i386.deb (163028 bytes)

Note: the version number in the online archive may not match the ones above, but it’s ok. Please download only the three packages listed above, no more no less. Otherwise, you will encounter some errors.

2. Follow the instructions in http://ubuntuforums.org/showthread.php?t=79896, then you can have GCC.installed.

3. Now, if you want to install g++, you have to install them by dpkg. Because if you use GUI-double-click to install g++-3.4_3.4.6-1ubuntu2_i386, ubuntu says “libstdc++6-dev_3.4.6-1ubuntu2_i386 is needed”. But if you double click libstdc++6-dev_3.4.6-1ubuntu2_i386 first, ubuntu says “g++-3.4_3.4.6-1ubuntu2_i386 is needed”.

4. Download the tow g++ deb packages mentioned in step 3 from http://archive.ubuntu.com/ubuntu/pool/main/g/gcc-3.4/

5. $sudo dpkg -i g++-3.4_3.4.6-1ubuntu2_i386.deb libstdc++6-dev_3.4.6-1ubuntu2_i386.deb

The following steps are from http://blog.sina.com.cn/s/blog_48a44f390100igad.html

    1. After the steps above, gcc3.4 is installed in your system. So now you have two versions of gcc in you system. The default is gcc 4.4, we should change it, in order to compile source codes of SESC by gcc 3.4
    2. List the all gcc you have in your system by: $ls /usr/bin/gcc* -l
    3. Add options for gcc 3.4 and gcc 4.4. The version number might not match yours. If not, change yours to the correct ones.
    $ sudo update-alternatives –install /usr/bin/gcc gcc /usr/bin/gcc-4.2 40
    $ sudo update-alternatives –install /usr/bin/gcc gcc /usr/bin/gcc-3.4 30
    4. Alternate to gcc 3.4

```
manio@jun-desktop:~/esesc/src/libsuc$ sudo update-alternatives --config gcc
[sudo] password for manio:
There are 2 alternatives which provide `gcc'.
Selection    Alternative
-----------------------------------------------
+        1    /usr/bin/gcc-4.2
*         2    /usr/bin/gcc-3.4
Press enter to keep the default[*], or type selection number:
```

    5. Enter 2, then gcc 3.4 will be chosen.
    6. Now gcc 3.4 is successfully installed in your system. From now on, you should use the gcc 3.4 to compile SESC source code.










