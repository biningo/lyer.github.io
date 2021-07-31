---
title: MySql和SQL总结
date: 2021-02-16
categories: [MySql]
tags: [数据库,MySql] 
draft: true
---

## 系统变量相关操作

查看系统变量和配置

```sql
show variables like 'default_storage_engine'; #session级别
show global  variables like 'default_storage_engine' #global级别
```

设置系统变量和配置

```sql
set default_storage_engine=MyISAM;
set global default_storage_engine=InnoDB;
set @@default_storage_engine=MyISAM;
set @@global.default_storage_engine=MyISAM;
```

查看系统状态变量

```sql
SHOW STATUS LIKE 'threads_connected'
```

查看字符集

```sql
SHOW CHARACTER SET #查看字符集字符集 CHARSET
SHOW COLLATION #查看字符集所有比较规则
```

​     

## 数据库操作

创建数据库指定字符集和比较规则

```sql
create database db2 charset utf8 collate utf8_bin;
```

修改数据库的字符集和比较规则

```sql
alter database db1 charset utf8;
```

删除数据库

```sql
drop database db1;
```

​     

## 表操作

创建表指定字符集、比较规则、存储引擎

```mysql
CREATE TABLE test(
    id int
) ENGINE = InnoDB #设置表的存储引擎
  DEFAULT CHARSET = utf8 #设置表的字符集、
  COLLATE=utf8_bin #设置字符集比较规则
```

修改表

```sql
alter table t1 charset=gbk;
```

其他建表语句

```mysql
DROP TABLE IF EXISTS category,article,tag,article_tag; #删除表
CREATE TABLE IF NOT EXISTS category
(
    id    INT PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(50) NOT NULL,
    UNIQUE uq_title (title)
) ENGINE = InnoDB
  CHARACTER SET = utf8;

CREATE TABLE article
(
    id         INT AUTO_INCREMENT,
    title      VARCHAR(100) NOT NULL,
    created_at DATETIME     NOT NULL,
    updated_at DATETIME     NOT NULL,
    deleted_at DATETIME DEFAULT NULL,
    author     VARCHAR(20)  NOT NULL,
    status     ENUM ('published','auditing','draft','deleted') NOT NULL,
    cid        INT          NOT NULL,

    PRIMARY KEY pk_id (id),
    FOREIGN KEY fk_cid (cid) REFERENCES category (id) ON DELETE CASCADE,
    UNIQUE uq_title (title) COMMENT '会自动创建唯一索引'
) ENGINE = InnoDB
  CHARACTER SET = utf8mb4;

CREATE TABLE tag
(
    id    INT PRIMARY KEY AUTO_INCREMENT,
    title varchar(20) NOT NULL,
    UNIQUE uq_title (title)
);

CREATE TABLE article_tag
(
    id  INT PRIMARY KEY AUTO_INCREMENT,
    tid INT NOT NULL,
    aid INT NOT NULL,
    FOREIGN KEY fk_tid (tid) REFERENCES tag (id),
    FOREIGN KEY fk_aid (aid) REFERENCES article (id),
    UNIQUE uq_tid_aid (tid, aid)
);
```

展示建表语句

```mysql
SHOW CREATE TABLE t1 #展示test表的建表语句
```

展示字段信息

```sql
DESC t1 #展示表的字段信息
```

修改表字段信息，只会部分修改不会全部修改

```sql
alter table t1 modify id varchar(10) primary key auto_increment;
```

在表中删除/添加字段

```sql
alter table t1 add column name varchar(20);
alter table t1 drop column name;
```

​     

## 索引

查看索引

```sql
show index from tb #查看索引
```

建立表时创建索引

```sql
create table tb(
    name varchar(200) NOT NULL ,
    index idx_name(name), #建表时就指定索引
    primary key pk_name(name)
)
```

创建索引

```mysql
#通过CREATE来创建索引
CREATE INDEX index_name ON table_name (column_list)
CREATE UNIQUE INDEX index_name ON table_name (column_list)

#通过ALTER来创建索引
ALTER TABLE table_name ADD INDEX index_name (column_list) #普通索引
ALTER TABLE table_name ADD UNIQUE (column_list) #唯一索引
ALTER TABLE table_name ADD PRIMARY KEY (column_list) #主键索引
```

删除索引

```sql
DROP INDEX index_name ON talbe_name
ALTER TABLE table_name DROP INDEX index_name
ALTER TABLE table_name DROP PRIMARY KEY #删除主键可以不指定索引名字 因为主键只有一个
```

​      

## 约束

请参考 **Mysql约束** 这篇文章

​    

## 事务

```mysql
SHOW VARIABLES LIKE '%AUTOCOMMIT%';
SET AUTOCOMMIT = 0; #取消事务自动提交 on|off
begin;
commit;
rollback;
```

​    

## explain

查看查询语句相信信息

​    

## 内置函数

TODO

## 参考

《SQL学习指南(第二版修订版)》

《MySql是怎样运行的》

《MySql必知必会》

《高性能MySql(第三版)》



