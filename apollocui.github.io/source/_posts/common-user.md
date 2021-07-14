---
layout: post
title: common_user
date: 2019-06-28 13:44:04
tags: common
password: cbb4693
---

## 常用命令
##### 处理僵尸进程
```
ps -A -o stat,ppid,pid,cmd | grep -e '^[Zz]'| awk '{print $2}' | xargs kill -9
```

##### 批量杀死进程
```
ps -ef|grep 进程名|grep -v grep|awk '{print "kill -9 "$2}'

ps axu|grep "inform_order_sender.php"|awk '{print $2}'|xargs kill

lsof -n | grep deleted  #查看删除占用

du -h --max-depth=1 #查看文件占用

ps aux|head -1; ps aux | sort -k4nr | head -10  #查看内存占用

echo "ABCDEFG" | awk -F "" '{print NF}' #shell计算字符串长度

awk '{print length($0)}' /etc/passwd  #计算文件每行字符串长度

str="ABCDEF";echo ${#str}  #获取字符串长度
```
##### 
```
```