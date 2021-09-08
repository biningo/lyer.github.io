---
title: Linux静态路由表
date: 2021-08-17
categories: [Linux]
tags: [Linux,网络]
---

## 查看路由表

```bash
route -n
netstat -rn
```

![](https://raw.githubusercontent.com/biningo/cdn/master/2021-04/route-table.png)

Destination是`0.0.0.0`表示是默认路由，只有其他路由都匹配不到时才会走这个路由

Gateway为`0.0.0.0`表示没有网关，静态路由表中的网关只能配置一个，配置多个则只会使用其中一个

​    

## 路由转发流程

- 工具目标IP个子网掩码匹配静态路由表中的条目
- 如果匹配到了则将数据包发送到路由表项对应的网卡上
- 如果路由表项都匹配不成功则走默认路由，也就是Destinaion为`0.0.0.0`的路由表项，因为默认路由配置了网关所以数据包会从网关发送出去，而在局域网内的通讯则不会走网关

比如我现在要访问`172.17.0.10`，则会匹配到docker0这个虚拟网桥，然后将数据包转发到docker0中由docker0进行转发

如果我现在要访问`192.168.0.10`，则会匹配到`192.168.0.0`这个条目走wlp0s20f3网络接口，这个网络接口是wifi，则会直接发送数据包到wifi猫中，因为是局域网地址所以不会通过网关转发到外网，直接转发到局域网内的目标主机中

如果我现在要访问`121.196.169.248`，则全部匹配不到会走默认路由，直接转发到网关上然后再转发到外网

​    

## route命令

```bash
route -n
route add default gw 10.0.0.1 dev eth0 #指定默认路由 并且设置网关和出口设备
route add -net 10.2.0.0/24 dev eth1 #指定其它路由
route del -net 10.0.0.0/24 #删除路由
```

​    

## 参考

[掌握Linux路由这一篇就够了！](https://zhuanlan.zhihu.com/p/61805945)