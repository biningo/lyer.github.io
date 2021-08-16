---
title: awk工具
date: 2021-08-17
categories: [Linux]
tags: [Linux,awk]
---

## awk和sed区别

他们都是Linux下的 **流处理工具**

- `awk`核心是**格式化**
- `sed` 核心是 **正则**

​    

## print输出和格式化

```bash
BEGIN {
        username="lyer"
        age=18
        OFS="-" #指定print分割符号，默认是\t
        print "hello,","world"
        print "I'm",username,age
        print "a","b","c"
}
# -16 不足的往右边填空格
# 16 不足的往左边填空格
{
        #printf "%-16s %-16s %-16s\n",$1,$2,$3
        printf "%16s %16s %16s\n",$1,$2,$3
}

END{
        a = sprintf("%16s %16s","hello","world")
        print a
}
```

​    

## NR输出行

输出第`1`行

```bash
awk 'NR==1' net.txt
```

输出`[1,4]`行

```bash
awk 'NR==1,NR==4' net.txt
awk 'NR>=1 && NR<=4' net.txt
```

输出行号，`$0`表示整行数据

```bash
awk 'NR==1,NR==5 {print NR,$0}' net.txt
```

范围打印，打印前`n`行

```bash
awk 'NR>=1250 {print NR,$0}' net.txt
```

`&&`和`||` 打印指定范围的行

```bash
awk 'NR<=3 || NR>=1250 {print NR,$0}' net.txt
awk 'NR>=1250 && NR<=1260 {print NR,$0}' net.txt
```

打印最后一行

```bash
awk 'END {print $0}' net.txt
```

处理最后`n`行，如果不知道有几行的行awk貌似不能处理最后n行，但是可以结合`tail`来进行处理

```bash
tail -n 1  net.txt | awk '{print $1,$3}'
```

跨行打印

```bash
awk 'NR==4 || (NR>=5 && NR<=7) {print NR,$0}' net.txt
```

打印奇偶行

```bash
awk 'NR<=10 && NR%2==0 {print NR,$0}' net.txt
```

​    

## NF输出列

输出倒数第n列

```bash
awk '{print $NF}' user.txt #倒数第1列
awk '{print $(NF-1)}' user.txt #倒数第2列
```

​    

## BEGIN和END

```bash
#!/bin/awk -f
#运行前
BEGIN {
    math = 0
    english = 0
    computer = 0
    printf "NAME    NO.   MATH  ENGLISH  COMPUTER   TOTAL\n"
    printf "---------------------------------------------\n"
}
#运行中
{
    math+=$3
    english+=$4
    computer+=$5
    printf "%-6s %-6s %4d %8d %8d %8d\n", $1, $2, $3,$4,$5, $3+$4+$5
}
#运行后
END {
    printf "---------------------------------------------\n"
    printf "  TOTAL:%10d %8d %8d \n", math, english, computer
    printf "AVERAGE:%10.2f %8.2f %8.2f\n", math/NR, english/NR, computer/NR
}
```

计算所有`.c`文件数量以及总大小

```bash
ls -l | awk 'NR>1 && $9 ~ /.c/ {sum+=$5} END{print sum}'
ls -l | awk '$9 ~ /.c/ {sum+=1} END{print sum}'
```

​     

## 分割符号

`-F`指定每行的分割符号，默认是以`\t`或则空格进行分割的

```bash
awk -F: '{print $1}' net.txt
```

`FS` 和-F的功能一样，指定分割列，下面的和上面的-F作用一样

```bash
awk 'BEGIN{FS=":"} {print $0}' a.txt
```

`RS`默认是`\n`，awk默认按照`\n`分割行，我们可以修改`RS`的值，一般需要在`BEGIN`中指定RS

```bash
awk 'BEGIN{RS=","} {print $0}' a.txt
```

`OFS`指定awk展示的分割符号，默认是`\t` ，也可以在Begin中指定

```bash
awk '{print $1,$2,$3}' OFS="-" net.txt
```

​    

## 正则匹配

匹配第`6`列中的FIN或TIME，`~`表示模式开始的标识，如果不以特定的列的话则不需要`~`标识，`//`中间填写正则表示式

```bash
awk '$6 ~ /FIN|TIME/ || NR==1 {print NR,$4}' OFS="\t" net.txt
awk '/WAIT/' netstat.txt
awk '/Alice/ || NR==1 {print $2}' user.txt
```

反向匹配

```bash
awk '$6 ！~ /FIN|TIME/ || NR==1 {print NR,$4}' OFS="\t" net.txt
awk '!/WAIT/' netstat.txt
```

​    

## 重定向和文件拆分

类似于SQL语句中的`group by`

```bash
awk 'NR!=1 {print $0 > $6}' net.txt #以$6为文件名进行group by创建文件
awk 'NR!=1 {print $1,$6 > $6}' net.txt
```

​    

## 环境变量

使用`ENVIRON`变量可以使用环境变量

```bash
awk '{print $0,ENVIRON["JAVA_HOME"]}' net.txt
```

​    

## 字符串相关操作

字符串拼接

```bash
END{
	s1="hello" "lyer"
	print s1
}
```

`length`获取字符串长度

```bash
END{
	print length(s1)
}
```

​    

## awk脚本

awk指令可以写在单独的脚本文件里去执行，只需要`-f`指定脚本文件即可

```bash
awk -f cal.awk net.txt
```

`if`语句

```bash
BEGIN{
    score = 79
    if(score>90){
        print "great!"
        print "=================="
    }
    else if (score>80){
    	print "good!"
    }
    else if (score>=60){
    	print "ok!"
    }
    else{
    	print "bad!"
    }
    #三目运算符
    msg = score>60?"successful":"failure"
    print msg
}
```

`for`循环和数组

```bash
BEGIN {
        count=10
        while(count>0){
                m=sprintf("count:%d",count)
                arr[m]="OK"
                count--
        }

        for(item in arr){
                print item,arr[item]
        }
        print "----------------------------"
        for(i=0;i<10;i++){
                print i
        }
}
```

`array`其实就是map

```bash
BEGIN {
        for(i=0;i<10;i++){
                arr[i]=sprintf("hello,world-%d",i)
        }
        print "length:",length(arr)
        for(i=0;i<length(arr);i++){
                print arr[i]
        }
        #输出是无序号的
        for(item in arr){
                print item,arr[item]
        }        
		print "------------------"
		delete arr[0] #删除第0个元素
		if (0 in arr){...} #判断key是否在数组中
		
		delete arr #删除数组所有元素
}
```

`switch`

```bash
switch(1){
	case 12: 
		print "OK"
		break;
	case 12: 
		print "OK"
		break;
	default:
		print "ok"
}
```

​     

## 其他

## cut

cut可以提供简单的列切割

```bash
-d #定义分割符  默认是空格
-f #指定显示的列
-c #以字符为单位切割
-b #字节为单位切割
```

```bash
cat cut.txt| cut -c-4 #[,4]
cat cut.txt| cut -c4- #[4,]
cat cut.txt| cut -c31-4 #1开始取4个
cat cut.txt| cut -d ':' -f 1
```

​    

## 参考

[awk简明教程](https://coolshell.cn/articles/9070.html)

[精通awk](https://www.bookstack.cn/books/junmajinlong-awk)

[awk 18个经典实战案例精讲](https://www.bilibili.com/video/BV1BJ411X7QN?from=search&seid=6111023761748603474)

[精通awk系列文章](https://www.junmajinlong.com/shell/awk/index/)

[sed和awk的区别](https://www.zhihu.com/question/297858714)

