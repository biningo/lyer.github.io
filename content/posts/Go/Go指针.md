---
title: Go指针
date: 2021-05-10
categories: [编程语言/Go]
tags: [Go]
---

## 返回局部变量的指针

Go支持垃圾回收，所以当一个函数返回了局部变量的地址这是合法的。这是Go和C指针的区别之一。Go编译器会作**内存逃逸**分析，如果一个局部遍历的指针被返回了则会将他的内存分配到堆空间

```go
type Stu struct {
	Name string
	Age  int
}

func NewStudent() *Stu {
	stu := Stu{}
	stu.Name = "lyer"
	stu.Age = 18
	return &stu //这是合法的
}

func main() {
	stu:=NewStudent()
	stu.Age = 21
}
```

​    

## Go指针的限制

- 普通类型的指针不能作算术运算

- 一个指针类型的值不能被随意转化为另外一个指针类型，也就是说每个类型的指针其实也相当于一个类型

- 指针只能在同类型比较(`==、!=`)

    ```go
    func main() {
    	s1:=&Stu{}
    	s1=  nil
    	s2:=&Stu{}
    	log.Println(s1==s2) //false
    }
    ```

- 指针的值不能赋给其他类型的指针

**综上所述: Go每个类型的指针都是一个独立的类型**

任意一个类型的指针都可以转化为`unsafe.Pointer`类型，此类型相当于`void*`

​    

## unsafe.Pointer和uintptr

`unintptr`属于一个可运算的指针类型，其长度每个平台不一样，比如`64位`平台则此类型必须能够表达所有的地址，所以长度为`int64`

**注意unintptr指向的地址不会被Go感知到，也就是说其指向的地址无法保证不被GC回收**

`unsafe.Pointer`指针相当于`void*`，其它所有类型的指针都可以转化为此类型，如果要进行运算的话则需要继续转化为`uintptr`

```go
func main(){
    arr := [3]int8{1, 2, 3}
	p := unsafe.Pointer(&arr)
	p1 := (*int8)(unsafe.Pointer(uintptr(p)))
	log.Println(*p1)

	p2 := (*int8)(unsafe.Pointer(uintptr(p) + 1)) //1字节
	log.Println(*p2)

	p3 := (*int8)(unsafe.Pointer(uintptr(p) + 2)) //2字节
	log.Println(*p3)
}
```

```go
//下面的程序需要了解一些struct内存对齐的知识
func main() {
	stu := Stu{1, 2, 3}
	log.Println(unsafe.Sizeof(stu)) //以8字节对齐 所以此处是16字节
	
    //偏移4字节
    p2 := (*int32)(unsafe.Pointer(uintptr(unsafe.Pointer(&stu)) + 4))
	log.Println(*p2)
	
    //偏移8字节
	p3 := (*int64)(unsafe.Pointer(uintptr(unsafe.Pointer(&stu)) + 8))
	log.Println(*p3)
}
```

我们也可以使用指针获取`slice`的len和cap

```go
type slice struct {
	array unsafe.Pointer //8 byte
	len   int //8 byte
	cap   int // 8 byte
}// 24 byte
```

```go
int main(){
    arr := []string{"a", "b", "c", "d"}// len:4 cap:4
    arr  =append(arr,"e") // len:5 cap:8
    //24 byte 所有的切片都是24
	log.Println(unsafe.Sizeof(arr)) 
	p := unsafe.Pointer(&arr)
	length := (*int)(unsafe.Pointer(uintptr(p) + 8))
	capacity := (*int)(unsafe.Pointer(uintptr(p) + 16))
	log.Println(*length, *capacity)
}
```

再来看一个`string`的例子

```go
//go中string的底层表示
type stringStruct struct {
	str unsafe.Pointer
	len int
}
```

```go
int main(){
	s := "hello,world"
    //所有的string都是16 byte 因为string对象有一个指向底层实际数据的指针
	log.Println(unsafe.Sizeof(s)) 
	p := unsafe.Pointer(&s)
    //移动一个Pointer的长度获取到len
	length := (*int)(unsafe.Pointer(uintptr(p) + 8)) 
	log.Println(*length) //11
}
```

可以看到上面需要手动计算偏移，如果对于一个`struct`的话就比较难计算比较麻烦，所以可以直接调用API函数计算

```go
type Stu struct {
	name  string //16
	age   int64 //8
	hobby []string //24
	n1    int64 //8
}
int main(){
	stu := Stu{}
	log.Println(unsafe.Sizeof(stu)) //56
    //0 16 24 48
	log.Println(unsafe.Offsetof(stu.name), unsafe.Offsetof(stu.age), unsafe.Offsetof(stu.hobby), unsafe.Offsetof(stu.n1))
}
```

