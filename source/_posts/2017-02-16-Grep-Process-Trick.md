---
title: Grep Process Trick
date: 2017-02-16 23:24:05
tags: [shell]
---

两个今天学到的 Shell tricks.
- grep process
- !$

## grep process

一个我们常用的命令

> $ ps -ef | grep "java"
>   502 13749 12934   0 11:29PM ttys002    0:00.00 grep java

这个命令本身没什么问题，主要是在输出上。
每次都会带上 grep 命令本身的进程。
直觉上我都会用一段代码去过滤这条，从而得到真正进程号。
今天看到一个简洁的写法

> $ ps -ef | grep "[j]ava"
>

解释下 [j]ava, 很有意思的办法，说出来也很好理解。
[j]ava 对于 grep 命令来说是 regex ,也就是按照 [j]ava, 进行过滤。
符合条件的当然是"java",这个最终的字符串。
但是 grep 命令在对应进程所展现出来的却是 grep "[j]ava"，却是不符合上面的正则

## !$

我经常打错命令，但是后面的参数却没有问题。
例如

> $ cd /usr/local/app/conf/a.conf

事实上我是想删除掉 a.conf 这个文件。以前的做法都是使用 iTerm 的快捷键快速移动光标到到行首病删除 cd。
这个我用的也很顺手，但是总有一些环境的 terminal 没有那么好用。
可以试试下面的命令

> $ cd !$