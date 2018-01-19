---
layout: post
title:  "虚拟机是怎么实现的?"
date:   2012-02-02
categories: OS
---

<span style="color: #222222;">1997年，斯坦福的Mendel Rosenblum带着Edouard Bugnion, Scott Devine在SOSP上发了篇论文，叫做Disco: running commodity operating systems on scalable multiprocessors （<a href="http://research.cs.wisc.edu/areas/os/Qual/papers/disco.pdf">http://research.cs.wisc.edu/areas/os/Qual/papers/disco.pdf</a></span><span style="color: #222222;">）。发了之后，我想他们应该是觉得这个主意太好了，就开了家公司，名叫VMWare。</span><br style="color: #222222;" /><br style="color: #222222;" />

<span style="color: #222222;">这篇论文起名叫Disco（迪士高）是因为虚拟机本身不是一个新的东西，大概在上世纪70年代就有了。作者们为了表示敬意，或者是显示这是一个复古的东西，就把这个项目取名为disco。这篇论文介绍了虚拟机关键技术，用来回答这个问题再合适不过了。（多年之后，OSDI上的另一篇论文（Memory Resource Management in VMware ESX Server）介绍了一些VMWare的改进。近年来论文越来越多。）</span><br style="color: #222222;" /><br style="color: #222222;" /><span style="color: #222222;">当初他们为什么要做虚拟机？简单说就是，新硬件层出不穷，但是OS赶不上。当初，他们想在Stanford的ccNUMA机器上跑IRIX（一个操作系统）。可是IRIX跑不起来。他们觉得修改OS或者写一个新的OS太难了（因为一个操作系统从出生到成熟要很长的时间，无数BUG要FIX，无数的新功能要增加。。。那样人家博士要怎么毕业。。）。所以，他们决定用虚拟机。</span><br style="color: #222222;" /><br style="color: #222222;" /><span style="color: #222222;">对于他们的项目，虚拟机大致有下面这些好处：</span>
<ol style="color: #222222;">
	<li>只要对商业OS做简单地修改，就能让他们在多个VM(Virtual Machine)间共享内存。</li>
	<li>Flexible。除了论文里的IRIX，实际上其他的OS也能跑。</li>
	<li>扩展性好。系统可以以虚拟机为单位扩展。</li>
	<li>fault-containment。每一个VM都是一个几乎独立的个体，一个坏了，不影响另一个。</li>
	<li>新老软件可共存。比如，新的软件只能在Linux-3.15跑。你可以用两个VM，一个是Linux2.6，一个是Linux-3.15。</li>
</ol>
<span style="color: #222222;">这篇文章回答下面这几个实现虚拟技术的关键问题：</span>
<ol style="color: #222222;">
	<li>VMM(Virtual Machine Monitor, 或者叫hypervisor)是怎么管住Guest OS的？或者说，皇上(VMM)是怎么防止大臣(OS)夺权的？</li>
	<li>有那么多个操作系统一起运行，内存是怎么管理的？</li>
	<li>多个VM之间是怎么分享资源的？或者说，1GB内存怎么当2GB用？</li>
</ol>
注意：如无特别说明，这篇文章里所说的处理器是指MIPS，而不是x86。
<h3>VMM(Virtual Machine Monitor, 或者叫hypervisor)是怎么管住Guest OS的？或者说，皇上(VMM)是怎么防止大臣(OS)夺权的？</h3>
<a href="http://www.manio.org/cn/wp-content/uploads/2014/06/new1.png"><img class="aligncenter wp-image-991 size-full" src="http://www.manio.org/cn/wp-content/uploads/2014/06/new1.png" alt="new1" width="200" height="278" /></a>

