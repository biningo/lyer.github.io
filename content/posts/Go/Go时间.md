---
title: Go时间
date: 2021-05-30
categories: [编程语言/Go]
tags: [Go,时间]
draft: true
---

```go
func main(){
    t := time.Now()
	st := t.Format("2006-01-02 15:04:05") //格式化
	fmt.Println(st)

	//毫秒 纳秒
	// 1s = 1000 ms 毫秒
	// 1ms = 1000 μs 微妙
	// 1μs = 1000 ns
	u1, u2 := t.Unix(), t.UnixNano()
	fmt.Println(u1, u2)
	
	t1 := time.Now()
	time.Sleep(time.Second)
	d := time.Since(t1) //计算t1到现在进过了多久 返回time.Duration
	fmt.Println(d)
}
```

时间戳转化为时间

```go
func main(){
    //时间戳->时间 传入s,ns
	s := time.Unix(time.Now().Unix(), 0).Local()
	s.Format("2006-04-02 15:01:05")
}
```

时间单位

```go
func main() {
	s := time.Now().Unix()
	ms := s * 1e3
	ns := s * 1e9
	ns1 := time.Duration(1) //Duration是以1ns为单位计算的
	fmt.Println(s, ms, ns, ns1.Nanoseconds())
}
```

