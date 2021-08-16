---
title: 微店SRE一面
date: 2021-08-12
categories: [面试]
tags: [面试]
draft: true
---

## 微店SRE一面

> 说一下TCP三次握手，为什么TCP三次握手是三次?

> TCP的`CLOSE_WAIT`状态在什么情况下会出现?

> go包引入其他包，然后`init`函数的执行情况

会按import的顺序进行执行init函数，同时比如我main包导入A包，A包又导入B包，B包又导入C包。每个包的init执行之前都会先执行import包的init，以此递归，比如上面的那个执行顺序就是 `C->B->A->main`

> go的数组和slice的不同？

> slice动态扩容的机制 *

> go的map是线程安全的吗？ go里面的map为什么不是线程安全的？go中map并发读写引发panic的原因， map的底层原理，sync.Map

> 你日常使用了哪些go的标准库

> go控制并发数如何处理

- channel里面预先放100个令牌，一个goroutine来取执行完毕之后再放回去
- 线程池，我们最多开启100个并发goroutinue，然后所有goroutine去同一个channel里面去(外部请求/外部数据)来响应请求，这样还需要一个接受请求并且把请求放入channel的goroutine

> 如何理解DevOps

> 对运维有什么理解

> 对SRE有什么理解

 面试官回答:

- 机器容器的扩缩容
- 稳定性的保障
- 程序成本、效率的计算和节约

> SRE和传统运维之间的区别

> 看你博客写一些docker相关的，你大概说一下docker相关的底层原理

https://mp.weixin.qq.com/s/d4VHKyGiCIHwrau74B4tPw

https://zhuanlan.zhihu.com/p/348438804

> Linux的cgroups底层是如何实现的

https://zhuanlan.zhihu.com/p/348438804

> k8s了解吗？ 说一下k8s有哪些组件，创建pod的大致流程

> 对未来的规划是什么，个人比较感兴趣的方向是什么

> 反问环节

- 对实习生或则应届生的培养方案

- 对于go版本升级的处理

