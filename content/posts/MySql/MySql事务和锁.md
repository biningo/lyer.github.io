---
title: MySql事务和锁
date: 2021-05-23
categories: [Mysql]
tags: [数据库,MySql]
---

## 为什么需要有事务

为了解决数据的一致性，保障数据的正确

​      

## 事务四大特性ACID

**原子性**  `Atomicity` 

事务是数据库的逻辑工作单位，不可分割，事务中包含的各操作要么都做，要么都不做

**一致性** `Consistency`

一致性是指事务将数据库从一种一致性状态变为下一种一致性状态，在事务开始之前和之后，数据库的完整性约束没有被破坏

比如转账的例子，两个账户的钱款总量在转账前后还是一致的，如果B余额增加了但是A余额没有扣除那么就会凭白无故多出一部分钱来，这就引发钱款不一致的状况了

再比如A的余额为0了，此时A还要给别人转账就不合适了，因为余额必须`>=0`。如果余额出现负数，这也导致了数据不一致状况

还有年龄不能为负数，红绿灯只有三种颜色，人民币最大面值为100等，这些约束都必须要符合真实世界的情况，都必须和真实世界保持一致性

**隔离性** `Isolation` 

每个事务都是隔离的，互相不影响，一共有`4`个隔离级别

不同事务在提交的时候，必须保证最终呈现出来的效果是串行的。比如两个顾客同时买一件衣服，衣服总量只有2件了。此时顾客A买了1件提交了，顾客B买了1件也提交了。那么最终的衣服总量必须减少为0，而不是1。如果在RR隔离级别下，顾客A和B在事务没提交之前看见的衣服总量都为2，因为他们是同时开启事务的，假设A先提交了，但是B看到的余量依旧是2件(RR隔离级别)，当B再买并且提交之后，则必须还剩余0件，而不是剩余1件，必须保障数据的一致性。也就是说虽然他们看起来是并行执行的事务，但是最终的效果一定要是串行的效果。

**持久性** `Durability`

事务一旦提交，它对数据库中的数据的改变就应该是永久性的，必须持久化到磁盘

​      

## MySql事务的实现

- 原子性: `undo log` 实现 (记录了事务修改之前的数据，当事务在执行过程中如果失败了那么当前事务就处于不一致的状态，这样可以回滚到上一个一致性的状态)
- 持久性: `redo log` 实现
- 隔离性: 锁+`MVCC`  实现
- 一致性: 通过原子性、持久性、隔离性实现

​      

## 隔离性和四大隔离级别

### 1、读未提交

`Read Uncommitted`

一个事务还没提交时，它做的变更对他事务可见

- 脏读
- 不可重复读
- 幻读

### 2、读已提交

`Read Committed` **Oracle默认的隔离级别**

一个事务只有提交时它做的变化才对其他事务可见，该级别会造成 **在事务中两次读取数据不一致的情况，也就是不可重复读**

- 不可重复读
- 幻读

### 3、可重复读

`Repeatable Read` **MySql、Innodb默认的隔离级别**

一个事务开始之后，其所看见的数据就是事务开始时候的数据，相当于给数据在事务开始时拍一个快照，在事务执行过程中看见的都是这个快照，即使是其他事务做了变更提交了对此事务也不可见

**Mysql的RR隔离级别下不会出现幻读问题**

### 4、串行化

事务必须串行化执行，一个事务只有等另外一个事务结束之后才可以开始，效率最低不支持并发，但是解决了事务隔离性的所3有问题

```sql
--锁住所有查询出来的行，因为下面这局查询了所有的行
--其他事务只能select，其他操作会阻塞，如果此时执行插入那是可以的，但是其他事务如果需要读取新插入的行则会阻塞
select * from stu

--锁行，其他事务无法对这行修改但是可以读，对于其他行则同时可以读或写
select * from stu where id=1

--锁行，其他事务无法对这行读或则写
update stu set name='A' where id=1 
```

​      

## 事务并发读写问题

### 1、脏写

> **一个事务修改了其他事务修改过的记录，就发生了脏写**

