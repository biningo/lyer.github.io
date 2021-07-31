---
title: MySql索引优化
date: 2021-05-06
categories: [Mysql]
tags: [数据库,MySql]
draft: true
---

## 索引扫描区间

范围查找，会查找到20的第一条记录然后在叶子节点往后遍历直到条件不成立

```mysql
select id,age,name from stu where age>=20 and age<=30
select id,age,name from stu where age<20 or age>30
```

IN查询 `[10,10) [20,20) [30,30)`

```mysql
select id,age,name from stu where age in (10,20,30)
```

!=查找，`(-,10) (10,+)`

```mysql
select id,age,name from stu where age!=10
```

Like模糊匹配，`[a,b)`

```mysql
select id,age,name from stu where name like 'a%'
```

联合索引的搜索区间，age相同的情况下name是有序的，所以可以形成搜索区间

```mysql
select id,age,name from stu where age=20 and name<'aa'
```

​    

## 索引用于order by 排序

索引本来就是有序的，所以当排序字段是索引时就不需要排序了，直接检索出来即可

```mysql
select id,age from stu where age<20 order by age;
```

联合索引排序，因为k1和k2确定的情况下是按照k3排序的，所以下面的sql也免去了排序

```mysql
select * from t where k1='a' and k2='b' order by k3;
```

​    

## 索引用于group by分组

k1是有序的所以直接检索出来即可轻松完成分组

```mysql
select count(*) from t group by k1;
```

​    







## 联合索引

用联合索引减少建立单独索引，利用最左原则有效的利用索引

## 索引下推

索引下推可以先根据索引查询结果过滤，然后将过滤的记录再进行回表，这样可以减少回表的次数

## 覆盖索引

利用覆盖索引特点减少回表查询

## 字符串索引

为字符串建立前缀索引，然后字符串左模糊查询可以利用索引，可以使用字符串倒序存储来有效利用索引

## 分页查询优化



## 建立索引的时机

- where

- order by
- group by

## 减少索引维护

- 索引数据类型不能太大
- 索引最好有序插入

- 主键索引能用小类型的数据就用小类型的数据，并且采用自增主键，这样可以减少插入数据时对B+树的维护动作
- 删除数据最好不要真正的删除，而是作一个删除标记然后在空闲的时间里定时的删除，因为删除数据可能会涉及到索引树的维护

## 参考

[https://www.bilibili.com/video/BV14v411z7M2?p=7](https://www.bilibili.com/video/BV14v411z7M2?p=7)

https://www.bilibili.com/video/BV1uK4y1a7Z6?p=6&t=432