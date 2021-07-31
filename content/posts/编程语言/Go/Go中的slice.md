---
title: Go中的slice
date: 2021-05-22
categories: [编程语言/Go]
tags: [Go]
---

## 数组

数组值拷贝

```go
func main(){
    a1:=[...]int{1,2,3}
	a2:=a1
	log.Println(a1,a1[0])
	log.Println(a2,a2[0]) //也可以通过下标访问
	a2[0] = 10
	log.Println(a) //a[0]=1
}
```

数组指针传递

```go
//通过指针访问数组
func main(){
    a:=[...]int{1,2,3}
    a2:=&a
    log.Println(a,a[0])
    log.Println(a2,a2[0])
    a2[0] = 10
    log.Println(a) //a[0]=10
}
```

​     

## len和cap的区别

**len** 代表底层数组可访问的范围，用索引访问不可越过这个界限

**cap** 代表底层数组的实际空间长度，不可用索引访问，如果`append` 元素时没有超过这个`cap`，则不再创建底层数组，直接在`len`空间后面扩展。否则开辟新的空间，同时增大cap（这里有一个增大规则），所以如果要频繁的扩容，适当设置大一些的cap能减少开销的，设置大的cap是为了防止多次扩容拷贝造成开销

```go
func main(){
    slice1 := []int{1,2,3} //len=3 cap=3
    slice2 := make([]int,2) //len=2 cap=2
    slice3 := make([]int,2,4) //len=2 cap=4
}
```

```go
func main(){
    arr:=make([]int,2,10) // len=2 cap=10
    arr[0]=1
    arr[1]=2
    //arr[2]=3 //报错
    
    // len=6 cap=10
    arr = arr[:6]  //扩容,不会再申请空间
    arr[2]=3 // [1,2,3,0,0,0]
    
    //会在len后面添加,如果len>cap则会进行扩容 此处不会扩容
    arr = append(arr,888)  // len=7 cap=10

    //arr=arr[:11] //报错 超过cap的大小，此时必须用过append进行扩容
}
```

```go
func main(){
    //下面的切片引用切片 指向同一个底层数组 cap右界限都是和父亲一样的
    a1 :=[]int{1,2,3,4,5,6,7,8} // len=8 cap=8
    a2 :=a1[:3] //[1,2,3] len=3 cap=8
    a3 :=a1[:5] //[1,2,3,4,5] len=5 cap=8
}
```

​        

## slice与数组的区别

数组

```go
array := [3]int{1,2,3}
array := [...]int{1,2,3} //不定大小的数组
```

切片

```go
func main(){
    //底层数组可见 修改切片会修改原数组 相当于原数组1-3的子数组指针
    arr := [5]int{1,2,3,4,5}
    slice :=arr[1:3] //cap=4 (5-1)  len=2 (3-1)
    slice[0]=555  //此时会影响原数组
    
    //append直接在len后面添加 因为还在cap范围内 所以会修改原数组
    slice = append(slice,777)
    slice = append(slice,888)
    //slice: [555,3,777,888] len=4 cap=4
    //arr: [1,555,3,777,888] len=5 cap=5
    
    //此时len已经超过了cap 所以会重新copy到新的空间 并且cap扩展为原来的2倍
    slice = append(slice,999)
    //slice: [555,3,777,888,999] len=5 cap=8
    //arr: [1,555,3,777,888] len=5 cap=5
    
    //此时修改slice已经不会影响原数组了
	slice[0]=666 // [666,3,777,888,999]
}
```

​    

## slice和可变参数

```go
func Args(arr ...int){
	//arr其实就是一个切片类型
	fmt.Println(reflect.TypeOf(arr).Kind()) //slice
}
```

​    

## slice的底层结构

`runtime/slice.go`

```go
type slice struct {
	array unsafe.Pointer //指向底层数组的首地址
	len   int
	cap   int
}
```

​     

## slice为什么是引用类型？

切片传入函数，同样也是值传递，会copy一份切片对应的`struct`到函数内

为什么又说是引用类型呢？为什么函数内部改变会影响原切片呢？

**根据上面的切片底层结构我们知道，切片有一个指向底层数组的指针，虽然切片是传值复制了一份，同时指向底层数组的指针也复制了一份，但指针始终是指向同一个地址的，那么我们改变切片的值其实就是改变底层数组的值。因为他们还是共享底层地址的。**

​    

## 空slice的判断

空切片可以判断它的长度是否为0，**但是判断为nil来判断这个切片是否为空是不准确的**

比如下面两种情况就要用`len`是否为0来判断，下面仅仅开辟了`slice`底层的结构体的内存空间，里面结构体属性都为默认值:

- `array unsafe.Pointer`指针为`nil`
- `len`和`cap`都为默认值`0`

```go
//下面两种创建slice的方法等价
empty1:=make([]int,0) //empty1 != nil
empty2:=[]int{} //empty2 != nil 
empty2:=new([]int) //empty3 != nil

slice:=make([]int,10)
empty3:=b[:0] // empty3 != nil
```