在任何隔离级别下都不允许发生脏写，也就是说如果两个事务操作了同一条记录则必须加锁来保障数据一致性

| 事务A                                                        | 事务B                                                 |
| ------------------------------------------------------------ | ----------------------------------------------------- |
| `update stu set name='A' where id=1;`  此时事务A会将这行锁住 |                                                       |
|                                                              | `update stu set name='AA' where id=1;`  此时事务B阻塞 |
| `commit`  --此时事务A会释放行锁                              |                                                       |
|                                                              | `commit`事务B得以执行                                 |

案例: 卖衣服

衣服数量为100，下面两个同时开启的并行事务A、B

事务A查询余量为100，于是买了100件 

```bash
update clothes set count=count-100 where count>=100;
```

事务B查询余量为100，于是也买了100件

```bash
update clothes set count=count-100 where count>=100;
```

此时衣服余量就会变为负数，为了防止此情况，事务B在`update`的时候必须阻塞等待事务A的提交，事务A提交之后余量变为了0，此时事务B再次执行update时由于`count=0`就会查询到余量不足，update影响了0条数据，以此防止了数据不一致的情况

### 2、脏读

**事务读取到了其他事务还没提交的修改**

`A`事务执行过程中，`B`事务做出的变化还没提交修改就被A事务读取到了，但是`B`事务的修改发生了回滚，那么`A`事务就读取到了脏数据，也就是说`A`事务查询出来的数据是不正确的

```bash
mysql> select * from clothes where id=1;
+----+-------+
| id | count |
+----+-------+
|  1 |   500 |
+----+-------+
```

| 事务A                                                       | 事务B                                                        |
| ----------------------------------------------------------- | ------------------------------------------------------------ |
| `select count from clothes where id=1`  count=500           |                                                              |
|                                                             |                                                              |
| `update clothes set count=count-100 where id=1;`  count=400 |                                                              |
|                                                             | `select count from clothes where id=1` count=400 ，将脏数据400返回了 |
|                                                             | `commit`                                                     |
| `rollback`  事务回滚，count=500                             |                                                              |

### 3、不可重复读

**事务读取到了其他事务已经提交的修改，同一个事务对于一条记录的两次查询结果出现不一致现象**

`A`事务在执行过程中读到了`B`事务已经提交了的修改，造成`A`事务在执行过程中两次读取数据不一致

> https://www.nowcoder.com/discuss/204259?type=1&order=3&pos=551&page=1

银行做活动例子:

事务a查询某地区余额`1000`以下送一包餐巾纸，生成名单

事务b小明余额`500`，存了`1000`，变成`1500` 

事务a查询`1000~2000`送一桶油，生成名单，这样小明收到了2个礼品

这里两次重复读取小明的记录，并且记录数据不一样，一次500，一次1500

理论上小明只能收到一个礼品，要么是餐巾纸要么是油，在**可重复读**的情况下则小明只能收到餐巾纸，因为小明充钱事务在活动事务开启之后则不能参与此次活动

### 4、幻读

幻读是指A事务在执行过程中查询一个范围的记录前后记录查询的结果不一样，会多出一行或则少掉一行，会出现幻行，Mysql通过**间隙锁和MVCC**解决了RR隔离级别下的幻读问题。其实幻读也是由不可重复读造成的，因为在同一个事务中读取到了其它提交的事务的数据

RR隔离级别下会发生幻读问题，因为添加一条数据更本不属于修改数据的范畴，RR级别下只是保证了其它事务对原有的数据修改不会被当前事务给查询到，并不能保证其他事务的插入数据不被当前事务查询。只是MySql通过**MVCC和锁**机制解决了RR隔离级别下的幻读问题

- 使用MVCC保证了事务始终读取的都是事务开启时的数据快照

- 使用间隙锁机制防止在当前事务执行过程中，其他事务进行插入操作而造成幻读

因为就单单MVCC并不能保证其他事务的插入操作不被当前事务读取，既然不能保证那么就加锁阻止其他事务插入，这样就避免了幻读问题，降低了并发性，另外的事务必须阻塞等待当前事务提交才可以成功执行插入操作

