---
title: git原理
date: 2021-04-30
categories: [git]
tags: [git]
draft: true
---

## git三大对象

git对象都保存在`objects`目录下，git三大对象分别为:

- `blob` 记录的是每个变更的真实文件的内容，blob不记录文件名只记录内容，如果两个文件内容相同则只会生成一个blob，每次对文件的修改都会生成一个完整的blob
- `tree` 记录的是文件名以及文件对应的blob对象指针和文件权限
- `commit` 记录tree对象、上一个commit版本、发布者等信息
- `tag` 使用`-a`创建的tag会创建对应的tag对象，轻量型tag只是对commit的一个命名，而重量型tag会创建tag对象

由于git将内容进行了压缩采用二进制存储，所以如果我们想要查看对象，查看对象内容和类型的相关命令

```bash
git cat-file -t 7b9473 #查看对象的类型
git cat-file -p 7b9473 #查看对象的内容
git ls-files -s #查看索引区的文件内容
```

​     

## git的index区域

git中有一个`index`文件，保存了目录下所有文件名、文件对应的blob对象、权限等信息，只会保存文件而不会单独保存目录

```bash
$ git ls-files -s #查看索引区中的内容
100644 07eb8ab765f8dcc9472f94e43cbe8e9b6f5cc47f 0	folder/hilyer
100644 2d832d9044c698081e59c322d5a2a459da546469 0	helloworld
```

`index`区域保存的就是项目下的所有文件的最新add时指向的blob，每次commit操作都是基于index生成一个快照，也就是生成tree对象以及commit对象。在tree对象中保存的就是当前index区域的信息，如果有目录则会以树形结构的tree来保存

每次add时就会更新index区域中文件对应的blob，使index区域的文件指向最新的blob对象。git可以将index区域的内容和工作区的内容进行比较就可以知道工作区哪些文件的内容发生了变化

​     

## git分支和tag实现原理

在`.git`目录下的`HEAD`文件中记录的是当前目录所在的分支的路径名，比如

```bash
$ cat .git/HEAD
ref: refs/heads/master
```

`.git/refs/heads`路径下面记录的就是所有的分支文件了，每个分支文件的名字就是分支名，内容就是对应的commit版本号

```bash
$ ls .git/refs/heads
dev  master
```

`.git/refs/tags`下面记录的就是所有tag的信息，和分支一样，文件名就是tag名字，内容就是commit版本号

```bash
$ ls .git/refs/tags
v1.0 v2.0
```

切换分支其实就是切换HEAD的指向，然后通过HEAD找到分支文件对应的commit，通过commit就可以找到tree对象，然后就可以找到所有的blob了

​    

## git垃圾对象

TODO

## 参考

[这才是真正的Git——Git内部原理](https://juejin.cn/post/6844904019245137927)

[三道 google 风格 git 面试题及其解答](https://juejin.cn/post/6844903876743659534)

[Git基本原理-麦兜搞IT](https://space.bilibili.com/364122352/channel/detail?cid=150242&ctype=0)

[learnGitBranching](https://github.com/pcottle/learnGitBranching)

[git-flight-rules](https://github.com/k88hudson/git-flight-rules)

[Git的奇技淫巧](https://github.com/521xueweihan/git-tips)

[Git常用命令参考手册](https://juejin.cn/post/6844904146571624461)

[Git基本原理介绍](https://www.escapelife.site/posts/da89563c.html)

