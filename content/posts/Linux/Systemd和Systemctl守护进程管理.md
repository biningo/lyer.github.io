---
title: Systemd和Systemctl守护进程管理
date: 2021-07-29
categories: [Linux]
tags: [Linux]
draft: true
---

## Systemd系统管理相关命令

`systemctl`  systemd的主命令，用于管理守护进程和系统进程

```bash
systemctl reboot #重启
systemctl status docker
systemctl start docker
```

`systemd-analyze` 查看进程启动耗时

```bash
systemd-analyze #查看系统启动耗时
systemd-analyze blame #查看每个服务启动耗时
```

`hostnamectl` 查看主机信息

```bash
hostnamectl
hostnamectl set-hostname lyer #设置主机名字
```

`localectl` 查看本地化设置

```bash
localectl
```

`timedatectl` 查看时区设置

```bash
timedatectl
timedatectl list-timezones #显示所有可用时区
```

`loginctl` 查看当前登入用户

```bash
loginctl
loginctl list-sessions 
loginctl list-users #列出所有登入用户
loginctl show-user lyer #列出指定用户的信息
```

​    

## systemctl管理守护进程

列出所有unit配置文件

```bash
systemctl list-unit-files
systemctl list-unit-files --type=service #指定type
```

​     

## systemd配置文件

`/usr/lib/systemd/system/`



## 巨人肩膀

[Systemd 入门教程：命令篇](https://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html)