**案例一** 银行做活动，并行事务`a、b、c` （MVCC防止幻读）

事务a查询某地区余额1000以下送一包餐巾纸 生成名单 

事务b新增了一个新用户小明，并存款500，并提交

事务c新增了一个新用户小李，并存款1500，并提交

事务a查询`1000~2000`送一桶油，生成名单，这样小明没有收到礼物，而同时注册的小李存了1500却收到了一桶油

**案例二** 新用户赠送VIP  （间隙锁防止幻读）

```sql
create table user( 
	id int primary key auto_increment,
    name varchar(20), --用户名
    rtime datetime,  --注册时间
    is_vip tinyint(1) --是否是VIP
);
```

假设有如下一个业务场景: 

一个视频网站搞活动，想要给最近7天内注册的新用户免费赠送一个月的会员，并且生成一条消息提示用户，提醒他已经升级为VIP了。如果在赠送VIP事务进行的同时又新注册了一个新用户，则在RC隔离级别下新用户会收到一条诡异的消息告诉他已经升级为VIP了，但是发现自己却还不是VIP

> 一下业务场景灵感来源此帖子https://www.v2ex.com/t/440873

```sql
--插入3个新用户
insert into user(name,rtime,is_vip) value('A',now(),0);
insert into user(name,rtime,is_vip) value('B',now(),0);
insert into user(name,rtime,is_vip) value('C',now(),0);

--插入3个老用户
insert into user(name,rtime,is_vip) value('D','2019-05-25',0);
insert into user(name,rtime,is_vip) value('E','2020-05-25',0);
insert into user(name,rtime,is_vip) value('F','2020-05-25',0);
```

```sql
+----+------+---------------------+--------+
| id | name | rtime               | is_vip |
+----+------+---------------------+--------+
|  1 | A    | 2021-05-25 14:35:29 |      0 |
|  2 | B    | 2021-05-25 14:35:32 |      0 |
|  3 | C    | 2021-05-25 14:35:34 |      0 |
|  4 | D    | 2019-05-25 00:00:00 |      0 |
|  5 | E    | 2020-05-25 00:00:00 |      0 |
|  6 | F    | 2021-01-01 00:00:00 |      0 |
+----+------+---------------------+--------+

--查询近7天内的数据
SELECT * FROM user WHERE date_sub(curdate(), interval 7 day) <= date(rtime); 

--更新操作
update user set is_vip=1 where date_sub(curdate(), interval 7 day) 
 <= date(rtime); --3
```

| 事务A (升级VIP)                                              | 事务B (新用户注册)                                           |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `update user set is_vip=1 where date_sub(curdate(), interval 7 day) <= date(rtime);` 赠送新用户VIP （没有使用索引字段则会锁所有的行来防止幻读） |                                                              |
|                                                              | `insert into user(name,rtime,is_vip) value('G',now(),0);` 新用户注册 （如果是在RR隔离级别则在此行会因为事务A加了间隙锁而被阻塞，直到事务A提交，防止了**幻读**现象）在RC隔离级别下此命令不会阻塞 |
|                                                              | `commit`                                                     |
| `SELECT id FROM user WHERE date_sub(curdate(), interval 7 day) <= date(rtime);` 在RC隔离级别下，可以查询到事务B新注册的用户ID，此时如果按照此ID集合发消息则事务B新注册的用户也会收到已经升级为VIP的消息，但是事务B注册的新用户却还不是VIP。在RR隔离级别下则查询不到事务B新注册的用户，则业务是正常的 |                                                              |
| `commit`                                                     |                                                              |

​     

## RR隔离级别案例演示

在RR隔离级别下查询的数据始终是和事务开启时是一致的，更新删除等操作则是操作最新的数据，这样才可以保障数据一致性

### 1、案例一

```bash
表a
+-------+------+------+-----+---------+-------+
| Field | Type | Null | Key | Default | Extra |
+-------+------+------+-----+---------+-------+
| id    | int  | YES  |     | NULL    |       |
+-------+------+------+-----+---------+-------+
```

假设`a`表初始数据为0条

