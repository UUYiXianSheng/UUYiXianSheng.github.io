---
layout: post
title: Linux修改系统时间
tags: Linux,date
---

### 介绍
本文将介绍在Linux(本文使用的是RedHat7.4)环境如何修改系统时间。注:修改时间需要root权限

1.查看当前系统时间
```     
$ date  
```    

2.局域网环境（无外网）
```     
$ date -s 10:10:10 
```    

或者date -s  完整日期时间（YYYY-MM-DD hh:mm[:ss]）
```     
$ date -s "2021-06-24 13:11:10"
``` 

然后执行如下命令将系统时间同步到bios。
```     
$ hwclock -w
``` 

3.通过ntpdate工具同步网络时间
首先安装ntpdate工具
```     
$ yum install -y ntpdate
```    

接着执行如下命令开始同步：
```     
$ ntpdate 0.asia.pool.ntp.org
```    

若上面的时间服务器不可用，也可以改用如下服务器进行同步：
```     
$ time.nist.gov
$ time.nuri.net
$ 1.asia.pool.ntp.org
$ 2.asia.pool.ntp.org
$ 3.asia.pool.ntp.org
```    

最后执行如下命令将系统时间同步到bios，防止系统重启后时间被还原。
```     
$ hwclock -w
```    
