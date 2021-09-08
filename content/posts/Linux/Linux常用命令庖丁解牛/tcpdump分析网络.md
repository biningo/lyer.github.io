---
title: tcpdump命令分析网络
date: 2021-08-23
categories: [Linux]
tags: [Linux]
---

## tcpdump概述

tcpdump是Linux下的网络抓包工具，使用 `libpcap` 库来抓取网络数据包

​    

## tcpdump用法

```bash
-n 不解析主机名 -nn 不解析端口和主机名
-i 指定监听的网卡 -i any 监听所有网卡 #默认不加-i则监听默认的网卡
-v 显示更多信息 -e 显示链路层信息
-A 显示ASCII -X 显示为16进制
```

抓取特定协议的数据包

```bash
tcpdump -nn -i wlp0s20f3 udp
```

抓取特定IP的数据包，也可以指定一个网段

```bash
tcpdump -nn -i any  host 192.168.0.1
tcpdump -nn -i any  host 192.168.0.0/16
tcpdump -nn -i any  src 192.168.0.1
tcpdump -nn -i any  dst 192.168.0.1
tcpdump -nn -i any  src 192.168.0.1 and dst 192.168.0.23
```

抓取指定端口的报文

```bash
sudo tcpdump -nn -i wlp0s20f3 tcp port 8080
sudo tcpdump -nn -i wlp0s20f3 tcp dst port 8080
```

将抓取到的数据包写入文件，`-s`指定抓取每个报文的大小，0则表示抓取整个报文，不加-s默认只抓取每个报文的前96byte

```bash
tcpdump -s0 -nn -i wlp0s20f3 -w ./tmp/temp/test.pcap
```

输出携带时间戳

```bash
tcpdump -nn -tttt  -i wlp0s20f3 host 192.168.0.1
```

​      

## 过滤器

条件修饰符

```bash
or  ||
and &&
! not
```

​     

## 命令

查看可以被监听的网卡设备

```bash
tcpdump -D
```

​    

## 巨人肩膀

[超详细的网络抓包神器 tcpdump 使用指南](https://juejin.cn/post/6844904084168769549)

[https://typefo.com/linux/tcpdump-tutorial.html](https://typefo.com/linux/tcpdump-tutorial.html)

[tcpdump简明教程](https://github.com/mylxsw/growing-up/blob/master/doc/tcpdump%E7%AE%80%E6%98%8E%E6%95%99%E7%A8%8B.md)