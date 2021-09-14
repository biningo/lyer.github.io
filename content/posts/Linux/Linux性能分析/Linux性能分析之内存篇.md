---
title: Linux性能分析之内存篇
date: 2021-08-19
categories: [Linux]
tags: [Linux]
draft: true
---

## 内存相关文件系统

记录了当前内存使用的统计信息，free命令就是读取的此伪文件的内容

```bash
/proc/meminfo
```

​     

## 内存分析常用命令

系统级memory分析命令

| command | desc                                |
| ------- | ----------------------------------- |
| free    | 展示Linux操作系统的整体内存统计信息 |
| vmstat  | 万金油命令                          |

进程级memory分析命令

| command  | desc                                         |
| -------- | -------------------------------------------- |
| top/ps   | 查看进程内存                                 |
| pmap     | 展示进程的内存映射信息                       |
| pidstat  | 主要用于监控进程的参数包括 CPU，IO，内存等等 |
| valgrind | 内存分配和内存泄露分析工具                   |

​     

## free

机器整体内存的统计

```bash
free -w
free -m/g #以M/G为单位 
free -h #打印符合人类阅读的单位(g)
```

free各个字段说明

| 字段      | 说明                                                         |
| --------- | ------------------------------------------------------------ |
| total     | 总内存                                                       |
| used      | 已被使用的内存                                               |
| free      | 闲置的内存                                                   |
| shared    | 多个进程共享内存的总额                                       |
| buffers   | 用来缓存写入磁盘的数据，定期的将buffer的数据刷入磁盘 **减少写磁盘的IO** |
| cache     | 是从磁盘读取的数据的缓存   **用来反复读，减少读磁盘IO**      |
| available | 新应用可使用内存的估计，估计了未使用和缓存内存，不过有些缓存不一定能够释放 |

> `buff/cache` 占比很大的原因是Linux对内存的设计决定的，Linux 的想法是内存闲置也没啥作用，还不如拿出来做系统缓存和缓冲区提高数据读写的速率，但是当系统内存不足时，buff/cache 会让出部分来，一个灵活的操作

`total`全部内存的计算

```bash
total = used + free + buffers + cache
```

`available`可使用内存的计算

```bash
available = free+(cache+buffers)的一部分 #因为有些buffer/cache没办法拿出来使用
```

​    

## vmstat

此命令展示的信息基本和`free`命令差不多，此命令统计指定时间间隔的统计信息

```bash
vmstat 1 5 #每1s执行一次 一共执行5次
```

​     

## top

此命令可以查看机器整体的CPU、内存以及各个进程的CPU和内存等信息，还可以进行进程排序

内存相关字段说明

| 字段 | desc                              |
| ---- | --------------------------------- |
| VIRT | 进程使用的虚拟内存总量(kb)        |
| RES  | 进程实际占用的内存,一般都是看这个 |
| SHR  | 进程占用共享内存大小              |
| %MEM | 进程实际占用的物理内存百分比      |

然后在top界面按`M`快捷键可以按照内存占比排序，按`R`则逆序

​     

## ps

```bash
ps -aux --sort 
```

| 字段 | desc                                 |
| ---- | ------------------------------------ |
| %MEM | 进程使用物理内存所占百分比           |
| VSZ  | 进程使用虚拟内存大小                 |
| RSS  | 进程使用物理内存大小，重点关注这个值 |

​    

## pmap

```bash
pmap -x <pid>
```

| 字段    | desc                                                      |
| ------- | --------------------------------------------------------- |
| Address | 占用内存的文件的内存起始地址                              |
| Kbytes  | 最大可以占用内存的字节数                                  |
| RSS     | 实际占用内存大小                                          |
| Dirty   | 脏页大小                                                  |
| Mapping | 占用内存的文件，[anon] 为已分配的内存，[stack] 为程序堆栈 |

​    

## pidstat

```bash
pidstat -r -p 3261 #查看启动到现在的值
pidstat -r -p 3261 2 10 # 2s查看10次
```

​    

## valgrind

```bash
valgrind --leak-check=full --tool=memcheck ./main
```

​    

## 巨人肩膀

[一文掌握 Linux 性能分析之内存篇](https://ctimbai.github.io/2018/05/18/tech/perf/%E4%B8%80%E6%96%87%E6%8E%8C%E6%8F%A1Linux%E6%80%A7%E8%83%BD%E5%88%86%E6%9E%90%E4%B9%8B%E5%86%85%E5%AD%98%E7%AF%87/) 【不错】

[Linux之《荒岛余生》（三）内存篇](https://mp.weixin.qq.com/s?__biz=MzA4MTc4NTUxNQ==&mid=2650519204&idx=1&sn=b367c1987fb8c985e83a6cb90f5436a6&scene=21#wechat_redirect)

[free内存使用查询](https://zj-linux-guide.readthedocs.io/zh_CN/latest/tool-use/[free]内存使用查询/#free)

[Linux性能调优之 内存](https://lework.github.io/2019/10/22/mem) 【不错】

[swap的罪与罚](https://blog.huoding.com/2012/11/08/198)

