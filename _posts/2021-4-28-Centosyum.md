---
layout: post
title: Centos挂载镜像文件制作yum源
tags: Centos,yum
---

### 介绍
本文将介绍在Centos7.*主机上面如何通过镜像文件制作yum源。



1.上传iso镜像文件至/usr/local/src目录

通过rz命令或者xftp工具等上传符合要求的镜像文件

2.创建镜像挂载目录 
```     
$ mkdir /mnt/CentOS7     
```    

3.挂载本地镜像文件

例:
```     
$ mount -t iso9660 -o loop /usr/local/src/CentOS-7-x86_64-Everything-1708.iso  /mnt/CentOS7    
```    
注意文件名和挂载目录根据实际的来!!!

4.配置开机自动挂在

```     
$ vi /etc/fstab    
```    
增加以下内容： 
```     
/usr/local/src/CentOS-7-x86_64-Everything-1708.iso /mnt/CentOS7 iso9660 defaults,ro,loop 0 0
```    

5.配置以及备份repo文件
备份：
```     
$ cd /etc/yum.repos.d
$ mkdir bak
$ mv *repo bak/
```    
 在当前目录下配置新的repo文件
```     
$ vim CentOS7-Localsource.repo    
```    
 添加以下内容

```     
[CentOS7-Localsource] 
name=CentOS7 
baseurl=file:///mnt/CentOS7 
enabled=1
gpgcheck=0   
```    
 
 
 
6.清除yum源缓存，重新构建yum源

清除yum源缓存：
```     
$ yum clean all   
```    

重新构建yum源
```     
$ yum makecache  
```    



