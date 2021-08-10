---
title: Go处理error
date: 2021-05-23
categories: [编程语言/Go]
tags: [Go]
---

## 大道至简的error

go的错误处理就只有一个`errors`包和一个`error`接口，这个接口只包哈一个`Error`方法，该方法返回一个string，这个包的代码很少，只有两个文件：

- `errors.go`
- `wrap.go`

go通过返回值来返回错误而不是通过`try/catch`，除非函数能保证一定能执行成功，否则每个函数都必须返回一个`error`

并且go的error建议只处理一次，也就是说如果你处理过了错误那么就不需要返回给上层了，如果将已经处理过的错误继续返回给上层则这个错误可能会被重复处理

```go
func f() error{
	return errors.New("error") //返回error接口
}
```

下面来看下`errors/errors.go`的源码，不过10行左右

```go
func New(text string) error {
	return &errorString{text} //返回errorString指针
}

//实现了error接口
type errorString struct {
	s string
}

//获取错误字符串的方法
func (e *errorString) Error() string {
	return e.s
}
```

​     

## 自定义error

我们只需要实现`error`接口也就是重写`Error()`方法即可自定义错误

```go
type ZeroDivisionError struct {
	msg  string
	code int
}
func (e ZeroDivisionError) Error() string {
	return fmt.Sprintf("[%d]:%s", e.code, e.msg)
}
type NullPointerError struct {
	msg string
}
func (e NullPointerError) Error() string {
	return fmt.Sprintf("NullPointerError:%s", e.msg)
}

func main(){
    e := IntNull()
	switch e.(type) {
	case NullPointerError:
		log.Println("Null:", e)
	case ZeroDivisionError:
		log.Println("Zero Division:", e)
	default:
		log.Println("undefined")
	}
}
```

也可以直接设置一个全局变量来表示error

```go
var (
	ErrOpenFile = errors.New("open file error")
	ErrReadFile = errors.New("read file error")
)
```

​    

## panic和recover

关于`panic`和`recover`，我们必须注意

- `panic` 只会触发当前 Goroutine 的 `defer`；
- `recover` 只有在 `defer` 中调用才会生效；
- `panic` 允许在 `defer` 中嵌套多次调用

```go
func main(){
    defer func() {
        e := recover() //panic传入什么 这里就接受什么
        err := e.(error) //都是interface类型
        log.Println(err)
    }()
    //....
    panic(errors.New("panic error"))
    //....
}
```

`panic`只会在当前`goroutinue`中生效，下面的panic并不会被捕获，`main`不会被打印

```go
func main() {
    defer func(){
        log.Println("main")
        recover()
    }
	go func() {
		panic("panic")
	}()
	time.Sleep(time.Second)
}
```

​     

## 优雅处理error

有时候，我们会写这样的一堆判断`error`的屎山代码：

```go
if err := a(); err != nil {
    return err
}
if err := b(); err != nil {
    return err
}
if err := c(); err != nil {
    return err
}
if err := d(); err != nil {
    return err
}
if err := e(); err != nil {
    return err
}
```

可以将所有相同的错误处理代码都封装为一个check函数

```go
//复杂检查是否有error 如果有则引发panic
func check(err error) {
	if err != nil {
		panic(err)
	}
}

//panic() + recover()
func task() (err error) {
	defer func() {
		if r := recover(); r != nil {
			return
		}
	}()

	_, err = a("one")
	check(err)

	_, err = b(1, 2)
	check(err)

	err = c()
	check(err)

	err = d()
	check(err)
	return nil
}

func a(msg string) (string, error) {
	return msg, errors.New("a error")
}

func b(x, y int) (int, error) {
	if y == 0 {
		return 0, errors.New("b error")
	}
	return x / y, nil
}

func c() error {
	return errors.New("c error")
}

func d() error {
	return errors.New("d error")
}
```

​    

## 只处理一次error

我们要么不处理错误直接返回给上层，要么只处理一次错误，下面的错误处理是不正确的，当前已经处理了还返回error的话则上层还会再处理一次或则多次，正确的写法应该返回`nil`

```go
func f() error {
	//....
	f, err := os.Open("abc")
	if err != nil {
		log.Println(err) 
		return err //返回nil  或则不处理错误直接返回error
	}
	f.Name()
	return nil
}
```

​    

## Wrap包装error

Go1.13之后，允许我们嵌套的将多个`error`合并为一个，将最终的`error`返回给最上层，其源代码在`errors/wrap.go`中

- `Is`：判断一个error对象是否是另外一个error对象的封装，大的是小的，小的不是大的
- `Unwrap`：去除一层`error`封装

```go
func main() {
	e1 := errors.New("a") 		  // a
	// %w 是wrap的意思
	e2 := fmt.Errorf("%w->b", e1) // b->a
	e3 := fmt.Errorf("%w->c", e2) // c->b->a
	e2 = errors.Unwrap(e3)        // b->a
	e1 = errors.Unwrap(e2)        // a
    //true true true
	fmt.Println(errors.Is(e2, e1), errors.Is(e3, e2), errors.Is(e3, e1))
}
```

​      

## 参考

[GO 编程模式：错误处理](https://coolshell.cn/articles/21140.html)

[Go语言技巧 - 2.【错误处理】谈谈Go Error的前世今生](https://www.bilibili.com/video/BV1vN411Z7Rv?from=search&seid=171776842596520823)

