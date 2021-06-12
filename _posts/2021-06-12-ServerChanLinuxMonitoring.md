---
layout: post
title: ServerChan实现服务器资源告警推送至企业微信
tags: ServerChan,Linux
---

### 介绍
本文将介绍ServerChan如何实现服务器如何将资源信息推送至企业微信/钉钉等App。

1.登录/注册Server酱

打开Server酱主页登录https://sct.ftqq.com/，我是使用GitHub账号登录的，没有的可
以换其他登录方式或者注册一个，GitHub地址:https://github.com/
![](/images/blog/ServerChanEnter.jpg)
登录成功之后点击打开消息通道，如图：
![](/images/blog/NewsAisle.jpg)

2.添加和获取企业微信群机器人/钉钉群机器人/飞书机器人信息

此处我使用的是企业微信机器人其他群聊机器人获取信息方式大致一样暂不一一介绍。
打开企业微信群聊->点击右上角->向下划点击群机器人->点击右上角添加即可->给机器人取一个名字,例:方糖。
![](/images/blog/EnterpriseWeChatRobot.jpg)
复制机器人的Webhook地址粘贴到Server酱的消息通道的企业微信群机器人WebHook URL内。

3.获取Server酱的SendKey

回到Server酱页面点击导航栏的SendKey，按照API调用实例试着调用，Server酱介绍的很详细我这里不再阐述。
![](/images/blog/ServerChanSendKey.jpg)

4.服务器实现告警和推送消息

