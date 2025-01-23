---
title: ubuntu启动就进入grub页面
date: 2017-12-23T20:01:46.478Z
tags: ubuntu linux
---
#### 一.发生现象   
在我给服务器装好了ubuntu系统16.04.3的系统后，修改了fstable配置项然后进行重启，然后就发现ubuntu系统起不来了，还原配置仍旧启动不起来。插上显示器就发现服务器进入了grub命令行页面。grub页面上面的标题显示Minimal BASH-like line editing is supported. For [duplicate]。以为是修改配置项出错，只能重新从U盘启动ubuntu 16.04系统。try install安装ubuntu系统，还原了各项配置项，仍旧无法启动。  
#### 二.解决方法 
在网上查找了各项资料。搜到一个可以外文资料，按照操作，成功修复操作系统。现在记录操作如下。  
1.制作u盘启动盘，启动ubuntu。  
2.通过u盘启动，进去ubuntu try intall，进入系统。  
3.确定自己的系统装在那个系统分区下面。  
4.执行命令,取代sda2用当前自己的系统分区

```  
sudo mount /dev/sda2 /mnt
for i in /sys /proc /run /dev; do sudo mount --bind "$i" "/mnt$i"; done
sudo chroot /mnt
update-grub

```   
附上原文地址：https://elementaryos.stackexchange.com/questions/2716/error-in-grub-minimal-bash-like-line-editing-is-supported-for



