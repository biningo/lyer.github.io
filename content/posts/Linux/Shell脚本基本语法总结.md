---
title: Shell脚本基本语法总结
date: 2021-03-19
categories: [Linux]
tags: [Linux,Shell]
---

## 基本语法

如果一行命令太长的话可以加`\`来分割多行命令

```bash
echo hello\
world
```

如果需要一行执行多条shell语句，则可以使用`;`分割，不管语句执行失败成功与否都会执行所有的语句

```bash
echo hello; ls -al ; echo ok
```

`||` 命令执行成功则不再继续执行

```bash
cd /abc || echo "dir is not exist"
```

`&&`  命令执行成功才会继续执行

```bash
cd /etc && ls
```

​    

## 快捷键

```bash
ctrl+a #光标到行首
ctrl+f #方向键->
ctrl+b #方向键<-
ctrl+e #光标到行尾
ctrl+u #剪切光标到行首
ctrl+k #剪切光标位置到行尾
ctrl+w #删除光标前一个单词
ctrl+y #粘贴
ctrl+d #退出当前shell
ctrl+z #暂停任务 bg fg可以恢复
ctrl+c #终止前台进程
ctrl+r #搜索历史命令
ctrl+p #上一条命令
ctrl+n #下一条命令
```

​     

## 模式扩展

`?` `*` 通配符

```bash
ls ?.txt #只列出 a.txt b.txt 等一个单词的文件
ls ??.txt #列出两个 ab.txt ...
ls * #列出所有文件
ls a*.txt #列出所有a开头的
```

列出所有隐藏文件

```bash
ls .*
```

匹配任意目录层级的文件

```bash
ls **/*.sh
```

`[]`方括号扩展 (和正则表达式类似)

```bash
ls [ab].txt #列出 a.txt 或则 b.txt
ls *[^abc]* #或*[!abc]* 不包含a、b、c的
ls [0-9].txt
ls [a-z].txt
ls [a-zA-Z0-9].txt
```

`{}`大括号扩展

```bash
echo d{a,e,i,u,o}g #dag deg dig dug dog
echo {a,b,c}.txt
echo {a..c} # a b c
echo {c..a} # c b a
echo {1..5} {5..1}
echo {j{p,pe}g,png} #嵌套 jpg jpeg png
echo {01..5} #有0 则每个都会输出0
mkdir {2007..2009}-{01..12} #建立时间目录 2007-01 ~ 2009-12
echo a{,b,c,d} #逗号前面可以为空 a ab ac ad
echo {a..d}{1..10} # a1,a2.....d10
echo {0..10..2} #设置step 0,2,4,6,8,10
```

`{}`用于for循环

```bash
#用于for循环
for i in {1..4};do
  echo $i
done

for i in {1..10..2};do
  echo $i
done
```

`()` 命令扩展、算数扩展

```bash
echo $(date +%F)
echo $((1+2))
```

​      

## 引号转义

单引号不会转义，双引号会解释变量为值，反引号相当于`$()`

```bash
echo "${SHELL}" #输出变量的值
echo '${SHELL}' #原样输出
echo `date +%F`
```

​     

## 变量

变量类型分为: 

- 全局 (`declare`或则直接写)
- 局部(函数内部) `local`声明
- 环境变量

全局变量可以被当前脚本所有范围内都访问到

```bash
#!/bin/bash
hello(){
        name="lyer"
        echo "hello"
}
hello
echo $name # lyer
```

如果想要让当前shell脚本的变量能让子shell都能访问到则需要`export`变量

```bash
name="lyer"
export name
```

local函数局部变量只能被当前函数访问到

```bash
#!/bin/bash
hello(){
        local name="lyer"
        echo "hello"
}
hello
echo $name # NULL
```

环境变量能被所有脚本访问到

```bash
#!/bin/bash
echo $PWD
```

只读变量

```bash
readonly name="AAA" #之后再次赋值会报错
#只读变量也可以后来才设置
age="18"
readonly age
```

删除变量（只读变量不能被删除）

```bash
#!/bin/bash
hello(){
        name="lyer"
        echo "hello"
        unset name
}
hello
if [ -z $name ];then #如果变量为空则为true
	echo "name is not exist."
else
 	echo "${name}"
fi
```

直接赋值空字符串也可以删除

```bash
name=
name=''
```

特殊变量有如下几个类型

| 量     | 含义                                                         |
| ------ | ------------------------------------------------------------ |
| `$0`   | 当前脚本的文件名                                             |
| `$<n>` | 传递给脚本或函数的参数。n 是一个数字，表示第几个参数。例如，第一个参数是$1，第二个参数是$2。 |
| `$#`   | 传递给脚本或函数的参数个数。                                 |
| `$*`   | 传递给脚本或函数的所有参数。                                 |
| `$@`   | 传递给脚本或函数的所有参数。被双引号`""`包含时，与 $* 稍有不同 |
| `$?`   | 上个命令的退出状态，或函数的返回值，用于判断上一个函数或则命令是否执行成功 |
| `$$`   | 当前Shell进程ID，对于 Shell 脚本，就是这些脚本所在的进程ID。 |

`${@}`和`${*}` 的区别，两个都是用于获取所有入参，区别在于`"${@}"`被双引号包裹会展开里面的所有的值，而`"${*}"`则会将所有值当做一个

> 如果不加双引号那么他们就没有区别，都能够被for循环读取

```bash
#下面只会执行一次循环
for i in "${*}";do
   echo $i    
done

#下面会输出每个参数
for i in "${@}";do
   echo $i  
done
```

变量替换

| 形式              | 说明                                                         |
| ----------------- | ------------------------------------------------------------ |
| `${var}`          | 变量本来的值                                                 |
| `${var:-word}`    | 如果变量 var 为空或已被删除(unset)，那么返回 word，但不改变 var 的值。 |
| `${var:+word}`    | 如果变量 var 存在，那么返回 word但不改变 var 的值            |
| `${var:=word}`    | 如果变量 var 为空或已被删除(unset)，那么返回 word，并将 var 的值设置为 word |
| `${var:?message}` | 如果变量 var 为空或已被删除(unset)，那么将消息 message 送到标准错误输出，可以用来检测变量 var 是否可以被正常赋值。 若此替换出现在Shell脚本中，那么脚本将停止运行 |

​    

## 算术运算

进行数学运算有如下四种方式

```bash
a=10
b=20
val=`expr $a + $b`
echo "a + b : $val"
```

```bash
a=10
b=20
val=$((a*b))
echo "a + b : $val"
```

```bash
a=10
b=20
let val=${a}*${b}
echo "a + b : $val"
```

注意`let`表示式如果有空格则需要用双引号

```bash
let a=1+2
let "b=1 + 2"
```

​     

## 字符串

字符串拼接

```bash
s1="abc"
s2="def"
s3=${s1}"-"${s2}
```

获取字符串的长度

```bash
name="lyer"
echo ${#name} #和array的操作一样
```

获取子串

```bash
${username:1} #[1,]
${username:1:2} #1开始获取2个 包括1
${username:1:-2} #[1,-2)
${username:-5:2} #从末尾-5位开始提取2位
${username: -3} #提取最后3个字符，注意冒号后面添加一个空格：txt
```

改变大小写

```bash
${username^^} #upper
${username,,} #lower
```

​     

## 数组

数组可以当map使用

```bash
#!/bin/bash
arr[0]="a"
arr[1]="b"
arr["name"]="lyer"
echo ${arr[1]} ${arr} #默认打印第0个
echo ${arr["name"]} # lyer
```

可以一次定义多个元素，对于数组的操作和字符串切割操作类似

```bash
arr=("a" "b" "c" "d" "e")
echo "size: ${#arr[*]}"  
echo ${arr} ${arr[*]:2:2} #a | c d
```

数组用户for循环

```bash
#依次打印数组元素
for i in ${arr[*]};do
   echo $i     
done
```

数组的copy

```bash
arr2=(${arr[*]})
arr2=(${arr[*]} a b c ) #也可以添加新的成员
arr2+=(d e f)
```

​    

## IF判断

逻辑判断

| 运算符     | 说明                                                |
| ---------- | --------------------------------------------------- |
| `-eq` `==` | 检测两个数是否相等，相等返回 true                   |
| `-ne` `!=` | 检测两个数是否相等，不相等返回 true                 |
| `-gt`      | 检测左边的数是否大于右边的，如果是，则返回 true     |
| `-lt`      | 检测左边的数是否小于右边的，如果是，则返回 true     |
| `-ge`      | 检测左边的数是否大等于右边的，如果是，则返回 true   |
| `-le`      | 检测左边的数是否小于等于右边的，如果是，则返回 true |

布尔判断

| 运算符 | 说明                                              |
| ------ | ------------------------------------------------- |
| `!`    | 非运算，表达式为 true 则返回 false，否则返回 true |
| `-o`   | 或运算，有一个表达式为 true 则返回 true           |
| `-a`   | 与运算，两个表达式都为 true 才返回 true           |

字符串

| 运算符       | 说明                                                         | 举例                   |
| ------------ | ------------------------------------------------------------ | ---------------------- |
| `=`          | 检测两个字符串是否相等，相等返回 true                        | [ $a = $b ] 返回 false |
| `!=`         | 检测两个字符串是否相等，不相等返回 true ,注意字符串不能使用`eq ne`等判断是否相等 | [ $a != $b ] 返回 true |
| `-z`         | 变量是否存在，检测字符串长度是否为0，为0返回 true            | [ -z $a ] 返回 false   |
| 直接写字符串 | 检测字符串是否为空，不为空返回 true                          | [ $a ] 返回 true       |

文件相关

| 运算符 | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| `-d`   | 检测文件是否是目录，如果是，则返回 true                      |
| `-f `  | 检测文件是否是普通文件（既不是目录，也不是设备文件），如果是，则返回 true |
| `-r `  | 检测文件是否可读，如果是，则返回 true                        |
| `-w `  | 检测文件是否可写，如果是，则返回 true                        |
| `-x`   | 检测文件是否可执行，如果是，则返回 true                      |
| `-s`   | 检测文件是否为空（文件大小是否大于0），不为空返回 true       |
| `-e`   | 检测文件（包括目录）是否存在，如果是，则返回 true            |

```bash
file="/tmp/a.txt"
if [ -e $file ];then
	echo "文件存在"
else
	echo "文件不存在"
fi	
```

c风格判断数字大小

```bash
if ((1<2)); then echo "OK"; fi    
```

多分支

```bash
if [ xxx ];then
	xxx
elif [ xx ]
	xxx
elif [ xx ]
	xxx
else
	xxx
fi
```

和`test`命令配合

```bash
if test $[2*3] -eq $[1+5]; then echo 'The two numbers are equal!'; fi;
```

​     

## 循环

Shell里面有三种循环

- while
- until 和`while`差不多,只是条件为false则循环,while是条件为真则循环
- for

```bash
for i in {1..5};do
	echo $i
done
```

```bash
count=5
while [ count -gt 0 ];do
	echo $count
	let count+=1
done
```

```bash
count=5
until [ count -le 0 ];do
	echo $count
	let count+=1
done
```

shift能将每次参数往前移动，shift后面可以加数字比如 `shift 4` 则这个时候$4就变成了$1

```bash
while [ ! -z $1 ];do
	echo $1
	shift
done
```

​    

## case

```bash
echo '输入 1 到 4 之间的数字:'
echo '你输入的数字为:'
read aNum
case $aNum in
    1)  echo '你选择了 1'
    ;;
    2)  echo '你选择了 2'
    ;;
    3)  echo '你选择了 3'
    ;;
    4)  echo '你选择了 4'
    ;;
a|b|c)
        echo "输入字母$aNum"
    ;;
    *)
        echo "其他"
    ;;
esac
```

​    

## 函数

函数调用和传参

```bash
hello(){
        username=$1
        age=$2
        echo "hello,world"
        echo "hello,$username,$age"
}
hello icepan 18
```

函数返回值只能返回数值类型代表函数执行成功还是失败，使用`$?`来查看上一个函数或则命令是否执行成功，如果是`0`表示执行成功，其他则表示执行失败

```bash
hello(){
	return 0
}
$? #0
```

因为函数只能返回数值，那么为了能返回其他值则可以借助全局变量，直接赋予全局变量值即可

```bash
name=""
age=0
hello(){
	name="lyer"
	age=18 
	#默认返回0
}
hello #调用函数
echo $name $age
```

也可以设置局部变量`local`而不影响外界的变量

```bash
name="a"
hello(){
	local name
	name="b"
}
hello
```

​    

## 引入脚本和export变量

```bash
source
. 
export
```

shell也可以和c一样import到一个shell进程中，这样就可以调用其他shell脚本的函数了，但是在引入的时候会执行整个脚本，可以用`source` `.` 等方式来引入，两种方式差不多，都会在当前进程中执行脚本并且设置变量和函数

```bash
#------------tmp.sh--------------
name="lyer"
hello(){
	echo "hello,world"
}
#---------------------------------
#引入到当前的shell终端
. tmp.sh
hello #直接调用函数

#-----------------------
#引入到其它shell脚本 
. tmp.sh
echo $name #使用其他脚本的变量
```

另外一个shell脚本中的变量只能在此进程中使用，子进程都无法使用，为了让子进程使用和本脚本的变量，则可以使用`export <变量>` 方式

```bash
name="aa"
export name
bash a.sh #在a.sh中可以使用name并且改变name  同时会影响
```

​    

## echo和printf

```bash
echo "hello,\t world" #转义
echo 'hello,\t world' #不转义
```

```bash
printf "%s,\t%s\n" "hello" "world" #单引号双引号一样
```

​     

## 参考

- https://wangdoc.com/bash/intro.html 【阮一峰Bash脚本教程】