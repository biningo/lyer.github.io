---
title: MySQL三大日志redolog、undolog、binlog
date: 2021-08-08
categories: [Mysql]
tags: [数据库,MySql] 
draft: true
---

## redo log

属于`InnoDB`存储引擎特有的日志，记录的是数据页的物理变更。InnoDB为了提高读写效率在内存中有数据页缓存，更新操作时为了减少磁盘的IO所以会先更新内存中的数据页，等适当的时机再将内存中的数据页持久化到磁盘，所以为了保障内存中的数据页在断电之后不丢失就产生了`redo log`

随着事务操作的执行就会生成redo Log，也就是说不是在事务提交时才记录日志的，在事务提交时会将产生Redo Log写入Log Buffer内存缓存区，并不是随着事务的提交就立即将日志进行持久化保存

日志持久化磁盘的策略可以通过下面参数进行配置: 

```bash
innodb_flush_log_at_trx_commit=2
```

`0` 每秒将内存缓存区的日志直接刷新到磁盘而不经过操作系统缓存，由后台线程每隔1秒执行一次。此策略不管是MySQL挂了还是服务器挂了都可能丢失一秒内的事务数据

`1` 每次事务提交都将内存缓存区的日志直接刷到磁盘而不经过操作系统缓存，此方法能保证完全不丢失数据但是磁盘IO消耗较大

`2` 每次事务提交都将内存缓存区的日志保存并由操作系统决定什么时候刷盘，同时每秒都由后台线程将日志通过`fsync`强制操作系统刷盘。这种是个折中的方案，如果MySQL挂了但是服务器没挂依然可以保证已经提交了的日志数据不丢失

当内存中的脏页写入到磁盘之后，Redo Log 的使命也就完成了，Redo Log占用的空间就可以重用（被覆盖写入）

​     

## binlog

属于`MySQL Server`层的日志，不管使用的是什么存储引擎都有binlog，记录了数据的逻辑变更语句。binlog会在整个事务都提交之后才会记录这个事务的所有操作到日志中，这和redo log是不同的

​     

## redolog和binlog的区别

- redo log属于Innodb特有，binlog属于Server层
- redo记录的是物理日志只记录对哪个页做了什么修改，binlog记录的是SQL逻辑日志某行要改为XXX
- redo日志空间有限采用循环写的方式，binlog一个日志文件写满了会开辟另外一个继续写不会覆盖原来旧日志
- redo log主要用于宕机之后内存数据的恢复，而binlog不仅仅可以用于数据恢复还可以用于主从数据同步
- redo log会记录事务执行过程中所有的变更，而binlog只有在事务提交之后才会记录这次事务所做的变更

​       

## log两阶段提交

由于redo log和binlog属于不同的部分，所以必须要保证redo log和binlog记录的数据一致。如果redo log已经提交了事务的数据没有被记录到binlog中也就是两个日志出现了数据不一致的情况，那么就会在主从同步的时候或则使用binlog做数据恢复的时候就会造成数据丢失的情况

先看两种情况

1. **先写redo log后写binlog**  假设在redo log写完，binlog还没有写完的时候，MySQL进程异常重启。由于我们前面说过的，redo log写完之后，系统即使崩溃，仍然能够把数据恢复回来，所以恢复后这一行c的值是1。 但是由于binlog没写完就crash了，这时候binlog里面就没有记录这个语句。因此，之后备份日志的时候，存起来的binlog里面就没有这条语句。 然后你会发现，如果需要用这个binlog来恢复临时库的话，由于这个语句的binlog丢失，这个临时库就会少了这一次更新，恢复出来的这一行c的值就是0，与原库的值不同。
2. **先写binlog后写redo log** 如果在binlog写完之后crash，由于redo log还没写，崩溃恢复以后这个事务无效，所以这一行c的值是0。但是binlog里面已经记录了“把c从0改成1”这个日志。所以，在之后用binlog来恢复的时候就多了一个事务出来，恢复出来的这一行c的值就是1，与原库的值不同。

两阶段提交的过程:

1. 客户端发送执行更新、插入语句的命令
2. MySQL将语句执行，把修改的语句更新到内存的数据页Buffer中
3. 记录redo log，并将记录状态设置为`prepare`
4. 通知Server，已经修改好了，可以提交事务了
5. 将更新的内容写入binlog并刷新到磁盘，同时将redo log修改为`commit`状态

总结起来就是先将redo log状态设置为prepared，等更新内容写入binlog后，在将redo log状态设置为commited

​     

## undo log

undo log主要记录了数据的逻辑变化，比如一条INSERT语句，对应一条DELETE的undo log，对于每个UPDATE语句，对应一条相反的UPDATE的undo log，这样在发生错误时，就能回滚到事务之前的数据状态

​    

## 三大日志和事务的关系

- undo log保障事务的原子性，要么全部成功，要么全部失败
- redo log和 binlog保障了事务的持久性，提交了的事务必须要持久化到磁盘

​    

## 巨人肩膀

[[MySQL学习笔记]二.redo log/binlog/两阶段提交](https://juejin.cn/post/6844904199952531463)

[MySQL Redo Log 重做日志](https://segmentfault.com/a/1190000039698853)