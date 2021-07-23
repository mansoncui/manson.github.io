---
layout: post
title: zabbix
date: 2019-06-25 17:36:16
tags:
    - [ zabbix ]
categories:
    - [ monitor ]
---
## zabbix 部署
```
环境：两台机器,搭建LAMP 平台

yum -y install  gcc gcc-c++  php-*   net-snmp  net-snmp-utils

rpm -ivh mysql-.........

rpm -ivh mysql-server............

修改数据库初始密码:mysql>set password=password("123");

yum -y install libcurl-devel  net-snmp-devel  mysql-devel

/etc/init.d/httpd start;chkconfig httpd on

/etc/init.d/httpd start;chkconfig httpd on
```
## 部署zabbix
```
1、配置和安装zabbix

#useradd zabbix

#tar -zxvf  zabbix-2.2.1.tar.gz

#cd zabbix-2.2.1

#vim config.sh

./configure --prefix=/usr/local/zabbix/   --enable-proxy  --enable-server  --enable-agent  --with-libcurl  --with-net-snmp

#sh  config.sh

#make && make install

安bin  sbin   #命令
share    #帮助文档
lib    #库文件
ext  #配置文件
我们只需要修改配置文件即可:装成功:ls /usr/local/zabbix/有以下文件：bin  etc  lib  sbin  share

#cp  misc/init.d/fedora/core/zabbix_*  /etc/init.d/    #*表示的是server和agentd

#chmod  +x   /etc/init.d/zabbix_*

#vim  /etc/init.d/zabbix_server

BASEDIR=/usr/local/zabbix  #修改安装目录

#vim  /etc/init.d/zabbix_agentd

BASEDIR=/usr/local/zabbix  #修改安装目录

service  zabbix_server  start

service  zabbix_agentd  start

chkconfig --add zabbix_server

chkconfig --add zabbix_agentd

2、拷贝zabbix网页目录到网站根目录下

#cp  -rp  frontends/php/     /var/www/html/zabbix

#chown apache:apache  -R   /var/www/html/zabbix/

3、给zabbix用户授权并创建库,把zabbix数据导入到刚刚创建的库中

#mysql -uroot -p密码

mysql>show create database mysql;  #查看字符集的关键字

mysql>create database zabbix  DEFAULT CHARACTER SET  utf8;  #创建zabbix库并指定字符集

mysql>grant all zabbix.*  to zabbix@localhost  identified by "密码";

cd  / root/zabbix-2.2.1/database/mysql/  #数据库目录下

数据导入数据库是按照一下顺序的

# mysql -uzabbix -pzabbix zabbix < schema.sql

# mysql -uzabbix -pzabbix zabbix < images.sql

# mysql -uzabbix -pzabbix zabbix < data.sql

4、vim /etc/php.ini

post_max_size = 8M   #把8M改为16M

max_input_time = 60  #把60改为300

max_execution_time = 30  #把30改为300

date.timezone = Asia/Shanghai

service  httpd  restart

#cd zabbix

#rpm -ivh  --nodeps php-mbstring-5.3.3-22.el6.x86_64.rpm  

#rpm -ivh  --nodeps  php-bcmath-5.3.3-22.el6.x86_64.rpm   #这两个包到网上下载

http://localhost/zabbix  

安装过程中:写监控端IP，数据库 名、用户名和密码都是zabbix，在数据库一起端口不要修改

4、进入到登录界面:用户名:admin  密码:zabbix

注释:

监控页面显示连接不上数据库,查看日志:tailf /tmp/zabbix_server.log

错误提示:23055:20160406:212209.070 [Z3001] connection to database 'zabbix' failed: [1045] Access denied for user 'root'@'localhost' (using password: YES)  23055:20160406:212209.071 Database is down. Reconnecting in 10 seconds

连接不上数据库:修改zabbix_server主配置文件 vim  /usr/local/zabbix/etc/zabbix_server.conf

103 DBUser=zabbix  #修改用户

111 DBPassword=zabbix  #去掉注释，加上密码
```

