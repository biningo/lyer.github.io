---
title: Vagrant总结
date: 2021-09-13
categories: [Vagrant]
tags: [Vagrant]
---

## 什么是Vagrant

Vagrant是一个配置虚拟机的工具，底层还是需要依赖虚拟机比如VirtualBox来创建虚拟机的，Vagrant只是调用了虚拟机提供的接口来快速配置虚拟机

Vagrant主要用于配置开发环境，而Docker用于在生产环境上部署

​    

## Vagrant基本使用

首先我们需要创建一个工作目录

```bash
mkdir test
```

然后需要初始化一个 `Vagrantfile` ，后面的`hashicorp/bionic64`就代表想要启动一个什么样的box，box其实就和docker镜像一样，是一种对虚拟机以及配置的打包

```bash
vagrant init hashicorp/bionic64
```

启动虚拟机

```bash
vagrant up 
```

销毁删除虚拟机

```bash
vagrant destroy
```

暂停/恢复虚拟机，实际使用此命令来暂停和启动虚拟机

```bash
vagrant suspends
vagrant resume
```

关机/启动虚拟机，如果虚拟机没创建则会自动创建

```bash
vagrant halt
vagrant up
```

修改Vagrantfile之后使之生效，相当于`halt+up`重启机器

```bash
vagrant reload
```



进入虚拟机

```bash
vagrant ssh
```

查看当前目录下虚拟机的状态，查看全局虚拟机的状态

```bash
vagrant status
vagrant global-status
```

​    

## 如何导入box

对于创建box对应的虚拟机，我们需要下载box然后才能基于box进行创建虚拟机

我们可以在Vagrantfile文件里面进设置相关的配置

```bash
config.vm.box = "ubuntu/trusty64" #回去官方的box仓库下载对应名字的box
config.vm.box = "http://example.com/virtualbox.box" #指定url
config.vm.box = "~/Downloads/virtualbox.box" #指定本地box磁盘路径
```

我们在初始化Vagrantfile的时候也可以指定box

```bash
vagrant init ubuntu/trusty64
vagrant init http://example.com/virtualbox.box
vagrant init ~/Downloads/virtualbox.box
```

​    







​    

