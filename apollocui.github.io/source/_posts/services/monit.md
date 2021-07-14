---
layout: post
title: monit
date: 2019-06-19 15:39:52
tags:
    - [ monit ]
categories:
    - [ services ]
password: cbb4693
---

## monit（monitrc config ）
```
set daemon  30              # check services at 30 seconds intervals
with start delay 30
set log syslog
set idfile /var/.monit.id
set statefile /var/.monit.state

set mailserver smtp.exmail.qq.com
  port 465
  username 'monit@kuaidihelp.com'
  password 'KdzTbsGhFdloSOL6'
  using SSL with timeout 5 seconds

set eventqueue
  basedir /var/monit
  slots 100

set mail-format {
  from:      monit@kuaidihelp.com
  reply-to:  monit@kuaidihelp.com
  subject:  Monit Alert
  message:
	$DATE
	$SERVICE $EVENT
	Monit $ACTION $SERVICE at $DATE  vpc_cloud_print_2_new: $DESCRIPTION.

	Yours sincerely,
			  monit
}

set alert cuibobo@kuaidihelp.com

include /etc/monit.d/*
```
---

## monit 监控各项服务

##### mysql56
```
check process mysql56 with pidfile /kuaibao/server/mysql56/mysql.pid
  start program = "systemctl start mysql"
  stop program  = "systemctl stop mysql"
```
---
##### php7
```
check process php7 with pidfile /kuaibao/server/php7/var/run/php-fpm.pid
  start program = "/etc/init.d/php-fpmd start"
  stop program  = "/etc/init.d/php-fpmd force-quit"

set daemon 15
##php7
check process php-fpm with pidfile /kuaibao/server/php7/var/run/php-fpm.pid
  start program = "/etc/init.d/php-fpmd start"
  stop program  = "/etc/init.d/php-fpmd force-quit"
  if failed port 80 protocol http request "/ping" status = 200 content = "pong" with timeout 3 seconds
          for 3 cycles then restart

## php状态
location ~ ^/(?:status|ping)$ {
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    fastcgi_pass   unix:/dev/shm/php-cgi.sock;
    allow 127.0.0.1;
    allow 10.20.0.0/16;
    deny all;
}
```
---
##### nginx
```
check process nginx with pidfile /kuaibao/server/nginx/logs/nginx.pid
  start program = "/kuaibao/server/nginx/sbin/nginx"
  stop program  = "/kuaibao/server/nginx/sbin/nginx -s stop"

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
##### 监控文件变化(参考文档:https://mmonit.com/wiki/Monit/ConfigurationExamples)
```
check file test.log with path /tmp/tmptest.log
if changed size then alert  或者
check directory root with path /root
if changed timestamp then alert
alert ****@******.com
```
---
##### 监控url:若连续5个周期打开url都失败（120秒超时，超时也认为失败）
```
if failed url http://127.0.0.1:4000/ timeout 120 seconds for 5 cycles then restart
if failed url http://127.0.0.1:5000/ timeout 120 seconds for 5 cycles then restart
```
---
##### 监控CPU核Memory
```
check system localhost
    if loadavg (1min) > 10 then alert
    if loadavg (5min) > 6 then alert
    if memory usage > 75% then alert
    if cpu usage (user) > 70% then alert
    if cpu usage (system) > 60% then alert
    if cpu usage (wait) > 75% then alert
```