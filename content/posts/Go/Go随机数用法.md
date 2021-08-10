---
title: Go随机数用法
date: 2021-02-15
categories: [编程语言/Go]
tags: [Go]
---

## 基本用法

go随机数在`math/rand`包下，go的随机数需要先给他一个`Seed`，`Seed`如果一样的话或则不设置的话每次生成的都是 **伪随机数** ，多次执行生成的都是一样的随机数序列，所以必须设定`Seed`而且还是以 **时间戳** 的方式来设置，如下生成 `[0,10)`之间的随机整数：

```go
rand.Seed(time.Now().UnixNano())
r:=rand.Intn(10) //[0,10) 返回int类型
r=rand.Int63n(10) //返回int64
```

如果要生成指定范围的随机整数，如下生成`[min,max)`之间的随机整数：

```go
rand.Seed(time.Now().UnixNano())
max:=10;min:=-10
rand.Intn(max-min)+min //[-10,10)
```

​    

## 随机负载均衡实现

我们实战一下，实现一个 **随机数负载均衡**

```go
type RandomBalance struct {
	curIndex   int
	hosts []string
}

func (r *RandomBalance) Add(host string) {
	r.hosts = append(r.hosts, host)
}

func (r *RandomBalance) Next() (string, error) {
	if len(r.hosts) == 0 {
		return "", errors.New("no host")
	}
	rand.Seed(time.Now().UnixNano())
	r.curIndex = rand.Intn(len(r.hosts))
	return r.hosts[r.curIndex], nil
}
```

```go
func main() {
	rb := RandomBalance{}
	for i := 1; i < 10; i++ {
		rb.Add("h" + strconv.Itoa(i))
	}
	for i:=0;i<10;i++{
		log.Println(rb.Next())
	}
}
```

​    

## cryptp包中的随机数

生成随机字节数组

```go
import (
	"crypto/rand"
    "encoding/base64"
	"encoding/hex"
    "math/big"
)
func main() {
	
	d64 := make([]byte, 64)
	rand.Read(d64) //传入的byte数组大小是多少就生成多少
	
    //序列化为16进制字符串
	h := hex.EncodeToString(d64) //128 每个字节是2位16进制
	fmt.Println(h)
	
    //序列化为base64
	b64 := base64.StdEncoding.EncodeToString(d64)
	fmt.Println(b64)
}
```

生成均匀分布的随机数字

```go
func main() {
    // [0,n)
	n1, _ := rand.Int(rand.Reader, big.NewInt(2)) 
    
    // [min,max)
    max := 3;min := -1
	n, _ := rand.Int(rand.Reader, big.NewInt(int64(max-min)))
	n2 := n.Int64() + int64(min) //生成的随机数
}
```

​    

## 加密解密和序列化

```go
import (
	"crypto/rand"
    "encoding/base64"
	"encoding/hex"
)
func main(){
	d64 := make([]byte, 64)
	rand.Read(d64)
	h := hex.EncodeToString(d64) //128 每个字节是2位16进制
	b64 := base64.StdEncoding.EncodeToString(d64)
   
    //序列化为sha256  返回256位(32byte)的加密字符串
	s256 := sha256.Sum256([]byte("hello,world")) 
	
    //将加密字符串序列化
    h = hex.EncodeToString(s256[:]) //使用fmt.Sprintf也可
	b64 = base64.StdEncoding.EncodeToString(s256[:])
}
```