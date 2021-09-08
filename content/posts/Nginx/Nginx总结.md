---
title: Nginx总结
date: 2021-06-10
categories: [Nginx]
tags: [Nginx] 
---

## Nginx命令

```bash
nginx #默认配置启动
nginx -c /etc/nginx.conf #指定配置文件启动
nginx -s reload  #重载配置,热重启  kill -s SIGHUB <pid>
nginx -s stop #关闭 kill -s SIGINT/SIGTERM <pid>
nginx -s quit #优雅关闭 kill -s SIGQUIT <pid> 
nginx -s reopen #重新打开日志文件(切割日志)
nginx -t #检查默认配置文件语法是否正确
nginx -t -c /etc/nginx.conf #指定配置文件路径
```

​     

## Location匹配规则

1. 寻找精确匹配，匹配成功则返回
2. 精确匹配不成功则匹配非正则的，进行最长匹配(和顺序无关)，如果最长匹配以`^~`开头则返回
3. 第二步匹配不到则进行正则匹配，正则匹配和配置文件的顺序有关，如果前面的正则匹配成功则会立即返回
4. 正则匹配不成功则继续进行第二步的最长匹配并且返回

匹配的修饰符，对于正则匹配，是一定要加的

```bash
^~ 开头匹配 用于不含正则表达式的前缀路径
~ 大小写敏感
~* 忽略大小写
! 反向匹配，！后面可以连接以上的规则    
```

案例一

```nginx
#访问/a/b/c 返回2
location /a/b/c {
    return 200 "1";
}
location ~ ^/a{
    return 200 "2";
}
```

案例二

```nginx
#访问/a/b/c 返回1
location ^~ /a {
    return 200 "1";
}
location ~ ^/a{
    return 200 "2";
}
```

案例三

```nginx
#访问/a/b/c  返回2
location /a {
    return 200 "1";
}
location ~ ^/a{
    return 200 "2";
}
```

案例四

```nginx
#访问/a/b/c 返回2
#访问/a/b 返回1
#访问 /a/c 返回3
location = /a/b{
    return 200 "1";
}
location ^~ /a/b{
    return 200 "2";
}
location ~ ^/a {
    return 200 "3";
}
```

​    

## 自定义错误页面

我们可以自定义404、500等错误页面

```nginx
server{
    listen 8080;
    #如果路径找不到就会到转移到/404.html
    #也可以重定向到一个URL
    error_page 500 503 502 https://github.com;
    location / {
        root /home/pb/program/nginx/conf/myconfig;
    	error_page 404 /404.html;
        autoindex on;
    }
}
```

```nginx
server{
  listen 8080;
  root /home/pb/program/nginx/conf/myconfig;
  autoindex on;
  error_page 404 /404.html;
  location /{
  }
}
```

​    

## Nginx访问权限

```nginx
deny   123.9.51.42; #拒绝单个IP
allow  45.76.202.231; #允许单个IP
deny   all; #拒绝全部
allow 10.1.1.0/16 #允许一个网段
```

> 注意`allow/deny`指令按照配置的前后顺序进行生效，前面配置的会先生效
>
> 有些指令采用下级覆盖上级的方式

如下表示只允许 10.76.202.231访问

```nginx
location / {
    allow  10.76.202.231;
    deny   all;
}
```

如下则会拒绝所有IP的访问，因为deny在前面已经生效了，后面配置allow都失效了，所以一般将allow配置在deny前面

```nginx
location / {
    deny   all;
    allow  10.76.202.231;
}
```

​    

## Nginx反向代理

访问8080会将我们代理到B站

```nginx
server{
    listen 8080;
    location / {
        proxy_pass http://www.bilibili.com;
    }
}
```

代理本地的后端服务器

```nginx
server{
    listen 8080;
    location / {
        proxy_pass localhost:9000/api;
    }
}
```

​    

## Nginx负载均衡

轮询

```nginx
upstream myserver{
    server 192.168.17.129:8000;
    server 192.168.17.129:8001;
    server 127.0.0.1:7071 down; #不参与负载均衡  
    server 127.0.0.1:7070 backup; #备份server  只有其他都忙了才访问 
}
```

权重

```nginx
#权重越高，在被访问的概率越大 如下分别是10% 10% 70%
upstream myserver{
    server localhost:8001 weight=1;
    server localhost:8002 weight=1;
    server localhost:8003 weight=8;
}
```

`IP Hash` 为每个客户端IP分配固定的服务器

```nginx
upstream myserver{
    ip_hash;
    server localhost:8001;
    server localhost:8002;
    server localhost:8003;
}
```

最少连接`least_conn`

```nginx
upstream myserver{
    least_conn;
    server localhost:8001;
    server localhost:8002;
    server localhost:8003;
}
```

`URL Hash`(三方) 为每个URL分配固定的服务器

```nginx
upstream myserver{
    hash $request_uri;
    server localhost:8001;
    server localhost:8002;
    server localhost:8003;
}
```

`Fair`(三方) 根据响应时间来访问，哪个机器响应快就哪个

```nginx
upstream myserver{
    fair;
    server localhost:8001;
    server localhost:8002;
    server localhost:8003;
}
```

访问入口

```nginx
server{
    listen 9000;
    location /{
        proxy_pass http://myserver;
    }
}
```

​        

## Nginx限流TODO



## Nginx对响应和请求的处理

Nginx添加响应头

```nginx
location dist/ {
	root /a;
	add_header Cache-Control no-cache;
	error_page 404   /page/404/index.html;
}
```

​    

## Rewrite模块

