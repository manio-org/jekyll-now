---
layout: post
title:  "How to read and manage research papers with printer, scanner, Evernote - a practical example"
date:   2013-07-12
categories: science
---


<h2>Read</h2>

If you are pursuing a PhD degree like I am, you need to read papers. The frequent problem that I meet is that I read and forget. A few days after reading a paper, I forget what it is about. This might be acceptable for certain papers that are not important to me. It is not acceptable for papers that are listed here http://research.cs.wisc.edu/areas/os/Qual/ . First of all, I have to pass the qualifying exam. Since I obviously cannot read all the fifty papers within 3 days before the exam, I have to distribute the papers to a long period. I am likely to forget some of the earlier ones. Second, these fifty papers and many other papers are important to my research/career. With a good understanding AND MEMORY of the papers, I might be able to develop better/newer ideas to solve the problems. Knowing what is old helps identifying what is new.

I want to remember the key ideas of a paper, the problems the authors try to solve and the challenges they meet. When I forget, I want to have something that can help me recall it. When I need to take an exam/talk the paper to someone else, I want to have something that can help me recall more details of the paper - what the structure of the paper is? What each paragraph is about? Do I understand them? What are the limitations? What's the extension of the paper?...

Although it is <strong>not about memory</strong>, this process of helping you recall the paper is expected (by me) to help getting better understanding of the paper. It is because you can only recall when you understand it before. The key of reading a paper is to "<strong>re-implement</strong>" the paper, or "<strong>say it in your own words</strong>". Often times, <strong>you thought you understood. But you don't really do until you can say it from your own word.</strong> And "say it in your own words" is equal to "answer an exam question", "tell someone you idea", or "write a paper". It forces you to truly understand the problem. One side benefit of "say it in your own words" is that you can review the problem by reading your own words, which is much faster than reading the paper again. To force you say it from your own words, you can imagine that you have an exam tomorrow and all the questions are from this paper.

<h2>Example</h2>

I take a paper I read recently as an example. I do the following when reading a paper.
<ul>
    <li>Put the paper to Evernote and tag it with "paper", "to.read.list". Set an appropriate priority for it.</li>
    <li>Print and mark on the paper while reading. Holding a paper paper is better than holding a mouse and looking at an e-paper because: 1. easier to draw graphs for understanding, 2. easier to write a few words (in a pdf editor, you need to find the mouse, move to a insert position, switch to keyboard, type, adjust the position. Often you have to use sticky note tool and CLICK to see your comments.) 3. less distraction. you mouse may "<em>accidentally</em>" move to Facebook, Twitter... or some notifications may pop out and drag you away. Shut down/sleep your computer while reading a paper. Regarding marks, somebody says using a highlighter is better than underlining. I will try that.</li>
    <li>Summarize each important section, paragraph and write it down on the margin. This is a part of "say it in your own words".
<ul>
    <li>Be critical. Imagine you are re-implementing the paper, you may find out the limitations, false assumptions/conclusions..</li>
    <li>Be creative. What are the extensions of this technique? How can it help my research?</li>
</ul>
</li>
    <li>Scan the paper and put it to Evernote. Tag it with "paper", "have.read". Evernote can search in your scanned PDFs.</li>
    <li>Write a summary of the whole paper in Evernote. This helps you recall the paper after you forget it. Sometimes, you can write down all your understandings and details if it is really an important paper. <strong>When you try to summarize, you are being tested if you have fully understood it.</strong> Doing it in Evernote is because typing large paragraphs is faster with a keyboard. And it is easier for Evernote to search.</li>
    <li>Sometimes, make a mindmap, scan it and put it to Evernote. Mindmap shows the structure of the paper well. Also, you can draw graphs to illustrate..</li>
</ul>
The paper that has been marked is <a href="wp-content/uploads/2013/07/ffs.pdf">here</a>. This is a summary that I wrote:

<hr />

A Fast file system for unix

This is a very important paper in FS field. It presents techniques to improve the Unix file system's throughput by introducing cylinder group and cluster related data into the same group, by increasing the block size for better locality (larger single transfers), by enabling two block sizes for better fragment management. It compares the old file system and the new file system (presented in this paper). The old file system has a very low utilization at about 2%. The new file system can have up to 47% due to less disk seeks.

Section 2 talks about the old file system, why the old one is bad. It serves as the motivation of the paper. Something needs to remember. Small block size is bad because the blocks are often allocated randomly due to a bad free block list and the allocation algorithm. In addition, when there are two files being written gradually, blocks are allocated for the two files interleavely. The blocks are likely to be not contiguous. Having bigger block size directly increases the contiguousness of the data, at the price of wasting more space potentially. The waste can be mitigated by using two block sizes, at the prices of bigger metadata and management cost. Bigger block size also reduce indirect blocks which introducing more seeks. The design of the things above is talked in 3.1.

The Section 3.2 describes how to design a CPU- and disk-aware file system. Consecutive allocation is NOT always the best! This is because of CPU. If a CPU has I/O channels, then consecutive blocks are good because CPU does not have to interrupt and prepare for a new I/O transfer. If CPUp does not have I/O channel, then blocks should be set apart in a proper distance. After the CPU is busy preparing and disk head moving, the head moves to the right position for the next access and can start accessing right there.

Layout policies include a global one and a local one. The global one tries to cluster related data and spread unrelated data. Basically, inodes of files in the same directory are placed in the same cylinder group. inodes of directories to where there are lots of free inode space and little existing directory. The local one optimizes locally, trying to allocate the next available inode for file. For data, it tries to place data in the same cylinder group.

Performance. In the old file system, write is faster than read because the data is buffered and the block allocation is simple. In the new file system, write is slower because block allocation is more complicated because of the fragment management.

Directory is organized as directory chunks. Chunk size is fixed. A chunk consists of directory entries with varying sizes.

<a href="http://ccr.sigcomm.org/online/files/p83-keshavA.pdf">This</a> article describes how to read papers by 3-pass technique.

