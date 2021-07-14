---
layout: post
title: nginx
date: 2019-06-19 11:01:01
tags:
    - [ nginx ]
categories:
    - [ services ]
password: cbb4693
---
## nginx location 优先级
1 Location是什么？
```
Location是Nginx中的块级指令(block directive),
通过配置Location指令块，可以决定客户端发过来的请求URI如何处理（是映射到本地文件还是转发出去）及被哪个location处理
```
2 location 语法
```
修饰符(modifier)         
location [ = | ~ | ~* | ^~ ]     uri     { ... } 
location根据不同的修饰符可以分为两大类
1. 前缀location(prefix location): 
   无修饰符的普通location
   带=的精准匹配location
   带^~的非正则表达式location
2.正则表达式location(regular expressions location):
   ~    区分大小写的正则location
   ~*   不区分大小写的正则location
```
3 Location基本匹配规则
```
匹配规则是指当请求到达nginx时，nginx如何决定该使用哪条location。
首先，nginx首先会检查所有的前缀location，从中选出最长前缀匹配（也就是修饰符后面的路径最长的）的location并记下。
然后，如果存在正则location时，按照其出现的顺序，依次匹配URI，找到匹配的正则location就不再继续往下，并选择该location作为最终的结果。(划重点：正则location出现的顺序很重要)
```
4 Location特殊匹配规则1
```
如果最长前缀匹配location的修饰符是^~时，就不会检查正则location了，直接选择该location为最终location
```
5 Location特殊匹配规则2
```
如果存在精准匹配location，且请求的uri跟其完全匹配，选择该精准匹配location作为最终的location
```
6 测试下自己的理解是否准确：
```
location / {
    [ configuration B ]
}

location /documents/ {
    [ configuration C ]
}

location ^~ /images/ {
    [ configuration D ]
}

location ~* \.(gif|jpg|jpeg)$ {
    [ configuration E ]
}

请求URI                        执行的规则
/                             A
/index.html                   B
/documents/document.html      C
/images/1.gif                 D
/documents/1.jpg  
```
---
## 参数优化
```
web端限流:http块
## limit config
limit_req_zone $binary_remote_addr zone=ip_r_limit_per_min:40m rate=40r/m;
limit_req_zone $binary_remote_addr zone=ip_r_limit:40m rate=20r/s;
limit_conn_zone $binary_remote_addr zone=ip_limit:40m;
limit_conn_zone $nginx_version zone=conn_limit:40m;
limit_conn_zone $uri zone=request_uri:60m;
## limit configuration

limit_req_status 429;
limit_conn_status 429;
error_page 429 @qps_limit;
#limit_req zone=ip_r_limit burst=30;


内部服务器限流:http块
## error page
error_page 429 = @qps_limit;
error_page 500 502 503 504 = @5xx;

## HTTPS header
#include https_headers.conf;

## limit config
## ip req
#limit_req_zone $binary_remote_addr zone=ip_r_limit:60m rate=10r/s;
## qps
#limit_req_zone $nginx_version zone=qps_limit:10m rate=800r/s;
## per ip connection
#limit_conn_zone $binary_remote_addr zone=ip_addr:30m;
## total connection
limit_conn_zone $nginx_version zone=conn_limit:10m;
## uri
limit_conn_zone $uri zone=request_uri:60m;

## http code
limit_req_status 429;
limit_conn_status 429;

#limit server total connection
limit_conn  conn_limit 500;
#setting for specific location
limit_conn request_uri 200;

##server内部配置
## for default server block
limit_conn  conn_limit 550;
include error_pages.conf;

##500错误返回
## default block error
error_page 500 502 503 504 @5xx;

## 5xx handler
location @5xx {
    return 200 '{"code":500,"msg":"5xx","data":{}}';
}

## 429 return 200
error_page 429 =200 @qps_limit;
location @qps_limit {
    return 200 '{"code":429,"msg":"429","data":{}}';
}

#openresty测试超时monit
location = /ping {
    content_by_lua_block {
       ngx.sleep(3)
       ngx.print("pong")
    }
}
##tengine测试超时
location = /ping {
    content_by_lua '
       ngx.sleep(4)
       ngx.print("pong")
    ';
}

```
---
## nginx log
```
IP相关统计
统计IP访问量（独立ip访问数量）

awk '{print $1}' access.log | sort -n | uniq | wc -l
查看某一时间段的IP访问量(4-5点)

grep "07/Apr/2017:0[4-5]" access.log | awk '{print $1}' | sort | uniq -c| sort -nr | wc -l  
查看访问最频繁的前100个IP

awk '{print $1}' access.log | sort -n |uniq -c | sort -rn | head -n 100
查看访问100次以上的IP

awk '{print $1}' access.log | sort -n |uniq -c |awk '{if($1 >100) print $0}'|sort -rn
查询某个IP的详细访问情况,按访问频率排序

grep '127.0.01' access.log |awk '{print $7}'|sort |uniq -c |sort -rn |head -n 100
页面访问统计
查看访问最频的页面(TOP100)

awk '{print $7}' access.log | sort |uniq -c | sort -rn | head -n 100
查看访问最频的页面([排除php页面】(TOP100)

grep -v ".php"  access.log | awk '{print $7}' | sort |uniq -c | sort -rn | head -n 100 
查看页面访问次数超过100次的页面

cat access.log | cut -d ' ' -f 7 | sort |uniq -c | awk '{if ($1 > 100) print $0}' | less
查看最近1000条记录，访问量最高的页面

tail -1000 access.log |awk '{print $7}'|sort|uniq -c|sort -nr|less
每秒请求量统计
统计每秒的请求数,top100的时间点(精确到秒)

awk '{print $4}' access.log |cut -c 14-21|sort|uniq -c|sort -nr|head -n 100
每分钟请求量统计
统计每分钟的请求数,top100的时间点(精确到分钟)

awk '{print $4}' access.log |cut -c 14-18|sort|uniq -c|sort -nr|head -n 100
每小时请求量统计
统计每小时的请求数,top100的时间点(精确到小时)

awk '{print $4}' access.log |cut -c 14-15|sort|uniq -c|sort -nr|head -n 100
性能分析
在nginx log中最后一个字段加入$request_time

列出传输时间超过 3 秒的页面，显示前20条

cat access.log|awk '($NF > 3){print $7}'|sort -n|uniq -c|sort -nr|head -20
列出php页面请求时间超过3秒的页面，并统计其出现的次数，显示前100条

cat access.log|awk '($NF > 1 &&  $7~/\.php/){print $7}'|sort -n|uniq -c|sort -nr|head -100
蜘蛛抓取统计
统计蜘蛛抓取次数

grep 'Baiduspider' access.log |wc -l
统计蜘蛛抓取404的次数

grep 'Baiduspider' access.log |grep '404' | wc -l
TCP连接统计
查看当前TCP连接数

netstat -tan | grep "ESTABLISHED" | grep ":80" | wc -l
用tcpdump嗅探80端口的访问看看谁最高

tcpdump -i eth0 -tnn dst port 80 -c 1000 | awk -F"." '{print $1"."$2"."$3"."$4}' | sort | uniq -c | sort -nr
```
