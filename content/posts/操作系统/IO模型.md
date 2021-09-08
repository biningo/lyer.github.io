---
title: IO模型
date: 2021-05-02
categories: [操作系统]
tags: [IO,Linux]
draft: true
---

## 直接IO和非直接IO

就拿磁盘来说，磁盘 I/O 是非常慢的，所以 Linux 内核为了减少磁盘 I/O 次数，在系统调用后会把用户数据拷贝到内核空间中缓存起来，只有缓存数据被取完的时候才发起磁盘 I/O 的请求。所以IO分为直接IO和非直接IO

- **直接 I/O**  不会发生内核缓存和用户程序之间数据复制，用户程序直接经过文件系统访问磁盘，缺点就是用户进程每次访问数据都会执行一次磁盘IO，除非用户自己做了缓存
- **非直接 I/O**  读操作时，数据从内核缓存中拷贝给用户程序。写操作时，数据从用户程序拷贝给内核缓存，再由内核决定什么时候写入数据到磁盘

内核会在一下几种情况下对真实的磁盘进行读写:

- 内核缓存区满了
- 用户主动调用`sync`
- 缓存数据超过指定时间
- ....

大部分程序都是采用缓冲IO来访问磁盘等数据，这样可以借助操作系统减少访问磁盘的次数。但是有些程序比如**数据库MySql**等则不会使用非直接IO，而是自己设计一套更加优秀的缓存机制，这样可以减少内核空间到用户空间拷贝数据的消耗。并且数据库都是有一套自己数据保存磁盘上的规则，比操作系统更加了解如何快速的访问数据

​      

## 缓冲IO和非缓冲IO

- **缓冲IO** 用户进程在自己空间也开辟缓存，每次系统调用的时候就会多获取一些数据到自己的缓存中去，这样可以减少系统调用的消耗

- **非缓冲IO** 则是每次读写数据都直接进行系统调用，系统调用结果直接返回到应用程序

​       

## 阻塞IO(BIO)和非阻塞IO(NIO)

**阻塞IO** 指的是用户线程在`read`系统调用时会一直等待内核读取数据，等内核读取数据到内核缓冲区(**比如从网卡读取数据、通过底层文件系统从磁盘读取数据到内核缓冲区**)然后将内核缓冲区数据copy到用户空间，这样整个`read`调用才会返回

```bash
阻塞IO等待时间=操作系统读取数据时间+内核缓冲区到用户缓存区copy的时间
```

![img](https://raw.githubusercontent.com/biningo/cdn/master/2021-04/BIO.png)

**非阻塞IO** 指的是用户线程在`read`系统调用之后可以立即返回，但是需要自己主动再次向内核询问数据是否读取到内核缓冲区中，这里需要多次询问直到数据读取完毕(在轮询期间可以干一些其它事情)，当数据准备好了之后，则需要阻塞的等待内核将内核缓冲区的数据copy到用户空间中(**注意这个过程是阻塞的**)

```bash
非阻塞IO等待的时间=不断轮询的消耗的时间+内核缓冲区到用户缓存区copy的时间
```

[![img](https://raw.githubusercontent.com/biningo/cdn/master/2021-04/NIO.png)](https://raw.githubusercontent.com/biningo/cdn/master/2021-04/NIO.png)

​    

## IO多路复用

> **为什么有IO多路复用**

在同步阻塞IO方案下，我们可以有下面两种方案来应对高并发场景

1. 一个请求必须等待另外一个请求结束之后才能被处理，这种完全没有并发处理能力
2. 每来一个请求就开一个线程进行处理，这种能同时处理多个请求，但是并发量如果很多那么就会开辟很多线程带来系统线程调度的消耗

于是就出现了IO多路复用，在一个线程里可以监听多个IO句柄处理多个请求，当有IO可读时则顺序处理那些可读的IO句柄。实现IO多路复用的前提是同步非阻塞IO，当IO不可读则需要立即返回去查看其他IO是否可读

TODO



数据从内核缓冲区copy到用户缓冲区等一系列操作，这个copy的过程还是需要阻塞等待的，所以多路复用是一个**同步非阻塞IO模型**

[![img](https://raw.githubusercontent.com/biningo/cdn/master/2021-04/MIO.png)](https://raw.githubusercontent.com/biningo/cdn/master/2021-04/MIO.png)

​     

## IO多路复用的系统调用函数

### select

```c
int select(
    int nfds, //注册fd的数量
    //fd_set是个数组 最大监听1024个fd
  	fd_set *readfds, //读事件 比如socket fd可读了
    fd_set *writefds, //写事件 比如socket缓冲区可写了
    fd_set *writefds, //异常事件
    struct timeval *timeout //超时时间 为NULL则表示如果没有fd可读则阻塞等待
);
```

- 监听的fd数量有限
- 每次需要将fd在用户空间到内核空间来回copy
- 产生惊群，只要有一个fd可读，那么用户进程必须遍历所有fd去检查到底是哪个fd可读

### poll

因为poll将fd集合用**链表**来保存，所以poll能监听的fd数量没有限制（小于操作系统的最大打开文件描述符个数），其他都和select区别不大

### epoll

```c
//初始化一个epoll句柄 返回一个句柄
//epoll_create1(EPOLL_CLOEXEC);
int epoll_create1 (int flags);

//epoll注册等操作
int epoll_ctl(
	int epfd, //create出来的epoll句柄
    int op, //操作类型 增加,删除..
    int fd, //待操作的fd
    struct epoll_event *events //epoll事件
)

    
int epoll_wait(
	int epfd,
    struct epoll_event *events,
    int maxevents //最大的事件数量
    int timeout
)
```

​        

## 异步IO(AIO)和同步IO

[![img](https://raw.githubusercontent.com/biningo/cdn/master/2021-04/io.png)](https://raw.githubusercontent.com/biningo/cdn/master/2021-04/io.png)

同步IO中用户程序还是会在内核缓冲区copy到用户缓存区的时候阻塞等待，而异步IO在 **内核数据准备+数据从内核态拷贝到用户态**这两个过程都不用等待，发起`aio read`就立即返回，用户不需要等待也不需要轮询，操作系统会在后台进行数据准备，数据准备完毕之后会将数据从内核态copy到用户数据区域，这一系列动作都完成之后就会通知用户程序，用户程序收到通知之后数据就已经在自己的用户空间了

[![img](https://raw.githubusercontent.com/biningo/cdn/master/2021-04/AIO.png)](https://raw.githubusercontent.com/biningo/cdn/master/2021-04/AIO.png)

​    

## 参考

[一口气搞懂「文件系统」，就靠这 25 张图了](https://mp.weixin.qq.com/s/qJdoXTv_XS_4ts9YuzMNIw)

[彻底理解 IO多路复用](https://juejin.cn/post/6844904200141438984)

[协程和IO多路复用更配哦~](https://www.bilibili.com/video/BV1a5411b7aZ?from=search&seid=16788754659546307329)