首先获取服务器信息，再用判断语句实现消息推送，我这里监控了cpu温度，内存和磁盘空间。读者有需求可以
自定义获取需要的信息，使用我的脚本需要安装sensors工具，这个是获取cpu温度的，下面的脚本供读者参考。
```     
#!/bin/bash
#统计系统信息
SERVERIP=''
USERNAME=''
#服务地址
SAVEPATH=''
#服务器ip
LOCAL_IP='*.*.*.*'
sys_dailyrate()
{
	#获取当前主机IP
	HOST_IP=`ifconfig | grep 'inet addr:' | awk '{print $2}' | cut -d':' -f2- | grep -v '127.0.0.1'`
	#获取当前主机名
	HOST_NAME=$USER
	#echo "HOST NAME: $HOST_NAME"
	#fetch and process memory information
	[ -f /proc/meminfo ] && {  #First judge whether the file exists
	#-e是模式的意思，常用来保护以破折号开头的模式。
	#-w是全字匹配。
	#-i忽略大小写。
	#-d指定分割符，-f为按照分割符取出的域，2-指的是域2及以后
	Buffers=`grep -we 'Buffers' /proc/meminfo | cut -d' ' -f2- | tr -d "[A-Z][a-z] "`
	Cached=`grep -we 'Cached' /proc/meminfo | cut -d' ' -f2- | tr -d "[A-Z][a-z] "`
	MemFree=`grep -ie 'MemFree' /proc/meminfo | cut -d' ' -f2- | tr -d "[A-Z][a-z] "`
	MemTotal=`grep -ie 'MemTotal' /proc/meminfo | cut -d' ' -f2- | tr -d "[A-Z][a-z] "`
	SwapCached=`grep -ie 'SwapCached' /proc/meminfo | cut -d' ' -f2- | tr -d "[A-Z][a-z] "`
	SwapFree=`grep -ie 'SwapFree' /proc/meminfo | cut -d' ' -f2- | tr -d "[A-Z][a-z] "`
	SwapTotal=`grep -ie 'SwapTotal' /proc/meminfo | cut -d' ' -f2- | tr -d "[A-Z][a-z] "`
	}
	MEMUSED="$(( ( ( ( $MemTotal - $MemFree ) - $Cached ) - $Buffers ) / 1024))"
	MEMTOTAL="$(( $MemTotal / 1024 ))"
	MEMFREE="$(( $MEMTOTAL - $MEMUSED ))"
	MEMPER="$(( ( $MEMUSED * 100 ) / $MEMTOTAL ))"
	[ "$SwapTotal" -gt "1" ] && {
	 SWAPUSED="$(( ( ( $SwapTotal - $SwapFree ) - $SwapCached ) / 1024 ))"
	 SWAPTOTAL="$(( $SwapTotal / 1024 ))"
	 SWAPFREE="$(( $SWAPTOTAL - $SWAPUSED ))"
	 SWAPPER="$(( ( $SWAPUSED * 100 ) / $SWAPTOTAL ))" 
	} || {
	 SWAPUSED="0"
	 SWAPTOTAL="0"
	 SWAPPER="0" 
	}
	
	#获取CPU使用率
	TIME_INTERVAL=5
	LAST_CPU_INFO=$(cat /proc/stat | grep -w cpu | awk '{print $2,$3,$4,$5,$6,$7,$8}')
	LAST_SYS_IDLE=$(echo $LAST_CPU_INFO | awk '{print $4}')
	LAST_TOTAL_CPU_T=$(echo $LAST_CPU_INFO | awk '{print $1+$2+$3+$4+$5+$6+$7}')
	sleep ${TIME_INTERVAL}
	
	NEXT_SYS_IDLE=$(echo $NEXT_CPU_INFO | awk '{print $4}')
	NEXT_TOTAL_CPU_T=$(echo $NEXT_CPU_INFO | awk '{print $1+$2+$3+$4+$5+$6+$7}')
	#系统空闲时间
	SYSTEM_IDLE=`echo ${NEXT_SYS_IDLE} ${LAST_SYS_IDLE} | awk '{print $1-$2}'`
	#CPU总时间
	TOTAL_TIME=`echo ${NEXT_TOTAL_CPU_T} ${LAST_TOTAL_CPU_T} | awk '{print $1-$2}'`
	CPU_USAGE=`echo ${SYSTEM_IDLE} ${TOTAL_TIME} | awk '{printf "%.0f", 100-$1/$2*100}'`
	#统计磁盘使用率
	#DISK_TOTAL=`df -k | grep -v  "tmpfs" | egrep -A 1 "mapper|sd" | awk 'NF>1{print $(NF-2)}' | awk -v total=0 '{total+=$1}END{printf "%.2f\n",total/1048576}'`
	DISK_TOTAL=`df -k |  egrep -A 1 "mapper|sd" | awk 'NF>1{print $(NF-4)}' | awk -v total=0 '{total+=$1}END{printf "%.2f\n",total/1048576}'`
	#DISK_USD=`df -k | grep -v "tmpfs" | egrep -A 1 "mapper|sd" | awk 'NF>1{print $(NF-3)}' | awk -v used=0 '{used+=$1}END{printf "%.2f\n",used/1048576}'` 
	DISK_USD=`df -k | egrep -A 1 "mapper|sd" | awk 'NF>1{print $(NF-3)}' | awk -v used=0 '{used+=$1}END{printf "%.2f\n",used/1048576}'` 
	DISK_FREE=`df -k | egrep -A 1 "mapper|sd" | awk 'NF>1{print $(NF-2)}' | awk -v free=0 '{free+=$1}END{printf "%.2f\n",free/1048576}'`
	DISK_USAGE=`echo $DISK_FREE $DISK_TOTAL | awk '{printf "%.0f", 100-$1/$2*100}'`
	#cpu温度
	CPU_T=`sensors |grep -n "Package id 0"| awk '{print $4}'|sed "s/+//g"|sed "s/.0.*//g"`
                #系统当前时间
	DATE=`date +%Y:%m:%d:%H:%M:%S`
	#打印log
	logPath=/home/liuyan/txt/
	log=$logPath$LOCAL_IP"_info.txt"
	echo "Time : $DATE" >>$log
	echo "Memory Total: $MEMTOTAL MB" >>$log
	echo "Memory Free: $MEMFREE MB" >>$log
	echo "Memory Used: $MEMPER%" >>$log
	echo "CPU Used: $CPU_USAGE%" >>$log
	echo "DISK Total: $DISK_TOTAL GB" >>$log
	echo "DISK Freee: $DISK_FREE GB" >>$log
	echo "DISK_USD: $DISK_USD" >>$log
	echo "CPU_T:" $CPU_T °C>>$log
	echo >>$log
	
	#scp $log  $USERNAME@$SERVERIP:$SAVEPATH
	#cp $log $SAVEPATH
	#监控内存使用情况
	Memopy=Warning:Memory_Usage_Over_80%!
	MOne=https://sctapi.ftqq.com/****************.send?title=${LOCAL_IP}"&"desp=$Memopy
	if [ $MEMPER -gt 80 ]
	then
		echo $DATE>>ser.log
		echo $Memopy>>ser.log
         	curl $Mone
	else
		echo $DATE>>ser.log
		echo "Memory health">>ser.log
	fi
	#监控CPU温度依赖sensors工具
	CPUT=Warning:CPU_Temperature_Over_60_Degrees!
	CTwo=https://sctapi.ftqq.com/****************.send?title=${LOCAL_IP}"&"desp=$CPUT
	#echo $CPU_T
	if [ $CPU_T -gt 60 ]
        then
                 echo $CPU >>ser.log
	         curl $CTwo
        else
                 echo "CPU health">>ser.log
        fi
	#监控磁盘空间
	DISK=Warning:Disk_Space_Usage_is_Over_80%!
	DThree=https://sctapi.ftqq.com/****************.send?title=${LOCAL_IP}"&"desp=$DISK
	#echo $DISK_USAGE
	if [ $DISK_USAGE -gt 80 ]
        then
                  echo $DISK >>ser.log
                  curl $DThree
		  echo >>ser.log
        else
                  echo "Disk health">>ser.log
		  echo >>ser.log
        fi

}
#while true; do
#统计系统信息 
sys_dailyrate


#休眠时间
#interval=600
#sleep $interval
#done
```    
这个脚本可以按照进程启动或者做成定时任务，提供给读者选择。