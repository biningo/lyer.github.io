---
title: Linux进程
date: 2021-04-30
categories: [操作系统]
tags: [Linux,OS]
draft: true
---

![](https://raw.githubusercontent.com/biningo/cdn/master/2021-04/task_struct.png)

​    ![](https://raw.githubusercontent.com/biningo/cdn/master/2021-04/pcb.png)

## task_struct各个字段

### ID

```c
pid_t	pid; //进程pid
pid_t	tgid; //进程属于哪个task group
```

如果是一个进程中的线程，则他们的`tgid`都指向父进程的`pid`，也就是该进程的主执行流，而父进程`tgid=pid` 

Linux中的线程就是通过`tgid`来实现了，属于同一个组的就是代表是同一个进程下的线程，这样我们就可以知道哪些是进程，哪些是线程了

### 信号

```c
//signal_struct记录了信号处理注册函数相关的结构体  里面记录了每个信号对应的处理函数
struct signal_struct		*signal; 

//哪些信号正在处理
struct sighand_struct		*sighand; 

//sigset_t 是个bitmap
sigset_t			blocked; //哪些信号暂不处理 
struct sigpending		pending; //待处理信号的信号队列 每引发一个信号就加入队列
```

Linux拥有信号处理机制，相当于软件层面的中断吧，信号处理是在用户空间完成的，使用的是用户栈不需要进入内核态，同时信号处理函数需要用户进程自己注册，如果没有注册则使用Linux内核提供的默认处理函数

信号其实就是一个数字，用数字标识一个信号，然后每个信号有对应的`handler`

### 进程状态

```c
volatile long state; //进程当前的状态 其实就是几个宏定义
int exit_state;
unsigned int flags;
```

主要有下面几个状态:

-1表示不可运行，0表示运行，>=1表示非运行状态

```c
/* Used in tsk->state: */
#define TASK_RUNNING                    0 //可运行 就绪态
#define TASK_INTERRUPTIBLE              1 //浅睡眠
#define TASK_UNINTERRUPTIBLE            2 //深睡眠
#define __TASK_STOPPED                  4  //停止
#define __TASK_TRACED                   8
/* Used in tsk->exit_state: */ //退出状态
#define EXIT_DEAD                       16 //进程退出状态
#define EXIT_ZOMBIE                     32
#define EXIT_TRACE                      (EXIT_ZOMBIE | EXIT_DEAD)
/* Used in tsk->state again: */
#define TASK_DEAD                       64
#define TASK_WAKEKILL                   128
#define TASK_WAKING                     256
#define TASK_PARKED                     512
#define TASK_NOLOAD                     1024
#define TASK_NEW                        2048
#define TASK_STATE_MAX                  4096
```

### 进程调度

```c
// 是否在运行队列上
int				on_rq;
// 优先级
int				prio;
int				static_prio;
int				normal_prio;
unsigned int			rt_priority;
// 调度器类
const struct sched_class	*sched_class;
// 调度实体
struct sched_entity		se;
struct sched_rt_entity		rt;
struct sched_dl_entity		dl;
// 调度策略
unsigned int			policy;
// 可以使用哪些 CPU
int				nr_cpus_allowed;
cpumask_t			cpus_allowed;
struct sched_info		sched_info;
```

### 亲缘关系

```c
struct task_struct __rcu	*parent; //父亲
struct list_head		children; //子进程链表头节点
struct list_head		sibling; //兄弟进程链表头节点(父亲进程的children)
struct task_struct		*group_leader; //该进程组的主线程
```

Linux进程可以看成一颗进程树，第一个用户级进程`init`进程`pid=1`，该进程可以完成一些操作系统启动的初始化工作，系统运行起来之后，还有一个作用就是会回收 **回收孤儿进程，清理僵尸进程** 等，此进程是所有用户进程的祖先

第一个内核级进程`kthreadd` `pid=2`

### 进程权限和所属用户

描述一些进程权限，是否有权打开一个文件、访问一个文件等，以及该进程属于哪个用户哪个用户组

### 虚拟内存管理

```c
struct mm_struct   *mm; //记录各个虚拟内存段的起始和结束地址
```

### 文件和文件系统

```c
struct fs_struct                *fs; //进程所处的根文件和文件系统
//进程打开的文件描述符列表 进程打开一个文件就有一个fd文件描述符,0 1 2 stdin stdout stderr
struct files_struct             *files; 
```

进程有属于自己的根文件系统，同时维护了进程本身的打开所有文件`fd`数组，fd其实就是文件打开列表的下标，数组元素都指向操作系统全局文件打开表，这个fd数组每个进程都是独立的，也就是说每个进程能能存在相同的fd，同一个文件可以被多个文件同时打开读写

​    

## 参考

[Linux进程调度](https://qiankunli.github.io/2019/05/01/linux_task_schedule.html)