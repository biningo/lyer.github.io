---
title: pidstat进程性能分析
date: 2021-09-04
categories: [Linux]
tags: [Linux]
draft: true 
---

## pidstat命令

主要用于监控全部或指定进程占用系统资源的情况，如CPU，内存、设备IO、任务切换、线程等

​    

## pidstat用法

pidstat第一次执行展示的是系统启动开始到现在的统计信息，如果是按指定时间间隔指定的则是展示距离上一次

```bash
pidstat #默认显示全部进程
pidstat -p 2642 #-p表示指定显示一个进程pid  默认展示-u CPU的统计信息
pidstat -p 2642 2 10  #每2s展示以一次 一共10次
pidstat -r -p 2642 #-r表示查看进程内存
pidstat -d -p 2642 #-d表示查看进程IO
```

​     

## 巨人肩膀

[pidstat 命令详解](https://commandnotfound.cn/linux/1/187/pidstat-命令)

​    



