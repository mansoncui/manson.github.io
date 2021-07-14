---
layout: post
title: nfs
date: 2019-06-19 15:38:31
tags:
    - [ nfs ]
categories:
    - [ services ]
#password: cbb4693
---
## nfs安装
##### 配置nfs
```
yum  -y install  nfs-utils   rpcbind
service  rpcbind  start
service   nfs   start 
vim /etc/exports
共享目录  app服务器地址 (rw,no_root_squash,anonuid=500,anongid=500)   #anonuid被共享所有者和所属组
exporter -rv  #刷新配置，并重启
eg:/kuaibao/www  192.168.1.0/24(insecure,rw,async,no_root_squash)
windwos挂载:mount -o  \\192.168.1.89\kuaibao\www  h:
```
---
##### 应用服务器操作
```
yum -y install  nfs-common     或  yum install nfs-utils
mkdir -p /webcache
mount   10.20.0.8(web服务器地址):/webcache /webcache
```
---
##### web和app之间内网服务器要免密码登录
```
su wanson
$ssh-keygen -t rsa  

注意:公网上开启nfs，在windows10上挂载，防火前开启2049,111,1011
服务端:
vim /etc/exports
/ebsig/data  *(insecure,rw,async,no_root_squash)

vim /etc/services   #mountd需要绑定这两个端口
mountd 1011/udp
mountd 1011/tcp
mountd 1012/udp
mountd 1012/tcp
这是默认mountd开启端口，需要注释:
#mountd          20048/tcp               # NFS mount protocol
#mountd          20048/udp               # NFS mount protocol

servics nfs  start 
servics rpcbind start 
```
