---
title: 'hdfs的JournalNode角色高可用理论原理'
date: 2018-03-20T22:37:40.000Z
tags: java hdfs
---
### 前言
在hdfs的namenode可以支持HA的时候，需要在两个namenode间进行共享editlog，hadoop社区提出了很多做法，可以通过配置nfs共享文件夹，也可以配置JournalNode去共享editlog。每次active的JournalNode都会写editlog到JournalNode，StandBy的JournalNode会去JournalNode拉取editlog。但是问题又来了，怎么保证JournalNode的高可用呢。社区的相关讨论都在这里https://issues.apache.org/jira/browse/HDFS-3077

### 理论基础
JournalNode采用的Quorum协议，去保证数据的一致性，通过添加奇数个节点去保证高可用。只要一半以上的节点可用，JournalNode就能正常提供服务。JournalNode每次写editlog都要保证写成功半数以上，才算写成功。

下面我来分析一下，为啥要半数以上的节点呢。这个是由Quorum协议规定的。Quorum协议非常简单理解。

它有三个变量。
N:复制节点数量  
R:成功读操作的节点数  
W:成功写操作的节点数   
 
假设有三个节点，那么我每次写2个节点，才算写成功。那么会有一个节点不包含更新的最新值，那么每次都读2个节点的话，那么我就可以读取到最新的值。所以JournalNode就利用这个原理，去保证数据的一致性和高可用。



