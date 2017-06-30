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






## How to build SESC utils

### What’s sescutils?

Sescutils is a group of tools for building programs to run on SESC. These tools include gcc, gdb, glibc, etc.

### Why do we need utils in SESC?

Actually, sescutils is a cross-compile toolchain, which is used to compile programs on PC for running on MIPS. SESC is a MIPS architecture simulator.

1. Download the sescutils from https://sourceforge.net/project/showfiles.php?group_id=49065
2. move the package to your $HOME. My $HOME=/home/manio. In the following parts of this tutorial, I will use /home/manio directly. You should
3. $cd $HOME
4. $tar jxvf sescutils.tar.bz2
5. make /bin/sh link to /bin/bash by:

```
sudo rm /bin/shsudo ln -s /bin/bash /bin/sh
```

Ubuntu uses dash, not bash. But the shell scripts in SESC are written by bash. So we have to change to bash.

6. modify file build-common in /home/manio/sescutils/build-mipseb-linux

```
 GNUSRC=$HOME/sescutils/src
 PREFIX=$HOME/sescutils/install
```

Note: GNUSRC is the position of the source codes. PREFIX is the position where the sescutils binary files will be installed to.

7. $cd $HOME/sescutils/build-mipseb-linux
8. $./build-1-binutils.
error:

```
gcc -W -Wall -Wstrict-prototypes -Wmissing-prototypes -g -O2 -o ar arparse.o arlex.o ar.o not-ranlib.o arsup.o rename.o binemul.o emul_vanilla.o bucomm.o version.o filemode.o ../bfd/.libs/libbfd.a ../libiberty/libiberty.a ./../intl/libintl.aarlex.o: In function `main':
/home/manio/sescutils/build-mipseb-linux/obj/binutils-build/binutils/arlex.c:1: multiple definition of `main'
arparse.o:/home/manio/sescutils/build-mipseb-linux/obj/binutils-build/binutils/arparse.c:1: first defined here
ar.o: In function `main':
/home/manio/sescutils/src/binutils/binutils/ar.c:342: multiple definition of `main'
arparse.o:/home/manio/sescutils/build-mipseb-linux/obj/binutils-build/binutils/arparse.c:1: first defined here
bucomm.o: In function `make_tempname':
/home/manio/sescutils/src/binutils/binutils/bucomm.c:425: warning: the use of `mktemp' is dangerous, better use `mkstemp' or `mkdtemp'
ar.o: In function `mri_emul':
```

Note: These errors are caused by the missing of bison and flex, which are used to parse the codes. So, we install them in the following steps. Before installing, make sure you can access the internet.

1. Install bison

```
$sudo apt-get install bison
```
2. $./build-1-binutils. Same error with 3
3. $sudo apt-get install flex
4. ./build-1-binutils
Success
5. $./build-2-gcc-core

Note: if you use gcc 4.x to compile sescutils, you will meet this error. To solve this problem, change to gcc3.4 by following the How to Install old gcc3.4 by deb in Ubuntu 8.04.
If you still can not get rid of the error, refer to the comments of this page. Your bison or flex should be the correct version.

```
gcc -c -g -O2 -DIN_GCC -DCROSS_COMPILE -W -Wall -Wwrite-strings -Wstrict-prototypes -Wmissing-prototypes -pedantic -Wno-long-long -Wno-error -DHAVE_CONFIG_H -I. -I. -I/home/manio/sescutils/src/gcc-3.4/gcc/gcc -I/home/manio/sescutils/src/gcc-3.4/gcc/gcc/. -I/home/manio/sescutils/src/gcc-3.4/gcc/gcc/../include c-parse.c -o c-parse.ogcc: c-parse.c: No such file or directory
gcc: no input files
make[1]: *** [c-parse.o] Error 1
make[1]: Leaving directory `/home/manio/sescutils/build-mipseb-linux/obj/gcc-core-build/gcc'
make: *** [all-gcc] Error 2
```

6. $./build-3-glibc

```
manio@jun-desktop:~/sescutils/build-mipseb-linux$ ./build-3-glibcchecking build system type... Invalid configuration `unknown-linux': machine `unknown' not recognized
configure: error: /bin/sh /home/manio/sescutils/src/glibc-2.3.2/scripts/config.sub unknown-linux failed
```

```
make: *** No rule to make target `all'. Stop.
```

Note: This error is caused by the wrong value of BUILD in build-common file. The uname –p command cannot output the right version of the building system, i.e. your PC.
To correct this problem, simply change the value of BUILD to “i686-pc-linux-gnu”. By the way, there is another value, host, which is the platform in which you want to run your compiled glibc. The target system is the platform in which you will run your program with glibc.

The typical toolchain is: build=your pc, host=your pc, target=arm/mips…

1. $./build-3-glibc
Success.
2. $./build-4-gcc
Success
3. $./build-5-gdb

ERROR

