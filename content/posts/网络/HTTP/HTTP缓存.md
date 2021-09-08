---
title: HTTP缓存
date: 2021-05-31
categories: [网络/HTTP]
tags: [HTTP,缓存]
---

## 浏览器缓存原理

浏览器会把请求的`URL`作为key，value就是此次请求对应的响应。每次查找缓存的时候就会到内存或则磁盘去查找key对应的响应信息即可

​      

## Cache-Control 强缓存

`Cache-Control` 控制浏览器缓存的时间，注意此头部是通用头部，请求和响应都可以有

如果是服务器响应头部的话则是告诉浏览器此信息可以缓存多久，下面的报文表示可以缓存`10s`

```http
GET /ping HTTP/1.1
Cache-Control: max-age=10
Content-Length: 4
Content-Type: application/json; charset=utf-8
Date: Tue, 01 Jun 2021 13:23:49 GMT
```

比如我现在一个重定向到此请求，则下次重定向就不需要再次请求`/ping`了，直接从缓存里面读取，下面是go后端演示程序

```go
func main() {
	r := gin.Default()
	r.GET("/ping", func(context *gin.Context) {
		context.Header("Cache-Control", "max-age=10")
		context.JSON(200, "OK")
	})

	r.GET("/a", func(context *gin.Context) {
		context.Header("Location", "/ping")
		context.Status(302) //临时重定向 不会被浏览器缓存 每次请求都会打到后端
	})

	r.GET("/b", func(context *gin.Context) {
		context.Header("Location", "/ping")
		context.Status(301) //永久重定向 会被浏览器缓存 下次请求此则直接从缓存中读取响应报文  然后直接重定向到/ping
	})
	r.Run(":8080")
}
```

用户主动刷新、地址栏重新输入、ctrl+f5强制刷新等动作浏览器会自动的在请头中设置Cache-Control的值为`max-age=0` 或则 `no-cache` ，表示直接请求服务器而不从缓存中读取，不管缓存是否过期都会请求服务器

如果是超链接、重定向、或则被动请求（js资源、css、图片）则不会自动加上`max-age=0`，缓存时间没有失效的话就从缓存中获取。这就是强缓存，即使服务器资源改变了，只要没有超过max-age则就会从缓存中拿

| Cache-Control值     | 含义                                                         |
| ------------------- | ------------------------------------------------------------ |
| max-age             | 单位是`s`，缓存的开始时间是报文创造时的时间而不是浏览器收到响应的时间 |
| no-cache(max-age=0) | 表示客户端不使用强缓存，直接请求服务器                       |
| no-store            | 禁止使用缓存，也不会进行协商缓存                             |
| private             | 专用于个人的缓存，中间代理、CDN 等不能缓存此响应             |
| public              | 响应可以被中间代理、CDN 等缓存                               |
| must-revalidate     | 在缓存过期前可以使用，过期后必须向服务器验证，进行协商缓存   |
| no-transform        | 不允许中间代理服务器对资源做转化                             |

​    

## 协商缓存

如果缓存时间过了，但是服务器资源还是没有改变，那么就还是可以继续用缓存的，而不需要服务器再次传递数据

当浏览器的强缓存失效的时候或者请求头中设置了`Cache-Control:max-age=0|no-cache` 不走强缓存，并且在请求头中设置了`If-Modified-Since` 或者 `If-None-Match` 的时候，会将这两个属性值到服务端去验证是否命中协商缓存

如果命中了协商缓存则会返回 `304` 状态表示缓存重定向，可以继续使用浏览器的缓存，并且响应头会设置 `Last-Modified` 或者 `ETag` 属性

请求头部:

![](https://raw.githubusercontent.com/biningo/cdn/master/img1/http-cache.png)

响应头部:

![](https://raw.githubusercontent.com/biningo/cdn/master/img1/http-cache2.png)

第一次请求服务端会把资源的最后修改时间放到`Last-Modified`响应头中

第二次发起请求的时候会将上一次响应头中的`Last-Modified`时间放到 `If-Modified-Since` 中传递给服务器，服务端根据`If-Modified-Since`字段（也就是上一次资源的修改时间）判断资源是否已经被修改过

如果`If-Modified-Since`和上一次资源修改时间相等则表示资源没有被修改，则返回 304表示浏览器可以继续使用缓存，浏览器收到304则会重定向到本地缓存中。如果资源已经被修改了则会直接返回新的资源并且附带上`Last-Modified`资源的更新时间

`ETag/If-None-Match` 是为了解决上面两个头部无法解决的问题:

- 资源改动间隔时间在`1s`内
- 资源虽然已经更新了但是内容还和之前的一样，这种情况还是可以继续使用缓存的

`ETag/If-None-Match` 的值是一串 hash 码，代表的是一个资源的摘要。当服务端的文件变化的时候，资源的摘要会随之改变

第一次发起请求服务器会将资源的摘要放到`ETag`中传递给浏览器

第二次发起请求时浏览器会将上次的`ETag`中的摘要放到`If-None-Match`中传递给服务器。服务器将浏览器传递上来的摘要和现在资源的摘要进行对比，如果相等则表示资源没有发生改变，还可以使用缓存。如果摘要不相等则表示资源已经发生改变，则服务器会重新传递资源

ETag 又有强弱校验之分，如果`ETag`是以 `W/` 开头的一串字符串，说明此时协商缓存的校验是弱校验的，只有服务器上的文件差异（根据 ETag 计算方式来决定）达到能够触发`ETag`值后缀变化的时候，才会真正地请求资源，否则返回 304 并加载浏览器缓存

**ETag是为了弥补Last-Modified的缺陷**

​    

## ETag解决了什么问题

1. `Last-Modified`  精确时间只能是秒级的，如果资源修改时间是在1s以内的话则不能察觉到资源已经修改了，还是会继续使用缓存，所以`ETag`可以解决这个问题
2. 有些文件虽然会修改，但是整体内容还是不变得，比如删除一个空行或则只是修改了文件的修改时间这种情况，这种的话则缓存还是有效的
3. 各个代理服务器和服务器的时间不一致，比如不在一个时区等，这些也只能通过ETag来判断是否可以使用缓存

**综上，ETag会比Last-Modified更加实用，所以Etag的优先级会更好，如果两个同时存在则优先考虑Etag**

​    

## 参考

[图解 HTTP 缓存](https://www.infoq.cn/article/aiwqlgtlk2eft5yi7doy)