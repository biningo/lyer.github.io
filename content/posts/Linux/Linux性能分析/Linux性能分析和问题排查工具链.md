---
title: Linux性能分析和问题排查工具链
date: 2021-08-19
categories: [Linux]
tags: [Linux]
draft: true  
---

## 网络

- xdp
- tcpdump
- netstat/ss

## CPU

- mpstat

- perf `sudo apt-get install linux-tools-common`
- [perf-tools](https://github.com/brendangregg/perf-tools)

## 内存

- gdb
- valgrind
- free
- memstat
- pmap
- top/ps 查看进程内存

## IO

- bcc
- bpftrace
- iostat
- iotop
- pidstat

## 进程

- ps
- pstree
- strace跟踪进程的系统调用
- ltrace跟踪

## 其他

- vmstat
- strace
- kprobe
- systemtap
- ulimit
- top
- pidstat

`uname` 打印操作系统的一些基本信息

```bash
uname -a
uname -r #查看linux内核版本
uname -v #操作系统版本
```

`uptime` 查看系统运行时间、用户数、整体CPU负载

```bash
uptime
```

