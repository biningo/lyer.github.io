---
title: Go并发模式和channel
date: 2021-05-22
categories: [编程语言/Go]
tags: [Go,并发,channel]
---

## channel实现互斥锁

传统的`sync.Mutex`互斥锁

```go
//如果不加锁那么最终结果可能不是10000
func main() {
	count := 0
	wg := sync.WaitGroup{}
	mu := sync.Mutex{}
	for i := 0; i < 10000; i++ {
		wg.Add(1)
		go func() {
			mu.Lock()
			count++
			mu.Unlock()
			wg.Done()
		}()
	}

	wg.Wait()
	fmt.Println(count) //10000
}
```

`channel`实现互斥锁

```go
func main() {
	count := 0
	wg := sync.WaitGroup{}

	//channel的大小表示资源数量 1表示只允许一个goroutine加锁
	lock := make(chan struct{}, 1)

	for i := 0; i < 10000; i++ {
		wg.Add(1)
		go func() {
			lock <- struct{}{} //加锁
			count++
			<-lock //解锁
			wg.Done()
		}()
	}

	wg.Wait()
	fmt.Println(count) //10000
}
```

​    

## channel实现同步

**一对一通知**

下面将会正确打印: `"hello,world"`

```go
func main() {
	c := make(chan struct{})
	go func() {
		fmt.Print("hello,")
		c <- struct{}{}
	}()
	<-c //等待上面的goroutine
	fmt.Println("world")
}
```

**多对一通知**

等待`N`个`goroutinue`执行完毕，下面会打印10句 `"hello,world"` （也可以使用`sync.WaitGroup`实现）

```go    
func main() {
    //创建带缓冲的channel
	c := make(chan struct{}, 10)
	for i := 0; i < cap(c); i++ {
		go func(num int) {
			fmt.Println(strconv.Itoa(num)+": ", "hello,world")
			c <- struct{}{}
		}(i)
	}
	for i := 0; i < cap(c); i++ {
		<-c
	}
	fmt.Println("OK")
}
```

`close`实现群发通知

```go

func main() {
	wg := sync.WaitGroup{}
	c := make(chan struct{})
	for i := 0; i < 10; i++ {
		wg.Add(1)
		go func(num int) {
			<-c
			fmt.Println(num, "OK")
			wg.Done()
		}(i)
	}
	time.Sleep(time.Second)
	close(c) //通知所有goroutinue可以继续执行了
	wg.Wait()
}
```

​     

## channel实现计数信号量

channel可以实现并发数量控制，资源限制等

```go
func main() {
	wg := sync.WaitGroup{}
	c := make(chan struct{}, 5) //资源数量为5
	for i := 0; i < 100; i++ {
		wg.Add(1)
		go func(id int) {
			c <- struct{}{} //channel满了则会阻塞
			fmt.Println(id, "进入")
			time.Sleep(time.Second * time.Duration(rand.Intn(3)))
			fmt.Println(id, "出去")
			<-c
			wg.Done()
		}(i)
	}
	wg.Wait()
}
```

​    

## channel实现线程池

**一对多模型** ，也可以看做是 **生产者消费者模型**

同时启动多个`gotoutinue`去`channel`拿任务

```go
func worker(wg *sync.WaitGroup, c <-chan string) {
	for data := range c {
		fmt.Println(data)
	}
	wg.Done()
}

func main() {
	c := make(chan string, 1000)
	//100个worker
	wg := sync.WaitGroup{}
	wg.Add(100)
	for i := 0; i < 100; i++ {
		go worker(&wg, c)
	}
	for i := 0; i < 10000; i++ {
		//往channel里面送数据
		c <- "data:" + strconv.Itoa(i)
	}
	close(c)
	wg.Wait()
	fmt.Println("finished!")
}
```

​    

## future/promise模型

可以使用`channel`实现**异步调用**`future/promise`模型

> **同步调用:** 我们平时写的普通函数都是同步调用模式，就是执行顺序都是确定的从上到下，一个函数执行完毕之后才可以继续执行下一个
>
> **异步调用:** 调用一个函数会立即返回，然后主执行流可以继续执行，函数什么时候执行完毕并不知道

```go
func httpRequest() <-chan string {
    //管道定义在main中也可以，通过参数传递进来即可
	future := make(chan string)
	go func() {
		time.Sleep(time.Second * 3) //模拟网络延迟
		future <- "hello,world"
	}()
	return future
}

func main() {
	future := httpRequest() //异步调用,会立即返回
	for {
		select {
		case response := <-future:
			fmt.Println(response) //hello,world
			return
		case <-time.After(time.Second):
			fmt.Println("wait....")
		}
	}
}
```

