---
title: Linux的Cgroups机制
date: 2021-08-12
categories: [Docker和容器]
tags: [docker,容器]
draft: true
---

## Cgroups概述

Cgroups全名 `Linux Control Group` ，作用就是限制一组进程所能使用的资源，比如CPU、内存、网络带宽等

Linux使用文件系统给用户暴露出来Cgroups的操作接口，路径如下

```bash
/sys/fs/cgroup
```

可以使用如下命令展示cgroup类型的文件系统

```bash
mount -t cgroup
```

​     

## Cgroups限制演示

下面就演示一下cgroups的CPU限制

1. 在CPU子系统下面创建文件夹

```bash
mkdir /sys/fs/cgroup/icepan
```

2. 启动一个死循环shell脚本，此时查看top命令可以看见此脚本将CPU干到了`100%`

```bash
while true;do : ;done &
[1] 148869
```

3. 然后在cgroups下输入限制条件

```bash
cat cpu.cfs_period_us #默认 100000
echo 20000 > cpu.cfs_quota_us
```

> `cpu.cfs_period_us` 是运行周期，`cpu.cfs_quota_us` 是在周期内这些进程占用多少时间，上面的意思就是在100000us时间内进程最多只能占用20000us (20%)

4. 再次使用top命令查看死循环进程发现CPU被限制到了20%

​      

## Cgroups相关概念

### 1、子系统

 Cgroups为每种资源限制类型都定义了一个子系统，比如CPU子系统、memory内存子系统等，内核也在更细化的拆分这些可以做出限制的子系统

子系统的限制需要配合内核模块进行，比如CPU子系统需要和Linux进程调度配合，Linux会读取CPU子系统的配置参数再结合进程调度进行限制进程在CPU上的占用时间。memory内存子系统则需要和Linux内存分配模块配合，当达到memory子系统配置的最大限制值的时候就需要kill进程

### 2、Hierarchy层级结构

cgroup可以是一个树型结构，比如我在cpu子系统下创建了A目录并且对CPU做了20%的限制，那么我在A目录下还可以继续创建目录，此目录也是一个cgroup并且继承了父亲的限制参数，如果子cgroup需要重写的话则不能超过过父亲

### 3、Control Group

就是子系统下的所有目录，表现出来就是目录下面会带有对应的配置文件

### 4、Tasks

就是被限制的进程

​    

## 巨人肩膀

[DOCKER基础技术：LINUX CGROUP](https://coolshell.cn/articles/17049.html)

[Linux资源管理之cgroups简介](https://tech.meituan.com/2015/03/31/cgroups.html)