| 事务A                     | 事务B                                                        |
| ------------------------- | ------------------------------------------------------------ |
| `insert into a value(1);` | `select count(*) from a;` 结果是0条                          |
| `commit;`                 |                                                              |
|                           | `select count(*) from a;` 结果是0条，杜绝了幻读问题          |
|                           | `update a set id=99;` 结果提示影响一行数据，因为这里必须基于最新的数据来操作 |
|                           | `select * from a`  显示99这行记录                            |
|                           | `select count(*) from a;` 此时`count=1`                      |
|                           | `rollback` 此时如果执行回滚 `id=1`又回滚到了事务A提交时的状态，相当于B事务没有发生一样 |

### 2、案例二

```bash
mysql> desc stu;
+-------+-------------+------+-----+---------+----------------+
| Field | Type        | Null | Key | Default | Extra          |
+-------+-------------+------+-----+---------+----------------+
| id    | int         | NO   | PRI | NULL    | auto_increment |
| name  | varchar(20) | YES  |     | NULL    |                |
+-------+-------------+------+-----+---------+----------------+
mysql> select * from stu;
+----+------+
| id | name |
+----+------+
|  1 | A    |
|  2 | A    |
|  3 | A    |
|  4 | A    |
|  5 | B    |
|  6 | B    |
|  7 | B    |
+----+------+
```

| 事务A                                         | 事务B                                                        |
| --------------------------------------------- | ------------------------------------------------------------ |
| `update stu set name='B' where id<=2;`        |                                                              |
| `select count(*) from stu where name='B';`  5 | `select count(*) from stu where name='B';`  3                |
| `commit`                                      |                                                              |
|                                               | `select count(*) from stu where name='B';`  3  ，在RR级别下还是3，无法看见A提交的更改，**解决了不可重复读的问题** |
|                                               | `update stu set name='C' where name='B';` 提示影响了5条数据，刚刚查询的还是3条数据，这里却影响了5条数据，因为这里读取到了A事务的更改，且必须读到，否则会造成数据的不一致问题 |
|                                               | `select count(*) from stu where name='C';` 5                 |
|                                               | `commit;`                                                    |

```bash
mysql> select * from stu;
+----+------+
| id | name |
+----+------+
|  1 | A    |
|  2 | A    |
|  3 | A    |
|  4 | A    |
|  5 | B    |
|  6 | B    |
|  7 | B    |
+----+------+
```

| 事务A                                          | 事务B                                                        |
| ---------------------------------------------- | ------------------------------------------------------------ |
| `select count(*) from stu where name='B'`   3  | `select count(*) from stu where name='B';`  3                |
| `insert into stu(name) value('B')`             |                                                              |
| `select count(*) from stu where name='B' `   4 | `select count(*) from stu where name='B';`  3                |
| `commit`                                       |                                                              |
|                                                | `select count(*) from stu where name='B';`  3 ，**杜绝了幻读问题** |
|                                                | `update stu set name='C' where name='B'` 提示影响4条数据     |
|                                                | `select count(*) from stu where name='C';`  4                |
|                                                | `select count(*) from stu` 8  此时数据的总条数也从7变成了8   |
|                                                | `commit`                                                     |

### 3、案例三

```bash
+----+------+
| id | name |
+----+------+
|  1 | A    |
|  2 | A    |
|  3 | A    |
|  4 | A    |
|  5 | C    |
|  6 | C    |
|  7 | C    |
|  8 | C    |
+----+------+
```

| 事务A                                                 | 事务B                                                        |
| ----------------------------------------------------- | ------------------------------------------------------------ |
| `select count(*) from stu where name='A'` 4           | `select count(*) from stu where name='A'` 4                  |
| `update stu set name='B' where name='A';` 影响4条数据 |                                                              |
|                                                       | `select count(*) from stu where name='A'` 4 ，事务A没提交，所以还是4条 |
| `commit`                                              |                                                              |
|                                                       | `select count(*) from stu where name='A'`4 ，在RR隔离级别下事务A提交了但是对事务B没有影响，还能查到4条记录 |
|                                                       | `update stu set name='D' where name='A';` 影响0条数据，因为`name='A'`的记录已经不存在了 |
|                                                       | `select count(*) from stu where name='A'` 在RR隔离级别下还能查到4条记录，这4条记录是事务开启前的数据的快照，此时这4条记录只能查不能改，因为真实的情况下并不存在`name=A`的记录，`name=A`的记录已经被修改了 |
|                                                       | `commit`                                                     |

