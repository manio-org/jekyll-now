---
layout: post
title:  "pthread_cond_wait 为什么需要传递 mutex 参数?"
date:   2012-02-02
categories: OS
---

（本文是我在<a href="http://www.zhihu.com/people/manio">知乎</a>上的回答，整理了一下）

<span style="color: #222222;">有几篇30多年前的论文极大地影响了现代操作系统中进程/线程的同步机制的实现，尤其是楼主问题中的实现。一篇是 Monitors: An Operating System Structuring Concept ( </span><a style="color: #225599;" href="http://www.vuse.vanderbilt.edu/~dowdy/courses/cs381/monitor.pdf" data-editable="true" data-title="vanderbilt.edu 的页面">http://www.vuse.vanderbilt.edu/~dowdy/courses/cs381/monitor.pdf</a><span style="color: #222222;">)，还有一篇是 Experience with Processes and Monitors in Mesa (</span><a style="color: #225599;" href="http://msr-waypoint.com/en-us/um/people/blampson/23-ProcessesInMesa/Acrobat.pdf" data-editable="true" data-title="msr-waypoint.com 的页面">http://msr-waypoint.com/en-us/um/people/blampson/23-ProcessesInMesa/Acrobat.pdf</a><span style="color: #222222;">)。另外，还有讨论semaphore，生产者/消费者问题，哲学家就餐问题等等的论文。</span><br style="color: #222222;" /><br style="color: #222222;" /><span style="color: #222222;">这里，我先介绍这两篇论文的内容，然后引出问题的答案。</span><br style="color: #222222;" /><br style="color: #222222;" /><span style="color: #222222;">第一篇是Tony Hoare写的，通常被叫做Hoare's Monitor。你可能不知道Tony Hoare是谁，但是你应该知道他发明的quicksort。你也应该知道他得过的那个图灵奖。Monitor是一种用来同步对资源的访问的机制。Monitor里有一个或多个过程(procedure)，在一个时刻只能有一个过程被激活(active)。让我来给个例子：</span>

```
MONITOR account {
    int balance; //initial value is 0;
    procedure add_one() {
        balance++
    }
    procedure remove_one() {
        balance--
    }
}
```

<span style="color: #222222;">如果这是一个monitor，add_one()和remove_one()中只能有一个中被激活的。也就是说，balance++和balance--不可能同时执行，没有race。</span><br style="color: #222222;" /><br style="color: #222222;" /><span style="color: #222222;">正如论文的标题所说的，monitor只是一个概念，他可以被几乎任何现代的语言实现。如果我们要用C++来实现Monitor，伪代码差不多就是这样(Java的Synchronization是Monitor的一个实现)：</span>

```
class Account {
  private:
    int balance; //initial value is 0;
    lock mutex;
  public:
    void add_one() {
        pthread_mutex_lock(&amp;mutex);
        balance++;
        pthread_mutex_unlock(&amp;mutex);
    }
    void remove_one() {
        pthread_mutex_lock(&amp;mutex);
        balance--
        pthread_mutex_unlock(&amp;mutex);
    }
};
```

<br style="color: #222222;" /><span style="color: #222222;">论文中也有条件变量（conditional variable），使用形式是cond_var.wait()和cond_var.signal()。让我们来看一下论文里最简单的一个例子。这是同步对单个资源访问的monitor。原文中的代码（好像）是用Pascal写的，我这里有C-style重写了一下。</span>

```
//注意这是一个monitor，这里的acquire()和release()不能也不会同时active。
Monitor SingleResource {
  private:
    bool busy;
    condition_variable nonbusy;
  public:
    void acquire() {
        if ( busy == true ) {
            nonbusy.wait()
        }
        busy = true
    }
    void release() {
        busy = false;
        nonbusy.signal()
    }
    busy = false; //initial value of busy
};
```

<span style="color: #222222;">需要特别注意，这是一个monitor（不是class）。其中的acquire()和release()不能也不会同时active。我们注意到，这里的nonbusy.wait()并没有使用lock作为参数。但是，Hoare其实是假设有的。只是在论文中，他把这个lock叫做monitor invariant。论文中，Hoare解释了为什么要在conditional wait时用到这个值（也就解释了楼主的问题）。我原文引用一下：</span>

> Since other programs may invoke a monitor procedure during a wait, a waiting program must ensure that the invariant t for the monitor is true beforehand.

<span style="color: #222222;">换句话说，当线程A等待时，为了确保其他线程可以调用monitor的procedure，线程A在等待前，必须释放锁。例如，在使用monitor SingleResource时，线程A调用acquire()并等待。线程A必须在实际睡眠前释放锁，要不然，即使线程A已经不active了，线程B也没法调用acquire()。（当然，你也可以不释放锁，另一个线程根本不检查锁的状态，然后进入对条件变量的等待!! 但是，首先，这已经不是一个monitor了，其次，看下文。）</span><br style="color: #222222;" /><br style="color: #222222;" /><span style="color: #222222;">pthread只是提供了一种同步的机制，你可以使用这种机制来做任何事情。有的对，有的错。Hoare在论文里的一段话也说更能解答楼主的问题：</span>


> The introduction of condition variables makes it possible to write monitors subject to the risk of deadly embrace [7]. <b>It is the responsibility of the programmer to avoid this risk, together with other scheduling disasters</b> (thrashing, indefinitely repeated overtaking, etc. [11]).


<span style="color: #222222;">有兴趣的同学可以读读这篇文章。</span><br style="color: #222222;" /><br style="color: #222222;" /><span style="color: #222222;">另外，为什么上面的代码里acquire()的实现使用的是：</span>
<pre lang="cpp" style="color: #222222;">if ( busy == true ) {</pre>
<span style="color: #222222;">而不是：</span>
<pre lang="cpp" style="color: #222222;">while ( busy == true ) {</pre>
<span style="color: #222222;">?</span><br style="color: #222222;" /><span style="color: #222222;">这是因为，在Hoare的假设里，当线程A调用nonbusy.signal()之后，线程A必须立即停止执行，正在等待的线程B必须紧接着立即开始执行。这样，就可以确保线程B开始执行时 </span><b style="color: #222222;"><i>busy</i>==false</b><span style="color: #222222;">。这正是我们想要的。</span><br style="color: #222222;" /><br style="color: #222222;" /><span style="color: #222222;">但是，在现代的系统中，这个假设并不成立。现代操作系统中的机制跟Mesa中描述的一致：在condvar.signal()被调用之后，正在等待的线程并不需要立即开始执行。等待线程可以在任何方便的时候恢复执行（优点之一：这样就把同步机制和调度机制分开了）。</span><br style="color: #222222;" /><br style="color: #222222;" /><span style="color: #222222;">在Mesa的假设下，上面的Monitor SingleResource的代码是错的。试想下面的执行过程：1. 线程A调用acquire()并等待，2. 线程B调用release()，2.线程C调用acquire()，现在</span><i style="color: #222222;">busy</i><span style="color: #222222;">=true，3. 线程A恢复执行，但是此时busy已经是true了! 这就会导致线程A和线程C同时使用被这个monitor保护的资源!!!</span>

```
void acquire() {
        if ( busy == true ) {
            nonbusy.wait()
        }
        //assert(busy != true)
        busy = true
    }
```

<span style="color: #222222;">在Mesa中，Butler Lampson和David Redell提出了一个简单的解决方案－把 if 改成 while。这样的话，在线程A恢复执行时，还要再检查一下</span><i style="color: #222222;">busy</i><span style="color: #222222;">的值。如果还不是想要的，就会再次等待。</span><br style="color: #222222;" />* * *

<span style="color: #222222;">下面是一位同学的代码，我觉得这是一个很好的例子，让我来试着讨论一下这段代码。</span>
```
// code 1                                    // line number
pthread_mutex_lock(mtx);                     // a1
if (pass == 0)                               // a2
    pthread_cond_unlock_and_wait(cv, mtx);   // a3
else                                         // a4
    pthread_mutex_unlock(mtx);               // a5
// mark                                      // a6


// code2                   // line number
pthread_mutex_lock(mtx);   // b1
pass = 1;                  // b2
pthread_mutex_unlock(mtx); // b3
pthread_cond_signal(cv);   // b4
```

<span style="color: #222222;">首先，假设没有“虚假唤醒（也就是说只要被唤醒，那么条件必然满足）”，这段代码也漏掉了一行重要的代码，就是</span>
`pass = 0`
<span style="color: #222222;">这相当于Hoare的论文中的</span>
`busy = true`
<span style="color: #222222;">这如果没有这行代码，这段代码的完整性就有待商讨。因为当code2第一次被执行后，所有执行code1的线程都不会等待（执行序列不会到达a3，他们可以在同一时间全部在a6），不能达到同步线程的作用。</span><br style="color: #222222;" /><br style="color: #222222;" /><span style="color: #222222;">另外，要在这个新的设计上加上pass = 0也很困难。试看下面的几个例子：</span><br style="color: #222222;" /><span style="color: #222222;">例子1:</span>
```
// code 1                                    // line number
pthread_mutex_lock(mtx);                     // a1
if (pass == 0)                               // a2
    pthread_cond_unlock_and_wait(cv, mtx);   // a3
else                                         // a4
    pthread_mutex_unlock(mtx);               // a5
pass = 0                                     // a6
```

<span style="color: #222222;">这个例子中的代码还是没有达到同步的作用，因为如果有10的线程，这十个线程还是有可能同时到a6之间。另外，a6没有得到mutex的保护，有race condition。</span><br style="color: #222222;" /><br style="color: #222222;" /><span style="color: #222222;">例子2:</span>

```
// code 1                                    // line number
pthread_mutex_lock(mtx);                     // a1
if (pass == 0)                               // a2
    pthread_cond_unlock_and_wait(cv, mtx);   // a3
    pass = 0                                 // a3'
else                                         // a4
    pass = 0                                 // a4'
    pthread_mutex_unlock(mtx);               // a5
```

<span style="color: #222222;">因为在这个设计里pthread_cond_unlock_and_wait()并不会在恢复执行时请求mtx（如果会，这段代码也是有问题的，会有死锁），所以，a3'没有得到mutex的保护，有race condition。</span><br style="color: #222222;" /><br style="color: #222222;" /><span style="color: #222222;">这段代码可以被改正，但是会跟原来设计十分相似。</span><br style="color: #222222;" /><br style="color: #222222;" /><br style="color: #222222;" /><span style="color: #222222;">我再在这里复制一下这段代码，讨论另一个问题。</span>

```
// code 1                                    // line number
pthread_mutex_lock(mtx);                     // a1
if (pass == 0)                               // a2
    pthread_cond_unlock_and_wait(cv, mtx);   // a3
else                                         // a4
    pass = 0                                 // a4'
    pthread_mutex_unlock(mtx);               // a5



// code2                   // line number
pthread_mutex_lock(mtx);   // b1
pass = 1;                  // b2
pthread_mutex_unlock(mtx); // b3
pthread_cond_signal(cv);   // b4
```


<span style="color: #222222;">这段代码很难做到没有“虚假唤醒（也就是说只要被唤醒，那么条件必然满足）”。试想下面的执行：</span>

```
TIME     THREAD1    THREAD2   THREAD3  COMMENT
t0       a1                            initially, pass == 0
t1       a2
t2       a3                            wait for cv
t3                  b1                    
t4                  b2
t5                  b3                 context switch to thread 3
t6                            a1
t7                            a2       pass=1
t8                            a4'      假设这里正确地将pass置为0
t9                            a5
t10      (a3)
```

<br style="color: #222222;" /><span style="color: #222222;">在t10时间，thread1看到的状态是pass==0，而不是它所期待的pass==1。这个情况会发生的根本原因为没有办法保证{b3;b4}的原子性。在这个情况下假设没有”虚假唤醒“有些牵强。</span><br style="color: #222222;" /><br style="color: #222222;" /><span style="color: #222222;">要让这段代码没有“虚假唤醒“，需要把b4放到b3前面（并且要控制好操作系统的调度）。</span>

```
// code2 (revision)
pthread_mutex_lock(mtx);  
pass = 1;                 
pthread_cond_signal(cv);  
pthread_mutex_unlock(mtx); 
```

<span style="color: #222222;">这样，有了mutex的保护，加上”等待的线程会立即执行“的假设，上面的代码才相对正确一些。</span>

