---
title: Python并发和GIL锁
date: 2021-01-31
categories: [编程语言/Python]
tags: [并发,Python]
---

## GIL全局解释器锁

在单核时代，Python为了解决多个线程并发访问数据造成数据不安全问题，在语言层面就实现了一种机制，就是给**一个进程中的多个并发线程**设置一把锁，只有抢到锁的线程才可以在CPU上执行，没有锁的线程只能等待。这样就可以控制在同一时刻内对数据的访问只有一个线程（**其实这也无法保证线程安全**）所以这把锁就叫做 **GIL锁**

也就是说Python的多线程并发在单核时代可以有效控制线程安全问题，但是到了多核时代，即使有多个核，同一时刻也只能有一个线程在执行（**因为同一个进程内的多个线程中只有一把GIL锁**）

比如现在有 `a、b、c、d`三个线程和`1、2、3、4`号CPU核，如果是其他语言，则在同一个时刻四个线程可以同时并发的跑在4个核上运行，但是Python因为有了一把**GIL锁** 现在`a`抢到锁了，那么`b、c、d`只能干巴巴的等待`a`主动释放锁才可以继续抢锁才有机会执行，即使有4个核也无法充分利用，所以语言层面上创建了4个线程但最终也相当于串行执行

```python
def my_task():
    i = 0
    for _ in range(10000000):
        i = i + 1

@metric
def f1():
    for t in range(2):
        t = threading.Thread(target=my_task)
        t.start()
        t.join()

@metric
def f2():
    arr = []
    for t in range(2):
        t = threading.Thread(target=my_task)
        t.start()
        arr.append(t)

    for t in arr:
        t.join()
```



​    

## GIL锁真的安全吗？

GIL锁其实并非安全，线程在下面三种情况下回主动释放锁：

- 不间断执行字节码`>1000`
- 执行时间`>15ms`
- `IO`操作

现在有一个全局变量`count=0`，假设`t1`线程拿到`count`准备`count+=1`的时候，`t1`的连续执行时间恰好`>15ms`了或则执行的字节码`>1000`了，此时`t1`就会主动释放锁，被`t2`抢到了，`t2`执行`count+=1`此时的`count==1`，后来`t1`再次执行的时候`count=0，count+=1`，`count==1`，正确的结果应该是`count=2`

也就是说GIL锁并不能百分之百保证线程安全，只有在循环比较短，执行代码比较少的情况下才可以百分之百保证线程安全

```python
count=0
def add_cpu(max_num):
    global count
    for i in range(max_num):
        count+=1

def f5(max_num):
    arr = []
    for t in range(10):
        t = threading.Thread(target=add_cpu,args=(max_num,))
        t.start()
        arr.append(t)
    for t in arr:
        t.join()
if __name__ == '__main__':
    count=0
    f5(1000)  #1000可以保证线程安全
    print(count) #count==1000

    count=0
    f5(100000) #100000无法保证线程安全
    print(count) #count!=100000
```

​    

## 如何避免GIL锁的影响

1. **IO密集型应用建议使用多线程，因为线程比进程轻量**：在`IO`密集型工作上，线程碰到`IO`就会立即释放锁让其他空闲的线程执行，而在CPU密集型则线程需要不断执行，直到达到释放锁的条件，所以 **IO密集型应用**使用多线程的效率还是高的，比如 **爬虫（等待网络IO）** 但是**CPU密集型**效率就不高了
2. 在CPU密集型应用则可以选择 **多进程、协程**代替 **多线程**

下面程序中，因为 **GIL锁**的原因：

- `f1`和`f2`执行时间应该是差不多的（CPU密集型）
- `f4`比`f3`执行时间要短 （IO密集型）

```python
#more CPU task
def my_task():
    i = 0
    for _ in range(10000000):
        i = i + 1

@metric #串行
def f1():
    for t in range(2):
        t = threading.Thread(target=my_task)
        t.start()
        t.join()

@metric
def f2(): #多线程
    arr = []
    for t in range(2):
        t = threading.Thread(target=my_task)
        t.start()
        arr.append(t)

    for t in arr:
        t.join()


#more IO task
def my_sleep():
    time.sleep(1)

@metric
def f3(): 
    for t in range(4):
        t = threading.Thread(target=my_sleep)
        t.start()
        t.join()

@metric
def f4():
    arr = []
    for t in range(4):
        t = threading.Thread(target=my_sleep)
        t.start()
        arr.append(t)
    for t in arr:
        t.join()
```

​    

## Python的协程

TODO

​    

## 参考

https://mp.weixin.qq.com/s/TBiqbSCsjIbNIk8ATky-tg