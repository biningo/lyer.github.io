---
title: Linux内存分析
date: 2021-08-19
categories: [Linux]
tags: [Linux]
draft: true
---

## 常用命令

- free



## free

机器整体内存的统计

```bash
free -w
free -m #以M为单位
free -h #打印符合人类阅读的单位
```

各个字段说明

| 字段      | 说明                                                         |
| --------- | ------------------------------------------------------------ |
| total     | 总内存                                                       |
| used      | 已被使用的内存                                               |
| free      | 闲置的内存                                                   |
| shared    | 多个进程共享内存的总额                                       |
| buffers   | 内核缓冲区使用的内存,准备要刷入磁盘的内存缓存数据            |
| cache     | 页缓存和`slabs`使用的内存，是从磁盘读取的数据的缓存          |
| available | 新应用可使用内存的估计，估计了未使用和缓存内存，不过有些缓存不一定能够释放 |

> `buff/cache` 占比很大的原因是Linux对内存的设计决定的，Linux 的想法是内存闲置也没啥作用，还不如拿出来做系统缓存和缓冲区提高数据读写的速率，但是当系统内存不足时，buff/cache 会让出部分来，一个灵活的操作

​    



## 巨人肩膀

[一文掌握 Linux 性能分析之内存篇](https://ctimbai.github.io/2018/05/18/tech/perf/%E4%B8%80%E6%96%87%E6%8E%8C%E6%8F%A1Linux%E6%80%A7%E8%83%BD%E5%88%86%E6%9E%90%E4%B9%8B%E5%86%85%E5%AD%98%E7%AF%87/)

[free内存使用查询](https://zj-linux-guide.readthedocs.io/zh_CN/latest/tool-use/[free]内存使用查询/#free)

[Linux性能调优之 内存](https://lework.github.io/2019/10/22/mem)

