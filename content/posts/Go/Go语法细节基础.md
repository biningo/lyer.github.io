---
title: Go语法细节基础
date: 2021-03-08
categories: [编程语言/Go]
tags: [Go]
---

## byte和rune

`byte`是`uint8`的内置别名，代表一个字节，我们可以将byte和uint8看作是同一个类型
`rune`是`int32`的内置别名，代表一个字符， 我们可以将rune和int32看作是同一个类型，代表一个Unicode字符，Unicode字符目前还没超过32位

```go
var b1 rune ='你' // var b byte = '你' 报错
var b2 rune = 20320 //你的Unicode值
var b3 byte = 'a'
```

## uintptr、int、uint

`uintptr`代表一个地址，所以必须要能寻找到计算机所能表示的最大地址，随意其大小随计算机的位数而确定，如果是64位则该类型的大小也是64位的

同理`int、uint`等也是相似的，随本机类型而定，64位机器则是`int64`

## type自定义类型

```go
type MyInt int64 //此类型是一个新的自定义类型 和int64类型不一样
type Int64=int64 //此类型和int64类型是一样的
```

## 类型0值

| 类型                   | 默认值 |
| ---------------------- | ------ |
| bool                   | false  |
| int、float64等数值类型 | 0      |
| string                 | ""     |

## 整数类型的表示

```go
d1:=0x10F //16进制 
d2:=0b111	//二进制
d3:=10_000_000 //可以用下划线分割方便观看 10000000
```

## 浮点数的表示

```go
//[float64]
d1:= .32 //0.32
d2:= 2. //2.0
d3:=2.3e3 //2300
d4:=2.3e-3 //0.0023
```

## 字符串的``反引号

```go
s:=`1111
2222
3333` //反引号不会进行转义
```

## 常量const和itoa

const定义的一组变量可以设置相同的值

```go
const (
	name1,age1="a",18
    name2,age2 //name2="a" age2=18 
    _,age3 // age3=18
)
```

只有const才可以设置`itoa`，该值初始值为0

```go
const a = itoa //0
const (
    _ = itoa
	one
    two
    test="test"
    four //4
    five1,five2=itoa,itoa //5,5
)
```

```go
const (
	a1,a2 = itoa*10,itoa*20 //0 0
    b1,b2 // 10 20
    c1,c2 // 20 40
)
```

```go
const(
    _ = iota
    kb = 1<<(iota*10) //1 << (10*1) 1K
    mb					//1 << (10*2) 1M
    gb				//1 << (10*3) 1G
    tb			//1 << (10*4) 1T
)
```

## 变量作用域

```go
int f(){
    x:=10
    //用一个花括号可以建立一个独立的作用域
	{
		x:=100
		log.Println(x)
	}
	log.Println(x)
}
```

## 匿名结构体

```go
type Book struct {
	author struct{ Name string }
	price  int
}
func main() {
	b := Book{}
	b.author = struct{ Name string }{Name: "dsada"}
}
```