<br style="color: #222222;" /><span style="color: #222222;">要理解VMM是怎么工作的，我们得先了解没有VMM的时候，系统是怎么工作的。在没有VMM的时候，计算机系统中的"应用”可分为用户进程(比如VIM)和操作系统。他们分别运行在不同的模式中（mode）。我们用一个类比来解释系统中的模式（mode）。模式这个机制是用来限制一段指令所在的环境的权限的，它是需要处理器的支持的。MIPS结构当时有三个模式：用户模式（user mode)，长官模式（supervisor mode，原谅我的翻译...）和内核模式（kernel mode）。这三种模式分别对应于人类社会里的民宅、官衙和金銮殿。显而易见，在金銮殿里的人拥有的权限最高，民宅里的最低。我们可以说，在没有VMM时，用户进程住在民宅里，县衙空着，OS住在金銮殿里（如图）。用户可以直接做一些不用什么特权的事情（unprivileged instructions），比如计算1+1。做这些事情不经过OS。但是，有些要特权才能做的事情，一定要经过OS。比如保存正在编辑的文档到硬盘（IO operations）。访问硬盘是特权操作是因为硬盘是共享的资源，要有人管着，不能让人乱来。试想，要是人人都能读写档案馆的档案，那不就乱套了。</span><br style="color: #222222;" /><br style="color: #222222;" /><img class="size-full wp-image-992 aligncenter" src="http://www.manio.org/cn/wp-content/uploads/2014/06/new2.png" alt="new2" width="300" height="289" /><br style="color: #222222;" /><span style="color: #222222;">VIM（用户进程）不能直接写磁盘上的文件。VIM必须请求在金銮殿里的操作系统来做。怎么请求呢？通过系统调用（system call）。系统调用的过程是这样的：比如要调用的是write(fd, buf, len, off)，首先把fd, buf, len, off放到stack里，然后再把一个write()函数对应的号码（system call number）放到stack里，最后调用一个特殊的指令使CPU进入内核模式(图中的圈1)。在x86结构中，这个指令是int（中断）。在MIPS结构中，这个指令是trap。这个指令会引导CPU执行一段代码（trap handler），依照stack里的system call number 找到对应的函数（在这里是write()的实现），然后调用这个函数（函数的参数在刚才的stack里）。在这里函数里，操作系统调用文件所在的文件系统里的write()实现，文件系统使用磁盘的驱动来最终实现。所以文件的操作完成后，操作系统调用与trap功能相反的一个指令，回到VIM程序的指令里(图中的圈2)。</span>

关于trap（陷阱）指令：把这个指令叫trap（陷阱）是非常形象又准确的。当trap指令执行里，就像是掉入了有更多特权的人（比如皇上）设置的陷阱里一样，任人摆布。

<br style="color: #222222;" /><img class="size-full wp-image-993 aligncenter" src="http://www.manio.org/cn/wp-content/uploads/2014/06/new3.png" alt="new3" width="200" height="145" /><br style="color: #222222;" /><span style="color: #222222;">操作系统维护自己的特权的过程大概就是这样。但是这个特权等级到底是怎么实现的呢？我们可以想像CPU里有一个特权状态（privileged state bit）。状态为开(On)的时候，CPU的特权等级高，你可以做任何事，包括转到低特权状态。关(Off)的时候，你所能做的事情就所限了。那怎么从off状态转到on状态呢？你必须到执行CPU的特殊指令，这个指令会把你带到一个特定的地方执行操作系统的指令，检查你是不是有进入高特权状态的权限。这就像是在机场，你可以很容易地从登机口到售票大厅，可是你要从售票大厅到登机口，你就得过安检。另外，为什么操作系统就能有高特权呢？Hmm...因为操作系统一开始就把那占了，之后运行的应用程序就只能听它使唤了。</span><br style="color: #222222;" /><br style="color: #222222;" /><span style="color: #222222;">现在，终于要说VMM（virtual machine monitor, 或者叫hypervisor）是怎么实现的了。如下图：</span><br style="color: #222222;" /><img class="size-full wp-image-994 aligncenter" src="http://www.manio.org/cn/wp-content/uploads/2014/06/new4.png" alt="new4" width="200" height="243" /><br style="color: #222222;" /><span style="color: #222222;">现在VMM进了金銮殿，有了最高的特权。操作系统被放到了官衙里（但是它自己并不知道）。用户进程还是在民宅里住着。要是现在用户进程VIM调用write()，会发生什么？在这种情况下，</span><span style="color: #222222;">用户进程会trap到VMM（拥有kernel mode）里去。但是，VMM并不知道如何处理write()。所以，VMM接下来会调用操作系统的里对应的trap handler，这个handler会执行文件系统的write()。 在执行过程中，文件系统的非特权指令可以直接在真的CPU上执行。要是文件系统运行特权指令，CPU会从操作系统（supervisor mode）trap到VMM（kernel mode）里去，由VMM仿真（emulate）操作系统要运行特权指令。之所以要由VMM来仿真，有几个理由。第一，VMM不相信操作系统。例如，在VMM上可能同时运行着几个操作系统，如果让其中一个操作系统执行直接内存访问的指令，它可能修改内存中另一个操作系统的代码或数据。第二，操作系统只了解虚拟的环境，并不知道实际的环境。例如，操作系统想要写/dev/sda3的第10个sector。但是，实际上/dev/sda3可能对应的是VMM里一个叫/fake_dev/sda3.data的文件。这个时候，我们就需要VMM来把操作系统的指令转化为实际环境中的操作了。</span>

