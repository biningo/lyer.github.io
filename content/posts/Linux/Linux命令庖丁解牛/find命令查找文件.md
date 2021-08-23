---
title: find命令查找文件
date: 2021-08-23
categories: [Linux]
tags: [Linux]
---

## find命令

`find`用于在任意的路径下过滤符合条件的文件

​     

## find基本用法

find后面可以追加多个路径，在多个路径下查找，但是如果不添加路径则表示在命令执行的路径下查找

```bash
find <查找路径>... <条件参数>...
```

列出指定路径下所有的文件

```bash
find ~/temp ~/bin
```

​     

## 按文件名查找

`-name`

```bash
find . -name "*.jpg" #查找以jpg结尾的文件
find . -name "?.jpg" #查找a.jpg这样的文件
find . -name "[abcdef].jpg" #查找a.jpg ... f.jpg
```

`-iname` 忽略大小写

```bash
find -iname "[a-z]*.txt" #a.TXT也可找到
```

​    

## 按时间查找

`-atime`: Access Time

`-mtime`: Modify Time

`-ctime`: Change Time

`+n` n天前 `-n` n天内

```bash
find . -atime -2 #查找2天内被访问过的文件
find . -atime +2 #查找2天前被访问过的文件
```

`-amin` 按分钟为单位查找

```bash
find . -amin -10 #查找10分钟以内被访问过的
```

`-newer` 找出比指定文件**修改时间**更长的所有文件

```bash
find . -newer /tmp/file.log
```

​     

## 按文件类型查找

`-type` 

```bash
find . -type d #查找所有目录 
find . -type f #查找所有普通文件
```

​     

## 按文件权限和用户查找

`-perm `按权限查找

```bash
find . -perm 644
```

`-user` `-group` 按用户、用户组查找

```bash
find . -type f -user tom #查看tom用户的所有普通文件
find . -type f -group root #查看root用户组的所有普通文件
```

​     

## 按文件大小

`-size `单位`cwbkMG`

`+nM` 大于n兆 `-nM`小于n兆

```bash
find . -size +100M #查找大于100M的文件
find . -size -10b #查找小于10字节的文件    
```

​     

## 路径匹配查找

`-path`

```bash
find $PWD -path "*internal*" #匹配带有internal的路径
find $PWD -path "*internal/*.yaml"
```

`-regex` 基于正则匹配路径

```bash
find $PWD -regex ".*\(\.yaml\|\.go\)$"
```

​    

## 查找条件

`-o` 或

```bash
find $PWD -name "[a-z]*.txt" -o -name "[0-9]*.txt"
```

`!` 反向查找

```bash
find . ! -size +10M #查找小于10M的文件
```

`-maxdepth <n>` 限制查找的目录深度

```bash
#一级目录查找: 只限于在当前目录下查找不允许再次递归
find $PWD -maxdepth 1  -regex ".*\(\.yaml\|\.go\)$" 
```

​     

## find执行命令

`-delete` 删除 (注意此命令不会提示，直接就删除了

```bash
find . -type f -name "*.txt" -delete #查找之后再删除
```

`-exec` 其中`{}`是搜索出来文件的替换符

```bash
find $PWD -name "[0-9]*.txt" -exec mv {} ./A/ \; #必须要加;结束 这里\代表转义;
find $PWD -name "[0-9]*.txt" -exec mv {} {}.bak \; #加bak后缀
find $PWD -name "[0-9]*.txt*" -exec cat {} \; #打印文件全部内容
find . -type f -name "[0-9]*.txt" -exec cat {} \; > all.txt #将结果的文件内容全部输出到all.txt
```

`-ok` 和exec命令一样，只不过这个命令会提示是否确定执行

```bash
find $PWD -name "[0-9]*.txt" -ok mv {} {}.bak \; #加bak后缀
```

​    

## 巨人肩膀

[find命令](https://man.linuxde.net/find)

man page
