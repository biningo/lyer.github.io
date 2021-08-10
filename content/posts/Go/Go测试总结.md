---
title: Go测试总结
date: 2021-02-15
categories: [编程语言/Go]
tags: [Go,测试]
---

## go中的测试

go的测试是是以 `xxx_test.go`结尾的，前面的名字和对应的测试文件名字一样，后面加个`test`，运行测试命令之后就会扫描所有测试文件进行测试

`xxx_test.go`测试文件中主要有如下几个部分:

| 类型     | 格式                    | 作用                           |
| -------- | ----------------------- | ------------------------------ |
| 单元测试 | 函数名前缀为`Test`      | 测试程序的一些逻辑行为是否正确 |
| 基准测试 | 函数名前缀为`Benchmark` | 测试函数的性能                 |
| 示例代码 | 函数名前缀为`Example`   | 为文档提供示例文档             |

​    

## 单元测试

```bash
$ go test -v #扫描当前包下所有的测试文件进行测试 并且输出详细信息
$ go test -v -test.run A #测试包含 A 字母的单元测试函数 [只能运行单元测试]
```

单元测试函数必须以 `t *testing`为参数，`t`主要用于报告测试结果是否正确以及日志记录，主要有如下几个方法: 

- `Error`
- `Log`

最常用

```go
//标记失败
func (c *T) Fail() //标记失败，但继续执行当前测试函数
func (c *T) FailNow() //标记失败，停止下面的执行
func (c *T) Failed() bool

//日志信息 go test如果测试成功的话，不会打印这部分内容，加上 -v则测试成功也会显示
func (c *T) Log(args ...interface{}) 
func (c *T) Logf(format string, args ...interface{})

// FailNow + Log
func (c *T) Fatal(args ...interface{})
func (c *T) Fatalf(format string, args ...interface{})

//Log + Fail
func (c *T) Error(args ...interface{})
func (c *T) Errorf(format string, args ...interface{})

//跳过
func (c *T) Skip(args ...interface{})
func (c *T) SkipNow()
func (c *T) Skipf(format string, args ...interface{})
func (c *T) Skipped() bool
```

请看下面的测试用例:

```go
func TestAdd(t *testing.T) {
    //测试数据 n1,n2测试数据 n3期望结果
	testData := make([]struct{ n1, n2, n3 int }, 100)
	rand.Seed(time.Now().UnixNano())
	for i := 0; i < len(testData); i++ {
		a, b := rand.Intn(1000), rand.Intn(1000)
		s := struct {
			n1, n2, n3 int
		}{
			a, b, a + b,
		}
		testData[i] = s
	}
    
    //运行多次测试
	for _, s := range testData {
		if result := Add(s.n1, s.n2); result != s.n3 {
			t.Errorf("add error:【%d】+【%d】=【%d】", s.n1, s.n2, s.n3)
		}
	}
	t.Logf("【%d】 ok", len(testData))
}
```

​    

## 测试覆盖率

```bash
go test -cover  #结果直接打印到屏幕 -test.run A
go test -cover  -covermode=atomic #选择测试模式 默认是set模式
go test -cover -coverprofile=cover.out #将测试结果输出到文件中
go test -cover  -run=Concurrent #指定测试某几个函数,写其他一部分名字即可

#下面是通过覆盖率生成的文件查看覆盖率报告
go tool cover -func=cover.out #展示每个函数的覆盖率情况 展示到屏幕中
go tool cover -func=cover.out -o cover.txt #将情况输出到文件
go tool cover -html=cover.out #将代码覆盖率文件以网页形式显示 会打开浏览器
go tool cover -html=cover.out -o cover.html #将覆盖率以html文件保存
```

覆盖率测试有三大模式:

- `set(默认)`: 每个语句是否执行？
- `count`: 每个语句执行了几次？
- `atomic`: 类似于count , 并发安全

下面看一个案例

```go
func Day(v int) string {
	switch v {
	case 0:
		return "SunDay"
	case 1:
		return "MonDay"
	case 2:
		return "TuesDay"
	case 3:
		return "WednesDay"
	case 4:
		return "ThursDay"
	case 5:
		return "FriDay"
	case 6:
		return "SaturDay"
	}
	return "No"
}
```

```go
type DayTest struct {
	V int
	S string
}

func TestDay(t *testing.T) {
	tests := []DayTest{
		{0, "SunDay"},
		{1, "MonDay"},
		{2, "TuesDay"},
	}
	for _, test := range tests {
		if s := Day(test.V); s != test.S {
			t.Errorf("%s,want:%s", s, test.S)
		}
	}

}
```

测试

```bash
# -race查看是否存在数据竞争
go test -race -coverprofile=coverage -covermode=atomic -v 
```

​    

## 基准测试

```bash
go test -v  -bench=. #-bench指定要测试的函数  .表示测试所有
go test -v -bench=Add #测试包含Add的基准测试函数
#-benchmem命令行标志参数将在报告中包含内存的分配数据统计
go test -bench=. -benchmem #-benchmem 展示内存分配情况
go test -bench=. -count 3 #-count指定测试函数的测试次数 默认1次
go test -bench=. -cpu 5 #-cpu 设置最大P的数量 默认和本机CPU个数一样 打印的名字后面的数字就是CPU的个数
go test -bench=. -cpu=1,2,3,4,5 #指定多个P 则每个P都会跑一遍
```

基准测试用于测试性能，以 `Benchmark`开头，`t`的函数和单元测试一样，执行`go test`是不进行基准测试的，需要指定`-bench`

```go
func BenchmarkAdd(b *testing.B) {
	for i := 0; i < b.N; i++ {
		Add(i, i)
	}
}
```

```bash
        `P的个数`	   `b.N的次数`  `单次执行测试函数的平均耗时`
BenchmarkAdd-8    	    1501	    669240 ns/op
```

下面是一个 **斐波那契数列** 计算的测试案例

```go    
func benchmarkFib(b *testing.B, n int) {
    for i := 0; i < b.N; i++ {
        Fib(n)
    }
}
//可以写多个性能测试函数
func BenchmarkFib1(b *testing.B)  { benchmarkFib(b, 1) }
func BenchmarkFib2(b *testing.B)  { benchmarkFib(b, 2) }
func BenchmarkFib3(b *testing.B)  { benchmarkFib(b, 3) }
func BenchmarkFib10(b *testing.B) { benchmarkFib(b, 10) }
func BenchmarkFib20(b *testing.B) { benchmarkFib(b, 20) }
func BenchmarkFib40(b *testing.B) { benchmarkFib(b, 40) }
```

```go
BenchmarkFib1-8         1000000000               2.03 ns/op
BenchmarkFib2-8         300000000                5.39 ns/op
BenchmarkFib3-8         200000000                9.71 ns/op
BenchmarkFib10-8         5000000               325 ns/op
BenchmarkFib20-8           30000             42460 ns/op
BenchmarkFib40-8               2         638524980 ns/op
PASS
ok      ./fib 12.944s
```

​    

## 参考

https://brantou.github.io/2017/05/24/go-cover-story

https://www.bookstack.cn/read/topgoer/b817b5a24fda9eed.md#brmnys

[生成漂亮的Go代码覆盖率报告](https://www.bilibili.com/video/BV1jK4y1U7cY?zw)