### 4、案例四

> 下面的现象不管在RR还是RC隔离级别上面都会出现
>
> 参考 [如果不能满足可重复读，有啥实际危害的例子？](https://www.v2ex.com/t/440873)

更新丢失问题

```bash
事务A: 老公去银行取钱(开启事务A)
事务B: (开启事务B)同时老婆在家里查询了下余额，发现余额还够
事务B: 于是在手机上下单买口红(提交了事务)
事务A: 老公取钱,银行会在卡上减金额
事务A: 显示余额，发现余额多扣了，老公就很疑惑: 明明只取了100怎么多扣了1000?

老公的最后显示余额的操作就无法看成是取款之后的余额了,要看成两件事情:取钱+买口红之后的余额
实际上数据还是一致的,钱余额还是一致的,这也是为了保证数据的一致性
```

```bash
+----+------+-------+
| id | name | money |
+----+------+-------+
|  1 | A    |  1000 |
---------------------
```

| 老公取钱(事务A)                                              | 老婆买口红(事务B)                                            |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `select * from account where id=1;` 发现余额为1000，准备取200 | `select * from account where id=1;` 发现余额为1000，可以买口红 |
|                                                              | `update account set money=money-500 where id=1;` 买口红减500，此时余额为500 |
|                                                              | `commit;`                                                    |
| `select * from account where id=1;` 在RR下余额为1000，在RC下余额为500 |                                                              |
| `update account set money=money-200 where id=1;` 取200       |                                                              |
| `select money from account where id=1;` 查询余额发现余额怎么变成了300，老公疑惑: 我明明才取了200怎么扣我700 (事后回家问老婆才发现老婆买了口红) |                                                              |
| `commit`                                                     |                                                              |

​    

## 什么时候代表事务结束

- `commit`提交事务的时候
- 再次执行`begin`开启事务时，如果上一个事务还没有提交则会进行自动提交

- `rollback`执行时则也代表此次事务结束了
- 出现`DDL`语句时也会提交事务 (`alter/drop table`等操作)

​        

## Mysql事务命令

```mysql
SHOW VARIABLES LIKE '%AUTOCOMMIT%';
SET AUTOCOMMIT = 0; #取消事务自动提交 on|off

SHOW VARIABLES LIKE 'transaction_isolation';  #查询当前的事务隔离级别
#修改系统的隔离级别  每次修改之后需要重新连接服务器才能生效
# read uncommitted 读未提交
# read committed 读已提交
# repeatable read 可重复读 (MySql默认的隔离级别)
# serializable 串行化
set global transaction isolation level repeatable read;

begin;
commit;
rollback;
```

​      

## InnoDB锁的分类

### 1、读写锁(共享锁和排他锁)

**读锁(S锁、共享锁)**

对于未加修饰的`select`语句则不会加任何锁，我们也可以显示的在select后面加上`share mode`表示加上读锁

InnoDB是支持行锁的，下面展示只在行上面手动加S锁，事务提交之后自动释放

```sql
--此时其他事务不能对这行进行修改操作，但还可以继续加S锁或普通的select
select * from stu where id=1 lock in share mode
```

表锁

```sql
--session1
LOCK TABLE stu READ;
select * from stu; --允许读
update stu set name='A' where id=1; --报错 不允许写

--session2
select * from stu; --允许读
update stu set name='A' where id=1; --阻塞 直到读锁释放

--session1
unlock tables; --此时session2的写操作得以执行
```

**写锁(X锁、排他锁)**

对于`update、insert、delete`语句会默认会加上写锁，普通`select`也可以加上`for update`显示的加上X锁

```sql
--select语句加X锁 此时其它事务不能对这行进行修改操作，也不能对这行加上S锁，否则阻塞
select * from table where id=1 for update 
```

```sql
--此update会自动在这行加上X锁，如果其他事务也更改这一行则会阻塞
update stu set name='A' where id=2;
```

表锁

```sql
--session1
LOCK TABLE stu WRITE，school WRITE; --同时给两张表加读锁

--session2
select * from stu; --读操作阻塞

--session1
unlock tables; --此时session2读操作得以执行
```

### 2、悲观锁和乐观锁

**悲观锁**

悲观锁会在每次执行之前先加锁，适合冲突大、并发量大的场景

```sql
--事务A
select * from stu for update;  --for udpate就表示加锁了

--事务B
select * from stu for update; --阻塞直到事务A提交 
```

**乐观锁**

在每次去拿数据的时候认为别人不会修改，不对数据上锁，但是在提交更新的时候会判断在此期间数据是否被更改，如果被更改则提交失败

乐观锁适合并发小，冲突小的场景，如果在冲突大的场景下每次执行完毕之后判断出已经被别人修改了则相当于白白执行了一次，会比较浪费资源

可以加一个`version`字段来实现乐观锁

```sql
--事务A
UPDATE stu SET age=age-10,version =version+1 WHERE version=0;
--事务B
UPDATE stu SET age=age-10,version=version+1 WHERE version=0;

--先执行的事务则成功，后执行的事务则条件判断失败,因为version已经被修改了
```

### 3、间隙锁

修改了一个范围的数据，则将这个范围边缘的数据都锁住不让插入不让修改，这样可以防止幻读现象

​     

## 事务隔离级别和锁

### RC隔离级别下和锁

住RC隔离级别下，只对修改过的行加锁

### RR隔离级别下和锁

RR隔离级别为了防止幻读问题，不仅仅只是对修改过的行加锁，在修改过的行的边缘或则间隙加锁，也就是说不允许在修改过的行中执行插入边缘行或则间隙行。这样就保证了在修改过程中不会有其他行插入而造成幻读问题

### 索引以及表锁行锁

当使用**普通索引**字段或则使用**唯一约束字段**为`where`时只会锁行

```sql
--事务A
update stu set name='E' where id=5; --锁行

--事务B
update stu set name='F' where id=6; --不会阻塞 (如果也是修改id=5则会阻塞)
```

而使用非索引字段当`where`条件则会进行全表扫描，此时就会锁住所有行

```sql
--事务A
update stu set name='B' where name='BB';   --锁表

--事务B
update stu set name='C' where id=3; --阻塞
```

### 范围锁

查询指定一个范围时也会造成此范围内的记录锁定，不在此范围内则不会加锁

```sql
--事务A
update stu set name='A' where id<5;

--事务B
update stu set name='G' where id=7; --不会阻塞
update stu set name='a' where id=1; --阻塞 直到A提交
```

​    

## 参考

[鲁班学院-周瑜MySql-索引事务和锁](https://www.bilibili.com/video/BV1uK4y1a7Z6?p=8) 【不错】

[鲁班学院-周瑜Mysql全解](https://www.bilibili.com/video/BV1W64y1u761?p=9) 【不错】

[一文彻底读懂MySQL事务的四大隔离级别](https://juejin.cn/post/6844904115353436174) 【写的很详细】

[mysql事务及锁机制](https://www.bilibili.com/video/BV13E411q7oV?from=search&seid=12578289313986651427)

[MySql数据库知识点之索引、事务、锁原理与用法解析，彻底扫清mysql知识盲区](https://www.bilibili.com/video/BV1uK4y1a7Z6?p=6&t=432)

[关于幻读，可重复读的真实用例是什么？](https://www.zhihu.com/question/47007926)

[有没有大佬知道数据库不可重复读有什么危害](https://www.nowcoder.com/discuss/204259?type=1&order=3&pos=551&page=1)

[如果不能满足可重复读，有啥实际危害的例子？](https://www.v2ex.com/t/440873)

[『MySQL』深入理解事务的来龙去脉](https://juejin.cn/post/6844903827611582471)

[MySQL数据库InnoDB引擎行级锁锁定范围详解](https://segmentfault.com/a/1190000013307132)

