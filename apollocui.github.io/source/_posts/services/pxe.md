---
layout: post
title: pxe
date: 2019-06-25 17:23:21
tags:
    - [ pxe ]
categories:
    - [ services ]
---

## pxe批量部署服务器
```
环境:准备两台机器(在虚拟机中操作)，先关闭iptables、selinux和NetworkManager

1、配置dhcp

yum -y install dhcp

cp /usr/share/doc/dhcp-4.1.1/dhcpd.conf.sample  /etc/dhcp/dhcpd.conf

vim  /etc/dhcp/dhcpd.conf

default-lease-time 600;
max-lease-time 7200;
subnet 192.168.1.0 netmask 255.255.255.0 {
  range 192.168.1.20 192.168.1.80;
  option routers 192.168.1.254;
 next-server 192.168.1.101;
filename "pxelinux.0";
}

/etc/init.d/dhcpd  start

chkconfig dhcpd on

2、yum  -y install tftp-server   tftp   syslinux

rpm -ql syslinux   #查看syslinux安装目录

cd  /usr/share/syslinux/

cp  -p pxelinux.0  /var/lib/tftpboot/

vim /etc/xinetd.d/conf

disable                 = no  #启用

cd /media/isolinux/

cp  -p  vmlinuz initrd.img splash.jpg   /var/lib/tftpboot/

 mkdir /var/lib/mysql/pxelinux.cfg

cp -p isolinux.cfg /var/lib/mysql/pxelinux.cfg/default

vim   /var/lib/mysql/pxelinux.cfg/default  #修改配置文件

default linux

append initrd=initrd.img  ks=ftp://192.168.1.101/pub/ks.cfg

services  xinetd start ;chkconfig   xinetd on 

3、yum -y install  vsftpd

service vsftpd on

mkdir  /var/ftp/pub/iso/

cp -rp /media/*   /var/ftp/pub/iso/

 chmod 644 /var/lib/tftpboot/pxelinux.cfg/default

4、yum -y install system-config-kickstart

system-config-kickstart

(1)基本配置中设置语言:中文简体  时区：shanghai    密码,高级配置:给安装后重新引导系统和在文本模式中执行安装(默认为图形安装)打上勾，后者可以不打勾；

(2)安装方法:选择ftp或是http。列如ftp: 第一个是写服务器地址，第二个是共享光盘镜像文件目录

(3)主引导记录、分区和磁盘标签都选第一个；新建 “/” ,"boot"和"swap"

(4)网卡配置:添加eth0网卡

(5)防火墙z选择禁用

(6)软件包安装:默认是最小化安装，可以选择包:

基本系统中:基本和网络文件系统

桌面:X窗口系统、字体、桌面、输入法和通用桌面

语言支持:中文

把文件保存到:/var/ftp/pub/

chmod   644  /var/ftp/pub/ks.cfg  
```
## 新建一台虚拟机(网络模式选为仅主机模式) 测试看是否成功
```
ks文件：

#platform=x86, AMD64, 或 Intel EM64T
#version=DEVEL
# Firewall configuration
firewall --disabled
# Install OS instead of upgrade
install
# Use network installation
url --url="ftp://192.168.1.101/pub/iso/"
# Root password
rootpw --iscrypted $1$KNdqC7g/$mVPG2RIICeyyECaglEkBy.
# System authorization information
auth  --useshadow  --passalgo=sha512
# Use text mode install
text
firstboot --disable
# System keyboard
keyboard us
# System language
lang zh_CN
# SELinux configuration
selinux --disabled
# Installation logging level
logging --level=info
# Reboot after installation
reboot
# System timezone
timezone  Asia/Shanghai
# Network information
network  --bootproto=dhcp --device=eth0 --onboot=on
# System bootloader configuration
bootloader --location=mbr
# Clear the Master Boot Record
zerombr
# Partition clearing information
clearpart --all --initlabel
# Disk partitioning information

part / --fstype="ext4" --size=50000 part /boot --fstype="ext4" --size=200 part swap --fstype="swap" --size=8129

%packages @base @basic-desktop @chinese-support @fonts @general-desktop @input-methods @internet-browser @network-file-system-client @x11

%end  
```
## 注释:
```
boot.msg （显示“[enter]”启动提示信息）

initrd.img（这是一个初始化文件，一个最小的系统镜像）

pxelinux.0     （这文件是为legcay启动，它是legcay的启动镜像）

pxelinux.cfg（该文件夹下放的是启动菜单，手动创建）

splash.jpg（背景图片，可以不要）

vesamenu.c32（legacy BIOS引导菜单工具，可从光盘或/usr/share/syslinux/中找到）

vmlinuz（内核文件）
```
