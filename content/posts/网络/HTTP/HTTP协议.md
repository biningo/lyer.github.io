---
title: HTTP协议
date: 2021-03-08
categories: [网络]
tags: [HTTP]
---

## 请求和响应报文

- 请求报文第一行必须指定 **请求的方法、资源路径、HTTP协议版本**

- 紧接着就是请求头信息

- 然后还需要一个空行

- 紧接着如果有请求体的话就是请求体，注意如果没有请求体也必须加一个空行

```http
GET /note/ef1b6cee.html HTTP/1.1
Host: www.ru23.com
Connection: keep-alive
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (iPhone; CPU iPhone OS 13_2_3 like Mac OS X) 
....各个请求头
<空行>
```

- 响应报文第一行必须指定 **HTTP协议版本、状态码、状态码的字符串简短描述**

- 紧接着就是响应头信息，响应头中必须指定`Content-Length`，否则客户端不知道报文的结束位置(特殊情况另作考虑)

- 然后还需要一个空行，空行的目的是为了更容易解析Header和Body，因为每个报文的Header大小是不确定的

- 如果有响应体的化就需要一个响应体，没有的话空行也是必须的

```http
HTTP/1.1 200 OK
Server: nginx/1.16.1
Date: Sun, 04 Oct 2020 02:27:01 GMT
Last-Modified: Sun, 30 Aug 2020 14:12:42 GMT
Content-Type: text/html
Content-Length: 40846
Accept-Ranges: bytes

<!DOCTYPE html>
<html>
<head><meta name="generator" content="Hexo 3.8.0"></head>
    <body>
    	<div>
    	.....
    	</div>
    </body>
</html>
```

​      

## URL编码

如果在浏览器中输入的URL含有特殊字符或则非`ASCII`比如中午，浏览器则会进行转义编码操作，编码方法就是将非ASCII或则特殊字符转化为 **16进制**，然后在前面加一个`%`

比如中文的话则会根据对应的UTF8编码的二进制转化为16进制然后加一个`%`，比如:

```bash
%E9%93%B6%E6%B2%B3
```

如果我们收到一个URL编码的query参数是这样的格式，那么可以用上述方法进行解码

​      

## HTTP请求方法

**请求方法单词必须全部大写**

| 方法名字  | 含义                                                         |
| --------- | ------------------------------------------------------------ |
| `GET`     | 获取资源，可以理解为读取或者下载数据                         |
| `HEAD`    | 只获取HTTP头部信息，不返回Body                               |
| `POST`    | 上传数据到server，数据保存在Body                             |
| `DELETE`  | 删除资源                                                     |
| `OPTIONS` | 列出服务器支持那些类型的HTTP请求，比如GET、POST，由响应报文头部的`Allow`字段列出 |
| `CONNECT` | 建立特殊的连接隧道                                           |
| `PUT`     | 更新资源，整个替换                                           |
| `PATCH`   | 更新资源，部分替换                                           |

​        

## HTTP内容协商

内容协商是指客户端希望得到什么类型的数据或则希望采用什么编码，服务器需要根据客户端的请求来做出合理的响应

客户端和内容协商的请求头都以 `Accept-`开头，服务器响应内容协商的相关头部都以`Content-`开头

`Accept` 允许的响应MINE类型

 `Content-Type`服务器响应的MINE类型，一般服务器的编码会在Cotent-Type头部指定，浏览器会根据`chartset`指定的编码来进行解码，默认都是UTF-8编码，所以如果Content-Type指定的charset和传递数据的编码方式不一致的话就会产生乱码

```bash
#q表示质量因子 也就是权重
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
```

```bash
#浏览器会采用charset中的编码方式进行解码
Content-Type: text/html; charset=utf-8 
```

`Accept-Encoding` 允许的内容编码 ，主要用于压缩  `Content-Encoding`  服务器采用的压缩编码

```bash
Accept-Encoding: gzip, deflate, br
```

```bash
Content-Encoding: gzip
```

`Accept-Language` 允许的语言 `Content-Language`服务器响应的语言

