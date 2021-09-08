---
title: 陌陌SRE二面
date: 2021-08-26
categories: [面试]
tags: [面试]
draft: true
---

> 未来的3~5的规划是什么

> 看你实习是微服务相关的，为什么会考虑做运维和SRE呢

> 用了多久的Linux，是什么发行版，用的习惯吗

> 正则表达式: 匹配行首 行位 空行^$ 匹配数字(\d [0-9])，复杂的正则有写过吗

> find查找 指定目录下的普通文件大于1M的普通文件 以pdf结尾的文件

Linux下的*号扩展 要加""

> HTTP请求报文手写

> 504状态码，其他状态吗

> 有排查过网络问题吗? 查DNS命令(有哪些?)，DNS查询过程

dig命令

nslookup

> DNS记录类型有哪些

A CNAME TXT

> Nginx的location的优先级和路径匹配

> 进程和线程的区别

> Python : 计算密集型和IO密集型以及GIL锁

> 你遇到最困难的问题

跨域

## eagle项目

改进的点:

- eagle挂了如何知道
- 遍历容器的消耗

- 提供一个web UI管理界面

