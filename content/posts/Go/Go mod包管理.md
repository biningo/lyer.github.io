---
title: Go mod包管理
date: 2021-02-12
categories: [编程语言/Go]
tags: [Go]
---

## Go modules相关的环境变量

```bash
GO111MODULE="on" #是否启用go module  go1.16默认开启
GOPROXY="https://goproxy.io,direct" #go拉去依赖的镜像站点
GOPRIVATE="" #私有go仓库相关配置
```

上面的环境变量可以使用下面的命令进行改写

```bash
go env -w GO111MODULE=on
```

或则直接设置环境变量

```bash
GO111MODULE=on
```

​     

## go.mod文件

```go
//每个模块都有一个名字,通常通过go mod init指定
module github.com/biningo/go-play
//指定go版本
go 1.15 
//指定依赖的库地址 <url> <version> 形式
require (
    github.com/gin-gonic/gin v1.6.3
)
//replace 替换 require 中声明的依赖，使用另外的依赖及其版本号 不经常使用
replace github.com/gin-gonic/gin v1.6.3 => github.com/gin-gonic/gin v1.6.2
//exclude 排除某些版本
exclude ithub.com/gin-gonic/gin v1.5.0
```

模块名字主要有如下几个作用：

- 作为模块的标识
- 作为模块的 import path，go会根据此导入到`$GOPATH/pkg/mod`路径下去寻找代码


go mod的`replace`命令主要用来替换原来的包，使用场景有如下几个

- 单纯的替换包
- 替换无法下载的包

​     

## go.sum文件

该文件记录格式如下

```bash
# <module> <version>/go.mod <hash>
github.com/gin-gonic/gin v1.6.3/go.mod h1:ahKqKTFpO5KTPHxWZjEdPScmYaGtLo8Y4DMHoEsnp14=
```

后面的`hash`就是`go get`命令生成的对代码压缩包的`hash`值

如果我们拿到一个开源项目，进行`build`下载依赖包进行构建的时候，如果本地缓存了相关的代码库，则取对应代码库的`zip`压缩包的`hash`值与当前项目的`go.sum`对应依赖的`hash`进行比较，如果一致则表示本地缓存的代码库和项目期望的代码库版本一致，则可以构建，如果不一致则表示本地缓存的版本不是我想要的则构建失败，需要下载项目指定的依赖版本进行构建

> go.sum存在的意义在于，我们希望别人或者在别的环境中构建当前项目时所使用依赖包跟go.sum中记录的是完全一致的，从而达到一致构建的目的

​        

## go包的版本

Go Modules的版本其实就是项目的`tag` ，如果没有`tag`则会自动生成`v0.0.0-commit日期-commitID`

​     

## go mod命令

初始化一个项目，也就是创建项目的`go.mod`文件，该文件记录了所有的依赖信息以及go的版本

```bash
go mod init github/biningo/bstree #一般以github仓库地址为项目名字
```

`go.mod`可能会记录一些没有用到的包，可以用如下命令来清除这些包或则下载一些不存在的包，有可能我们从github上拉别人的代码就需要下载一些其它库

```bash
go mod tidy
```

可以将依赖的包的代码都复制到本项目下的`ventor`目录下，可以使用如下命令

```bash
go mod ventor
```

​    

## go list命令

用于列出当前项目的`go mod`文件依赖了哪些模块

```bash
go list all #列出依赖了的标准库 不会列出其他的mod模块
go list -f {{.xxxx}} #指定输出格式（包括标准库 不包括mod模块）
go list -json #以json格式输出项目依赖信息(包括标准库 不包括mod模块)
go list -m all  #查看所有依赖以及间接依赖mod模块当前版本（不包括标准库）
go list -u -m all #查看所有依赖以及间接依赖当前版本和可升级版本
go list -m -versions #github.com/gin-gonic/gin 查看某些模块的所有版本
```

可以执行如下命令将`Go Modules`的所有依赖信息都输出到文件

```bash
go list -m -json all > go.list
```

​     

## 搭建go私有仓库

有时候我们这公司内部开发一些代码库，这时候我们就需要搭建公司私有的代码仓库了

https://www.szyhf.org/2017/07/12/%E5%BD%93go-get%E9%81%87%E4%B8%8Agitlab/

http://holys.im/2016/09/20/go-get-in-gitlab/

https://segmentfault.com/a/1190000018414744    

## 巨人肩膀

[Go Modules 包管理工具的理解与使用](https://www.infoq.cn/article/xyjhjja87y7pvu1iwhz3)