| flag      | 说明                                             |
| --------- | ------------------------------------------------ |
| last      | nginx重定向`location`区段，能够直接返回200状态码 |
| break     | nginx重定向资源路径，能够直接返回200状态码       |
| redirect  | 返回302临时重定向                                |
| permanent | 返回301永久重定向                                |

```nginx
location /first {
    rewrite /first/(.*) /second/$1 last;
    return 200 "hello,first\n";
}
location /second {
    rewrite /second/(.*) /third/$1 last;
    return 200 "hello,second\n";
}
location /third{
    return 200 "hello,third\n";
}
```

​    

## Nginx日志

Nginx日志分为

- 访问日志
- 错误日志

```nginx
access_log  "/usr/local/nginx/logs/access_log"  main;
error_log  "/usr/local/nginx/logs/error_log"  error;
```

设置缓存区大小`buffer`以及缓存区的有效时间`flush`，下面表示缓存区大小为200k，并且定时10s将缓存区日志刷新一次磁盘

- 缓存区满了刷新磁盘
- 定时刷新磁盘

> 如果不设置buffer和flush则表示每次请求都会直接写入日志

```nginx
access_log conf/myconfig/access.log test buffer=200k flush=10s;
```

自定义日志格式

```nginx
log_format test
    '[$time_local] $remote_addr '
    '"$http_referer" "$http_user_agent"';
access_log /home/pb/program/nginx/conf/myconfig/access.log test;
```

常用的`format`参数（如果值不存则使用`-`代替）【自行官网查看】

错误日志

```nginx
error_log logs/error.log error;
```

level可以是`debug`, `info`, `notice`, `warn`, `error`, `crit`, `alert`,`emerg`中的任意值

​        

## Nginx日志切割

Nginx日志路径可以在`nginx.conf`中配置，但是随着日志的写入，日志文件会变得很大。我们可以手动进行转移日志文件

```bash
mv access.log ~/back/access.log
nginx -s reopen #重新打开日志，此时日志文件被我们转移了，所以会新创建一个日志文件
```

我们也可以通过Shell脚本来进行日志转移

```bash
#!/bin/bash
oldAccessLog="/home/pb/program/nginx/conf/myconfig/access.log"
newAccessLog=$oldAccessLog-$(date +%Y%m%d%H)
echo $oldAccessLog
echo $newAccessLog
mv $oldAccessLog $newAccessLog
nginx -s reopen
```

​    

## Nginx动静分离

静态资源就是图片、js、css等资源，动态资源就是后端查询数据库生成的数据

动静分离就是将这两部分的资源分开，可以通过Nginx的`Location`将动态请求和静态请求分离开来

我们可以讲静态资源直接通过Nginx处理并返回给客户端，而将动态资源通过反向代理请求后端的服务器比如Go语言HTTP服务器、Tomcat等

如果可以的话可以直接将静态资源放到阿里云OSS或则其他云厂商进行CDN加速

​    

## Nginx架构TODO

​    

## Nginx变量

```nginx
server{
    listen 8000;
    location /{
        set $username "lyer";
        set $log_file "my.log";
        access_log /conf/myconfig/a/$log_file  combined;
        return 200 "OK $username\n";
    }
}
```

常见的一些系统内置变量

TODO

​    

## Nginx配置文件

默认配置文件的路径

```bash
$NGINX_HOME/conf/nginx.conf
```

​    

## Nginx解决跨域问题

方案一: 反向代理，将前端发送的请求反向代理到后端服务器中

```nginx
# localhost:8080/api/ping ===> localhost:9090/api/ping
location ^~ /api {
	proxy_pass http://localhost:8080;
}
```

上面的配置的问题就是说 `/api/xxx`路径会被转发到`localhost:8080/api/xxx`，这样我们的后端服务器就需要为路径增加`/api`前缀了，我们还可以通过`rewrite`指令进行改写

```nginx
# localhost:8080/api/ping ===> localhost:9090/ping
location ^~ /api {
    rewrite ^/api/(.*)$ /$1 break;
    proxy_pass http://localhost:9090;
}
```

方案二: 添加同源策略相关的`HTTP Header` ，比如在前端的nginx配置中添加Header即可，也可以在后端服务器上添加跨域请求头

浏览器会先发送OPTION请求到跨域的后端服务器上进行检测，如果后端运行则会响应这个OPTION请求，并且填充相关的响应头部

```nginx
server {
    listen 10000;
    add_header 'Access-Control-Allow-Origin' $http_origin;
    add_header 'Access-Control-Allow-Credentials' 'true';
    add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
    add_header 'Access-Control-Allow-Headers' $http_access_control_request_headers;
    add_header 'Access-Control-Expose-Headers' 'Content-Length,Content-Range';
    access_log logs/test.log combined;
    location / {
        if ($request_method = 'OPTIONS') {
            add_header 'Access-Control-Max-Age' 1728000;
            add_header 'Content-Type' '';
            add_header 'Content-Length' 0;
            return 204;
        }
        proxy_pass http://localhost:9090;
    }
}
```

​        

## 参考

[Nginx工作流程简介](https://zhuanlan.zhihu.com/p/136180893)

[初探 Nginx 架构](https://wiki.jikexueyuan.com/project/nginx/nginx-framework.html)

[前端必会的 Nginx入门教程](https://juejin.cn/post/6844903701459501070)

[Nginx从入门到实践](https://segmentfault.com/blog/siguoya-nginx) 【思否】

[给Nginx配置日志格式和调整日期格式](https://www.cnblogs.com/Serverlessops/p/13410327.html)

[Nginx 限流](https://colobu.com/2015/10/26/nginx-limit-modules/)

[Nginx 从入门到实践，万字详解！](https://juejin.cn/post/6844904144235413512#heading-0)

《Nginx从入门到实战》(慕课网)

