---
title: DNS协议
date: 2021-02-26
categories: [网络]
tags: [网络协议]
---

## DNS作用

- **解析域名为IP** (网络上只能通过IP通讯)
- **负载均衡** (一个域名绑定多个IP,采用轮询方式访问多个IP)
- **灵活配置IP** (当要更换IP时直接换IP,依旧不影响原来域名的访问)

​     

## DNS记录类型

下面列出常见的记录类型

| 类型  | 解释                                                         |
| ----- | ------------------------------------------------------------ |
| A     | IP地址记录`Address` 记录域名对应的IP                         |
| AAAA  | IPV6的地址记录                                               |
| NS    | DNS服务器记录（Name Server），记录上一级DNS服务器地址，该记录只能设置为域名，不能设置为IP地址 |
| CNAME | 规范名称记录（Canonical Name），返回另一个域名，即当前查询的域名是另一个域名的跳转 |
| SRV   | 用于服务发现和负载均衡                                       |

​      

## DNS查询过程

![](https://raw.githubusercontent.com/biningo/cdn/master/img/image-20200615110750393.png)

![](https://raw.githubusercontent.com/biningo/cdn/master/img/image-20200614203436848.png)

​    

本地接受到DNS查询返回的报文之后就会调用操作系统的DNS报文解析程序`glibc/musl`来解析报文获取IP地址，其配置文件为`/etc/resolv.conf`

本地还有`/etc/hosts`文件也可以记录IP和域名的映射关系，查询DNS服务器之前会先查询这个文件以及本地DNS缓存

​    

## dig命令

该命令是dns查询工具，类似的工具还有`nslookup`

```bash
dig cname www.baidu.com #查询www.baidu.com的CNAME跳转到那个URL
dig ns baidu.com #查询哪个DNS服务器记录了这个域名解析地址
dig baidu.com #查询baidu.com的IP地址(A记录)
dig +shore baidu.com #返回简短信息
dig +trace +additional baidu.com #返回复杂的追踪信息
dig @8.8.8.8 baidu.com #指定DNS服务器去查询
dig @8.8.8.8 -p 53 baidu.com #还可以指定查询服务器的端口 默认是53
dig +x 111.111.111.111 #反向解析
```

​    

## DNS协议和报文格式

DNS协议使用`53`端口，由于广域网中不适合传输过大的 UDP 数据包，因此规定当报文长度超过了 `512` 字节时，应转换为使用 TCP 协议进行数据传输

可能会出现如下的两种情况：

- 客户端认为 UDP 响应包长度可能超过 512 字节，主动使用 TCP 协议
- 客户端使用 UDP 协议发送 DNS 请求，服务端发现响应报文超过了 512 字节，在截断的 UDP 响应报文中将 `TC` 设置为 1 ，以通知客户端该报文已经被截断，客户端收到之后再发起一次 TCP 请求

DNS的报文格式:

![](https://raw.githubusercontent.com/biningo/cdn/master/img/dns.png)

由 `12 `字节长的首部和 4 个其他字段组成

首部字段:

![](https://raw.githubusercontent.com/biningo/cdn/master/img1/dns-header.png)

响应报文:

![](https://raw.githubusercontent.com/biningo/cdn/master/img1/dns-reply.png)

​    

## 参考

https://www.ruanyifeng.com/blog/2016/06/dns.html

https://itbilu.com/other/relate/EyxzdVl3.html

https://draveness.me/dns-coredns

https://draveness.me/whys-the-design-dns-udp-tcp

http://jaminzhang.github.io/dns/DNS-Message-Format (DNS数据包解析)

https://www.cnblogs.com/dongkuo/p/6714071.html (DNS服务器实现)

https://www.cnblogs.com/kirito-c/p/12076274.html (CoreDNS和k8s)