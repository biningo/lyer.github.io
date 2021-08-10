---
title: Linux常用命令总结
date: 2021-02-23
categories: [Linux]
tags: [Linux]
draft: true        
---

## 用户和用户组管理

### 1、用户

用户添加

```bash
useradd abc
useradd abc -g g1 #指定组
useradd abc -G g1,g2 #指定多个组
```

用户删除

```bash
userdel abc #删除用户 保留家目录
userdel -r abc #删除用户 同时删除家目录
```

修改用户

```bash
#第一次创建用户必须设置密码  只有root才可以设置密码
passwd abc #不加用户名就是修改当前用户密码
passwd -l abc #锁定用户
passwd -u abc #解锁
passwd -d abc #设置无密码登入
passwd -f abc #强迫下次登入需要重新设置密码

#usermod和useradd用法一样
usermod -g g2 abc #指定abc的用户组为g2
usermod -aG docker $USER #将当前用户加入 docker组
```

用户登入注销

```bash
login abc
logout abc #exit也可
```

其他

```bash
id sys #查看用户的 group和uid等 不知道用户名则是当前用户
```

### 2、用户组

用户组的添加删除

```bash
groupadd g1
groupdel g1
```

group的修改

```bash
groupmod -n g2 g1 #将g1名字修改为g2
groupmod -g 111 lyer #将lyer组的gid修改为111
```

如果一个用户创建时没有指定组则会默认创建一个和用户名字一模一样的组

### 3、相关文件

`/etc/passwd` 记录用户账户信息 uid、gid、用户名、主目录...

```bash
#用户名:x占位符表示密码:uid:gid:家目录:用的shell
lyer:x:1000:1000::/home/lyer:/bin/bash
```

`/etc/shadow` 包含用户密码信息

```bash
#用户名:加密之后的密码:....一些标识信息
```

`/etc/group`  记录用户组，创建用户的时候linux给每个用户初始化一个组，gid和uid一样

```bash
#组名:x占位符表示密码:gid:组内成员uid(被隐藏,不可见)
lyer:x:1000:
```

### 4、常见问题

> 添加用户不会创建家目录

编辑`/etc/login.defs` 设置 `CREATE_HOME yes`
