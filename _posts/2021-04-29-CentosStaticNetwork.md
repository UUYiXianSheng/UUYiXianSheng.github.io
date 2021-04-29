---
layout: post
title: Centos7*配置静态ip
tags: Centos,Network
---

### 介绍
本文将介绍在Centos7.*系统上面如何配置静态网络ip。

1.查看网口信息
```     
$ ifconfig   
```    
2.确认要改的ip

可以使用ping命令，例：
```     
$ ping 172.25.3.11    
```    
3.编辑配置文件

按照实际的网口来，例：
```     
$ vim /etc/sysconfig/network-scripts/ifcfg-ens192   
```    

内容如下，请按照实际要求配置
```     
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
IPADDR=172.25.3.11
NETMASK=255.255.255.0
GATEWAY=172.25.3.254
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=eno1
UUID=25a98524-aa29-4f17-af93-1afd6afd415a
DEVICE=eno1
ONBOOT=yes 
```    
4.重启网络
```    
service network restart
```    


