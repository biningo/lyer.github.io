---
title: top命令统计进程信息
date: 2021-08-22
categories: [Linux]
tags: [Linux]
draft: true 
---

## top概述

`top`命令用于实时显示进程的动态

​    

## top用法

```bash
top #默认按照CPU占用率排序
top -c #显示命令路径 默认只显示命令名字
top -d 1 #修改刷新间隔1s top进入之后按s再进行修改也可
top -n 5 #更新5次之后退出
top -H #显示线程  默认只显示进程
top -p 12355 #指定某个pid的进程
```

|       |                       |
| ----- | --------------------- |
| 数字1 | 查看每个CPU的统计信息 |
| s     | 修改更新时间          |
| P     | 按%CPU排序            |
| M     | 按%MEM排序            |
| T     | 按TIME+排序           |
| R     | 逆序                  |

更详细用法查看man page

​     

## 统计信息详解

第1行: 输出结果和`uptime`命令一致

第2、3行为CPU信息

`Tasks`

```bash
total   进程总数
running   正在运行的进程数 
sleeping   睡眠的进程数 
stopped   停止的进程数 
zombie   僵尸进程数 
```

`CPU ` (全部CPU统计信息的平均值，按`1`则可以列出每个CPU的统计信息的平均值)

```bash
us   用户空间占用CPU百分比 
sy   内核空间占用CPU百分比 
ni   用户进程空间内改变过优先级的进程占用CPU百分比 
id   空闲CPU百分比 
wa   等待输入输出的CPU时间百分比 
hi   硬中断（Hardware IRQ）占用CPU的百分比
si   软中断（Software Interrupts）占用CPU的百分比
st   (Steal time) 是当 hypervisor 服务另一个虚拟处理器的时候，虚拟 CPU 等待实际 CPU 的时间的百分比
```

第4、5行为内存统计信息

`Mem`

```bash
total  物理内存总量
free   空闲内存总量
used   使用的物理内存总量
buff/cache 用作内核缓存的内存量
```

`Swap`

````bash
total  交换区总量 
free   空闲交换区总量 
used   使用的交换区总量
avail  缓冲的交换区总量
````

各个进程展示的字段含义

```bash
PID 
USER  
PR 优先级
NI nice值
VIRT 进程使用的虚拟内存总量(kb)  
RES 进程使用的未被换出的物理内存大小
SHR 共享内存大小
S  进程状态
%CPU  上次更新到现在的CPU时间占用百分比
%MEM  进程使用的物理内存百分比
TIME+ 进程使用的CPU时间总计，单位1/100秒
COMMAND 命令名/命令行
```

​     

## 进程的状态

| 状态                     | 描述                       |
| ------------------------ | -------------------------- |
| R (TASK_RUNNING)         | 可执行状态                 |
| S (TASK_INTERRUPTIBLE)   | 可中断的睡眠状态           |
| D (TASK_UNINTERRUPTIBLE) | 不可中断睡眠状态           |
| T (TASK_STOPPED)         | 暂停状态或跟踪状态         |
| Z (EXIT_ZOMBIE)          | 退出状态，进程成为僵尸进程 |

​    

## 巨人肩膀

[Linux 常用命令之 top 命令详解](https://learnku.com/articles/30384)
