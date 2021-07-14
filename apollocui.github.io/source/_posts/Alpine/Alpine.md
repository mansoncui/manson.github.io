---
title: Alpine命令详解
date: 2019-06-11 16:45:26
tags:
    - [Alpine]
categories:
    - [Alpine]
comments: true
---
1.apk update
$ apk update #更新最新镜像源列表

2.apk search
$ apk search #查找所以可用软件包
$ apk search -v #查找所以可用软件包及其描述内容
$ apk search -v 'acf*' #通过软件包名称查找软件包
$ apk search -v -d 'docker' #通过描述文件查找特定的软件包

3.apk add
$ apk add openssh #安装一个软件
$ apk add openssh openntp vim   #安装多个软件
$ apk add --no-cache mysql-client  #不使用本地镜像源缓存，相当于先执行update，再执行add

4.apk info
$ apk info #列出所有已安装的软件包
$ apk info -a zlib #显示完整的软件包信息
$ apk info --who-owns /sbin/lbu #显示指定文件属于的包

5.apk upgrade
$ apk upgrade #升级所有软件
$ apk upgrade openssh #升级指定软件
$ apk upgrade openssh openntp vim   #升级多个软件
$ apk add --upgrade busybox #指定升级部分软件包

6.apk del
$ apk del openssh  #删除一个软件
