---
title: mpstat命令统计CPU信息
date: 2021-08-19
categories: [Linux]
tags: [Linux]
draft: true 
---

## mpstat概述

`mpstat`  全称Multiprocessor Statistics，该命令用于报告CPU一些统计信息

​    

## mpstat用法

```bash
mpstat #查看所有CPU整体的统计信息
mpstat 2 10 # 2s统计一次 一共统计10次
mpstat -P 0 #查第0个CPU的统计信息
mpstat -P 0-3 #查看[0,3]号CPU
mpstat -P 0,3 #查看 0和3 CPU的信息
mpstat -P 1 2 10 #统计1号CPU 2s统计一次 一共统计10次
```

其他高级用法参见man page

​    

## 统计参数

展示出来的统计参数如果不加上`internal`的话则是系统开机到现在的平均值，如果加上`internal` 第一行的信息自系统启动以来的平均信息，从第二行开始，输出为前一个 interval 时间段的平均信息

| 参数    | 释义                                                         |
| :------ | :----------------------------------------------------------- |
| CPU     | 处理器 ID                                                    |
| %usr    | 在 internal 时间段里，用户态的 CPU 时间%，不包含 nice 值为负进程 |
| %nice   | 在 internal 时间段里，nice 值为负进程的 CPU 时间（%）        |
| %sys    | 在 internal 时间段里，内核占用时间（%）                      |
| %iowait | 在 internal 时间段里，CPU花费多少硬盘 IO 等待时间（%）       |
| %irq    | 在 internal 时间段里，硬中断时间（%）                        |
| %soft   | 在 internal 时间段里，软中断时间（%）                        |
| %steal  | 显示虚拟机管理器在服务另一个虚拟处理器时虚拟 CPU 处在非自愿等待下花费时间的百分比 |
| %guest  | 显示运行虚拟处理器时 CPU 花费时间的百分比                    |
| %gnice  |                                                              |
| %idle   | 在 internal 时间段里，CPU 除去等待磁盘 IO 操作外的因为任何原因而空闲的时间闲置时间（%） |

​    

## 巨人肩膀

[mpstat 使用介绍和输出参数详解](https://wsgzao.github.io/post/mpstat/)

