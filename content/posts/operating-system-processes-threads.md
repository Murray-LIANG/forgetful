---
title: "Processes and Threads"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "processes",
    "threads",
    "multi-processing",
    "multi-threading",
]

comment: true
toc: true
auto_collapse_toc: true
---
# Processes and Threads

**进程**是某个程序的一次运行活动，是**资源分配**和调度的一个独立单位。**线程**是进程中的一个控制线
程，**最小的独立运行的基本单位**。进程中所有线程共享进程的资源，如**虚拟地址空间**，**文件描述符
**。每个线程有自己的**程序计数器**，**寄存器**和**栈**等。

# Multiprocessing and Multithreading
1. 单进程，单线程，MS-DOS大致是这种
2. 多进程，单线程，多数Unix及Linux是这种
3. 多进程，多线程，Windows，Solaris 2.x和OS/2是这种
4. 单进程，多线程，VxWorks是这种

# Thread的种类
## User Level Threads
仅存于用户空间中。对于他们的创建，撤销，之间的同步和通信等功能**无需系统调**用来实现。在同一个进程中
的用户级**线程切换不需要内核**支持，而内核完全不知道用户线程的存在，而是**视作一个进程**，所以**调
度是以进程为单位**进行的。

- 优点
1. 线程切换不需要转到内核空间，节省了宝贵的内核空间
2. 调度算法进程专用，由用户程序制定

- 缺点
1. 系统调用阻塞，同一个进程**一个线程阻塞，整个进程都会阻塞**
2. **一个进程只能在一个CPU**上获得执行

![](/images/operating-system-processes-threads-user-level-threads.jpg)

就线程的同时执行而言，任意给定时刻每个进程只能有一个线程在运行，而且只有一个处理器内核会被分配给该进
程。对于一个进程，可能有成千上万个用户级进程，但是他们对系统资源没有影响。运行时**库**调度并分派这些
线程。库调度器从进程中的多个线程中选择一个线程，然后该线程和该进程允许的一个内核线程关联起来。

## Kernel Supported Threads
内核空间为每个内核支持的线程设置了一个线程控制块，内核则根据该控制块来感知线程的存在并控制。

所以，
- 优点
1. 在多核处理器上，内核可以调用同**一个进程中的多个线程**同时工作
2. 如果**一个进程中哦某个线程阻塞了，其他线程仍然可以得到工作**

- 缺点
1. 相对于用户线程来说，**切换开销大**，需要从用户态，进入内核态并且由内核进行切换，因为线程调度和管
理都在内核实现。

![](/images/operating-system-processes-threads-kernel-supported-threads.jpg)

内核级线程是内核对象，常驻内核空间。用户线程与之一一对应。运行时 库会为每个用户线程分配一个内核级线
程。


## 混合方式
![](/images/operating-system-processes-threads-mixed-mode-threads.jpg)

结合了以上两种线程。库和操作系统都可以管理线程。**进程有着自己的内核线程池**。可运行的用户线程由运行
时库分派并标记为准备好执行的可用线程。操作系统选择用户线程并将它映射到线程池中的可用内核线程。

内核线程不会被销毁或者重建，只是在必要时被分配给不同的用户级线程，而不是每当创建新的用户级线程都会创
建新的内核线程。

# 进程和线程的上下文
进程的上下文多数信息都与地址空间的描述有关，包括：
- 指向可执行文件的指针
- 栈
- 静态和动态分配的变量的内存
- 状态
- 优先级
- 程序IO状态
- 授予权限
- 调度信息
- 审计信息
- 有关的资源信息
    - 文件描述符
    - 读写指针
- 有关事件和信号的信息
- 寄存器组
    - 栈指针
    - 指令计数器

而线程的上下文信息比较少，大部分都是共享进程的：
- 栈
- 状态
- 优先级
- 寄存器组
    - 栈指针
    - 指令计数器

# Go中Goroutine使用epoll
Gorouines与kernel threads是多对多的mapping。所以必须有个机制来让内核通知用户级的线程管理器关于
内核线程的使用情况。

## Scheduler Activation
在**BSD**中，有一个机制叫**Scheduler Activation**，使用它可以在用户级线程做**blocking**
system call时，让内核创建一个**新的内核级线程**给进程去执行其他没有阻塞的用户级线程，然后内核通过
**upcalls**来通知用户级线程管理器有线程阻塞，**新的内核线程已经分配用户级线程管理器去执行新的线程
**。

http://www.it.uu.se/education/course/homepage/os/vt18/module-4/implementing-threads/


## Linux中没有Scheduler Activation
Go也并没有利用Scheduler Activation，而是使用了**epoll**来避免给每个**阻塞**的用户级线程分配内
核线程。

https://groups.google.com/forum/#!topic/golang-dev/8U7zlE1OD3U
