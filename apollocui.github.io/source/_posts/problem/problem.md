---
layout: post
title: 守护进程的文件数量限制
date: 2021-08-03 16:00:00
tags:
    - [ systemd ]
categories:
    - [ problem ]
---
## 故障现象
故障现象

c++开发终端服务 进行并发压测，抓包报如下错误:
<img src="/images/problem/system-fd.png" width=100% height=50% align=left/>
<img src="/images/problem/system-压测.png" width=100% height=50% align=left/>

## 处理过程
修改 c++ 终端服务 systemd 启动脚本，添加如下两行:
```
DefaultLimitNOFILE=10240000 # 修改文件句柄的限制
DefaultLimitNPROC=10240000 # 修改进程树的限制
```
## 重启服务
```
systemctl daemon-reload 
systemctl restart edxxxx-xxp
```
systemctl show edxxxx-xxp | grep -i limit
<img src="/images/problem/system-open-file.png" width=100% height=50% align=left/>

## 重新压测正常
<img src="/images/problem/system-并发压测.png" width=100% height=50% align=left/>

