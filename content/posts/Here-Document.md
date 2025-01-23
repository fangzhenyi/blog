---
title: Here Document了解一下
date: 2018-06-23T20:01:46.000Z
tags: ubuntu linux
---
### 发现问题
经常在编写shell脚本的时候，结果脚本里面有一个命令需要从命令行输入内容才能继续执行。比如有如下脚本1.sh，有如下代码，假设我们想静默的调用这个脚本：

```shell
#!/bin/bash
echo "用户名"
read username
echo $username
echo "密码"
read password
echo $password

```
但是当我们执行./1.sh的时候，我们需要从键盘输入一些数据，脚本才可以继续向下执行。那我们怎么才能预定义好要输入的数据，让这个程序静默执行呢。
### here document来解决问题。
Here Document 是在Linux Shell 中的一种特殊的重定向方式，它的基本的形式如下

```
cmd << delimiter
  Here Document Content
delimiter
```
因此我们执行上面那个脚本，可以这样执行：

```
./1.sh << delimiter
  fzy
  123
delimiter
```
可以得到如下结果：

```
promote:~ fangzhenyi$ ./1.sh << delimiter
>   fzy
>   123
> delimiter
用户名
fzy
密码
123
```
但是，假设我们执行这个脚本，只需要一行呢，这个时候该怎么办呢（编写dockerfile很多命令都需要一行，换行会带来很多莫名其妙的问题）。一行的话，我们可以采用如下写法。

```
promote:~ fangzhenyi$ echo -e "fzy\n123" | ./1.sh 
用户名
fzy
密码
123
```






