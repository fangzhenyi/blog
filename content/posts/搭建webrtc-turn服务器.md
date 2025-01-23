---
title: webrtc搭建turn服务器
date: 2016-07-16T11:29:35.478Z
tags: webrtc
---

### 1.搭建turn服务器目的

部署WebRTC 或 SIP p2p 方案时经常会遇到p2p 无法穿透的环境，

这时就是TunServer 的用武之地了。

这里我们使用turnserver-0.7.3

### 2.下载confuse依赖库

```
wget http://savannah.nongnu.org/download/confuse/confuse-2.7.tar.gz

tar zxvf confuse-2.7.tar.gz

cd confuse*

./configure

make && make install
```


### 3.下载turnserver

```
wget http://downloads.sourceforge.net/project/turnserver/turnserver-0.7.3.tar.bz2 
tar jxvf turnserver-0.7.3.tar.bz2

cd turnserver*

./configure

make && make install
```

### 4.编辑配置文件

将extra 中的配置文件模版拷贝到/etc目录下,假设您的ip 是 1.2.3.4

1，配置文件

```
cp extra/turnserver.conf.template /etc/turnserver.conf 
vi /etc/tunserver.conf
```

修改

listen_address = { "1.2.3.4" }

修改 ## Daemon mode. daemon = true # 修改为后台服务方式

修改带宽限制

```
##Allocation bandwidth limitation (in KBytes/s). ## 0 value means bandwidth quota disabled. bandwidth_per_allocation = 1024   
##Restricted user bandwidth (in KBytes/s). ## 0 value means bandwidth limitation disabled. restricted_bandwidth = 0
```

2，认证用户文件

```
cp extra/turnusers.txt.template /etc/turnusers.txt 
vi /etc/turnusers.txt
```


添加一行或多行认证信息格式为 用户名:密码:domain:authorized 例如下面的行: 700:700pass:domain.org:authorized

添加完成后，就可以在webrtc 里面使用stun 和tunserver 了。
 
```var configuration = { 
'iceServers': [{ { 'url' : 'stun:1.2.3.4'} , { 'url' : ‘turn:700@1.2.3.4',credential : '700pass'} }] };
```