```
checking for wctype... yeschecking for library containing gethostbyname... none required
checking for library containing socketpair... none required
checking for library containing waddstr... no
checking for library containing tgetent... no
configure: error: no termcap library found
make: *** [configure-gdb] Error 1
```

Note: A lib is missing. Install it by the following steps.

1. $ sudo apt-get install libncurses5-dev
2. $./build-5-gdb
Success.
3. Now, the sescutils is completely built. You can use it to build programs for SESC later.







## How to Build SESC source code

1. Install CVS
$sudo apt-get install cvs
2. Download SESC source code by CVS
$cvs -d:pserver:anonymous@sesc.cvs.sourceforge.net:/cvsroot/sesc login
Note: just press Enter when password is requested.
$cvs -z3 -d:pserver:anonymous@sesc.cvs.sourceforge.net:/cvsroot/sesc co -P sesc
3. Move the source code to $HOME/esesc
4. Read the $HOME/esesc/REAMME. It will give you some steps to install SESC.
5. $cd ~
6. $mkdir sesc-build
7. $cd sesc-build
8. $../esesc/configure
Note: if you download SESC in Windows, you will meet the following error.
ERROR: elif unexpected…
REASON: cannot use DOS text format in Linux
SOLVE: open configure by vim, :set ff=unix
9. $sudo apt-get install binutils
Note: this binutils is for building SESC source code. The binutils in sescutils is for building programs running on SESC.
10 $make
ERROR: USHRT_CHAR undefined
SOLUTION: include limits.h file in esesc/src/libcore/FetchEngine.cpp file. Add
#include <limits.h>
11. $make
ERROR:
/home/manio/SESC/build/../esesc/src/libmint/subst.cpp:52:26: error: linux/dirent.h: No such file or directory
SOLUTION:
– Do not include linux/dirent.h, use dirent.h instead
12. $make
ERROR‘uint32_t’ was not declared in this scope
SOLUTION: add #include <stdint.h> wherever you see this error.
Reference:
http://www1.cs.columbia.edu/~youngjin/discus/messages/54/59.html?1259617974
Hi Alejandro,It maybe caused by the strictness of the C compiler provided in Ubuntu 9.10.
(I had almost the same thing with 9.10.)As you mentioned, please try to include the
following statement anytime you have the above-mentioned error:
#include <stdint.h>
13. $make
ERROR: “/usr/bin/ld: cannot find -lz”
SOLUTION: sudo apt-get install zlib1g-dev
14. SUCCESS
15. Test installation:
$make testsim

Output

```
manio@ubuntu:~/SESC/build$ make testsimmake[1]: `sesc.mem’ is up to date.Generating sesc.conf from: /home/manio/SESC/build/../esesc/confs/mem.conf
cp /home/manio/SESC/build/../esesc/confs/mem.conf sesc.conf
cp /home/manio/SESC/build/../esesc/confs/shared.conf .
./sesc.mem -h0x800000 -csesc.conf /home/manio/SESC/build/../esesc/tests/crafty < /home/manio/SESC/build/../esesc/tests/tt.in
static[0x1008db40-0x101b3dd4] heap[0x101b4000-0x109b4000] stack[0x109b4000-0x111ac000] -> [0x41000000-0x4211e4c0]
Crafty v14.3
sesc_simulation_mark 0 (simulated) @30176641
White(1): sesc_simulation_mark 1 (simulated) @30225840
White(1): pondering disabled.
sesc_simulation_mark 2 (simulated) @30299339
White(1): noise level set to 0.
sesc_simulation_mark 3 (simulated) @30321649
White(1): search time set to 99999.00.
sesc_simulation_mark 4 (simulated) @30423769
White(1): verbosity set to 5.
sesc_simulation_mark 5 (simulated) @30449198
White(1): sesc_simulation_mark 6 (simulated) @30751014
White(1): search depth set to 2.
sesc_simulation_mark 7 (simulated) @30767155
White(1):
clearing hash tables
depth time score variation (1)
1 ###.## -0.67 axb5 c6xb5
1 ###.## -0.08 a4a5
sesc_simulation_mark 8 (simulated) @33091197
1-> ###.## -0.08 a4a5
2 ###.## – a4a5
2 ###.## -0.65 a4a5 f6f5
2 ###.## -0.58 axb5 c6xb5 Ne4c5
2 ###.## -0.46 Rf1c1 f6f5
2 ###.## -0.43 Ra1c1 f6f5
sesc_simulation_mark 9 (simulated) @37838384
2-> ###.## -0.43 Ra1c1 f6f5
time:### cpu:### mat:-1 n:833 nps: ####
ext-> checks:18 recaps:4 pawns:0 1rep:6
predicted:0 nodes:833 evals:372
endgame tablebase-> probes done: 0 successful: 0
hashing-> trans/ref:17% pawn:83% used:w0% b0%
White(1): Ra1c1
time used: ###.##
sesc_simulation_mark 10 (simulated) @38879583
Black(1): execution complete.
```

Here is another howto for installing SESC, by Girish Venkatasubramanian.