```bash
Accept-Language: zh-CN,zh;q=0.9,en-US;q=0.8,en;q=0.7
```

```bash
Content-Language: zh-CN
```

另外服务器必须指定`Content-Length`来表示响应Body的长度，如果是 **Chunk**传输不知道长度则必须指定`Transfer-Encoding:chunked`

```bash
Content-Length: 1000 #byte
```

`Vary` 列出了服务器参考了哪些Header字段来进行内容协商

```bash
Vary: Accept-Encoding,Accept
```

常见的的 **MINE** 类型如下:

| MINE                                                         | 描述       |
| ------------------------------------------------------------ | ---------- |
| `text/html` `text/css` `text/plain`纯文本                    | 文本类型   |
| `image/gif` `image/png` ...                                  | 图片类型   |
| `audio/mp4` `audio/mp3` ...                                  | 音视频类型 |
| `application/json`  `application/pdf` `application/javascript` `application/octet-stream`未知的二进制流类型 | 应用类型   |

​    

## HTTP头部字段和常见头部

| 类型     | 含义                                                     |
| -------- | -------------------------------------------------------- |
| 通用字段 | 在请求头和响应头里都可以出现                             |
| 请求字段 | 仅能出现在请求头里，进一步说明请求信息或者额外的附加条件 |
| 响应字段 | 仅能出现在响应头里，补充说明响应报文的信息               |
| 实体字段 | 属于通用字段，但专门描述 body 的额外信息                 |

头部字段是不区分大小写的，比如`Host`和`host`是一样的，但是一般建议使用大写，并且我们不仅仅可以使用HTTP规范的头部，还自定义头部字段，自定义头部需要注意以下几个规则:

- 头部字段名字不允许出现空格，可以允许 `-` 来分割不同的单词，但是不允许使用 `_` 下划线，比如`User-Name`是合法的，`User_Name`就是非法的
- 字段名后面必须紧接着`:`，不能有空格
- 头部字段顺序没有意义，可以随意排列
- 有些头部字段可以重复设置例如 `Set-Cookie`，有些则只能设置一个比如`Host`

**下面来看一些常见头部**:

`User-Agent` 客户端类型，浏览器类型，浏览器内核版本信息，操作系统类型等

```bash
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/84.0.4147.89 Safari/537.36
```

`Host` 表示请求的域名，DNS获取到IP只是请求到那台机器，因为一个机器可能会开好几个服务和域名，所以需要Host来标识是哪个服务

```bash
Host: v.qq.com
```

> nginx中配置的`server_name`需要和HTTP头部的`Host`字段相匹配这样才可以访问，如果不匹配则会走默认的server_name，这样可以实现一台主机多个域名

`Date` 表示报文创建的时间，**GMT** 格式

```bash
Date: Sun, 04 Oct 2020 02:27:01 GMT
```

`Referer` 求来自哪个页面，由浏览器来自动添加

```bash
Referer:https://www.google.com
```

`Server` 服务器类型

```bash
Server: nginx
```

`Allow` 服务器运行的请求类型

```bash
Allow: GET,POST,OPTIONS      
```

​     

## Body压缩

报文压缩、加密等都是对于HTTP报文的`Body`部分进行的，**压缩和加密** 其实都是对Body的一种编码方式，比如启用 `gzip`压缩就是对Body部分进行`gzip`格式的编解码，这样就达到了压缩的效果，进行压缩或则加密的报文都需要消耗一定的CPU资源

压缩`Body`的时候服务器需要在响应的Header指定`Content-Encoding`:

```http
HTTP/1.1 200 ok
Content-Encoding: gzip
....
```

这样客户端就可以根据这个字段来知道Body是否被编码，以此来解码

客户端在请求的时候也可以设置Header指定`Accept-Encoding`字段表明自己可以接受哪些编码方式，服务器就可以根据客户端的请求来改变相应的响应策略

```http
GET /a/b.html HTTP/1.1
Accept-Encoding: gzip,deflate,compress
...
```

​        

## HTTP文件下载

