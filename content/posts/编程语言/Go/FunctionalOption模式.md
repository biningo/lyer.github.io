---
title: Functional Option模式
date: 2021-07-22
categories: [编程语言/Go]
tags: [Go]
---

当我们创建一个struct时里面有些属性是必须要指定的，但是有些属性是可以有默认值而非必须指定的。比如一个请求HTTP的Client结构体

```go
type Client struct{
    Host string
    Port string
    Timeout int
    Protocol string //http https
}
```

其中假如Host和Port是必须指定的，其他两个属性都有默认值，由于go没有函数重载，则需要给每一种情况都创建一个构造函数。为了应对这种可选项，我们就可以使用Functional Option风格

```go
type Client struct {
	Host     string
	Port     uint16
	Timeout  int
	Protocol string
}

type Option func(*Client)

func NewClient(host string, port uint16, opts ...Option) *Client {
	client := &Client{
		Host:     host,
		Port:     port,
		Timeout:  10,
		Protocol: "http",
	}

	for _, o := range opts {
		o(client)
	}
	return client
}

func WithTimeout(timeout int) Option {
	return func(client *Client) {
		client.Timeout = timeout
	}
}

func WithProtocol(protocol string) Option {
	return func(client *Client) {
		client.Protocol = protocol
	}
}

func main() {
	client := NewClient("localhost", 8080, WithProtocol("https"), WithTimeout(20))
	fmt.Println(client)
}
```

还可以将所有可选配置单独建立一个option结构体

```go
type Client struct {
	Host   string
	Port   uint16
	option option
}
type option struct {
	Timeout  int
	Protocol string
}

type Option func(*option)

func NewClient(host string, port uint16, opts ...Option) *Client {

	option := option{
		Timeout:  10,
		Protocol: "http",
	}

	for _, o := range opts {
		o(&option)
	}

	client := &Client{
		Host:   host,
		Port:   port,
		option: option,
	}
	return client
}

func WithTimeout(timeout int) Option {
	return func(opt *option) {
		opt.Timeout = timeout
	}
}

func WithProtocol(protocol string) Option {
	return func(opt *option) {
		opt.Protocol = protocol
	}
}
```

​    

## 参考

[GO 编程模式：FUNCTIONAL OPTIONS](https://coolshell.cn/articles/21146.html)