<span style="color: #222222;">仿真完特权指令之后，CPU会回到操作系统。当write()这个系统调用执行完之后，操作系统会调用一个特权指令（在MIPS里是rett指令），试图回到user mode。这个时候CPU又会trap到VMM，由VMM来完成最终的模式转换。</span><br style="color: #222222;" /><br style="color: #222222;" /><span style="color: #222222;">为什么VMM能够知道操作系统的trap handler在哪？系统中先有VMM。当你在VMM上安装操作系统的时候，操作系统会尝试调用特权指令安装trap handler。因为有最高特权的VMM实际上是监视着这一切的，所以它可以记录下trap handler的位置就行了。</span><br style="color: #222222;" /><br style="color: #222222;" /><span style="color: #222222;">总的说来，VMM占据了CPU的最高特权，使得在更低特权等级的操作系统无法进行有害操作，并且在实际环境中完成操作系统的需求。</span><br style="color: #222222;" /><b style="color: #222222;"></b>
<h3></h3>
<h3>有那么多个操作系统一起运行，内存是怎么管理的？</h3>
在没有VMM的时候，系统中有两种内存地址：虚拟地址(virtual address)和物理地址(physical address)。从虚拟地址到物理地址的转换有两种方式。方式一：在TLB(translate lookside buffer，硬件实现)查找。方式二：在页表(page table)中查找，找到之后把结果放到TLB中去。系统会先尝试方式一，要是找不到（TLB miss），就用方式二。

<br style="color: #222222;" /><img class="size-full wp-image-996 aligncenter" src="http://www.manio.org/cn/wp-content/uploads/2014/06/new6.png" alt="new6" width="500" height="65" /><br style="color: #222222;" /><span style="color: #222222;">在有了VMM之后，系统中有三种内存地址：虚拟地址(virtual address)，物理地址(physical address)和机器地址(machine address)。机器地址才是真正与内存条上的地址一一对应的。物理地址只是操作系统认为的物理地址。</span><br style="color: #222222;" /><img class="size-full wp-image-995 aligncenter" src="http://www.manio.org/cn/wp-content/uploads/2014/06/new5.png" alt="new5" width="400" height="254" /><br style="color: #222222;" /><span style="color: #222222;">当操作系统试着要使用特权指令来完成一个虚拟地址到物理地址的转换时（TLB miss），VMM就介入了（VMM监视着所有对特权寄存器的操作）。VMM会先使用操作系统内的代码来先完成虚拟地址到物理地址的转化（因为VMM并不知道这个映射关系）。然后，操作系统认为自己已经完成了转化，尝试去更新TLB（特权操作）。这个时候，VMM会介入，用一个叫个pmap的映射表找到物理地址对应的机器地址，用机器地址替换掉物理地址，然后把TLB更新为虚拟地址到机器地址的映射。之后，所有对这个虚拟地址的访问都会被转换为对相应机器地址的访问。（注意，MIPS用的是software-reloaded TLB，x86用的是hardware-reloaded TLB）</span>

&nbsp;
<h3>多个VM之间是怎么分享资源的？或者说，1GB内存怎么当2GB用？</h3>
<span style="color: #222222;">我们知道，每一个虚拟机都要占用大量的内存空间。在内存有限的情况下，怎么在一台机器运行更多的虚拟机？幸运的是，不用的虚拟机之间在内存中数据可能会完全一致（比如，系统文件在内存中的缓存）。如要我们可以只在内存中保留一份数据，我们就行节省很多空间。Disco使用虚拟IO设备和虚拟网络设备来节省内存空间。</span>

<br style="color: #222222;" /><span style="color: #222222;"><img class="size-full wp-image-997 aligncenter" src="http://www.manio.org/cn/wp-content/uploads/2014/06/new7.png" alt="new7" width="400" height="189" /></span>

<span style="color: #222222;">虚拟IO设备：当两个虚拟机从同一个磁盘上读同一个文件时，VMM会intercept DMA，然后就会发现这两个VM在使用同样的数据。这份数据只需要在机器内存里保存一份，然后修改pmap，使得两个VM的物理地址指向同一个机器地址就可以了。当任何一个VM更新这份数据，VMM会给它一份新的拷贝，原来的那份不做更改（copy on write机制）。　</span><br style="color: #222222;" /><br style="color: #222222;" /><span style="color: #222222;">虚拟网络设备：当使用NFS从VM1向VM2复制文件时，文件并没有被真正地复制。虚拟网络设备会更新VM2上的pmap，使之指向在内存中的文件，使得VM2上的操作系统认为自己已经有了这个文件。</span><br style="color: #222222;" /><br style="color: #222222;" /><span style="color: #222222;">后来，VMWare还有用hash来找相同的内存页然后再共享的技术。</span><br style="color: #222222;" /><br style="color: #222222;" /><span style="color: #222222;">（完）</span>