​    

## 生产者消费者模型

**多对多模型**， 可以启动多个消费者和多个生产者

```go
func Producer(prefix string, out chan<- string) {
	for i := 0; i < 100; i++ {
		out <- prefix + ":" + strconv.Itoa(i)
	}
}

func Consumer(in <-chan string) {
	for v := range in {
		fmt.Println(v)
	}
}

func main() {
	c := make(chan string, 100)
	go Producer("one", c)
	go Producer("two", c)
	go Producer("three", c)

	for i := 0; i < 5; i++ {
		go Consumer(c)
	}

	//一直阻塞 直到Ctrl+C 退出
	sig := make(chan os.Signal)
	signal.Notify(sig, syscall.SIGINT, syscall.SIGTERM)
	fmt.Printf("quit (%v)\n", <-sig)
}
```

​    

## sub/pub发布订阅模型

**一对多模型**

```go
type Subscriber struct {
	Name      string
	C         chan interface{}
	Topics    map[string]struct{}
	Publisher *Publisher
}

func NewSubscriber(name string, cap int, publisher *Publisher) *Subscriber {
	return &Subscriber{
		Name:      name,
		C:         make(chan interface{}, cap),
		Topics:    map[string]struct{}{},
		Publisher: publisher,
	}
}

func (s *Subscriber) Subscribe(topics ...string) {
	for _, topic := range topics {
		s.Topics[topic] = struct{}{}
	}
	s.Publisher.Subscribers[s.Name] = s
}

type Publisher struct {
	TimeOut     time.Duration
	Subscribers map[string]*Subscriber
}

func NewPublisher(timeout time.Duration) *Publisher {
	return &Publisher{
		TimeOut:     timeout,
		Subscribers: map[string]*Subscriber{},
	}
}

func (p *Publisher) Send(msg interface{}, topics ...string) {
	for _, topic := range topics {
		for _, sub := range p.Subscribers {
			if _, ok := sub.Topics[topic]; ok {
				select {
				case sub.C <- msg:
				case <-time.After(p.TimeOut):
				}
			}
		}
	}
}


func main() {
	pub := NewPublisher(time.Second * 5)
	sub1 := NewSubscriber("one", 100, pub)
	sub2 := NewSubscriber("two", 100, pub)

	sub1.Subscribe("golang", "java")
	sub2.Subscribe("java")

	go func() {
		for {
			select {
			case v := <-sub1.C:
				fmt.Println(sub1.Name, v)
			}
		}
	}()

	go func() {
		for {
			select {
			case v := <-sub2.C:
				fmt.Println(sub2.Name, v)
			}
		}
	}()

	time.Sleep(time.Second * 2)
	pub.Send("hello,golang", "golang")
	pub.Send("hello,java", "java")
	pub.Send("hello,python", "python")
	
    //一直阻塞 直到Ctrl+C 退出
	sig := make(chan os.Signal)
	signal.Notify(sig, syscall.SIGINT, syscall.SIGTERM)
	fmt.Printf("quit (%v)\n", <-sig)
}
```

​    

## 快速响应

我们在请求某个资源的时候可以多开几个冗余goroutinue去请求，只取最快的那个即可，这样可以提升速度，因为goroutinue比较轻量所以也不会消耗多大的资源

```go
func request(c chan<- string) {
	time.Sleep(time.Second * time.Duration(rand.Intn(5)))
	c <- "data"
}

func main() {
	rand.Seed(time.Now().UnixNano())
    
    //注意需要开辟有缓存大小的channel,防止发送线程阻塞
	size := 5
	c := make(chan string, size) 
	start := time.Now()
	for i := 0; i < cap(c); i++ {
		go request(c)
	}
	data := <-c //只取一个数据
	fmt.Println(time.Since(start), data)
}
```

我们也可以使用缓冲为1的`channel`再配合`select`也可实现 **最快响应**

```go
func request(c chan<- string) {
	time.Sleep(time.Second * time.Duration(rand.Intn(5)))
	select {
	case c <- "data":
	default:
		return
	}
}

func main() {
	rand.Seed(time.Now().UnixNano())
	c := make(chan string, 1)
	start := time.Now()
	for i := 0; i < 100; i++ {
		go request(c)
	}
	data := <-c
	fmt.Println(time.Since(start), data)
}
```
