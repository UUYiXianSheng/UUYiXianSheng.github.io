---
layout: post
title: yum安装gcc、libc-heard时报错
tags: gcc
---

### 介绍

背景：Centos7.4安装NVIDIA的Tesla驱动时报错，缺少libc-heard文件，遂安装gcc、libc-heard，安装时又报错如下图。

![](/images/blog/LinuxYumINstallGccERR.jpg)

1.处理方式

执行命令： 

```     
$ yum distro-sync
```    

此命令是把本地的软件版本变成和yum源上最新的版本一致。可能升级也可能降级。关键是看你本地已安装软件版本和yum源上版本。对软件包组不生效。

2.再执行： yum install 进行安装即可

```     
$ yum install gcc libc-heard
```    
