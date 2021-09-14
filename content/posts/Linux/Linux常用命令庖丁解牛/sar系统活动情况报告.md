---
title: sar系统活动情况报道
date: 2021-09-10
categories: [Linux]
tags: [Linux]
---

## sar命令

sar是系统运行状态统计信息的工具，指定时间取样然后报告系统情况

使用sar命令之前需要开启`sysstat`允许收集信息的配置

```bash
vim /etc/default/sysstat
```

​     

## 查看CPU信息

`-u` 查看CPU利用率

```bash
sar 1 5 #默认就是-u
sar -u 1 5 # 1s展示一次 一共5次
sar -u 1 5 -P 0 #指定CPU核
```

`-q` 查看CPU负载

```bash
sar -q 1 5
```

`-I` 查看CPU中断情况

```bash
sar -I ALL
sar -I SUM
```

`-w` 查看CPU上下文切换情况

```bash
sar -w 1 5
```

​    

## 查看内存信息

`-r` 查看内存利用率

```bash
sar -r 1 5
```

`-S` 查看swap区大小  `-W` 查看交换区使用情况

```bash
sar -W
sar -S
```

`-B` 查看内存页情况

```bash
sar -B
```

​    

## 查看I/O信息

`-b` 查看IO读写情况

```bash
sar -b
```

`-d`  查看物理设备的读写情况

```bash
sar -d 
```

​    

## 查看网络信息

`-n` 查看网络IO情况

```bash
sar -n DEV #查看各个网卡的读写情况
```

-n可以指定很多参数，比如DEV、TCP等

​    

## 使用ksar可视化

首先需要讲sar的统计信息文件导出，文件在`/var/log/sysstat/sa11` ，一般sa名字后面跟的是这个月的号

```bash
sar -A -f /var/log/sysstat/sa11 > ./sar-monitor
```

然后就可以启动ksar选择导入此文件即可可视化展示

​    

## 巨人肩膀

[和sar比起来，其他Linux命令都是猹](https://juejin.cn/post/6916300737194491912)

