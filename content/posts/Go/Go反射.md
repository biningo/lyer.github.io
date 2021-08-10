---
title: Go反射
date: 2021-03-29
categories: [编程语言/Go]
tags: [Go]
draft: true
---

## 什么是反射

**反射** 是指一类应用，它们能够**自描述和自控制**。也就是说，这类应用通过采用某种机制来实现对自己行为的描述（self-representation）和监测（examination），并能根据自身行为的状态和结果，调整或修改应用所描述行为的状态和相关的语义

go官方自带的`reflect`包就是反射相关的，go反射主要有下面两大对象:

- **Value** 值对象，通过`reflect.ValueOf`获取
- **Type** 类型对象，通过`reflect.Typeof`获取

​    

## Typeof

获取一个对象的类型对象Type，`Type`是一个接口，go中每种类型都实现了`Type`接口，比如`chanType`表示`chan`的Type对象

通过Type对象就可以获取类型相关的信息，比如 **Name()** 类型的名字、**PkgPath()** 包路径、**Size()**此类型占用的字节大小等，`Type`接口的方法不是每种类型都可以调用的，比如 **Len()** 方法只能`arrayType`类型才可以调用，表示数组的长度 ，**Size()** 方法则只可以由定长的类型才可以调用，比如数组、string、int64等，而slice、map就不可以调用Size方法，会报错

```go
func main() {
	var i interface{}
	i = [2]int{1,2} //int类型默认和平台位数一致 这里为int64
	t := reflect.TypeOf(i)
	log.Println(t.Len(),t.Size()) //2 8*2=16
}
```

​    

## ValueOf

ValueOf就和TypeOf同理了，这里不多赘述，重点说明下面几个问题

`Interface()` 将一个对象转化为`interface{}`空接口类型

`Elem()` 如果值是一个指针的话，调用此方法获取指针指向的值的`Value`对象

`Recv()` 获取一个chan的值

`Call()` 调用一个函数，传入各个参数的Value对象

`Field()` 获取一个struct对象的各个属性的Value对象

​    

## 反射原理

go在`1.5`之后就实现了 **自举**，自举过程如下:

- 先用C和汇编写一个Go的编译器，用C的编译器编译成可执行文件，此二进制文件就可以编译go代码了
- 用Go写一个Go的编译器，用上述编译器编译成二进制文件，此二进制文件就是用Go来实现的Go编译器

go的反射与 **interface和unsafe.Pointer** 结合的比较紧密 TODO

​    

## 参考

- https://i6448038.github.io/2020/02/15/golang-reflection 【go反射实现原理】
- https://juejin.cn/post/6844903559335526407 【Golang的反射reflect深入理解和示例】
- https://colobu.com/2019/01/29/go-reflect-performance/ 【Go Reflect 性能】

[深度剖析Reflect + 实战案例](https://learnku.com/articles/55022)