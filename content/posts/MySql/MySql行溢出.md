---
title: MySql行溢出
date: 2021-05-03
categories: [Mysql]
tags: [数据库,MySql] 
draft: true
---

## MySql单行大小

MySql规定一行数据记录最大长度不能超过`65535byte`，单行记录的大小为这行的行头信息大小和所有列最大能保存的大小加起来的总和（这里计算排除TEXT，BOLB类型)）

下面我使用ASCII编码创建一个表，为了方便里面只设置一个varchar字段，下面的建表是不成功的，虽然`<=65535` 但是一行除了记录字段数据之外还会有额外的行记录头部信息等

```sql
create table test(name varchar(65535)) charset=ASCII;
```

下面修改为`65532`则刚好能创建表

```sql
create table test(name varchar(65532)) charset=ASCII;
```

因为这里只有一个字段，并且可以为NULL，记录varchar长度需要2字节，所以这里行记录必须保存`3byte`的头部信息，所以varchar实际数据只能存储`65535-3=65532byte`的长度了，因为记录头信息固定为5byte，所以不会计算在行记录最大大小65535之内

​     

## 行溢出

同时如果一行记录超过了一个页的大小`16kb`则会产生行溢出，也就是说这行记录会保存在多个数据页中，这就是行溢出

同时Mysql规定一页必须存放至少`2`条数据，产生溢出之后就只存前756字节，同时还会保存下一页的地址