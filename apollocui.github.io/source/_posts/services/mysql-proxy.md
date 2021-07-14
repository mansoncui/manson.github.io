---
layout: post
title: mysql_proxy
date: 2019-06-25 17:10:54
tags:
    - [ mysql读写分离 ]
categories:
    - [ services ]
---
## mysql 读写分离
```
环境: 四台机器，两台数据库服务器，一台代理，一台客户端测试

注意:先把代理建完并测试以后，再搭建主从数据库服务器

一、搭建代理服务器

1、下载mysql-proxy包，并解压到/usr/local/,重命名为mysqlproxy(重命名根据自己情况，不一定)

-P  #指定代理服务器的ip地址和端口号

-r  #指定从数据库服务器的IP地址和端口(读操作)

-b  #指定主数据库服务器的IP地址和端口(写操作)

-s  #指定lun脚本文件的路径

--keepalive  #若进程奔溃，自动重启此进程（可写可不写)

lun脚本存放的目录:/usr/local/share/doc/mysql-proxt/rw--splitting.lua   #区分读写操作的

/usr/local/bin/mysqlproxy/bin/mysql-proxy     #启动命令

/usr/local/bin/mysqlproxy/bin/mysql-proxy   -P 指定代理服务器的IP地址和端口  -r 从数据库服务器IP地址和端口  -b  主数据库的IP地址和端口   -s   /usr/local/share/doc/mysql-proxt/rw--splitting.lua    &

关闭命令端口: pkill  进程名或 kill -9 pid

二、搭建主从数据库

主从授权用同一个用户，建议为：proxyuser ，地址为:%(代表所有地址)

主从数据库的搭建和之前一样

三、客户端测试

mysql -h代理IP   -u授权用户  -p授权密码

注释:前5个终端读写操作都是在主数据库服务器上，超过五个时（不包括第五个）读写操作正常分开，除非重启该服务，否者读写永久生效
```