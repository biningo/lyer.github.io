---
title: Go的defer
date: 2021-02-12
categories: [编程语言/Go]
tags: [Go]
---

## defer执行时机

```go
for i:=1;i<10;i++{
    defer log.Println(i)
}
```

上面那段简单的代码基本就可以说明**多个defer时的执行顺序了**

当代码中出现**defer**时，会将**defer**要执行的函数压人栈，然后等函数执行完毕再执行**defer栈**中的内容

go1.13以前用堆分配，加入到链表中，再尾递归调用，go1.13在栈上分配，如果defer过多则还是会在堆上用链表来管理

go1.14则做了进一步优化，defer的开销基本很小了

​    

## defer的估值时刻

**defer**分为**进入阶段**和**退出阶段** ，defer延迟的只是函数体的执行，并不延迟函数的初始化

```go
//defer初始化值和位置有关 推迟执行的仅仅是函数体
func main(){
    j:=10
    defer func(jj int) {
        log.Printf("j=%d jj=%d\n",j,jj) //j=99 jj=10
    }(j) //传入10
    j=99
}
```

​    

## 防止defer内存泄漏

下面这段代码会严重占用内存栈，造成短暂内存泄漏，有大量的文件句柄没有被释放

```go
//内存泄漏
 func writeManyFiles(files []os.File) error {
	 for _, file := range files {
         //......
         defer file.Close()
	 }
	 return nil
 }
```

用函数包裹之后每循环一个就关闭一个文件句柄

```go
//防止内存泄漏
 func writeManyFiles(files []os.File) error {
	 for _, file := range files {
	 	if err:= func() error {
			f, err := os.Open(file)
			defer f.Close()
			if err != nil {
				return err
			}else {
				return nil
			}
		}();err!=nil{
			return err
		}
	 }
	 return nil
 }
```

​    

## defer和闭包

下面打印的都是`4`

```go
func P(){
	for i:=0;i<4;i++{
		defer func (){
			log.Println(i)
		}()
	}
}
```

因为闭包是引用传递的，defer又是压栈延迟执行的，所以当  **i=4时，循环结束，defer开始执行，这时开始延迟调用函数，i已经是4了**

可以用传入参数的方式来避免这种情况：

```go
for i:=0;i<4;i++{
    defer func(j int) {
        log.Println(j) //output: 3 2 1 0
    }(i)
}
```

​    

## defer需要注意的点

1. `defer`引发的内存泄露问题
2. `defer`估值时刻
3. `defer`结合闭包

