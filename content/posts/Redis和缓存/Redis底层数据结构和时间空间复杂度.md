---
title: Redis底层数据结构和时间空间复杂度
date: 2021-06-10
categories: [Redis]
tags: [Redis]
draft: true
---

## redis数据类型和底层数据结构

| 数据类型 | 底层数据结构                                                 |
| -------- | ------------------------------------------------------------ |
| string   | (object对象和SDS分配在一起)embstr、RAW(object对象和SDS分开分配)  底层都是SDS |
| list     | quicklist(链表节点是ziplist)                                 |
| set      | hashtable、intset(有序、存整数, 类型升级无法降级)            |
| sort set | skiplist、ziplist(有序、连锁更新问题)                        |
| hash     | hashtable、ziplist                                           |

​      

## embstr的优点

当字符串长度`<32byte`时会使用embstr来保存，也就是会将object的底层指针指向的sds内存和`redisObject`分配的内存分配在一块连续的空间 (也是采用C柔性指针的特性)

![](https://raw.githubusercontent.com/biningo/cdn/master/2021-04/embstr.png)

优点如下:

- 只需要一次内存分配和释放
- 连续的内存空间更有利于缓存

​      

## 字典和hash表

Hash表使用拉链法解决冲突

Hash表的负载因子 `已保存节点数量/容量`，负载因子如果`>=1`时则肯定就会发生hash碰撞

渐进式rehash，如果`rehashidx=1`则代表此字典正则rehash

-  如果集中式一次性全部rehash到新的数组如果key很多的话则会有很大的计算量，造成短暂性的阻塞而无法对外服务，所以采用渐进式的rehash
- 增加只在新表里增加，删除则会查找两个表，这样旧表只会减少而不会增加。并且在每次对key查找的时候 都会附带rehash一两个值，这样就把rebash分摊到每次操作上面





