---
title: MySql备份与恢复
date: 2021-05-09
categories: [MySQL]
tags: [数据库,MySql,备份] 
---

## mysqldump备份与恢复

备份所有数据库`--all-databases`

```bash
#备份所有数据库
mysqldump -uroot -p --all-databases > ./all.sql
```

备份指定的数据库

```bash
#备份d1、d2 库
mysqldump -uroot -p --databases d1 d2   > ./db.sql
```

备份指定数据库中的表，恢复时必须use到指定的数据库下

```bash
#备份test库中的a b c 表
mysqldump -uroot -p --databases test --tables a b c   > ./db.sql
```

恢复数据

```bash
mysql> source /home/lyer/tmp/temp/db.sql
```

​      

## MySql备份脚本

```bash
#!/bin/bash
# -------------------------------------------------------------------------------
# FileName:    mysql_backup.sh 
# Describe:    Used for database backup
# Revision:    1.0
# Date:        2020/08/11
# Author:      wang

# 设置mysql的登录用户名和密码(根据实际情况填写)
mysql_user = "root"
mysql_password = "yourpassword"
mysql_host = "localhost"
mysql_port = "3306"
backup_dir = /data/mysql_backup

dt=date +'%Y%m%d_%H%M'
echo "Backup Begin Date:" $(date +"%Y-%m-%d %H:%M:%S")

# 备份全部数据库
mysqldump -h$mysql_host -P$mysql_port -u$mysql_user -p$mysql_password -R -E --all-databases --single-transaction > $backup_dir/mysql_backup_$dt.sql

find $backup_dir -mtime +7 -type f -name '*.sql' -exec rm -rf {} \;
echo "Backup Succeed Date:" $(date +"%Y-%m-%d %H:%M:%S")
```

​     

## binlog恢复

一般通过备份恢复的数据还是不全的，全量备份之后的数据可以通过`binlog`来恢复，`binlog`的作用有以下几个

- 主从同步（`master`发送自己的`binlog`到从机，从机通过此日志进行同步）
- 数据恢复（使用`mysqlbinlog`工具来恢复数据）

从指定时间点和结束时间点开始，恢复这段时间内的数据

```bash
mysqlbinlog --start-datetime="2021-05-09 22:34:00" --stop-datetime="2021-05-09 22:36:00" --database=test binlog.000001 | mysql -uroot -p
```

按`position`偏移位置恢复数据

1. 查看当前position

```sql
show master status;
```

![](https://raw.githubusercontent.com/biningo/cdn/master/2021-04/binlog-1.png)

2. 删除数据并且恢复

```sql
mysqlbinlog   --start-position=5671  --stop-position=5973  $MYSQL_HOME/data/binlog.000018  > tag.sql
```

```bash
mysql> source /dxxxx/tag.sql
```

​      

## 参考

[MySQL备份脚本，应该这么写](https://juejin.cn/post/6860665123796287501)

[mysql数据恢复，binlog详解](https://juejin.cn/post/6844903897249628174#heading-4)