---
title: Linux文件管理
date: 2021-08-10
categories: [Linux]
tags: [Linux]
---

## 文件类型

> Linux一切皆文件，很多东西都可以使用VFS抽象为文件来实现

| 文件类型   | 符号        | 举例                                       |
| ---------- | ----------- | ------------------------------------------ |
| 普通文件   | `-`         | 纯文本文件、二进制可执行文件、压缩文件.... |
| 目录文件   | `d`         | /                                          |
| socket文件 | `s`         | /                                          |
| 管道文件   | `p`         | /                                          |
| 软链接文件 | `l` (L小写) | /                                          |

查看文件类型的方式主要有下面几个命令

```bash
ls -l a.txt
file a.txt
stat a.txt #查看文件的详细信息
```

​     

## 文件时间

| 文件时间 | 描述                                                |
| -------- | --------------------------------------------------- |
| Access   | 文件最近被访问的时间                                |
| Modify   | 文件最近被修改的时间，`ls -l`默认列出的就是修改时间 |
| Change   | 文件对应的inode节点被改动的时间(chmod、chown)       |

对于一个首次创建的文件，那么上面三大时间都是一样的

> 使用`touch`一个已经存在的文件则不会清空文件，而是将文件的三大时间都修改为最新

使用`ls`命令查看文件的三大时间

```bash
ls -l filename #列出文件的 mtime (Modity) 等价于`ll`
ls -lc filename #列出文件的 ctime(Change Time)
ls -lu filename #列出文件的 atime(Access Time)
ls --time-style long-iso #指定时间格式 long-iso最完美
```

> cat一个文件，只会第一次变化，当再次cat的时候如果Access时间是新于Modify和Change的时间的则文件不会变化

同时还需要注意，往文件里面追加内容或则使用sed等更改文件内容的命令会同时更新文件的Modify和Change时间。这是因为文件内容变了那么文件的一些inode属性就变了，最直接的大小变了，此时 Change时间也会改变     

​     

## 文件权限

![](https://raw.githubusercontent.com/biningo/cdn/master/2021-04/linux-file1.png)

| 符号   | 描述                  |
| ------ | --------------------- |
| 2      | inode硬链接数         |
| osmond | 文件所属者            |
| family | 文件所属组            |
| 4096   | 文件大小 kb           |
| 时间   | ls不加参数默认是mtime |
| docs   | 文件名                |

文件权限计算方法如下

```bash
ower:group:other
rwxr-x-wx
421401021 --> 753
```

​     

## find文件查找

> 按时间

`-atime`: Access Time

`-mtime`: Modify Time

`-ctime`: Change Time

`+n` n天前 `-n` n天内

```bash
find . -atime -2 #查找2天内被访问过的文件
find . -atime +2 #查找2天前被访问过的文件
```

> 按文件类型

`-type` 

```bash
find . -type d #查找所有目录 
```

> 按文件权限

`-perm`

```bash
find . -perm 644
```

> 按文件大小

`-size `单位`cwbkMG`

`+nM` 大于n兆 `-nM`小于n兆

```bash
find . -size +100M #查找大于100M的文件
find . -size -10b #查找小于10字节的文件
```

> 按文件名查找

`-name`

```bash
find . -name "*.jpg" #查找以jpg结尾的文件
find . -name "?.jpg" #查找a.jpg这样的文件
find . -name "[abcdef].jpg" #查找a.jpg b.jpg c.jpg d.jpg e.jpg f.jpg
```

> 其他

`!` 反向查找

```bash
find . ! -size +10M #查找小于10M的文件
```

多目录下查找

```bash
find d1 d2 -size +10M
```

