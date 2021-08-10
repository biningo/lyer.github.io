---
title: 优雅的写Go
date: 2021-02-14
categories: [编程语言/Go]
tags: [Go]
draft: true
---

## 静态代码检查工具

### goimports

`goimports` 是 Go 语言官方提供的工具，主要有如下几个作用：

- 自动**格式化 Go 语言代码**
- 对所有引入的包进行管理，自动增删依赖的包引用、将依赖包按字母序排序并分类

`goimports` 就是等于 `gofmt+依赖包管理`

用法和`go fmt`差不多，这里不再赘述。另外，在CI/CD中不应该加入这些检查，因为这是开发者的本职工作，应该由开发者来规范代码

```bash
goimport -w . #格式化 并且修改原文件
```

### go vet和golint

**代码语法检查、代码风格检查**，官方提供

### golangci-lint

强大的go开源静态代码分析，用于`CI`防止不规范代码合并，主要有几个用途：

- 代码规范
- 代码风格统一
- 语法错误、冗余语法等

```bash
golangci-lint run ./...
golangci-lint run dir1 dir2/... dir3/file1.go
```

具体见官网：https://golangci-lint.run/usage/performance

​    

## go代码报告评分平台

https://goreportcard.com

​        

## Go Code Review Comments总结

### 1、注释规则

注释开头第一个单词就是方法名或者变量名，接下来再描述实际的内容，并且以`.`结尾代表这句话是一个完整的句子

所有导出的包都应该都包级doc注释，如果注释很多的话可以单独在包下创建`doc.go`用于专门存放包注释

```go
// Request represents a request to run a command.
type Request struct { ...
// Encode writes the JSON encoding of req to w.
func Encode(w io.Writer, req *Request) { ...
```

### 2、Context

`Context`应该作为方法的第一个参数，并且不能保存在`struct`中而是以函数参数的方式进行传递

```go
func F(ctx context.Context, /* other arguments */) {}
```

### 3、Rand生成随机数

使用`crypto/rand`代替`math/rand`生成密钥

```go
func Key() string {
    buf := make([]byte, 16)
    _, err := rand.Read(buf)
    if err != nil {
        panic(err)  //
    }
    return fmt.Sprintf("%x", buf) //格式化为16进制
    // or hex.EncodeToString(buf)
    // or base64.StdEncoding.EncodeToString(buf)
}
```

### 4、空slice

声明一个空的`slice`时建议使用

```go
var t []string // nil
```

而不是使用

```go
t := []string{} // []
```

如果如果在`JSON`格式化场景下则可能需要使用第二种方法生成一个`[]`

### 5、panic和error

使用返回`error`的方式代替`panic`，除非真的造成了不可修复的影响

处理`error`时不应该使用`_`丢弃，而是去处理他

### 6、错误信息

错误的信息不应该大写，除非是一些专有名词，并且不应该加`.`句号

```go
fmt.Errorf("something bad") //正确
fmt.Errorf("Something bad") //错误
```

### 7、接口

如果接口没有被任何`struct`继承则删除这个多余的接口

### 8、命名规则

Go遵循的是混合大小写的规则命名，也就是驼峰命名，而`.go`文件命名则遵循`snake`命名，并且命名越简短越好

```go
//文件命名
hello_world.go

//变量或则函数命名
maxLength //不可导出
CountLength //可导出
```

包名不应该存在冗余

```bash
chubby.ChubbyFile --> chubby.File
```

### 9、函数形参

不要为了节省内存就都传入指针变量，有些参数如果不允许被函数内部修改则必须让传入非指针变量，一些容量较大的`struct`则可以传入指针以节省空间

### 10、函数方法

函数方法中的接受者命名不要写成

```go
me/this/self
```

而是以struct名字的前几个字母即可

```go
// 如 type Client struct
c/cli
```

​    

## Effective Go总结

TODO

​    

## Go项目目录结构

`/pkg`

 本地代码共享库，同时被外部直接引用，和项目无关的一些公共库

`/internal`

 内部代码库，不能被外部引用。比如我们写的应用代码应该放在`/internal/app`下，应用程序的共享库代码放在 `/internal/pkg`

`/examples` 

示例代码库

`/cmd`  

存储项目的可执行文件，比如`main.go` ，代码应该尽可能少，其他应用相关的代码应该写到`/internal`下

`/scripts` 

存放项目的一些脚本代码，比如安装的shell脚本装，分析等操作的脚本

`/api`

对外提供的API，比如gRPC应用的proto文件应该存放在此目录下

`/build`

CI/CD相关的代码和打包构建项目相关的代码

`/config`

配置文件相关，比如`yaml`

`/deployments` 或则 `/deploy`

IaaS，PaaS，系统和容器编排部署k8s相关的配置和模板（docker-compose，kubernetes/helm，mesos，terraform，bosh)

`/docker`

docker相关的文件和代码

`/assets`

静态资源文件

`/docs`

设计和文档

`/githooks`

git钩子

`/tools`

和项目相关的工具代码，这些代码可以引用`/pkg` `/internal`的库函数等

`/Makefile`

`/Dockerfile`

`/Jenkinsfile`

## 参考

[Go Code Comments中文翻译](https://www.zybuluo.com/wddpct/note/1264988)

[Effective Go中午翻译](https://learnku.com/docs/effective-go/2020/semicolon/6240)

[如何写出优雅的Go代码](https://draveness.me/golang-101/)

[golang 编程规范 - 项目目录结构](https://makeoptim.com/golang/standards/project-layout#go-目录)

https://www.jianshu.com/p/ca38dcdaf6bb

https://supereagle.github.io/2019/10/03/golang-lint/

