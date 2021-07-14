---
layout: post
title: php
date: 2019-06-19 10:29:22
tags:
    - [ php-fpm ]
categories:
    - [ services ]
---
## php7 安装
```
```
---
## php 进程相关知识

php官网学习配置地址:https://php.net/manual/zh/install.fpm.configuration.php

1.通过命令查看服务器上一共开了多少的 php-cgi 进程
```
ps -fe |grep "php-fpm"|grep "pool"|wc -l
```
---
2.查看已经有多少个php-cgi进程用来处理tcp请求
```
netstat -anp|grep "php-fpm"|grep "tcp"|grep "pool"|wc -l
```
---
3.查看每个PHP-FPM进程的内存占用
```
ps -ylC php-fpm –sort:rss
```
---
4.查看消耗内存最多的前 40 个进程
```
ps auxw|head -1;ps auxw|sort -rn -k4|head -40
```
---
5.查看PHP-FPM的平均内存占用：
```
ps --no-headers -o "rss,cmd" -C php-fpm | awk '{ sum+=$1 } { printf ("%d%s\n", sum/NR/1024,"M") }'
```
---
6.判断php-fpm进程是否打满
```
curl http://127.0.0.1/status
pool:                 www
process manager:      static
start time:           11/Jun/2019:16:39:47 +0800
start since:          669307
accepted conn:        11725653
listen queue:         0
max listen queue:     0
listen queue len:     0
idle processes:       298
active processes:     2
total processes:      300
max active processes: 152
max children reached: 0
slow requests:        9837

解释:当listen queue不为零时，表示当前请求等待队列,当前进程数被使用完了
Active connections 表示Nginx正在处理的活动连接数4016个

server 表示Nginx启动到现在共处理了 947407306 个连接
accepts 表示Nginx启动到现在共成功创建 947407306 次握手
handled requests 表示总共处理了 925780898 次请求
请求丢失数 = 握手数 - 连接数 ，可以看出目前为止没有丢失请求

Reading:Nginx 读取到客户端的 Header 信息数
Writing:Nginx 返回给客户端 Header 信息数
Waiting:Nginx 已经处理完正在等候下一次请求指令的驻留链接
```
####php status
```
php-fpm status状态值详解

pool：fpm池子名称，大多数为php7

process manager：进程管理方式，值：static，dynamic or ondemand

start time：启动日期，如果reload了php-fpm，时间会更新

start since：运行时长

accepted conn：当前池子接受的请求数

listen queue：请求等待队列，如果这个值不为0，那么要增加FPM的进程数量

max listen queue：请求等待队列最高的数量

listen queue len：socket等待队列长度

idle processes：空闲进程数量

active processes：活跃进程数量

total processes：总进程数量

max active processes：最大的活跃进程数量（FPM启动开始算）

max children reached：进程最大数量限制的次数，如果这个数量不为0，那说明你的最大进程数量太小了，需要设置大点
```
