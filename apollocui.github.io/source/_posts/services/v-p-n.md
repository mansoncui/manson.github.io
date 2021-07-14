---
layout: post
title: v-p-n
date: 2019-06-28 13:55:24
tags: 
    - [ v-p-n ]
categories:
    - [ services ]
---
## vpn 安装
   ##### 更换内核，安装锐速加速软件(可以跳过)
```
[root@vultr_test ~]# rpm -qa|grep kernel
kernel-3.10.0-862.el7.x86_64
kernel-tools-libs-3.10.0-862.14.4.el7.x86_64
kernel-3.10.0-862.14.4.el7.x86_64
kernel-tools-3.10.0-862.14.4.el7.x86_64
[root@vultr_test ~]# rpm -ivh http://soft.91yun.org/ISO/Linux/CentOS/kernel/kernel-3.10.0-229.1.2.el7.x86_64.rpm --force
Retrieving http://soft.91yun.org/ISO/Linux/CentOS/kernel/kernel-3.10.0-229.1.2.el7.x86_64.rpm
Preparing...                          ################################# [100%]
Updating / installing...
   1:kernel-3.10.0-229.1.2.el7        ################################# [100%]
[root@vultr_test ~]# rpm -qa|grep kernel
kernel-3.10.0-862.el7.x86_64
kernel-tools-libs-3.10.0-862.14.4.el7.x86_64
kernel-3.10.0-862.14.4.el7.x86_64
kernel-tools-3.10.0-862.14.4.el7.x86_64
kernel-3.10.0-229.1.2.el7.x86_64

降内核:
rpm -ivh http://xz.wn789.com/CentOSkernel/kernel-3.10.0-229.1.2.el7.x86_64.rpm --force

rpm -qa | grep kernel

wget -N --no-check-certificate https://raw.githubusercontent.com/wn789/serverspeeder/master/serverspeeder.sh  #锐锋加速VPN

bash serverspeeder.sh
```
---
   ##### 安装工具
```
[root@vultr_test ~]# yum -y install net-tools vim
[root@vultr_test ~]# wget --no-check-certificate https://bootstrap.pypa.io/get-pip.py
[root@vultr_test ~]# python2.7 get-pip.py`

安装并配置ss
(1)安装
[root@vultr_test ~]# pip install shadowsocks

(2)配置
[root@vultr_test ~]# vim /etc/shadowsocks.json
{
    "server":"0.0.0.0",
    "server_port":8765,
    "password":"bd87d3600c6baffe20f0a03ebf17985f",
    "timeout":300,
    "method":"rc4-md5"
}

{
    "server":"0.0.0.0",
    "server_port":8765,
    "password":"ef487f64b19e581554564ab8635c8995",
    "timeout":60,
    "method":"rc4-md5"
}

(3)启动
[root@vultr_test ~]# ssserver -c /etc/shadowsocks.json -d start 
[root@vultr_test ~]# netstat -anpt |grep python
tcp        0      0 0.0.0.0:8765            0.0.0.0:*               LISTEN      1162/python2.7  

(4)关闭防火墙
[root@vultr_test ~]# systemctl stop firewalld
[root@vultr_test ~]# systemctl disable firewalld
```
---
##### 优化并添加monit监控
```
#!/bin/sh
### BEGIN INIT INFO
# Provides:          ssserver
# Required-Start:    $remote_fs $network
# Required-Stop:     $remote_fs $network
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: starts crontab-ui
# Description:       starts the crond Process Manager daemon
### END INIT INFO

prefix=/usr/bin/ssserver

case $1  in 
	start)
	${prefix} -c /etc/shadowsocks.json -d start  > /dev/null 2>&1 &
	if [ $? -eq 0 ];then
		echo "start ssserver ....... ok"
	else
		echo "start ssserver ....... fail"
	fi
	;;

	stop)
	${prefix} -c /etc/shadowsocks.json -d stop
	if [ $? -eq 0 ];then
		echo "stop ssserver ....... ok" 
	else
		echo "stop ssserver .......fail"
	fi
	;;
	status)
	process=`pgrep -f '/usr/bin/ssserver' | wc -l` 
	if [ ${process} -eq 1 ];then
		echo "ssserver is running ........!!!"
	else
		echo "ssserver is not running ........!!!"
	fi	
	;;
	
*)
	echo "Usage: $0 {start|stop|status}"
	;;
esac
```
---
##### monit 配置
```
####ss
check process ssserver with  matching "/usr/bin/ssserver"
      start program = "/etc/init.d/ss start"
      stop program = "/etc/init.d/ss stop"

新版本安装ss

参考链接:https://www.e-learn.cn/content/qita/838991
CENTOS7安装ShadowSocksR与加速软件锐速
```

## 安装ssr
##### 下载安装脚本
```
[root@VPS ~]# wget –no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocksR.sh
[root@VPS ~]# chmod +x shadowsocksR.sh

安装ss
[root@VPS ~]# ./shadowsocksR.sh 2>&1 | tee shadowsocksR.log

安装完以后会有一点延迟。刚开始需要等一下看看
ssr相关操作
启动：/etc/init.d/shadowsocks start
停止：/etc/init.d/shadowsocks stop
重启：/etc/init.d/shadowsocks restart
状态：/etc/init.d/shadowsocks status
 
配置文件路径：/etc/shadowsocks.json
日志文件路径：/var/log/shadowsocks.log
代码安装目录：/usr/local/shadowsocks
```
---
##### 安装锐速破解版
```
1.一键拉取安装脚本
[root@VPS ~]# wget -N --no-check-certificate https://raw.githubusercontent.com/wn789/serverspeeder/master/serverspeeder.sh
注：锐速只支持kvm,不支持openvz 

2.运行脚本
[root@California_VPS ~]# chmod +x serverspeeder.sh
[root@California_VPS ~]# bash serverspeeder.sh

注：如果内核不支持，转3 
3.手动更换内核 
(1) CentO S7.3的内核3.10.0-514.16.1.el7.x86_64暂不支持安装锐速，故更换为3.10.0-229.1.2.el7.x86_64
rpm -ivh http://soft.91yun.org/ISO/Linux/CentOS/kernel/kernel-3.10.0-229.1.2.el7.x86_64.rpm --force       ##更换内核
rpm -qa | grep kernel
如果看到上述箭头所示，代表内核安装成功。 
以上转换内核安装不成功，下面亲测可以用：
wget -N --no-check-certificate https://freed.ga/kernel/ruisu.sh && bash ruisu.sh

(2) 重启查看内核版本
reboot
uname -r

(3) 重新安装锐速

wget -N --no-check-certificate https://github.com/91yun/serverspeeder/raw/master/serverspeeder.sh && bash serverspeeder.sh
```