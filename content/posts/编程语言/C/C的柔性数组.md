---
title: C的柔性数组
date: 2021-05-01
categories: [编程语言/C]
tags: [C]
---

## 什么是柔性数组

> **结构中最后一个元素允许是未知大小的数组，这个数组就是柔性数组**

但结构中的柔性数组前面必须至少一个其他成员,柔性数组成员允许结构中包含一个大小可变的数组，`sizeof`返回的这种结构大小不包括柔性数组的内存。包含柔数组成员的结构体用`malloc`函数进行内存的动态分配,且分配的内存应该大于结构的大小以适应柔性数组的预期大小

​     

## 为什么需要柔性数组

C中的结构体都是固定大小的，但是有些时候我们需要一个可变大小的结构体，比如有时候需要在结构体中存放一个长度动态的字符串

```c
typedef struct mystr
{
    int len; //记录字符串长度
    char *data;//底层的char数组指针
}mystr;
```

我们需要为`data` malloc一段内存，然后通过这个指针访问这段内存。

首先我们按照常规的做法，不做任何处理，直接`malloc`，如下

```c
typedef struct mystr
{
    int len;
    char* data;
}mystr;

int main(int argc, char const *argv[])
{
    char* c = "hello,world";
    
    //分别分配内存
    mystr* s = (mystr*)malloc(sizeof(mystr));
    s->data = (char*)malloc(strlen(c)+1); //+1是为\0分配的 strlen不会将\0计算进来
    strcpy(s->data,c);
    s->len = strlen(s->data);
    printf("len:%d data:%s\n",s->len,s->data); //11 hello,world
    
    //分别释放空间
    free(s->data);
    free(s);
    return 0;
}
```

可以看到上面的操作比较麻烦，结构体和内部的`data`指针分配内存和释放内存操作都是分开的，`data`数据区和结构体不是连续的两块内存，这样会带来两个问题:

- 操作不方便(上面说了，分配和释放都需要手动进行两次)
- 两次malloc分配的内存不是连续的，容易造成内存碎片，并且多次的释放和分配内存也会带来一定的消耗

为了让他们两个内存连续，也就是说为了能够实现动态长度的`struct`，分配只需要分配一次，释放也只需要拿到struct的指针即可释放所有内存

**方案一: 指针运算**

```c
typedef struct mystr
{
    int len;
    char* data;
}mystr;


int main(int argc, char const *argv[])
{
    char* c = "hello,world";
    //直接malloc一块连续的内存 然后强转为mystr指针
    mystr *s = (mystr*)malloc(sizeof(mystr)+strlen(c)+1);
    s->data=NULL; //此data指针闲置不用
    
    //mystr结构体大小为单位进行+1 结构体之后就是str的内存区域 
    //此处将数据copy到此区域
    strcpy((char*)(s+1),c);
    s->len = strlen(c);

    //int所占字节数，64位机器为4字节
    printf("sizeof(int): %ld\n", sizeof(int));
    //内存对齐 此处是16字节
    printf("sizeof(mystr): %ld\n", sizeof(*s));
    //s的起始地址 data指针的地址 
    printf("mystr:%p  data: %p\n",s,&(s->data));
    //数据，null，此处为空，故此变量已经被浪费。访问对应字符串数据需要(char *)(p+1)
    printf("p->data: %s\n", s->data);
    //偏移后，对应的字符串
    printf("data msg: %s\n", (char*)(s+1)); 
    printf("data msg: %s\n",(char*)(&(s->data))+8); 
    
    free(s);
}
```

**方案二: 柔性数组**

方案一需要进行指针运算，并且还会浪费一个data指针的内存空间，所以于是就有了下面 **柔性数组** 的方案，这是c语法层面支持的，最后那个数组指针不占用空间，并且可以直接通过指针访问结构体后面的数据

```c
typedef struct mystr
{
    int len;
    char data[];
}mystr;

int main(int argc, char const *argv[])
{
    char* c = "hello,world";
    //直接malloc一块连续的内存 data指针不占用任何空间
    mystr *s = (mystr*)malloc(sizeof(mystr)+strlen(c)+1);
    strcpy((char*)(s+1),c); //将数据copy到结构体后面那片预先分配的内存区域
    s->len = strlen(c);
    //获取对应的字符串
    printf("data msg: %s\n",s->data);    
    //(char*)s + size(int)也可
    printf("data msg: %s\n",(char*)s + sizeof(int));
    //直接通过数组下标来访问,也可其实就是指针的偏移
    printf("w: %c\n",s->data[7]);
	//反向获取到len
    printf("len: %d\n",*((char*)&(s->data)-4));
    free(s); //直接释放所有
}
```

​    

## 总结

柔性数组是C语法上提供的一个支持，目的是为了应对一种结构体需要动态增加的场景，主要解决了如下几个问题:

- **结构体内数据内存不连续问题**
- **多次malloc个free带来的性能开销**
- **不方便编程，每次都要进行指针运算才能获取到结构体后面的数据**
- **指针浪费**

柔性数组在`Redis`中的`sds`动态字符串结构中广泛使用，Redis的sds返回给应用层的是底层的char数组，同时因为有了动态数组的特性，就可以直接通过底层char数组就可以反向获取到sds结构体的相关信息比如`len、flags、alloc`等

这样即解决了C字符串类型的不足:

- 无法获取长度，获取长度必须遍历char数组直到遇到`\0`结束符
- 存在二进制`\0`安全问题

又能兼容C语言字符串处理函数，而不需要自己写一套专门正对sds的处理函数

具体参看Redis源码`sds.h`和`sds.c`

​    

## 参考

[C语言柔性数组](https://developer.aliyun.com/article/31693)

[C 语言之柔性数组](https://www.jianshu.com/p/686507f7b863)

[深入浅出C语言中的柔性数组](https://www.cnblogs.com/jinxiang1224/p/8468206.html)