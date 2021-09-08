---
title: iptables总结
date: 2021-08-17
categories: [Linux]
tags: [Linux,iptables]
---

## iptables和netfilter

`netfilter` 是 Linux 内置的一种防火墙机制，用于过滤网络数据包，而`iptables`则是一个用于配置netfilter防火墙的命令行工具

​    

## iptables命令格式

![](https://raw.githubusercontent.com/biningo/cdn/master/2021-04/iptables-command.png)

`iptables`命令格式

```bash
iptables [-t 表名] 管理选项 [链名] [匹配条件] [-j 控制类型]
```

- 不加`-t`指定表名则默认操作的是`filter`表
- 不加指定的链名则会展示表的所有的链规则

​    

## 四表和五链和数据流向

> **注意，在OUTPUT之后会通过route表进行路由选择，选择出口的网卡**

![](https://raw.githubusercontent.com/biningo/cdn/master/2021-04/iptable-link.png)

iptables匹配规则如下，按规则的排列顺序逐一匹配，只要匹配到了就不会再继续往下匹配，就传递给下一个链了，所以我们需要将优先级更高的规则防止在前面

​    

## 查看iptables

查看主要是使用`-L`命令，表示`list`的意思

查看`nat`表的OUTPUT链规则

```bash
iptables -t nat -nvL OUTPUT
```

- `n`不解析域名，直接显示IP
- `v`显示详细信息
- `L` 表示`list`展示的意思

> 一般都会加上`-nvL`来查看iptables规则

查看filter表的OUTPUT规则

```bash
iptables -nvL OUTPUT
```

​    

## 编写iptables规则

`-A`命令添加iptables规则

`-j`后面加控制类型

filter主要有如下控制类型

| 控制类型 | 解释                                                    |
| -------- | ------------------------------------------------------- |
| ACCEPT   | 允许传递数据包                                          |
| DROP     | 丢弃数据包，不响应                                      |
| REJECT   | 拒绝数据包，响应RST                                     |
| LOG      | 在`/var/log/messages`文件中记录日志信息，然后再继续传递 |

```bash
iptables -A INPUT -j DROP #丢弃所有数据包
iptables -A INPUT -j ACCEPT #接受所有数据包
```

`-I`表示插入iptables规则，插入到指定的行，原来的行后移

```bash
iptables -I INPUT 2 -s 10.10.10.1 -j ACCEPT #允许10.10.10.1主机访问本机
```

`-R`替换iptables规则

```bash
iptables -R INPUT 2 -s 194.168.1.5 -j ACCEPT #替换第2行
```

`-D`表示删除规则，后面更上规则的行号

```bash
iptables -nvL INPUT --line-number
iptables -D INPUT 3 #删除第3行
```

`-P`设置链的默认规则，只能设置`ACCEPT`和`DROP` ，都匹配不上时就会按此默认规则进行处理

```bash
iptables -P INPUT ACCEPT #默认全部允许通过
```

`-F`清空某个链的全部规则Flush

```bash
iptables -F INPUT #清空filter表上INPUT规则
iptables -F #清空filter表中所有链上的规则
```

​    

## 规则匹配条件

默认不加条件则表示无条件，匹配所有

```bash
iptables -A INPUT  -j  DROP #丢弃所有包
```

我们可以让iptables只对特定的数据包应用规则，只需要在添加规则后面加上条件即可，比如加上数据包的目的端口、源端口、目的IP、源IP等条件

```bash
-p 协议名 tcp udp icmp
-i 进入的网络接口 只能用于入端链
-o 流出的网络接口 只能用户出端链
-s 源IP、域名、网段 不配置则表示匹配任何地址
-d 目标IP、域名、网段 不配置则表示匹配任何地址
--sport 源端口、端口范围
--dport 目的端口、端口范围
--sport 23
--sport 2000:3000 #2000~3000
--sport :2000 # 0~2000
-m multiport --dport 80,443,110 #端口列表
-m iprange --src-range 192.168.8.100-192.168.8.123 #ip范围匹配
-m iprange --dst-range 192.168.8.100-192.168.8.123
```

> `--sport` 和 `--dport` 必须配合 `-p` 参数使用

```bash
iptables -A INPUT -s 192.10.0.0/24 -p tcp -i eth0  -j DROP
iptables -A INPUT -s 192.10.0.0/24 -p tcp ！ -i eth0  -j DROP #!反向匹配
iptables -A OUTPUT -s 192.10.0.0/24 -p tcp -o eth0 -j DROP
iptables -R INPUT 1 -s 10.0.0.0/24 -p tcp  --dport 2000:3000 -j DROP
```

​    

## NAT转化

nat表主要有如下控制类型

| 控制类型   | 解释                                                         |
| ---------- | ------------------------------------------------------------ |
| DNAT       | 目的IP转化，比如容器的端口映射可通过此来实现                 |
| SNAT       | 源IP转化，比如容器需要访问外网则需要修改源IP为出口网卡的IP，数据包回来之后内核会自动转化回来 |
| MASQUERADE | 动态SNAT，适用于动态IP，网卡IP会变化的场景。一般需要SNAT的场景都使用这个 |
| REDIRECT   | 端口转发                                                     |

### SNAT

SNAT全称   `Source Network Address Translation` 源地址转换

作用就是将数据包的源IP修改，可以在iptables的`nat`表的`OUTPUT`、`POSTROUTING`链中添加SNAT规则进行转化

- 内网服务访问外网，容器访问外网

```bash
iptables -t nat -nvL #查看nat表链条规则
sudo iptables -t nat -A POSTROUTING -s 172.20.0.0/24 -o wlp0s20f3 -j SNAT --to-source 192.168.0.23

iptables -t nat -A POSTROUTING -s 172.20.0.0/24 -o wlp0s20f3 -j MASQUERADE #动态IP SNAT，不需要写死IP，会自动SNAT出口网卡的IP
```

### DNAT

DNAT全称为 `Destination Network Address Translation`  目标地址转换

作用就是修改目标IP，可以在nat表的`INPUT`、`PREROUTING`中修改目标IP，一般是在PREROUTING中进行修改

- 外网服务访问内网服务，内网穿透

```bash
iptables -t nat -A PREROUTING -i eth0 -d 192.168.0.23 -p tcp --dport 8080 -j DNAT --to-destination 172.17.0.2:80
```

​    

## iptables规则备份和恢复

`iptables-save`

```bash
iptables-save #直接打印屏幕
iptables-save > iptables_rules.txt #持久化到文件
```

`iptables-restore`

```bash
iptables-restore < iptables.txt
```

​     

## ufw和iptables

ubuntu下iptables规则是不持久化的，直接保存在内存里

ufw是ubuntu下的防火墙操作工具，底层还是编写iptables的filter表规则

TODO

​     

## 巨人肩膀

[iptables超干货直播（110分钟），慎入！](https://www.bilibili.com/video/BV1LE411J72q)

[讲讲路由表和iptables](https://www.bilibili.com/video/BV1hJ411a7P1?from=search&seid=9137634896289872994)

[iptables 防火墙（一）| 四表/五链、数据包匹配流程、编写 iptables 规则](https://juejin.cn/post/6972296377401344031)

[iptables 防火墙（二）| SNAT / DNAT 策略及应用](https://juejin.cn/post/6972688208375070728)

[iptables 防火墙（三）| 规则的导出 / 导入、使用防火墙脚本程序](https://juejin.cn/post/6973309007062630407)

[iptables系列教程（二）| iptables语法规则](https://juejin.cn/post/6844904159192301582)

[Linux iptables命令详解](https://juejin.cn/post/6922354523826552846)

