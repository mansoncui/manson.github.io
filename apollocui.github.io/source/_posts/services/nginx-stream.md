---
layout: post
title: nginx 部署 stream
date: 2021-07-22 18:00:00
tags:
    - [ nginx ]
    - [ stream ]
categories:
    - [ services ]
#password: cbb4693
---
<!-- more -->
## 系统环境
```
操作系统: centos 7.9
nginx version: nginx/1.20.1
```
## nginx install 
```
yum -y install nginx
systemctl start nginx && systemctl enable nginx 
```
## nginx 安装stream模块
### nginx stream 描述
````
Nginx 的 TCP/UDP 负载均衡是应用 Stream 代理模块（ngx_stream_proxy_module）和 Stream 上游模块（ngx_stream_upstream_module）实现的。Nginx 的 TCP 负载均衡与 LVS 都是四层负载均衡的应用，所不同的是，LVS 是被置于 Linux 内核中的，而 Nginx 是运行于用户层的，基于 Nginx 的 TCP 负载可以实现更灵活的用户访问管理和控制。
````
### 安装nginx stream
````
yum -y install nginx-mod-stream

stream {

    log_format proxy '$remote_addr [$time_local] '
                 '$protocol $status $bytes_sent $bytes_received '
                 '$session_time "$upstream_addr" '
                 '"$upstream_bytes_sent" "$upstream_bytes_received" "$upstream_connect_time"';

    access_log /var/log/nginx/tcp-access.log proxy ;

upstream tcp_9093 {
	server 192.168.3.105:8000;
	server 192.168.3.114:8000;
}

    server {
        listen 9093;
        proxy_pass tcp_9093;
        allow 192.168.7.0/24;
        deny   all;
    }

    server {
        listen 9095;
        proxy_pass 192.168.7.219:9098;
        allow 192.168.7.0/24;
        deny   all;
    }
}
````
### 重启nginx
```
nginx -t  #检查配置文件是否有语法错误
nginx -s reload 或 systemctl reload nginx 

参考地址: 
https://pkgs.org/download/nginx-mod-stream
https://centos.pkgs.org/7/epel-x86_64/nginx-mod-stream-1.20.1-2.el7.x86_64.rpm.html
```


