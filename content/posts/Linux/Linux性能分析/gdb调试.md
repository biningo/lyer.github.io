---
title: GDB调试
date: 2021-09-04
categories: [Linux]
tags: [Linux]
draft: true 
---

## gdb调试

运行相关

```bash
r (run) #运行
c (continue) #运行到下一个断电处或结束
n (next) #单步调试 不会进入函数
s (step) #单步调试 会进入函数
until #执行循环体直到结束后停止
j (jump) #跳转到指定的行/函数等
```

断点相关

```bash
b <n> (break n)  #在n行处设置断点
b <func> #在函数func中设置断点
disable <n> #断点暂时不可用 n是断点编号
enable <n> #恢复断点
clear <n/func> #清除断点  n是断点的行或则函数
info b #展示所有断点
delete b #清空所有断点
```

查看运行时信息

```bash
p 变量 (print) #打印变量值
p <函数>  #打印函数
p <函数>(入参) #调用函数获取返回值并打印
p x=10 #调试过程中修改x的值

display <变量> #让其在每次调试时都展示变量
wahere/bt #堆栈列表
info locals #打印堆栈所有变量的值
whatis <变量/函数> #查询变量或函数的类型
info func <函数名> #查看一个函数
watch <变量> #监视一个值,发生变化则报告
x <变量> #查看在内存中的地址
```

窗口

```bash
layout src #显示源代码窗口
layout asm #显示反汇编窗口
layout regs #显示源代码/反汇编和CPU寄存器窗口
layout split #显示源代码和反汇编窗口
```

​      

## cgdb

一个基于gdb的更方便的调试工具

​        

## 巨人肩膀

[gdb 调试利器](https://linuxtools-rst.readthedocs.io/zh_CN/latest/tool/gdb.html)

[GDB 调试指南](https://juejin.cn/post/6844903950697644040)

[这可能是你最想要的一份GDB使用指南 ](https://juejin.cn/post/6931942984376139783) 【不错】

