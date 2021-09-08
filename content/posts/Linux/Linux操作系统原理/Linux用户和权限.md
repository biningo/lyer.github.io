---
title: Linux权限和用户
date: 2021-06-03
categories: [Linux]
tags: [Linux,权限]
---

## 用户和用户组

一个用户必须属于某个组，创建新用户的时候如果没有指定用户组则会创建一个同名的默认用户组

用户组和用户的关系是多对多的关系，一个组里可以有多个用户，同理一个用户也可以属于多个用户组，但是每个用户都会有一个默认组，而其他组则是附加组在`/etc/passwd`文件中记录的用户组就是用户所属的默认组，而在`/etc/group`中会记录每个用户组里面的其它用户成员列表(用户组的主成员不记录在此)

`/etc/passwd`记录的是用户相关的一些信息，每个用户占用一行

```bash
用户名:口令(都为x):用户ID:用户组ID:用户注释:用户家目录:shell解释器
testuser:x:1001:1001:TmpUser:/home/testuser:/bin/sh
```

`/etc/group`记录的是用户组相关的信息，每个用户组占一行

```bash
组名:口令(都为x):用户组ID:组内用户列表
audio:x:29:pulse,pb #主用户不会记录在这里
```

`/etc/shadow` 记录的是用户加密之后的密码信息

`/etc/default/useradd` 新用户默认设置的配置文件

```bash
SHELL=/bin/sh #新用户的默认shell
EXPIRE=2020/06/08 #新用户的默认过期时间 不配置则永久
....
```

​      

## 用户和用户组相关的Linux命令

> 我们直接修改配置文件也是可以的，但是Linux也为我们提供了一些命令，本质上都是修改底层的配置文件

新建/删除/修改用户

```bash
useradd/userdel/usermod
```

用户组增删改

```bash
groupadd/groupdel/groupmod
```

修改用户密码

```bash
passwd
```

为用户添加到指定的组中 (直接修改`/etc/group`命令也可)

```bash
#添加用户到指定的用户组中 不加-a则表示替换
#这里需要注意: 用户的默认组是不会被修改掉的 这个操作只能修改附加组
usermod -a -G g1,g2,g3
```

查看用户信息

```bash
id/who/users/whoami/last/lastb/groups
```

​         

## sudo命令和sudoers配置

`sudo`的作用是一个用户借用其它用户的权限来执行相关命令，需要配置`/etc/sudoers`文件，Linux会判断是否符合sudo文件的配置才会决定是否能够借权

```bash
用户名 主机=(用户:用户组) NOPASSWD:命令列表
#pb用户可以在 所有主机登入 执行 所有用户:所有组 的所有命令 并且不需要输入密码
pb ALL=(ALL:ALL) NOPASSWD:ALL
#表示admin组内的用户规则
%admin ALL=(ALL:ALL) ALL
```

下面再来看几个案例

```bash
#相当于(root:root)
peter ALL=(root) NOPASSWD: ALL
lucy ALL=(ALL) chown,chmod,useradd
papi ALL=/usr/sbin/*,/sbin/*,!/usr/sbin/fdisk
```

​    

## 用户/用户组和文件权限

![](https://raw.githubusercontent.com/biningo/cdn/master/2021-04/linux-file1.png)

| 权限         | 文件             | 目录                         |
| ------------ | ---------------- | ---------------------------- |
| `r` 读权限   | 可以读取文件内容 | 可以列出目录下一级层的文件   |
| `w` 写权限   | 可以修改文件内容 | 可以在目录下删除或则创建文件 |
| `x` 执行权限 | 可以执行文件     | 可以`cd`进入目录             |

<img src="https://raw.githubusercontent.com/biningo/cdn/master/2021-04/linux-file2.png" style="zoom:50%;" />

```bash
#增加权限
chmod u+x
chmod a+x #a= u+g+oaa

#减少权限
chmod a-x

#数字修改权限
chmod 753

#直接赋予权限
chmod u=rwx,g=xw

#递归修改权限
chmod -R a+x
```

新创建的文件的默认权限是需要`umask`掩码进行计算的

```bash
umask #002
umask -S #以字符方式显示  u=rwx,g=rwx,o=rx

#默认权限计算公式
666-
002
-----
664  rw-rw-r--
```

​    

## 参考

[Linux用户和权限管理看了你就会用啦](https://juejin.cn/post/6844903619305668615)

[sudo使用](https://www.cnblogs.com/jing99/p/9323080.html)