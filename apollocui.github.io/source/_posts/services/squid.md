---
layout: post
title: squid
date: 2019-06-25 17:19:18
tags:
    - [ squid ]
categories:
    - [ services ]
---
## squid

##### squid 正向代理
```
环境:准备三台服务器，一台web服务器，squid服务器，客户端。关闭selinux和iptables

1、yum -y install squid   #squid代理服务器

vim  /etc/squid/squid.conf

visible_hostname   www.baidu.com  #(主机名没改，就不用写，改了就要写)

cache_mem   64 MB    #高速缓存

 cache_dir  ufs  /var/spool/squid  100 16 256  #缓存目录  ufs文件格式   100是缓存大小  16是一级子目录   256是二级子目录

http_port 80   vhost    # 把端口改为80，把httpd服务关闭

cache_peer 网站服务器的地址  parent  80 0 originserver   #定义网站服务器地址，当squid缓存中没有时，访问web网站服务器，并存储在squid代理服务器上

service   squid   restart

chkconfig squid on

2、yum  -y install httpd  elinks     #web网站服务器

service httpd start

chkconfig httpd on

echo "1111"   >  /var/www/html/index.html

elinks --dump http://localhost/     或   curl  http://localhost

3、客户测试:http://squid服务器的IP地址
```

##### squid 反向代理
```
环境:准备三台服务器，一台web服务器，squid服务器，客户端。关闭selinux和iptables

1、yum -y install squid   #squid代理服务器

vim  /etc/squid/squid.conf

visible_hostname   www.baidu.com  #(主机名没改，就不用写，改了就要写)

cache_mem   64 MB    #高速缓存

 cache_dir  ufs  /var/spool/squid  100 16 256  #缓存目录  ufs文件格式   100是缓存大小  16是一级子目录   256是二级子目录

http_port 80   vhost    # 把端口改为80，把httpd服务关闭

cache_peer 网站服务器的地址  parent  80 0 originserver   #定义网站服务器地址，当squid缓存中没有时，访问web网站服务器，并存储在squid代理服务器上

service   squid   restart

chkconfig squid on

2、yum  -y install httpd  elinks     #web网站服务器

service httpd start

chkconfig httpd on

echo "1111"   >  /var/www/html/index.html

elinks --dump http://localhost/     或   curl  http://localhost

3、客户测试:http://squid服务器的IP地址
```