```bash
title: Linux命令查看手册
date: 2021-02-23
categories: [Linux]
tags: [Linux]
draft: true
```

## 导论

bash快捷键

```bash
ctrl+a #光标到行首
ctrl+e #光标到行尾
ctrl+u #剪切整行
ctrl+k #剪切光标位置到行尾
ctrl+w #删除光标前一个单词
ctrl+y #粘贴
ctrl+d #退出当前shell
ctrl+z #暂停任务 bg fg可以恢复
ctrl+c #终止前台进程
ctrl+r #搜索历史命令
```

开关机

```bash
#-h:halt的意思
#shutdown会进行一些清理工作 然后调用init 0
shutdown -h now #立刻关机
shutdown -h 1 #1分钟之后 shutdown不加参数默认就是这个
shutdown -h 11:00 #11点关机
shutdown -r  now #立刻重启
sync #将内存数据同步到磁盘 关机前都要执行这个命令
last #查看关机和启动的日志
```

其它

```bash
ctrl+x+e #如果需要在命令行输入多行命令可以通过此快捷键召唤vim输入命令 然后保存就出现在命令行上了
```

​        

## 文件和目录

### mkdir

```bash
-p  #递归创建 如果存在则忽略不会报错
```

### tree

```bash
-a #显示所有文件
-d #只显示目录
-i #不显示树枝
```

### ls

```bash
-l #长格式输出
-t #按最后修改时间降序(默认按照文件名)
-r #反序 ps: -tr按照时间升序号
-h #以人类可读的方式显示大小等信息
-a #列出所有文件
-S #根据文件大小降序
--time-style long-iso #指定时间格式 long-iso最完美

#--------不常用----------
--time atime#默认输出的是修改的时间,指定输出哪种时间 atime ctime
-u #按最后访问时间atime降序
-c #按最后状态修改时间ctime降序
-p #目录后面加上'/'
-F #在每个文件下加上文件类型指示符号 @/|=*....
-i #显示inode号
-d #只显示当前目录本身信息
```