下面的就是为`nil` ，因为下面仅仅只是声明，`struct`的默认值为`nil`

```go
var empty2 []int // empty2 == nil
```

​      

## slice扩容机制

切片在`cap<=1024`的时候扩容是2倍扩容，当`cap>1024`则扩容就是按照 `1.25~2`倍来扩容，这和go的内存分配机制有关(Go会管理一些空闲的内存块，数组的实际空间必须是完整的内存块大小)，如果预估的容量的大小匹配到了一个指定的内存块，则新容量大小会以这个内存块大小为主

扩容的时候会将原来的地址数组的元素重新copy到扩容后新的底层数组空间上，也就是如果发生扩容那么底层数组就会改变

```go
a1 := []int{1, 2}   //[1,2] len=2 cap=2
a2 := append(a1, 3) //[1,2,3] len=3 cap=4 此时底层数组已经改变

//底层数组改变 不会影响a1
// a1: [1,2]
// a2: [999,2,3]
a2[0] = 999
```

如果我们slice的cap足够的话，我们也可以直接扩展len大小而不用append，当然使用append则更加方便而不需要考虑len

```go
func main(){
	slice:=make([]int,0,10)//len=0
	slice = slice[:5] //len=5
	for i:=0;i<len(slice);i++{
		slice[i] = i
	}
}
```

​        

## slice的copy

需要注意的是copy并不会创建空间，所以需要将目标的空间提前创建好，如果len空间不足的话则会截断

```go
func main() {
	s1 := []int{1, 2}
	s2 := make([]int, 1, 5)
	copy(s2, s1) //s1->s2
    //s1: [1,2]
    //s2: [1] 由于len(s2)=1 所有后面的截断了
}
```

下面在 `i` 的位置插入一个元素，其他元素向后移动，前提是len够,否则后面的元素会截断

```go
arr:=[]int{1,2,3,4,5,6}//len=6 cap=6
arr=append(arr,0) //扩充一个空间 len=7 cap=12

//将第i个元素向右边移动
// [1,2,2,3,4,5,6]
copy(a1[i+1:],a1[i:])
arr[i] = 88 //[1,88,2,3,4,5,6]
```

​      

## slice内存泄露问题

删除可以仅仅移动底层数组指针，**但是这种方式有一个缺陷，会造成内存泄漏，无法被GC回收不需要的内存**

```go
arr:=arr[1:]
```

下面这种方式先nil，然后再删除，不会内存泄漏

```go
arr[0]=nil
arr=arr[1:]
```

但是上面的方式比较麻烦，如果元素较多则需要一个个手动的进行置`nil`，下面使用`append`防止slice内存泄露，因为`append`会在目标slice底层cap不足时新开辟内存再进行copy，所以不会造成内存泄露问题

```go
//因为a[:0]的cap=0 所以这里新开辟了一块内存
//将a[:0]替换为 []int{} 也可，原理都是一样的
a = append (a[:0], a[1:]...) // 删除开头1个元素
a = append (a[:0], a[N:]...) // 删除开头N个元素
a = append (a[:0], a[:2]...) // 保留开头2个元素
a = append (a[:0], a[:len(a)-N]...) // 删除结尾N个元素
```

```go
func main() {
	slice := []int{1, 2, 3, 4, 5, 6, 7}
	slice = append([]int{}, slice[1:]...) // [2,3,4,5,6,7]
	slice = append([]int{}, slice[:3]...) // [2,3,4]
}
```

​     

## slice花式操作

**头部尾部插入**

```go
func main(){
    s:=[]int{1,2}
    elements:=[]int{3,4,5,6}

    // Push (插入到结尾)
    //超过cap的值会自动扩容 此处最好用elements为左值
    s = append(s, elements...)  //s = append(elements,s...)
    // Unshift（插入到开头）
    s = append(elements, s...) //在s的头部插入
}
```

> 插入的时候当cap不足时，会自动扩容，增加元素赋值的消耗，应该避免这样大容量的copy，并如果真的需要，则可以预先开辟大一点的`cap`，并且用大容量为 **左值** 来减少扩容拷贝的消耗

**删除操作**

```go
func main(){
    arr :=[]string{1,2,3,4,5,6}
    arr = append(arr[:2], arr[5:]...) //[1,2,6] 删除[2,4]范围内的元素
}
```

**切片实现栈和队列**

```go
func main() {
	q := []int{1, 2, 3, 4, 5}
	q = append(q, 6) // [1,2,3,4,5,6]
	q = append([]int{0}, q...) // [0,1,2,3,4,5,6]
	q, val := q[1:], q[0] // [1,2,3,4,5,6] 0
	q, val = q[:len(q)-1], q[len(q)-1] // [1,2,3,4,5] 6
}
```

​        

## for-range遍历slice

```go
func main() {
	arr := []int{9, 8, 7, 6, 5}
	for index := range arr {
		fmt.Println(arr[index])
	}
	for index, val := range arr {
		fmt.Println(index, val)
	}
}
```

​    

## slice并发

注意，go里面的特殊容器都是线程不安全的，所以在并发场景下 需要加锁进行控制

