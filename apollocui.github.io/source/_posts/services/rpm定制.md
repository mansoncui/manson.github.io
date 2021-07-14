---
layout: post
title: rpm定制
date: 2019-06-19 10:45:55
tags:
    - [ rpm定制 ]
categories:
    - [ services ]
---
## rpm包定制

##### 安装依赖软件包和检查ruby*是否存在
```
yum -y install ruby rubygems ruby-devel

rpm -qa ruby rubygems ruby-devel
```
---
##### 查看当前使用的rubygems仓库和yum源仓库
```
gem sources list

# 添加阿里云的Rubygems仓库，外国的源慢，移除原生的Ruby仓库
gem sources -a http://mirrors.aliyun.com/rubygems/
gem sources --remove https://rubygems.org/

# 安装fpm，gem从rubygem仓库安装软件类似yum从yum仓库安装软件。首先安装低版本的json，高版本的json需要ruby2.0以上，然后安装低版本的fpm，够用。
gem install json -v 1.8.3
gem install fpm -v 1.3.3
```
---
##### 安装构建包命令
```
yum -y install rpm-build

fpm参数详解:
参数		参数说明
-s			指定源类型
-t			指定目标类型，即想要制作为什么包
-n			指定包的名字
-v			指定包的版本号
-C			指定打包的相对路径
-d			指定依赖于哪些包
-f			第二次打包时目录下如果有同名安装包存在，则覆盖它
-p			输出的安装包的目录，不想放在当前目录下就需要指定
--post-install		软件包安装完成之后所要运行的脚本；同--after-install
--pre-install		软件包安装完成之前所要运行的脚本；同--before-install
--post-uninstall	软件包卸载完成之后所要运行的脚本；同--after-remove
--pre-uninstall		软件包卸载完成之前所要运行的脚本；同--before-remove
```
---

##### fpm制作包案列:
```
openresty制作rpm包
fpm -s dir -t rpm -n openresty -v 1.13.6.2 -d 'pcre-devel,openssl-devel,GeoIP-devel,libuuid-devel' --post-install /kuaibao/shell/openresty_install.sh -f /kuaibao/server/openresty

mysql制作rpm包
fpm -s dir -t rpm -n php-mysql -v 5.6 -d 'ncurses-devel,cmake,make,ncurses'  -f /kuaibao/server/mysql56/

php制作rpm包
fpm -s dir -t rpm -n php -v 7.1.18 -d 'libxml2,libxml2-devel,libxml2-python,freetype-devel,openldap,openldap-clients,openldap-devel,openldap-servers,libmcrypt,gd-devel,libmcrypt-devel,libcurl-devel,openssl-devel,libxslt-devel' --post-install /kuaibao/shell/php_install.sh  -f /kuaibao/server/php7/

lsyncd制作rpm包
fpm -s dir -t lsyncd -n lsyncd -v 1.1.10 -d 'lua,lua-devel,ncurses-devel,cmake,make,ncurses'   -f /kuaibao/server/lsyncd/

yum命令安装rpm包
yum -y localinstall nginx-1.6.2-1.x86_64.rpm
这个命令会自动先安装rpm包的依赖，然后再安装rpm包
```
---
##### 参考文档
```
参考地址:https://cloud.tencent.com/developer/article/1008084
        https://www.zyops.com/autodeploy-rpm/
        https://www.liuliya.com/archive/289.html
        https://www.cnblogs.com/reblue520/p/7017889.html（比较全）
简易搭建yum仓库:http://www.cnblogs.com/clsn/p/7757868.html
FPM的github：https://github.com/jordansissel/fpm
（fpm是ruby写的，因此系统环境需要ruby，且ruby版本号大于1.8.5）
```