![](https://raw.githubusercontent.com/biningo/cdn/master/2021-04/linux_file.png)

### stat

通过`stat`可以查看文件的详细信息

```bash
stat a.txt
```

### cp

```bash
cp -r abc ../home #递归copy abc->../home拷贝目录
cp -i  #如果有覆盖则提示是否需要覆盖
cp -d #如果copy的是符号连接则copy的是原文件 -d表示仅仅copy符号链接
cp -a #拷贝所有 -r -d 结合 包括创建时间等
```

### mv

```bash
mv -i #覆盖提示
mv -f #强制覆盖
```

### rm

删除应对策略:

- 禁止使用`rm -rf`
- `rm`指令用`mv`代替，将文件移动到其它路径

```bash
rm -f #强制删除
rm -i #删除确认
rm -r #递归删除
```

### rmdir

删除空目录

```bash
rmdir -p A/AA/AAA  #递归删除 如果父子目录都为空
```

### *ln

建立软硬链接

```bash
ln target newfile #建立硬连接 目录不可以建立硬连接
ln -s target newfile #建立软连接 目录可以建立软连接
```

### *find

`-atime`: 访问时间

`-mtime`: 修改时间

+1: 1天前    -1: 1天内

```bash
find . -atime -2 #查找两天内被访问过的文件
find . -mtime -5 #查找两天内被修改过的文件
```

`-type`按类型查找

```bash
find . -type f #f|d|s|p 普通文件|目录|socket|管道
```

反向查找`!`

```bash
find . ! -type d #查找不是目录的文件 
```

`-perm`按照目录权限来查找

```bash
find . -perm 700
```

`-size`按照大小来查找`cwbkMG`

```bash
find . -size +100M #查找大于100M的文件
find . -size -10b #查找小于10字节的文件
```

`-path` `-prune` 忽略搜索路径

```bash
#-prune 和-path结合使用表示忽略搜索路径  -path为真时执行-prune忽略,为假时执行-pint 这句话其实是 -path "xxx" -a -prune -o -print的缩写
find $PWD -path "$PWD/pkg" -prune -o -print
```

`-user`查找指定用户的文件

```bash
find .  -user abc
```

`-newer`查找比指定文件新的文件

```bash
ind . -newer act ! -newer caddy #查找比act新 比caddy旧的文件
```

`-maxdepth`指定查找的目录深度

```bash
find . -maxdepth 1 #查找当前路径下所有一级深度的文件
```

`-name`按照文件名进行查找

```bash
find . -name "*.jpg" #查找以jpg结尾的文件
find . -name "?.jpg" #查找a.jpg这样的文件
find . -name "[abcdef].jpg" #查找a.jpg b.jpg c.jpg d.jpg e.jpg f.jpg
```

`-exec`执行: 如果文件很多则效率很低，exec和xargs的区别就是exec是逐一执行的xargs是一起执行的

```bash
find . -type f -exec ls -l {} \; #最后\是转义;号的 -exec必须以;结尾 {}是占位符
#和下面的命令效果一样
find . -name "go*"  | xargs ls -l
```

`find`结合`xargs`

```bash
find . -type f | xargs -i mv {} dir/ #-i表示将查询出来的插入到{}中
find . -type f | xargs -p rm #-p表示有提示
find . -type f | xargs  tar -zcvf file.tar.gz #将当前路径下的文件都打包
find . -type f | xargs sed -i "s/hello/hi/g" #将所有文件中的hello替换为hi
```

多条件

```bash
find src1 src2 -name README.md #也可以指定多个目录
```

通过find移动文件的3种方法

```bash
find . -name "a" | xargs -i  mv {} dir/
find . -name "a" | xargs -i  mv -t dir/ #mv -t表示反转
mv `find . -name "c"` ./B #反引号表示解析命令结果
```

### *xargs

```bash
cat a | xargs #表示合并为一行 这里没有接命令 所以会输出到屏幕上
cat a | xargs -n 2 #只显示两列 每行最多输出2个

find -name "*.js" | xargs wc -l # 等价于 wc -l a.js b.js c.js ...
cat download.txt | xargs wget #批量下载
find . -name "*.js" | xargs -i mv {} dir1/ #{}代表占位符 -I [] 指定占位符
find . -name "*.js" | xargs -ip mv {} dir1/ #-p表示询问
```

### basename、dirname

```bash
basename /a/b/c #显示文件名或者目录名  c
dirname /a/b/c #显示目录路径 /a/b
```

### lsattr、chattr

[TODO]列出和改变文件的扩展熟悉

展示和改变文件的扩展属性

### file

```bash
file A #显示文件的类型
file -b A #不输出文件名 只显示精简信息
file *
```

### md5sum

生成文件的md5值，可以用于校验备份文件是否损坏

```bash
md5sum file #计算文件的md5值
md5sum file > md5sumVal
md5sum -c md5sumVal #检查md5值和现在文件的md5值是否一样 也就是文件是否改变
```

### *chown

更改文件所属的用户或则用户组

```bash
chown 用户 a.txt
chown :组 a.txt
chown 用户:组 a.txt
chown -R #递归更改
```

### *chmod

更改文件的读写权限

```bash
chmod u+x
chmod a+x #a=u+g+oaa
chmod a-x
chmod 653 #用数字代替
chmod u=rwx,g=xw
chmod -R #递归
```

### umask

定义了文件创建时的默认权限的掩码

```bash
666 #默认初始权限
022 - #掩码
-----
644

umask #默认是数值方式输出
umask -S #以字符方式输出掩码
umask 003 #修改掩码
```

> 文件权限和数字的一个对应可以查看这个网址 https://linxz.github.io/tianyizone/linux-chmod-permissions.html

​    

## Linux信息显示和搜索文件

### *uname

显示系统相关的参数信息

```bash
uname -a #显示所有
uname -n #主机名称
uname -v #内核版本
```

### *hostname

```bash
hostname #显示主机名称
hostname -I #显示主机所有IP
```

### *du

统计磁盘使用情况

```bash
du -h #统计当前路径下的文件磁盘使用情况 -h表示文件大小以人类可读的方式展示
du -a #显示所有文件
du -h test #指定目录
du -s #当前目录总大小
```

### *date

显示系统时间

```bash
date +%Y #2021
date +%Y-%m-%d-%H:%M:%S #完整显示
date +%Z #显示时区
date +%R # %H:%M
date +%F # %Y-%m-%d
date '+%F %R' #%Y-%m-%d %H:%M:%S
date +%N # 显示纳秒
```

### cal

```bash
cal #显示当前月份的日历
cal 2020 #显示2020年的日历
cal 5 2020 #显示5月日历 可以查看星期几
```

### *watch

监视命令的执行

```bash
watch -n 1 -d netstat -ant #监视网络 -n表示变化间隔1s -d表示高亮变化了的部分
watch -n 1 -d cat a.txt #监视文件的变化
```

### which和whereis

查找可执行命令或则文件的路径，在PATH路径下查找

```bash
which bash
which git
```

查看路径以及man路径

```bash
whereis bash
```

### locate和updatedb

TODO

​    

## 压缩与备份

### **tar

打包和压缩工具，可以仅仅打包，也可以压缩并打包

```bash
#z:gzip压缩 v:详细信息 f:指定压缩名字

# c:创建一个新的tar包
tar -czvf  a.tar.gz ./* #压缩当前文件夹下的所有文件 并且压缩包的名字为a.tar.gz
du -h c-learn.tar.gz #压缩之后可以使用du命令对比大小

# t:不解压查看压缩包内容
tar -tzvf a.tar.gz

# x:解压
tar -xzvf a.tar.gz #默认解压到当前目录下
tar -xzvf a.tar.gc -C /tmp/ #指定解压路径
```

### unzip

解压zip文件

```bash
unzip -O cp936 'Redis 5设计与源码分析.zip' # -0是为了防止乱码
```

### scp

ssh方式传输文件

```bash
scp ./a.pdf root@aliyun:/root/ #上传文件到远程服务器
scp root@aliyun:/root/lyer-home/images ./images #从远程服务器中下载文件到本地
```

### **rsync

文件同步工具，如果目标文件不存在则会自动创建

同步本地文件

```bash
#也可用于本地两个文件同步 source文件夹带斜杠就是同步文件夹下面的文件
# a:递归  v:详细
# 没带斜线则会将目录本身复制到目录下

#将data1目录下的文件全部同步到data2目录下
rsync -av data1/ data2/
#将data1复制到data2目录下
rsync -av data1 data2/
#以data1目录为准  两个目录完全一致 会将data2目录下的不一致的删除
rsync -av data1/ --delete  data2/ #--delete让两个目录完全一致 不同的会被删除
```

同步远程文件

```bash
rsync -av 192.168.0.23:/data1/ data2/  #拉取
rsync -av data1/ 192.168.0.23:/data2/  #推送
```

​    

## 用户管理及用户信息查询

### 几个重要的文件

```bash
/etc/passwd #用户信息文件
/etc/shadow #用户密码文件
/etc/gshadow #组密码文件
/etc/group #用户组文件
/etc/login.defs #用户定义文件
/etc/default/useradd #创建用户的默认配置文件
/etc/skel #会将此目录下的文件复制到用户的家目录下
/etc/sudoers #sudo文件
```

`/etc/passwd`

```bash
testuser:x:1001:1001:TmpUser:/home/testuser:/bin/sh
```

| 用户名   | 密码标识(都为x) | 用户ID | 用户组ID | 用户说明 | 用户家目录     | shell解释器 |
| -------- | --------------- | ------ | -------- | -------- | -------------- | ----------- |
| testuser | x               | 1001   | 1001     | TmpUser  | /home/testuser | /bin/sh     |

`/etc/group`

| 组名   | 口令 | 组标识号 | 组内用户列表 |      |      |      |
| ------ | ---- | -------- | ------------ | ---- | ---- | ---- |
| docker | x    | 134      | u1           | u2   | u3   |      |

### useradd

useradd如果不指定用户组的话默认会创建一个和用户名相同的用户组

```bash
useradd -m testuser #-m家目录如果不存在则自动创建
useradd -s bash testuser #-s指定用户使用的sh类型
useradd -g testgroup -u 100 testuser #-g指定用户组 -u指定UID
useradd -e "2020-06-07" testuser #-e指定用户的过期时间
useradd -c TempUser testuser #-c指定用户的简短注释
```

### userdel

```bash
userdel -r testuser #删除目录的同时删除其关联的文件
userdel -f testuser #强制删除 即使用户已经登入
```

### *usermod

修改用户，和useradd参数一样

### groupadd和groupdel

```bash
groupadd testgroup
groupadd -g 1002 testgroup
groupdel testgroup #删除用户组
```

### *passwd

```bash
passwd #修改当前用户的密码
passwd testuser #修改指定用户的密码
echo "123456" | passwd --stdin testuser #一条设置密码

#5天内不能更改密码
#30天之后必须修改密码
passwd -n 5 -x 30 testuser

#账户过期前7天告诉用户
#过期后60天禁止用户登入
passwd -w 7 -i 60 testuser
```

### chage

修改用户密码有效期

### chpasswd

批量更新用户密码

### *su和sudo

`su`切换用户 `sudo`根据`/etc/sudoers`文件借权代替执行

```bash
su root #切换到root用户
sudo ls /etc
```

### *id

```bash
id #显示当前用户和用户组信息
id testuser #指定用户名
```

### *w、who、users、whoami、last、lastb

```bash
#显示已经登入的用户
w
who
users
whoami #显示当前用户名
last #显示历史登入列表
lastb #显示登入失败记录
lastlog
```

​    

## ?磁盘与文件系统管理

## ?进程管理

进程信息都保存在`/proc`下，进程信息展示都是通过读取目录下的信息进行展示的

### *ps

```bash
ps -e #显示所有进程
ps -l #详细显示进程状况
ps -f #显示UID PPID C 等详细栏位
ps f #显示进程树结构
```

展示当前系统的所有进程

```bash
ps -ef #另外一种风格的进程信息
ps -aux #显示BSD风格的进程信息
```

查找指定进程

```bash
ps -ef | grep ssh 
```

### *top

实时查看系统资源情况

```bash
top #默认按照CPU占用率排序
top -c #显示命令路径
top -d 1 #修改刷新间隔1s top进入之后按s再进行修改也可
top -n 5 #更新5次之后退出
top -H #显示线程  默认只显示进程
top -p 12355 #指定某个pid的进程
按键1 查看单独CPU的信息
按键s 修改更新时间
```

### *pstree

显示某个进程的进程树

```bash
pstree 15835
-a #显示进程完整信息
-p #显示pid
```

### pgrep

ps -ef和grep效果一样

```bash
pgrep docker #只显示pid
pgrep docker -a #等同于 ps -ef | grep docker
```

### *kill

```bash
kill -l #列出所有信号
kill -15 12322 #发送指定信号到进程
kill -9 1111 #强制终止
```

### killall

通过进程名字终止进程，如果有多个子进程则需要执行多次

```bash
killall mysql
```

### *pkill

通过进程名终止进程包括子进程，只需要执行一次

```bash
pkill mysql
```

### nice和renice

忽略

### *nohup

 `no hang up`（不挂起），用于在系统后台不挂断地运行命令，退出终端不会影响程序的运行

```bash
nohup a.sh > /dev/null  2>&1 &
```

### jobs fg bg & nohub 

`&`  `jobs` `fg` `bg` `nohub`

```bash
./exe & #后台运行 但是日志还会打印在标准输出
./exe > log.txt 2>&1 & #0:标准输入 1:标准输出 2:标准错误 2>&1是将标准出错重定向到标准输出 最终结果就是`标准输出`和`错误`都被重定向到`log.txt`中
etcd > /dev/null  2>&1 & #后台运行 同时输出日志重定向到垃圾桶
nohup etcd > /dev/null 2>&1 & #以守护进程方式运行 不会随着终端退出而退出
jobs #查看后台运行中的进程

#ctrl+z暂停任务 ctrl+c终止任务
fg $1 #将后台任务放到前台执行
bg $1 #将一个暂停的任务放在后台执行
```

### **strace

跟踪进程TODO

### **ltrace

跟踪进程使用了哪些系统调用TODO

### **runlevel

输出当前系统的运行级别

```bash
runlevel
```

### **init

TODO

### **systemctl

管理系统服务(守护进程)，systemctl命令是service命令和chkconfig命令的集合和代替

```bash
#系统服务都在此路径下:/usr/lib/systemd
systemctl start docker
systemctl stop docker 
systemctl restart docker
systemctl status docker
#开启/关闭服务开机启动
systemctl enable docker
systemctl disable docker
systemctl is-enabled docker

systemctl list-unit-files #列出所有守护进程的信息
```

​     

## ?网络管理

### *ifconfig

```bash
ifconfig #查看所有网卡信息
ifconfig wlp0s20f3 #查看指定网卡信息
ifconfig wlp0s20f3 up/down #开启/关闭网卡
ifconfig wlp0s20f3 192.168.0.24/24 #为网卡分配IP 同时也指定了子网掩码
ifconfig wlp0s20f3 192.168.0.24/24 up
```

### ifup和ifdown

激活/关闭网卡，和`ifconfig up/down` 命令作用一样

```bash
ifup eth0
ifdown eth0
```

### *route

显示本机的静态路由表

```bash
route -n
```

### arp

显示本机ARP缓存

```bash
arp -n #显示IP
```

### *ip

TODO

### **netstat

```bash
-a #显示所有socket
-i #显示网卡
-n #直接显示IP 不显示域名
-u #只查看UDP
-t #只查看TCP
-s #查看统计信息
-4 #只显示ipv4的套接字
-6 #只显示ipv6的套接字
-l #只查看监听中的socket
-p #显示那个PID在监听

netstat -an #显示所有连接信息
netstat -ant
netstat -anu
netstat -antp

netstat -i #显示本机的网络接口
netstat -r #显示路由表
```

### **ss

`ss`   是`Socket Statistics`的缩写，用来查看本机网络连接状况，和netstat一样，但是查询速度更快，用法和netstat一样

```bash
ss -an #所有
ss -ntlp #TCP
ss -antp #TCP
ss -nulp #UDP
ss -ntulp #TCP UDP正则监听的
ss -s #查看网络统计信息
```

### *ping

```bash
-c 2 #发送次数2
-i 1 #发送间隔1s

#发送3个包 每隔2s发送 包大小为1024字节
ping -c 3 -i 2 -s 1024 google.com 
```

### traceroute

### telnet

### arping

### nc

### wget

下载文件工具，支持FTP、HTTP

```bash
-q #静默模式 关闭下载日志输出
-b #后台下载
-O #指定下载文件名
wget https://mirrors.tuna.tsinghua.gz
wget -O a.tar https://mirrors.tuna.tsinghua.gz
```

### utw

`utw` (centos下为`firewall`，ubuntu下为`utw`)    ubuntu下防火墙iptables工具

默认的配置文件`/etc/default/ufw `

```bash
#注意 没有出现在iptables中的一律不允许访问
ufw enable/disable
ufw status

#日志文件储存在 /var/logs/ufw
ufw logging on/off #启用关闭日志
ufw logging low|medium|high #设置日志级别

ufw allow 1000 #开放1000短端口
ufw deny 1000 #关闭1000端口 有数据包直接丢弃 无响应
ufw reject 1000 #关闭1000端口 有数据包接受则返回一个拒接包
ufw delete  reject 1000 #删除规则

ufw allow ssh #允许指定端口号或则对应的服务名 这里和22一样
ufw allow 80/tcp #还可以指定TCP或则UDP 只允许80端口上的TCP包
ufw allow http/tcp

ufw allow from 192.168.1.1 #允许指定IP连接到任何端口
ufw allow from 192.168.1.0/24 #允许特定子网连接到任何端口
ufw allow from 192.168.1.1 to any port 8080 #允许指定IP连接到8080端口 tcp/udp都可
ufw allow from 192.168.1.1 to any port 8080 proto udp #指定协议UDP

ufw status numbered #列出每个规则的序号 可以按照序号进行删除
ufw delete 1
```

### dig

`dig` 域名查询工具

```bash
dig @8.8.8.8 baidu.com #指定向8.8.8.8DNS服务器查询
dig @8.8.8.8 -p 53 baidu.com #还可以指定查询服务器的端口 默认是53
dig +trace baidu.com #显示整个DNS查询过程
dig +trace +additional baidu.com #显示更详细的信息
dig +short baidu.com #简化查询结果 只显示查询得到的IP
dig +x 111.111.111.111 #反向解析
dig ns baidu.com #查看这个域名的ns记录
dig a baidu.com #查看A记录
dig cname baidu.com
```

​      

## ?Linux系统管理命令

开关机

```bash
#-h:halt的意思
#shutdown会进行一些清理工作 然后调用init 0
shutdown -h now #立刻关机
shutdown -h 1 #1分钟之后 shutdown不加参数默认就是这个
shutdown -h 11:00 #11点关机
shutdown -r  now #立刻重启
#重启
reboot #init 6

sync #将内存数据同步到磁盘 关机前都要执行这个命令
last #查看关机和启动的日志
```

## ?系统内置命令

## 参考

[https://github.com/xjh22222228/linux-manual#cp](https://github.com/xjh22222228/linux-manual#cp)

[看完这篇Linux基本的操作就会了](https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247484231&idx=1&sn=4cf217a4d692a7aba804e5d96186b15b&chksm=ebd74246dca0cb5024de2f1d9f9e2ecb631e49752713c25bbe44f44856e919df5a973049c189#rd)

《跟老男孩学Linux运维:核心命令系统实战》    

《鸟哥的Linux私房菜》

《Linux命令行与Shell脚本编程大全》

[应该知道的LINUX技巧](https://coolshell.cn/articles/8883.html)

[28个UNIX/LINUX的命令行神器](https://coolshell.cn/articles/7829.html)
