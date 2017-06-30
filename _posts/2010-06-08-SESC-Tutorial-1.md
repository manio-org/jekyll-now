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