## 在页面中配置监控:
```
1、管理(菜单最后一个)-》用户(users)-》创建用户主群（左面）选择用户（user）-》双击admin-》zh_CN  #页面改为中文

2、配置本机监控本机:要运行/etc/init.d/zabbix_agentd start（谁被监控服务器监控谁就要起代理服务程序）

# vim /usr/local/zabbix/etc/zabbix_agentd.conf

LogFile=/tmp/zabbix_agentd.log

Logfile=/var/log/zabbix/zabbix_agentd.lg

修改日志文件目录:mkdir /var/log/zabbix/ chown zabbix:zabbix  /var/log/zabbix/

81 Server=127.0.0.1,192.168.1.254  #监控服务器的IP地址

122 ServerActive=192.168.1.254:10051  #监控服务器的IP地址和端口号

134 Hostname=Zabbix serve

/etc/init.d/zabbix_agentd restart

默认就是自己监控自己:组态-》主机->双击zabbix_server->主机—>最下方修改受监测中-》save

选择监测中—》最新组态-》

2、监控远端某台服务器:（先配置被监控端） tar -zxvf  zabbix-2.2.1.tar.gz useradd zabbix cd zabbix-2.2.1

./configure --prefix=/usr/local/zabbix --enable-agent make  &&  make install

cp   misc/init.d/fedora/core/zabbix_agentd /etc/init.d/

chmod +x  /etc/init.d/zabbix_agentd

chkconfig --add zabbix_agentd

chkconfig zabbix_agentd on

vim /etc/init.d/zabbix_agentd

BASEDIR=/usr/local/zabbix  #修改安装目录

service  zabbix_agentd start

vim /usr/local/zabbix/etc/zabbix_agentd.conf  #修改配置文件

ServerActive=192.168.1.254:10051 #监控端服务器IP地址和端口

Hostname=Zabbix client 100 Server=127.0.0.1,192.168.1.254

service  zabbix_agentd  restart

在监控端配置被自己监控服务器:192.168.1.100 组态-》主机-》创建主机（右面）-》主机名（server100）-》linuxserver-》被监控的IP地址，其他的默认-》状态（受监控中）-》save

双击server100-》模板-》添加

3、自定义监控项:监控192.168.1.101上面的用户数

（1、）在客户端(被监控端)，定义监控命令 cd /usr/local/zabbix/etc/

vim zabbix_agentd.conf

255 UnsafeUserParameters=1  #开启允许自定义命令

243 Include=/usr/local/zabbix/etc/zabbix_agentd.conf.d/

定义命令格式:UserParameter=<key>,<shell command>

cd  /usr/local/zabbix/etc/zabbix_agentd.conf.d/

vim mon.user.numbers

UserParameter=mon.user.num,wc -l  /etc/passwd  |  awk '{print $1}'

service zabbix_agentd restart

cd /usr/local/zabbix/

./bin/zabbix_get  -s  127.0.0.1  -k mon.user.num   #测试上方是否正常使用

（2）使用被监控端定义命令，对客户机做监控 在监控端调用该命令:在192.168.1.10操作 cd /usr/local/zabbix/bin/

./zabbix_get  -s  192.168.1.101  -k mon.user.num

创建监控模板： a、组态->模板-》创建模板（右上角）-》模板名（monitorusernum）-》可见名称（monitorusernum）-》linux server -》 一下默认-》保存

b、应用集-》创建应用集-》名称（monusernum101） c、在应用集下面创建项目-》 创建监控项-》名称（monuserbig42）->类型（zabbix代理）-》键值mon.user.num(自己创建)-》十进制-》后面默认（使用自己创建应用集）-》save

组态-》用户-》模板（刚刚自己自定义模板添加进去）

d、在监控客户端4.101时使用刚才自己定义模板

四、监控报警: 当监控到客户端192.168.4.205的用户数大于42个  报警  并发送报警邮件

定义触发器: 点击触发器-》创建触发器-》名称（mon.user.big42）->表达式（添加） -》项目（自己刚刚创建的）       最末T值是>N         N 值（插入）         选择严重
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

组态-》动作->创建动作->名称（to-mail自己定义、邮件主题、邮件正文）  

  默认接收人:server100userbig42    默认信息不变

 条件   默认就可以不用设置

 操作    -新的     操作类型 ：送出信息         送到用户：admin     仅送到：  email     条件：  不用设置    添加   存档
 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

定义邮件的用户(发件人) 管理-用户--用户（右上方 选择 ）--admin  ---  示警媒介----添加          

　　　　　　　　　　　　　　   类型:email       收件人：root@localhost             

　　　　　　　　　　　　  添加        存档

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++      
```
## 配置邮件 (或者直接用Python写个脚本来发送)
```
指定邮件服务器 管理---示警媒介类型---Email--名称:Email        类型;电子邮件        邮件服务器名:localhost        SMTP  HELO:localhost        SMTP电邮:zabbix@localhost(发件人地址)

在监控服务器 本机运行邮件服务（postfix \dovecot）

yum -y install postfix  yum -y install dovecot /etc/init.d/postfix restart /etc/init.d/dovecot restart

测试监控报警配置是否成功
```