HTTP下载文件时必须要设置的两个头部:

`Content-Type` 用于指定待下载文件的类型

`Content-Disposition` 用于指定下载后文件名以及相应的编码规则

- `attachment`表示浏览器会将数据下载下来，如果不设置则不会进行下载
- `inline` 表示响应中的消息体会以页面的一部分或者整个页面的形式展示

```bash
Content-Type: application/pdf
Content-Disposition: attachment; filename="filename.jpg" 
```

​     

## FORM表单数据

`Content-Type`在Post提交的时候也会用到，告诉服务器提交的信息属于Form表单编码还是JSON格式，并且POST请求也需要指定`Content-Length`

HTML表单里可以根据`<form enctype="application/form-data">` 来指定表单上传的编码方式，POST提交的数据主要有如下几个类型

```bash
Conent-Type: mutipart-data
Content-type: x-www.form-urlencoded #form不指定enctype默认就是这种编码
Conent-Type: application/json
```

### 1、x-www-form-urlencoded

提交的表单数据按照 `key=val&key=val`方式进行编码

```http
POST http://www.example.com HTTP/1.1
Content-Type: application/x-www-form-urlencoded;charset=utf-8

name=lyer&age=18
```

### 2、multipart/form-data

需要在`Content-Type`中指定`boundary`分隔符

```bash
POST http://www.example.com HTTP/1.1
Content-Type:multipart/form-data; boundary=-------XXXX

-------XXXX
Content-Disposition: form-data; name="username"

lyer
-------XXXX
Content-Disposition: form-data; name="age"

18
-------XXXX
Content-Disposition: form-data; name="file"; filename="a.png"
Content-Type: image/png

<二进制数据>
-------XXXX
```

​      

## HTTP长连接keep-alive

`Connection` 默认值是`keep-alive`

```bash
Connection:close #立即关闭此次TCP连接 即不支持长链接
```

​     

## HTTP队首阻塞

HTTP是 **请求-应答** 模式，如果同时发送多个请求那么当浏览器收到多个应答的时候就无法辨别那个应答属于哪个请求，所以HTTP协议必须是一收一发的，一个请求发送之后必须等待其应答回来之后才可以发送下一个请求，每个请求必须排队发送

**所以每个请求需要加入一个请求队列，只有等前面请求的应答收到之后此请求才可以发送**

如果队首有请求阻塞了，那么后面所有的请求都必须等待，这就是 **队首阻塞问题**

为了解决队首阻塞问题造成的速度慢，有如下几个方法:

- 浏览器多开几个线程并发进行请求后端服务器资源，此方法问题就是后端服务器接收到的并发数量巨大: **用户数并发请求线程数** 造成服务器压力太大，所以一般浏览器只会开`6~8`个线程左右的进行并发请求
- **域名分片** 也就是给一台机器一个服务多开几个域名，这样分别将用户的请求负载均衡到各个域名上去请求同一台机器，这样并发也上去了

​      

## HTTP代理

`Via` 保存经过的代理节点的主机名字或则域名

![](https://raw.githubusercontent.com/biningo/cdn/master/img1/image-20201007095736179.png)

```bash
Via: nginx-1,nginx-2
```

`X-Forwarded-For` 保存经过代理的IP

```bash
X-Forwarded-For: 192.168.0.10,192.168.0.12
```

​      

## HTTP安全之Basic认证

Basic认证安全性比较低，但是其实现比较简单，只需要加一个头部即可

比如现在访问 `/api`资源，服务器发现没有通过认证，于是范围如下报文

```http
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Basic realm=<认证范围,随便写>
```

现在浏览器都实现了Basic认证弹窗，当收到这样的报文之后浏览器就会弹窗要求用户输入密码，如果严重通过则可以访问，如果不通过则继续返回`401`或则返回`403`

用户输入密码之后，浏览器会在头部加上`Authorization: base64编码(username:pwd)`

​    

## 参考

- https://www.cnblogs.com/lightsong/p/4172979.html 【第三方Cookie】
- https://www.zoo.team/article/http-cache 【缓存】
