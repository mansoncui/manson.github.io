---
layout: post
title: centos6-7
date: 2019-07-10 09:53:02
tags:
   - [ centos6-7 服务进程优化 ]
categories:
    - [ linux操作性能优化 ]
---

## centos6 和 7 服务进程关闭
```
服务名称	        功能	                            		默认 	建议 	备注说明
NetworkManager	    用于自动连接网络，常用在Laptop上			开启	关闭	对服务器无用
abrt-ccpp	                                            		开启	自定	对服务器无用
abrt-oops	                                            		开启	自定	对服务器无用
abrtd	                                                		开启	自定	对服务器无用
acpid	            电源的开关等检测管理，常用在Laptop上		开启	自定	对服务器无用
atd	                在指定时间执行命令							开启	关闭	如果用crond，则可关闭它
auditd				审核守护进程								开启	开启	如果用selinux，需要开启它
autofs				文件系统自动加载和卸载						开启	自定	只在需要时开启它，可以关闭
avahi-daemon		本地网络服务查找							开启	关闭	对服务器无用
bluetooth			蓝牙无线通讯								开启	关闭	对服务器无用
certmonger														关闭	关闭	
cpuspeed			调节cpu速度用来省电，常用在Laptop上			开启	关闭	对服务器无用
crond				计划任务管理								开启	开启	常用，开启
cups				通用unix打印服务							开启	关闭	对服务器无用
dnsmasq	dns cache												关闭	关闭	DNS缓存服务，无用
firstboot			系统安装后初始设定							关闭	关闭	
haldaemon			硬件信息收集服务							开启	开启	
ip6tables			ipv6防火墙									开启	关闭	用到ipv6网络的就用，一般关闭
iptables			ipv4防火墙									开启	开启	ipv4防火墙服务
irqbalance			cpu负载均衡									开启	自定	多核cup需要
kdump				硬件变动检测								关闭	关闭	服务器无用
lvm2-monitor		lvm监视										开启	自定	如果使用LVM逻辑卷管理就开启
matahari-broker													关闭	关闭	此服务不清楚，我关闭
matahari-host													关闭	关闭	此服务不清楚，我关闭
matahari-network												关闭	关闭	此服务不清楚，我关闭
matahari-service												关闭	关闭	此服务不清楚，我关闭
matahari-sysconfig												关闭	关闭	此服务不清楚，我关闭
mdmonitor			软raid监视									开启	自定	
messagebus			负责在各个系统进程之间传递消息				开启	开启	如停用，haldaemon启动会失败
netconsole														关闭	关闭	
netfs				系统启动时自动挂载网络文件系统				开启	关闭	如果使用nfs服务，就开启
network				系统启动时激活所有网络接口					开启	开启	网络基础服务，必需！
nfs					网络文件系统								关闭	关闭	nfs文件服务，用到就开启
nfslock				nfs相关										开启	关闭	nfs相关服务，用到就开启
ntpd				自动对时工具								关闭	自定	网络对时服务，用到就开启
ntpdate				自动对时工具								关闭	关闭	
oddjobd				与D-BUS相关									关闭	关闭	
portreserve			RPC 服务相关								开启	自定	可以关闭
postfix				替代sendmail的邮件服务器					开启	自定	如果无邮件服务，可关闭
psacct				负荷检测									关闭	关闭	可以关闭
qpidd				消息通信									开启	开启	
quota_nld														关闭	关闭	可以关闭
rdisc			    自动检测路由器								关闭	关闭	
restorecond			selinux相关									关闭	关闭	如果开启了selinux，就需开启
rpcbind															开启	开启	关键的基础服务，nfs服务和桌面环境都依赖此服务！相当于CentOS 5.x里面的portmap服务。
rpcgssd				NFS相关										开启	关闭	NFS相关服务，可选
rpcidmapd			RPC name to UID/GID mapper					开启	关闭	NFS相关服务，可选
rpcsvcgssd			NFS相关										关闭	关闭	NFS相关服务，可选
rsyslog				提供系统的登录档案记录						开启	开启	系统日志关键服务，必需！
saslauthd			sasl认证服务相关							关闭	关闭	
smartd				硬盘自动检测守护进程						关闭	关闭	
spice-vdagentd													开启	开启	
sshd				ssh服务端，可提供安全的shell登录			开启	开启	SSH远程登录服务，必需！
sssd															关闭	关闭	
sysstat															开启	开启	一组系统监控工具的服务，常用
udev-post			设备管理系统								开启	开启	
wdaemon															关闭	关闭	
wpa_supplicant		无线认证相关								关闭	关闭	
ypbind				network information service客户端			关闭	关闭	
```