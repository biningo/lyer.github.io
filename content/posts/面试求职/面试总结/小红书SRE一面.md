---
title: 小红书SRE一面
date: 2021-08-16
categories: [面试]
tags: [面试]
draft: true
---

> 聊项目

> 你的项目是如何部署的

> 给你两台机器实现你的程序高可用

> 如何保障nginx的高可用

> nginx四层负载均衡和七层负载均衡，nginx七层有了解吗 【不会】

> 你知道jwt的实现方式是什么？ jwt用于防范什么攻击?  【不会】

- CSRF
- XSS

> 讲一下lambda的背景和目标

> 找到一百万随机大小的中位数 [295. 数据流的中位数](https://leetcode-cn.com/problems/find-median-from-data-stream/)

> 一个大日志，如何排查昨天的问题 (查看大日志的方法)

日志切割https://www.cnblogs.com/fengzhilaoling/p/12269885.html

https://blog.csdn.net/weixin_45312167/article/details/93626313

```bash
sed -n '/2019-06-25 16:00:00/,/2019-06-25 16:10:00/'p info.log > 20190625160000.log
```



> 十万行日志，里面有IP，如何实现unique和sort的功能

计数排序

> 查找文件夹下创建3天以内的文件

https://blog.csdn.net/whatday/article/details/109253401

find命令

