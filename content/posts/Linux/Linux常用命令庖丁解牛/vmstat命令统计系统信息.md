---
title: vmstat命令统计系统信息
date: 2021-08-27
categories: [Linux]
tags: [Linux]
draft: true 
---

## vmstat命令

`vmstat` 全称 Virtual Meomory Statistics (虚拟内存统计)，是实时的系统监控工具

vmstat可以查看 内存、CPU、IO、系统运行信息等一些实时的数据，可以指定时间间隔来监控

​    

## vmstat用法

```bash
vmstat -w #以友好的格式打印
vmstat -t #附带打印时间戳
vmstat 1 #每1s采集一次 一直采集
vmstat 1 10 #每1s采集一次 一共采集10次
```

```bash
vmstat -f #查看系统启动到现在fork的进程数
vmstat -s #打印系统事件
vmstat -d #查看磁盘使用信息统计
vmstat -D #磁盘整体统计
```

​    

## 统计信息详解

### procs

| 列   | 描述               |
| ---- | ------------------ |
| `r`  | 等待运行的进程数   |
| `b`  | 休眠状态下的进程数 |

### memory

| 列       | 描述                 |
| -------- | -------------------- |
| `swpd`   | 使用的虚拟内存量     |
| `free`   | 空闲内存量           |
| `buff`   | 用作缓冲区的内存量   |
| `cache`  | 用作高速缓存的内存量 |
| `inact`  | 非活动内存的数量     |
| `active` | 活动内存量           |

### swap

