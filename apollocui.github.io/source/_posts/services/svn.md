---
layout: post
title: svn
date: 2019-06-20 13:46:22
tags:
    - [ svn ]
categories:
    - [ services ]
password: cbb4693
---

## 部署svn

##### 安装apache
```
yum -y install httpd
```
##### 安装svn模块
```
yum -y install mod_dav_svn subversion  
ls /etc/httpd/modules/ | grep svn   #查看svn模块
```
##### 新增配置文件
```
cat /etc/httpd/conf.d/subversion.conf
LoadModule dav_svn_module modules/mod_dav_svn.so
LoadModule authz_svn_module modules/mod_authz_svn.so
<Location /manson/svn>
DAV svn
SVNParentPath /manson/svn   #svn的根目录SSLRequireSSL                #SSL访问权限
AuthType Basic               #Basic认证方式
AuthName "Authorization SVN"   #认证时显示的信息
AuthUserFile /manson/svn/htpasswd      #用户文件&密码
AuthzSVNAccessFile /manson/svn/auth.conf  #访问权限控制文件
Require valid-user            #要求真实用户,不能匿名
</Location>
```
##### 创建仓库
```
mkdir /manson/svn
svnadmin create /manson/svn/test
chown -R apache.apache /manson/svn
```
##### 创建用户文件htpasswd和权限控制文件auth.conf
```
htpasswd -c  /manson/svn/htpasswd  用户名  #首次创建用户名
htpasswd -b  /manson/svn/htpasswd  用户名  密码  #追加方式创建用户名
svnadmin load /new/path/to/your/repository < ./repository.clear-dump   #导入数据
svnadmin dump /mnt/backup/var/www/svn/android  >  /ebsig/android.dump  #导出数据
```
##### 安装PHP和svnadmin
```
yum -y install php
wget http://sourceforge.net/projects/ifsvnadmin/files/svnadmin-1.6.2.zip/download
unzip iF.SVNAdmin-stable-1.6.2  #解压到网站根目录/var/www/html/
mv svnadmin /var/www/html/
chown -R apache.apache svnadmin
cd /var/www/html/svnadmin
chmod -R 777 data
```
##### 启动httpd服务
```
service httpd start  
```
##### 网页登录
```
http://IP/svnadmin #(默认的账户为admin/admin)
![svn](images/svn.jpg)
```