---
title: Etcd导论
date: 2021-07-22
categories: [Etcd]
tags: [etcd]
draft: true
---

## Etcd原理

Etcd属于CAP中的`CP`，任何时刻无论从哪个节点拿到的数据都是一致的

Etcd保存的key是有序的

​      

## Etcd几大功能模块

- kv存取，根据prefix获取
- lease租约机制
- watch监控机制
- auth用户认证
- snapshot备份和恢复
- endpoint集群管理
- txn事务

​      

## Etcd备份恢复操作

https://zhuanlan.zhihu.com/p/101523337

备份文件

```bash
etcdctl snapshot save ./my.db
```

恢复备份

```bash
etcdctl snapshot restore ./my.db
```

恢复之后就需要重启etcd服务

