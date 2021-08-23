---
title: sed工具
date: 2021-08-22
categories: [Linux]
tags: [Linux,sed]
draft: true
---

> **在使用sed和awk命令时，需要注意他们都是以行为单元进行操作的，也可以指定为特殊的分割单元**

## awk和sed区别

他们都是Linux下的 **流处理工具**

- `awk`核心是**格式化**
- `sed` 核心是 **正则**

​    

## sed命令格式

![](https://raw.githubusercontent.com/biningo/cdn/master/2021-04/sed.png)

​    

## sed常见参数

```bash
-i #对原文件进行修改(默认不会修改原文件而是直接输出到屏幕)
-e #有多个正则则需要写-e
-n #仅显示script处理后的结果 p指令需要加这个  其他不需要吧
```

​    

## 范围

指定行范围

```bash
sed -n '1 p' txt # 1
sed -n '1,3 p' txt # [1,3]
sed -n '1,+2 p' txt # [1,3]
```

奇偶行

```bash
sed -n '1~2 p' txt # 奇数行
sed -n '2~2 p' txt # 偶数行
```

`$`表示到最后一行

```bash
sed -n '2,$ p' txt
```

正则匹配范围

```bash
sed -n '/a/,+2 p' txt # /a/匹配规则开始到之后的2行
sed -n '/a/,/c/ p' txt # /a/ 到 /c/
```

​    

## 操作

### s文本替换

将全部的`hello`替换为`hi`，结果重定向到b.txt

```bash
sed 's/hello/hi/g' a.txt > b.txt
sed -i 's/hello/hi/g' a.txt #修改原文件
```

在每一行前面添加`lyer`内容

```bash
sed 's/^/lyer/g' a.txt
```

在每一行结尾添加`lyer`内容

```bash
sed 's/$/lyer/g' a.txt
```

去掉`html`的tag

```bash
sed 's/<[^>]*>//g' html.txt
```

替换指定行的内容

```bash
sed '1s/hello/hi/g' a.txt #替换第1行
sed '1,3s/hello/hi/g' a.txt #替换[1,3]行
```

只替换每行的第`n`列

```bash
hello,world hello,world hello,world
hello,world hello,world hello,world
hello,world hello,world hello,world
-----------------------------------
sed  's/hello/hi/2' a.txt #只替换每行的第2个
sed  's/hello/hi/2g' a.txt #替换每行的2列之后的所有
```

上面的范围可以应用与s命令，也就是说可以指定替换的范围

```bash
hi,world
hello,world
hi,world
hello,world
--------------------------
#将hi开头的行中的world->lyer g表示将行中的全部替换
sed  '/^hi/ s/world/lyer/g' txt
```

`&`表示匹配到的内容，比如下面可以将匹配到的内容用`""`标注出来

```bash
sed  's/hi/"&"/g'  txt
sed 's/.*/"&"/' file #将每行都使用“”包裹
```

### a和i插入指令

`a`表示在目标行后面追加，`i`表示在目标行前面插入

```bash
sed '2a hello,append' txt #在2行后面插入语句
sed '2i hello,append' txt #在第二行前面插入语句
```

使用正则匹配进行追加、插入

```bash
sed '/hello/a Hello,World' txt #在有hello的行后面插入
sed '/hello/i Hello,World' txt #在有hello的行前面插入
```

对每行都进行插入操作

```bash
sed '/.*/a -------------------' txt
```

### c替换行命令

替换指定的行，替换多行为1行

```bash
sed '1c hi,world' txt #替换第一行
sed '1,3c hi,world' txt #替换1~3行为1行
```

将正则匹配到的行都替换为后面的

```bash
sed '/hello/c hi,world' txt
```

### d删除行命令

和上面的差不多

```bash
sed '2d' txt
sed '/hello/d' txt
```

### p打印命令

使用`p`命令时需要`-n`，不然会输出两遍

```bash
sed -n '/hello/p' txt
```

查看第`1~3`行、`4~5`行

```bash
sed -n -e '1,3 p' -e '4,5 p' txt
```

### w写入文件命令

`w`命令可以讲匹配到的行写入另外一个文件

```bash
sed -n '/a/,+2 w a.txt' txt #将匹配到的文件写入a.txt
```

​     

## sed高级部分

TODO

​    

## 参考

[sed简明教程](https://coolshell.cn/articles/9104.html)

[Linux生产环境上，最常用的一套“Sed“技巧](https://juejin.cn/post/6844903848885092365#heading-1)

