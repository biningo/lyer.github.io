---
title: lsof查看打开文件
date: 2021-09-11
categories: [Linux]
tags: [Linux]
draft: true
---

## lsof命令

用于查看当前系统打开的文件，Linux下一切皆文件，比如网络socket这些都是使用文件进行抽象的，所以此工具也可以用来查看网络情况

​    

## lsof基本用法

指定文件查看是哪个进程打开了

```bash
lsof /temp
```

`-i`参数查看指定条件的打开文件，只要包含的都会列出来

```bash
lsof -i :7890 #查看指定端口
lsof -i TCP #查看指定网络协议
lsof -i TCP:7890 #TCP:7890
```

`-c` 列出指定进程所打开的文件

```bash
lsof -c redis
```

`-p` 指定进程号

```bash
lsof -p 32321
```

`-u` 指定用户

```bash
lsof -u root
```

其他

```bash
-g #列出 group id
```

​      

## 字段解释

| output  | desc                                             |
| ------- | ------------------------------------------------ |
| COMMAND | 进程名字                                         |
| FD      | 文件描述符(一个整数)  有三大模式 `u`读写   `r w` |
| TYPE    | 文件类型                                         |
| DEVICE  | 磁盘名字                                         |
| SIZE    | 文件大小                                         |
| NAME    | 打开文件的确切名字                               |

​    
