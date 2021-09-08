---
title: C语法总结
date: 2021-05-16
categories: [编程语言/C]
tags: [C]
---

## 函数指针和回调函数

- **指针函数**  返回指针的函数

```c
char * sayHello(){
    char *msg = (char*)malloc(sizeof(13));
    msg = "hello,world\n";
    return msg;
}

int main(int argc, char const *argv[])
{

    char *msg;
    msg = sayHello();
    printf("%s\n",msg);
}
```

- **函数指针**  保存函数入口地址的指针，可用于直接设置CPU的PC，直接跳转到目标函数代码指向，目标代码还可以是一段汇编程序，这样也可以实现C和汇编混合编程

```c
void echo(char *msg)
{
    printf("%s\n", msg);
}
int main(int argc, char const *argv[])
{
    //void:返回值  (*func):函数指针写法 (char *msg):形参
    void (*func)(char *msg); 
    
    //可以直接赋予一个函数的地址 或则void*地址都可
    //赋予一段汇编代码地址起始处也可
    func = &echo;            
    func("hello,world");     //直接跳转到地址入口执行
}
```

下面将四则运算法则传入函数中进行回调

```c
int sum(int i1,int i2){
    return i1+i2;
}

int sub(int i1,int i2){
    return i1-i2;
}
int mul(int i1,int i2){
    return i1*i2;
}
int div(int i1,int i2){
    return i1/i2;
}
void func(int i1,int i2,int (*operate)(int,int)){
    printf("i1:%d i2:%d = %d\n",i1,i2,operate(i1,i2));
}

int main(int argc, char const *argv[])
{
    func(8,4,sum);
    func(8,4,sub);
    func(8,4,mul);
    func(8,4,div);
    return 0;
}
```

函数指针可以用于结构体，实现类似于了类方法的效果

```c
typedef struct Stu
{
    char *name;
    void (*say)(char *msg);
}Stu;

void say(char *name){
    printf("hello,%s\n",name);
}

int main(int argc, char const *argv[])
{
    Stu stu;
    stu.name = "lyer";
    stu.say = &say;
    stu.say(stu.name);
    return 0;
}
```

​    

## extern和static

`extern`关键字的作用就是告诉编译器此值在其它文件中定义了，这里只是作个声明

```c
//sum.c
//表示a b的值由外部定义了
extern int a; 
extern int b;
int sum(){
    return a+b;
}

//main.c
#include <stdio.h>
extern int sum(); //表示sum是外部定义的 因为这里没有.h头文件
int a = 10;
int b = 20;
int main(int argc, char const *argv[])
{
    int c = sum();
    printf("%d\n", c); //30
    return 0;
}
```

再看一个案例

```c
//a.c
char a = 'A'; 

//main.c
int main(){
    extern char a;  //引用a.c中定义的全局变量
    printf("%c", a);
}
```

`static`关键字修饰的全局变量表示禁止被外部的`extern`所引用，表示这个变量的作用域仅仅在此文件内

```bash
static int age=19;
```

比如我在上面的`a`前加一个`static`那么main中就无法引用了，就会报错

此关键字如果修饰函数则表示此函数不可以被外部调用

```c
static void hello(){
    printf("hello\n");
}
```

​    

## enum枚举

如果没有枚举的话，我们其实也可以用宏定义来实现，只是代码不够优雅

```c
#include <stdio.h>
enum DAY{
    MON,THU,WED
};

int main(int argc, char const *argv[])
{
    enum DAY day=THU;
    printf("%d\n",day); //1
    return 0;
}
```

​    

## 巨人肩膀

[C 语言教程](https://wangdoc.com/clang/intro.html)

