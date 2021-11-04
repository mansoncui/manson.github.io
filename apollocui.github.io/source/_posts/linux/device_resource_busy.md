---
title: home 目录不能重命名
date: 2021-11-04 15:44:00
tags:
    - [ mv ]
    - [ Device or resource busy ]
categories:
    - [ Device or resource busy ]
---

## 给home 做软连接出现问题
### 问题现象
```
需求: home 目录需要做软链接，把 /home 进行重命名失败
现象如下图:
```
<img src="/images/linux/home_device_resource_busy.jpg" width=100% height=20% align=left/>

### 查找原因
```
lsof +d /home   #查看是哪个进程占用该目录

kill -9 pid  #把该进程停掉

grep -h home /proc/*/task/*/mountinfo | sort -u
```
<img src="/images/linux/home_process.jpg" width=100% height=20% align=left/>

## 解决方法
```
service NetworkManager stop

```
