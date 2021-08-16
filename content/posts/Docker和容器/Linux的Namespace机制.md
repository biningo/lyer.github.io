---
title: Linux的Namespace机制
date: 2021-08-12
categories: [Docker和容器]
tags: [docker,容器]
---

## 几个重要的Namespace

| Namespace          | 系统调用参数  | 作用                                               |
| ------------------ | ------------- | -------------------------------------------------- |
| Mount  namespaces  | CLONE_NEWNS   | 隔离挂载点(隔离文件系统),挂载之后才会生效          |
| UTS namespaces     | CLONE_NEWUTS  | 隔离hostname                                       |
| IPC namespaces     | CLONE_NEWIPC  | 隔离IPC，只有在同一个Namespace下的进程才能相互通信 |
| PID namespaces     | CLONE_NEWPID  | 隔离进程 PID，两个namespace之间pid可以重复         |
| Network namespaces | CLONE_NEWNET  | 隔离网络设备、端口，有独立的协议栈                 |
| User namespaces    | CLONE_NEWUSER | 隔离用户和用户组                                   |

后面的内核版本又新加入了以下namespace

| Namespace        | 作用                |
| ---------------- | ------------------- |
| Cgroup namespace | 隔离 Cgroups 根目录 |
| Time Namespace   | 隔离系统时间        |

​      

## 相关的系统调用

`clone` 用于创建一个进程，这里可以传入参数来设置namespace

`unshare` 让一个进程脱离一个namespace

`setns` 让一个进程加入一个namespace

​     

## namespace案例

我们可以用go、c等语言实现创建namespace等，这里用Linux自带的`unshare`命令来实现一下

```bash
unshare --pid --fork --mount-proc /bin/bash
```

然后就会进入一个隔离的pid namespace中，此时可以发现bash程序的pid=1，可以在外部查看这个bash的pid，然后进入`/proc/<pid>/ns`中执行`ls -l`查看namespace id和外部不一样

要验证其他namespace都可以使用`unshare`命令来验证

```bash
unshare --mount --fork /bin/bash
unshare --uts --fork /bin/bash
unshare --ipc --fork /bin/bash
unshare --user -r /bin/bash
unshare --net --fork /bin/bash
```

​      

## 相关的文件系统

`/proc/<pid>/ns`

​     

## chroot命令和Mount Namespace





## Linux内核实现

Linux为了用namespace进行隔离进程视图那么首先需要知道这个进程属于哪个namespace，所以在`task_struct`中有下面这个字段用来表示此进程属于的namespace

```c
struct task_struct {
......
	/* Namespaces: */
	struct nsproxy	*nsproxy;
......
}
```

这个`nsproxy`的结构体里面持有各个namespace的指针

```c
struct nsproxy {
	atomic_t count;
	struct uts_namespace *uts_ns;
	struct ipc_namespace *ipc_ns;
	struct mnt_namespace *mnt_ns;
	struct pid_namespace *pid_ns_for_children;
	struct net 	     *net_ns;
	struct time_namespace *time_ns;
	struct time_namespace *time_ns_for_children;
	struct cgroup_namespace *cgroup_ns;
};
```

然后如果需要fork子进程则会判断clone函数是否传入的参数，如果没有传入参数则子进程就会复制父进程的namespace和父进程处于同一个namespace下

​     

## 巨人肩膀

[DOCKER基础技术：LINUX NAMESPACE（下）](https://coolshell.cn/articles/17029.html)

[DOCKER基础技术：LINUX NAMESPACE（上）](https://coolshell.cn/articles/17010.html)

[5分钟快速了解Docker的底层原理](https://mp.weixin.qq.com/s/d4VHKyGiCIHwrau74B4tPw)