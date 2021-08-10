---
title: Go文件读写和IO操作
date: 2021-05-29
categories: [编程语言/Go]
tags: [Go]
---

## IO接口

`io.Reader` 读IO接口

```go
type Reader interface {
	Read(p []byte) (n int, err error)
}
```

`io.Writer` 写IO接口

```go
type Writer interface {
	Write(p []byte) (n int, err error)
}
```

`io.Closer` 关闭IO接口

```go
type Closer interface {
	Close() error
}
```

​    

## 文件读取

方式一: 无缓冲直接读取

```go
func fileDemo1() {
	file, err := os.Open("/go-test-learn/io/t1.go")
	CheckError("", err)
	defer file.Close()

	content := []byte{}
	buf := make([]byte, 100) //每次读取的最大byte数组
	for {
        n, err := file.Read(buf) //返回真实读取的数据字节大小 (<=len(buf))
		if err == io.EOF { //err=io.EOF表示已经读取到结束了
			break
		}
		CheckError("", err)
		content = append(content, buf[:n]...)
	}
	//fmt.Println(string(content))
}
```

方式二: 缓冲IO

缓冲IO在调用`Read()`时会预先读取一部分数据到自己的缓存区，这样就不需要每次都调用Read去读取数据了，直接返回缓存区里的数据即可，只有缓存区里的数据不足才会调用底层的Read

```go
func fileDemo2() {
	file, err := os.Open("/go-test-learn/io/t1.go")
	CheckError("", err)
	defer file.Close()
    
    //包装file为缓冲reader
	reader := bufio.NewReader(file) 

	content := []byte{}
	buf := make([]byte, 100)

	for {
		n, err := reader.Read(buf)
		if err == io.EOF {
			break
		}
		CheckError("", err)
		content = append(content, buf[:n]...)
	}
	//fmt.Println(string(content))
}
```

方式三: `io.ReadAll()`方法，`ioutil.ReadAll()`也是间接调用此方法

```go
func fileDemo3() {
	file, err := os.Open("/home/pb/data/go-code/go-test-learn/io/t1.go")
	CheckError("", err)
	defer file.Close()
	
    //也是有缓冲的读取
	content, _ := io.ReadAll(file) //ioutil.ReadAll(file)同理
	if err == nil {
		fmt.Println(string(content))
	}
}
```

文件按行读取

```go
func fileDemo5() {
	file, err := os.Open("/go-test-learn/io/t1.go")
	CheckError("", err)
	defer file.Close()

	reader := bufio.NewReader(file)
	for {
        //读取到\n就结束  返回的string包含\n
		line, err := reader.ReadString('\n')
		if err == io.EOF {
			break
		}
		CheckError("", err)
		fmt.Print(line)
	}
}
```

​    

## 文件写入

方式一: `ioutil.WriteFile()`

```go
func writeDemo1() {
	content := []byte("hello,world")
    //文件不存在的话会自动创建 存在的话则会覆盖里面的内容
	ioutil.WriteFile("/io/a.txt", content, 0644) 
}
```

方式二: `io.WriteString()` 需要预先打开文件

```go
func writeDemo2() {
	file, err := os.OpenFile("/go-test-learn/io/a.txt", os.O_WRONLY|os.O_APPEND, 0644)
	CheckError("", err)
	defer file.Close()
    
    //返回写入的字节大小
	_, err = io.WriteString(file, "hello,world\n")
	fmt.Println(err)
}
```

方式三:  `Write()` 直接写入

```go
func writeDemo3() {
	file, err := os.OpenFile("/home/pb/data/go-code/go-test-learn/io/a.txt", os.O_WRONLY|os.O_APPEND, 0644)
	CheckError("", err)
	defer file.Close()
    
	file.Write([]byte("hello,go\n"))
}
```

方式四: `bufio.Writer`缓冲写入

写入的时候不会立即调用`Write()`写入，而是先写入到缓冲区里面，直到缓存区写满了才调用`Write()`一次性写入

```go
func writeDemo4() {
	file, err := os.OpenFile("/go-test-learn/io/a.txt", os.O_WRONLY|os.O_APPEND, 0644)
	CheckError("", err)
	defer file.Close()

	writer := bufio.NewWriter(file) //包装为有缓冲的Writer
	writer.WriteString("hello,java\n")
    writer.Flush() //需要调用Flush()刷新，否则可能不会被写入
}
```

​    

## 几个特殊的IO方法

`io.ReadAll()` 读取reader中的全部数据

```go
content, _ := io.ReadAll(file) //ioutil.ReadAll(file)同理
```

`io.ReadFull` 填充传入的字节数组，返回读取的字节数

- 如果传入的字节数组空间足够大则`error`为`ErrUnexpectedEOF`，并且返回实际读取的字节数
- 如果传入的字节数组不足，则`error`为`nil`，并且返回实际读取的字节数

```go
func main() {
	file, _ := os.Open("/a.txt")
	data := make([]byte, 20)
	n, err := io.ReadFull(file, data)
	fmt.Println(string(data), n, err == io.ErrUnexpectedEOF)
}
```

`io.Copy` 负责在两个IO之间copy数据

```go
func main() {
	file1, _ := os.Open("/a.txt")
	file2, _ := os.OpenFile("/b.txt",os.O_WRONLY|os.O_CREATE|os.O_APPEND, 0664)
	defer file1.Close()
	defer file2.Close()

	reader := bufio.NewReader(file1)
	writer := bufio.NewWriter(file2)
    //文件复制: 将file1的内容copy到file2中
	n, err := io.Copy(writer, reader) //返回copy的字节数
}
```

​    

## 参考

[Golang 读、写文件](https://segmentfault.com/a/1190000017918542)