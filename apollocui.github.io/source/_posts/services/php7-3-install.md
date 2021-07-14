---
layout: post
title: php7.3-install
date: 2019-07-16 14:41:40
tags:
    - [ php7 ]
categories:
    - [ services ]
---
## php7.3
##### 安装libzip 不然会报错
```
wget https://nih.at/libzip/libzip-1.2.0.tar.gz
tar -zxvf libzip-1.2.0.tar.gz
cd libzip-1.2.0
./configure
make && make install

错误信息:configure: error: off_t undefined; check your library configuration 
#解决方法
# 添加搜索路径到配置文件
echo '/usr/local/lib64
/usr/local/lib
/usr/lib
/usr/lib64'>>/etc/ld.so.conf
# 更新配置
ldconfig -v

#解决方法：手动复制过去
cp /usr/local/lib/libzip/include/zipconf.h /usr/local/include/zipconf.h

```
##### php7.3 安装配置
```
php下载地址:https://www.php.net/distributions/php-7.3.7.tar.gz

wget https://www.php.net/distributions/php-7.3.7.tar.gz

tar -zxvf  php-7.3.7.tar.gz

cd  php-7.3.7

./configure  --prefix=/usr/local/php7 --with-config-file-path=/usr/local/php7/etc --with-gd --with-jpeg-dir --with-png-dir --with-zlib --with-freetype-dir --with-libxml-dir --enable-shared  --with-iconv --without-pdo-sqlite --with-gettext=/usr --enable-soap --with-curl --enable-mbregex --enable-fpm --with-fpm-user=www --with-fpm-group=www --enable-mbstring --with-mhash --enable-pcntl --enable-sockets --enable-dom --with-xsl --enable-opcache --with-openssl --enable-simplexml --enable-zip --enable-shmop --enable-ftp --enable-bcmath --with-pdo_mysql --with-mysqli=mysqlnd

make -j 4

make install
```
##### 安装php7错误信息总结
```
configure: error: libxml2 not found. Please check your libxml2 installation.
yum install -y  libxml2-devel

configure: error: Please reinstall the BZip2 distribution
yum install -y  bzip2-devel

configure: error: cURL version 7.15.5 or later is required to compile php with cURL support
yum install -y  curl-devel

configure: error: jpeglib.h not found.
yum install -y  libjpeg-devel

configure: error: png.h not found.
yum install -y libpng-devel

configure: error: freetype-config not found.
yum install -y freetype-devel

configure: error: xslt-config not found. Please reinstall the libxslt >= 1.1.0 distribution
yum install -y libxslt-devel

configure: error: Please reinstall the libzip distribution
yum install -y libzip-devel
```
