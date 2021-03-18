---
layout:     post
title:      网络IO之阻塞、非阻塞、同步、异步
subtitle:   网络IO
date:       2019-11-7
author:     Jow
header-img: img/about-bg-walle.jpg
catalog: 	 true 
tags:
    - c++
    - network

---

### 目录
1. 前言
2. 数据流向
3. 网络IO模型的详细分析

> 当你觉得真的很难的时候，就是需要迎头而上的时候，慢慢的你就会发现，自己真的成长进步了许多。


## 前言

这段时间一直在研究win C++网络编程，通过一点时间对TCP的学习发现网上的例子真的非常的简单，只是讲了怎么用，但是其中很多的细节和坑都没有讲解的很清楚，其中阻塞这块将的就非常的模糊，经过前面的实验和总结，今天打算根据别人的讲解自己再做一次深入的理解。掺杂着部分自己的理解，但是大部分还是总结。

一般讲解的IO模型包含五种，阻塞、非阻塞、多路、信号驱动以及异步，信号在十几种并不是很常用，所以在这里主要讲解其他四种。
## 数据流向

网络IO操作实际过程涉及到内核和调用这个IO操作的进程。以read为例，read的具体操作分为以下两个部分:

1. 内核等待数据可读
2. 将内核读到的数据拷贝到进程

具体流程如下图：
![](https://images0.cnblogs.com/blog/305504/201308/12224938-4db3844232b84fb284d057a21df5f149.png)

再说一下IO发生时涉及的对象和步骤。对于一个network IO（以read举例），它会涉及到两个系统对象，一个是调用这个IO的process（or thread），另一个就是系统内核（kernel）。当一个read操作发生时，它会经历两个阶段：

1. 等待数据准备（waiting for the data to be ready)
2. 将数据从内核拷贝到进程中(Copying the data from the kernel to the process)。

## 网络IO模型的详细分析
常见的IO模型有阻塞、非阻塞、IO多路复用以及异步。

**blocking IO**

在Linux中，默认情况所有的额socket都是blocking，一个典型的读写操作流程大概是这样子：
![](http://hi.csdn.net/attachment/201007/31/0_1280550787I2K8.gif)

当用户调用了recvfrom这个系统调用，kernel就开始了IO的第一个阶段：准备数据。对于network io来说，很多时候数据在一开始还没有到达，这个时候kernel就要等待足够的数据到来。而在用户进程这边，整个进程就会被阻塞，当kernel一直等到数据准备好了，它就会将数据从kernel中拷贝到用户内存，然后kernel返回结果，用户进程才接触block的状态重新运行起来。

所以，blocking IO的特点就是在IO执行的两个阶段都被block了。

**non-blocking IO**

linux下，可以通过设置socket使其变为non-blocking。当对一个non-blocking socket执行读操作时，流程是这个样子：

![](http://hi.csdn.net/attachment/201007/31/0_128055089469yL.gif)

从图中可以看出，当用户进程发出read操作时，如果kernel中的数据还没有准备好，那么它并不会block用户进程，而是立刻返回一个error。从用户进程角度讲 ，它发起一个read操作后，并不需要等待，而是马上就得到了一个结果。用户进程判断结果是一个error时，它就知道数据还没有准备好，于是它可以再次发送read操作。一旦kernel中的数据准备好了，并且又再次收到了用户进程的system call，那么它马上就将数据拷贝到了用户内存，然后返回。

所以，用户进程其实是需要不断的主动询问kernel数据好了没有。

**IO multiplexing**

IO multiplexing这个词可能有点陌生，但是如果我说select，epoll，大概就都能明白了。有些地方也称这种IO方式为event driven IO。我们都知道，select/epoll的好处就在于单个process就可以同时处理多个网络连接的IO。它的基本原理就是select/epoll这个function会不断的轮询所负责的所有socket，当某个socket有数据到达了，就通知用户进程。它的流程如图：

![](http://hi.csdn.net/attachment/201007/31/0_1280551028YEeQ.gif)

当用户进程调用了select，那么整个进程会被block，而同时，kernel会“监视”所有select负责的socket，当任何一个socket中的数据准备好了，select就会返回。这个时候用户进程再调用read操作，将数据从kernel拷贝到用户进程。

这个图和blocking IO的图其实并没有太大的不同，事实上，还更差一些。因为这里需要使用两个system call (select 和 recvfrom)，而blocking IO只调用了一个system call (recvfrom)。但是，用select的优势在于它可以同时处理多个connection。（多说一句。所以，如果处理的连接数不是很高的话，使用select/epoll的web server不一定比使用multi-threading + blocking IO的web server性能更好，可能延迟还更大。select/epoll的优势并不是对于单个连接能处理得更快，而是在于能处理更多的连接。）

在IO multiplexing Model中，实际中，对于每一个socket，一般都设置成为non-blocking，但是，如上图所示，整个用户的process其实是一直被block的。只不过process是被select这个函数block，而不是被socket IO给block。

**Asynchronous I/O**
linux下的asynchronous IO其实用得很少。先看一下它的流程：

![](http://hi.csdn.net/attachment/201007/31/0_1280551287S777.gif)

用户进程发起read操作之后，立刻就可以开始去做其它的事。而另一方面，从kernel的角度，当它受到一个asynchronous read之后，首先它会立刻返回，所以不会对用户进程产生任何block。然后，kernel会等待数据准备完成，然后将数据拷贝到用户内存，当这一切都完成之后，kernel会给用户进程发送一个signal，告诉它read操作完成了。

A synchronous I/O operation causes the requesting process to be blocked until that I/O operation completes;An asynchronous I/O operation does not cause the requesting process to be blocked; 

各个IO Model的比较如图所示：
![](http://hi.csdn.net/attachment/201007/31/0_1280551552NVgW.gif)

经过上面的介绍，会发现non-blocking IO和asynchronous IO的区别还是很明显的。在non-blocking IO中，虽然进程大部分时间都不会被block，但是它仍然要求进程去主动的check，并且当数据准备完成以后，也需要进程主动的再次调用recvfrom来将数据拷贝到用户内存。而asynchronous IO则完全不同。它就像是用户进程将整个IO操作交给了他人（kernel）完成，然后他人做完后发信号通知。在此期间，用户进程不需要去检查IO操作的状态，也不需要主动的去拷贝数据。