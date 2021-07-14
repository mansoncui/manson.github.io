---
title: docker
date: 2019-06-11 16:41:23
tags:
    - [ docker ]
    - [docker-compose]
    - [docker-volume]
categories:
    - [ docker ]
---

## docker

##### docker常识积累

```
docker CE社区版官方免费支持(7个月)
docker EE企业版官方免费支持(24个月)

国内支持仓库云服务商: 时速云镜像仓库、网易云镜像服务、DaoCloud 镜像市场、阿里云镜像

###centos 安装docker-ce

$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
安装依赖包:
sudo yum install -y yum-utils \
           device-mapper-persistent-data \
           lvm2

添加yum源:
sudo yum-config-manager \
    --add-repo \
    https://mirrors.ustc.edu.cn/docker-ce/linux/centos/docker-ce.repo

官网源:
# $ sudo yum-config-manager \
#     --add-repo \
#     https://download.docker.com/linux/centos/docker-ce.repo

如果需要测试版本的 Docker CE 请使用以下命令：
$ sudo yum-config-manager --enable docker-ce-test

如果需要每日构建版本的 Docker CE 请使用以下命令：
$ sudo yum-config-manager --enable docker-ce-nightly

更新 yum 软件源缓存，并安装 docker-ce
$ sudo yum makecache fast
$ sudo yum install docker-ce

使用脚本自动安装
$ curl -fsSL get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh --mirror Aliyun

启动 Docker CE
$ sudo systemctl enable docker
$ sudo systemctl start docker

添加内核参数
如果在 CentOS 使用 Docker CE 看到下面的这些警告信息：

WARNING: bridge-nf-call-iptables is disabled
WARNING: bridge-nf-call-ip6tables is disabled
请添加内核配置参数以启用这些功能。

$ sudo tee -a /etc/sysctl.conf <<-EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

然后重新加载 sysctl.conf 即可
$ sudo sysctl -p

查看镜像大小

docker system df   #查看镜像、容器、数据卷所占用的空间

docker images 等于 docker image ls  #查看镜像实际占用磁盘空间

docker image ls -f dangling=true  #显示虚悬镜像(虚悬镜像:是指没有仓库名和标签的镜像)

docker image prune  #删除虚悬镜像

# docker image ls -f before=crond_new:1.0.1  #查看crond_new:1.0.1之前建立镜像

# docker image ls -f since=crond_dev:1.0.0   #查看crond_dev:1.0.0之后建立镜像

# docker image ls -q  #只列出镜像ID

# docker image ls --format "{{.ID}}: {{.Repository}}" #只列出镜像ID和仓库名

# docker image ls --format "table {{.ID}}\t{{.Repository}}\t{{.Tag}}"  #指定间隔符号

# docker image ls --digests  #获取精确镜像摘要 

# docker rm node@sha256:b4f0e0bdeb578043c1ea6862f0d40cc4afe32a4a582f3be235a3b164422be228  #根据镜像摘要进行删除

# docker image rm $(docker image ls -q redis)  #删除redis实列
```

##### docker volume

```
docker volume ls

docker volume inspect my-vol

docker run -d -P --name web --mount source=my-vol,target=/webapp training/webapp python app.py  #挂载实列

docker inspect web  #查看逻辑卷信息

docker volume rm my-vol  #删除逻辑卷

docker volume prune  #删除无主逻辑卷

docker run -itd --name test_local --restart=always --mount type=bind,source=/opt/app,target=/kuaibao/www alpine /bin/sh  #挂载本地目录至容器中，和-v选项区别是:本地目录不存在时会报错，而-v则会自己创建

docker run -itd --name test --restart=always --mount source=my-vol,target=/kuaibao/www alpine /bin/sh  #挂载本地创建的volume挂载至容器中

Docker 挂载主机目录的默认权限是 读写，用户也可以通过增加 readonly 指定为 只读。

docker run -itd --name test_read --restart=always --mount type=bind,source=/opt/app,target=/kuaibao/www,readonly alpine /bin/sh  #只读模式,如下测试
[root@k8s-slave ~]# docker exec -it test_read touch /kuaibao/www/1.txt
touch: /kuaibao/www/1.txt: Read-only file system

docker run -itd --name test_file --restart=always --mount type=bind,source=$HOME/.bash_history,target=/root/.bash_history alpine /bin/sh  #挂载文件至容器中
```

##### docker 选项

```
EXPOSE  记录服务可用端口，但是并不创建喝宿主机之间的映射
--export 运行时暴露端口，但是并不创建喝宿主机之间的映射
-p  创建端口映射规则，比如 -p ip:hostPort:containerPort| ip::containerPort | hostPort:containerPort | containerPort 
    必须指定containerPort,如果没有指定hostPort，Docker 会自动分配端口
	
-P  将Dockerfile里暴露的所有容器端口映射到动态分配的宿主机端口上

--link 在消费和服务容器之间创建链接，比如 --link name:alias
       这会创建一系列环境变量，并在消费者容器的/etc/hosts文件里添加入口项
	   必须暴露或发布端口
```

##### docker log

```
docker logs [OPTIONS] CONTAINER
  Options:
        --details        显示更多的信息
    -f, --follow         跟踪实时日志
        --since string   显示自某个timestamp之后的日志，或相对时间，如42m（即42分钟）
        --tail string    从日志末尾显示多少行日志， 默认是all
    -t, --timestamps     显示时间戳
        --until string   显示自某个timestamp之前的日志，或相对时间，如42m（即42分钟）

案列:
#查看指定时间后的日志，只显示最后100行：
docker logs -f -t --since="2018-02-08" --tail=100 CONTAINER_ID

#查看最近30分钟的日志
docker logs --since 30m CONTAINER_ID

#查看某时间之后的日志：
docker logs -t --since="2018-02-08T13:23:37" CONTAINER_ID

#查看某时间段日志
docker logs -t --since="2018-02-08T13:23:37" --until "2018-02-09T12:23:37" CONTAINER_ID
```

##### docker-compose

```
docker-compose create	创建所有的服务

docker-compose start	启动被停止或未启动的服务

docker-compose up	    创建所有服务并且启动服务，即同时执行了create和start命令

docker-compose stop	    停止所有服务

docker-compose kill	    强行停止所有服务

docker-compose rm	    删除停止的服务

docker-compose restart	重启所有服务

docker-compose down	    停止、删除所有的服务以及网络、镜像

docker-compose up -d nginx   #构建建启动nignx容器

docker-compose exec nginx bash  #登录到nginx容器中

docker-compose down  #删除所有nginx容器,镜像

docker-compose ps  #显示所有容器

docker-compose restart ngin  #重新启动nginx容器

docker-compose run --no-deps --rm php-fpm php -v  #在php-fpm中不启动关联容器，并容器执行php -v 执行完成后删除容器

docker-compose build nginx  #构建镜像

docker-compose build --no-cache nginx  #不带缓存的构建

docker-compose logs nginx  #查看nginx的日志

docker-compose logs -f nginx  #验证（docker-compose.yml）文件配置，当配置正确时，不输出任何内容，当文件配置错误，输出错误信息

docker-compose pause nginx  #暂停nignx容器

docker-compose unpause nginx #恢复ningx容器

docker-compose rm nginx  #删除容器（删除前必须关闭容器）

docker-compose stop nginx  #停止nignx容器

docker-compose start nginx  #启动nignx容器
```

##### 说明:以下是制作php的crond计划任务镜像

```
命令:docker run -d --name crond  -v  /www/:/www/ web:1.0.0

FROM alpine:edge
MAINTAINER Cuibobo <cuibobo@kuaidihelp.com>

ENV LANG=C.UTF-8
ENV PHP_FPM_USER="www" 
ENV PHP_FPM_GROUP="www" 
ENV PHP_FPM_LISTEN_MODE="0660"
ENV PHP_MEMORY_LIMIT="512M"
ENV PHP_MAX_UPLOAD="50M"
ENV PHP_MAX_FILE_UPLOAD="200"
ENV PHP_MAX_POST="100M"
ENV PHP_DISPLAY_ERRORS="On"
ENV PHP_DISPLAY_STARTUP_ERRORS="On"
ENV PHP_ERROR_REPORTING="E_COMPILE_ERROR\|E_RECOVERABLE_ERROR\|E_ERROR\|E_CORE_ERROR"
ENV PHP_CGI_FIX_PATHINFO=0
ENV TIMEZONE="Asia/Shanghai"

COPY ./service.sh /root/
COPY ./ErrorLogWriterDev.phar /root/
COPY ./rsyslog.conf /opt/

RUN  apk update && \
     apk add --no-cache tzdata  && \
	apk add --no-cache php7-fpm \
		       php7 \
                       php7-mcrypt \
                       php7-soap  \
                       php7-openssl \
                       php7-gmp \
                       php7-pdo_odbc \
                       php7-json \
                       php7-dom \
                       php7-pdo \
                       php7-zip \
		       php7-mysqli \
		       php7-phar \
                       php7-sqlite3 \
                       php7-apcu \
		       php7-pdo_pgsql \
                       php7-bcmath \
                       php7-gd \
                       php7-odbc \
                       php7-pdo_mysql \
                       php7-pdo_sqlite \
                       php7-gettext \
                       php7-xmlreader \
                       php7-xmlrpc \
                       php7-bz2 \
                       php7-sockets \
                       php7-xmlwriter \
		       php7-xsl \
		       php7-tokenizer \
		       php7-ftp \
		       php7-posix \
                       php7-fileinfo \
		       php7-shmop \
		       php7-mbstring \
                       php7-iconv \
                       php7-pdo_dblib \
                       php7-curl \
		       php7-session \
                       php7-ctype \
		       php7-simplexml \
                       php7-redis && \
		sed -i "s|;listen.owner\s*=\s*nobody|listen.owner = ${PHP_FPM_USER}|g" /etc/php7/php-fpm.d/www.conf && \
sed -i -e "s/;daemonize\s*=\s*yes/daemonize = no/g" /etc/php7/php-fpm.conf && \
sed -i -e "s/;slowlog/slowlog/g" /etc/php7/php-fpm.d/www.conf && \
sed -i "s|;listen.group\s*=\s*nobody|listen.group = ${PHP_FPM_GROUP}|g" /etc/php7/php-fpm.conf && \
sed -i "s|;listen.mode\s*=\s*0660|listen.mode = ${PHP_FPM_LISTEN_MODE}|g" /etc/php7/php-fpm.conf && \
sed -i "s|user\s*=\s*nobody|user = ${PHP_FPM_USER}|g" /etc/php7/php-fpm.conf && \
sed -i "s|group\s*=\s*nobody|group = ${PHP_FPM_GROUP}|g" /etc/php7/php-fpm.conf && \
sed -i "s|;log_level\s*=\s*notice|log_level = notice|g" /etc/php7/php-fpm.conf && \
sed -i "s|display_errors\s*=\s*Off|display_errors = ${PHP_DISPLAY_ERRORS}|i" /etc/php7/php.ini && \
sed -i "s|display_startup_errors\s*=\s*Off|display_startup_errors = ${PHP_DISPLAY_STARTUP_ERRORS}|i" /etc/php7/php.ini && \
sed -i "s|error_reporting\s*=\s*E_ALL & ~E_DEPRECATED & ~E_STRICT|error_reporting = ${PHP_ERROR_REPORTING}|i" /etc/php7/php.ini && \
sed -i "s|;*memory_limit =.*|memory_limit = ${PHP_MEMORY_LIMIT}|i" /etc/php7/php.ini && \
sed -i "s|;*upload_max_filesize =.*|upload_max_filesize = ${PHP_MAX_UPLOAD}|i" /etc/php7/php.ini && \
sed -i "s|;*max_file_uploads =.*|max_file_uploads = ${PHP_MAX_FILE_UPLOAD}|i" /etc/php7/php.ini && \
sed -i "s|;*post_max_size =.*|post_max_size = ${PHP_MAX_POST}|i" /etc/php7/php.ini && \
sed -i "s|;*cgi.fix_pathinfo=.*|cgi.fix_pathinfo= ${PHP_CGI_FIX_PATHINFO}|i" /etc/php7/php.ini && \
cp /usr/share/zoneinfo/${TIMEZONE} /etc/localtime && \
echo "${TIMEZONE}" > /etc/timezone && \
sed -i "s|;*date.timezone =.*|date.timezone = ${TIMEZONE}|i" /etc/php7/php.ini && \
        mkdir /www && \
        apk del tzdata curl && \
	adduser -D -g 'www' www && \
        chown -R www:www /www && \
	apk add rsyslog logrotate && \
	rm -f /etc/rsyslog.conf && \
	mv /opt/rsyslog.conf /etc/ && \
        apk add --no-cache --repository http://dl-4.alpinelinux.org/alpine/edge/testing gnu-libiconv && \
	rm -rf /var/cache/apk/*
ENV LD_PRELOAD /usr/lib/preloadable_libiconv.so php

WORKDIR /www
VOLUME ["/www"]

EXPOSE 80
CMD ["/root/service.sh"]

Alpine操作系统
源码安装swoole需要依赖
apk add  gcc g++ make php7-dev php7-pear 
php7-dev  #phpize
php7-pear #pecl 
pecl install swoole-4.2.8.tgz  #选择mysqld和socket
```
##### php+openresty(alpine)
```
server.sh 可以根据自己情况编写，但是要以php7-fpm结尾
打包镜像
docker built -t web:1.0.0 -f Dockerfile . --no-cache
docker 运行实例
docker run -d --privileged=true --name test --restart=always  -p 80:80 -v /data/:/www/  web:1.0.0

FROM alpine:edge
MAINTAINER Cuibobo <cuibobo@kuaidihelp.com>

ENV PHP_FPM_USER="www" 
ENV PHP_FPM_GROUP="www" 
ENV PHP_FPM_LISTEN_MODE="0660"
ENV PHP_MEMORY_LIMIT="512M"
ENV PHP_MAX_UPLOAD="50M"
ENV PHP_MAX_FILE_UPLOAD="200"
ENV PHP_MAX_POST="100M"
ENV PHP_DISPLAY_ERRORS="On"
ENV PHP_DISPLAY_STARTUP_ERRORS="On"
ENV PHP_ERROR_REPORTING="E_COMPILE_ERROR\|E_RECOVERABLE_ERROR\|E_ERROR\|E_CORE_ERROR"
ENV PHP_CGI_FIX_PATHINFO=0
ENV TIMEZONE="Asia/Shanghai"

ARG RESTY_VERSION="1.13.6.2"
ARG RESTY_OPENSSL_VERSION="1.0.2p"
ARG RESTY_PCRE_VERSION="8.42"
ARG RESTY_J="3"
ARG RESTY_CONFIG_OPTIONS="\
    --with-file-aio \
    --with-http_addition_module \
    --with-http_auth_request_module \
    --with-http_dav_module \
    --with-http_flv_module \
    --with-http_geoip_module=dynamic \
    --with-http_gunzip_module \
    --with-http_gzip_static_module \
    --with-http_image_filter_module=dynamic \
    --with-http_mp4_module \
    --with-http_random_index_module \
    --with-http_realip_module \
    --with-http_secure_link_module \
    --with-http_slice_module \
    --with-http_ssl_module \
    --with-http_stub_status_module \
    --with-http_sub_module \
    --with-http_v2_module \
    --with-http_xslt_module=dynamic \
    --with-ipv6 \
    --with-mail \
    --with-mail_ssl_module \
    --with-md5-asm \
    --with-pcre-jit \
    --with-sha1-asm \
    --with-stream \
    --add-dynamic-module=/opt/ngx_http_substitutions_filter_module-master \
    --with-stream_ssl_module \
    --with-pcre=/tmp/pcre-${RESTY_PCRE_VERSION} \
    --with-openssl=/tmp/openssl-${RESTY_OPENSSL_VERSION} \
    --with-threads \
    "
ARG RESTY_CONFIG_OPTIONS_MORE=""
ARG RESTY_ADD_PACKAGE_BUILDDEPS=""
ARG RESTY_ADD_PACKAGE_RUNDEPS=""
ARG RESTY_EVAL_PRE_CONFIGURE=""
ARG RESTY_EVAL_POST_MAKE=""

LABEL resty_version="${RESTY_VERSION}"
LABEL resty_openssl_version="${RESTY_OPENSSL_VERSION}"
LABEL resty_pcre_version="${RESTY_PCRE_VERSION}"
LABEL resty_config_options="${RESTY_CONFIG_OPTIONS}"
LABEL resty_config_options_more="${RESTY_CONFIG_OPTIONS_MORE}"
LABEL resty_add_package_builddeps="${RESTY_ADD_PACKAGE_BUILDDEPS}"
LABEL resty_add_package_rundeps="${RESTY_ADD_PACKAGE_RUNDEPS}"
LABEL resty_eval_pre_configure="${RESTY_EVAL_PRE_CONFIGURE}"
LABEL resty_eval_post_make="${RESTY_EVAL_POST_MAKE}"
	
COPY ./ngx_http_substitutions_filter_module-master /opt/ngx_http_substitutions_filter_module-master
COPY ./echo-nginx-module /opt/echo-nginx-module
COPY ./service.sh /root/
COPY ./old /opt/old
COPY ./host /opt/host
COPY ./rsyslog.conf /opt/rsyslog.conf

RUN  apk update && \
     apk add --no-cache tzdata  && \
	apk add --no-cache php7-fpm \
		       php7 \
                       php7-mcrypt \
                       php7-soap  \
                       php7-openssl \
                       php7-gmp \
                       php7-pdo_odbc \
                       php7-json \
                       php7-dom \
                       php7-pdo \
                       php7-zip \
		       php7-mysqli \
                       php7-sqlite3 \
                       php7-apcu \
		       php7-pdo_pgsql \
                       php7-bcmath \
                       php7-gd \
                       php7-odbc \
                       php7-pdo_mysql \
                       php7-pdo_sqlite \
                       php7-gettext \
                       php7-xmlreader \
                       php7-xmlrpc \
                       php7-bz2 \
                       php7-sockets \
                       php7-xmlwriter \
		       php7-xsl \
		       php7-tokenizer \
		       php7-ftp \
		       php7-posix \
                       php7-fileinfo \
		       php7-shmop \
		       php7-mbstring \
                       php7-iconv \
                       php7-pdo_dblib \
                       php7-curl \
		       php7-session \
                       php7-ctype \
                       php7-redis && \
		sed -i "s|;listen.owner\s*=\s*nobody|listen.owner = ${PHP_FPM_USER}|g" /etc/php7/php-fpm.d/www.conf && \
sed -i -e "s/;daemonize\s*=\s*yes/daemonize = no/g" /etc/php7/php-fpm.conf && \
sed -i -e "s/;slowlog/slowlog/g" /etc/php7/php-fpm.d/www.conf && \
sed -i "s|;listen.group\s*=\s*nobody|listen.group = ${PHP_FPM_GROUP}|g" /etc/php7/php-fpm.conf && \
sed -i "s|;listen.mode\s*=\s*0660|listen.mode = ${PHP_FPM_LISTEN_MODE}|g" /etc/php7/php-fpm.conf && \
sed -i "s|user\s*=\s*nobody|user = ${PHP_FPM_USER}|g" /etc/php7/php-fpm.conf && \
sed -i "s|group\s*=\s*nobody|group = ${PHP_FPM_GROUP}|g" /etc/php7/php-fpm.conf && \
sed -i "s|;log_level\s*=\s*notice|log_level = notice|g" /etc/php7/php-fpm.conf && \
sed -i "s|display_errors\s*=\s*Off|display_errors = ${PHP_DISPLAY_ERRORS}|i" /etc/php7/php.ini && \
sed -i "s|display_startup_errors\s*=\s*Off|display_startup_errors = ${PHP_DISPLAY_STARTUP_ERRORS}|i" /etc/php7/php.ini && \
sed -i "s|error_reporting\s*=\s*E_ALL & ~E_DEPRECATED & ~E_STRICT|error_reporting = ${PHP_ERROR_REPORTING}|i" /etc/php7/php.ini && \
sed -i "s|;*memory_limit =.*|memory_limit = ${PHP_MEMORY_LIMIT}|i" /etc/php7/php.ini && \
sed -i "s|;*upload_max_filesize =.*|upload_max_filesize = ${PHP_MAX_UPLOAD}|i" /etc/php7/php.ini && \
sed -i "s|;*max_file_uploads =.*|max_file_uploads = ${PHP_MAX_FILE_UPLOAD}|i" /etc/php7/php.ini && \
sed -i "s|;*post_max_size =.*|post_max_size = ${PHP_MAX_POST}|i" /etc/php7/php.ini && \
sed -i "s|;*cgi.fix_pathinfo=.*|cgi.fix_pathinfo= ${PHP_CGI_FIX_PATHINFO}|i" /etc/php7/php.ini && \
cp /usr/share/zoneinfo/${TIMEZONE} /etc/localtime && \
echo "${TIMEZONE}" > /etc/timezone && \
sed -i "s|;*date.timezone =.*|date.timezone = ${TIMEZONE}|i" /etc/php7/php.ini && \
		apk add --no-cache --virtual .build-deps \
        build-base \
        curl \
        gd-dev \
        geoip-dev \
        libxslt-dev \
        linux-headers \
        make \
        perl-dev \
        readline-dev \
        zlib-dev \
        ${RESTY_ADD_PACKAGE_BUILDDEPS} \
    && apk add --no-cache \
        gd \
        geoip \
        libgcc \
        libxslt \
        zlib \
        ${RESTY_ADD_PACKAGE_RUNDEPS} \
    && cd /tmp \
    && if [ -n "${RESTY_EVAL_PRE_CONFIGURE}" ]; then eval $(echo ${RESTY_EVAL_PRE_CONFIGURE}); fi \
    && curl -fSL https://www.openssl.org/source/openssl-${RESTY_OPENSSL_VERSION}.tar.gz -o openssl-${RESTY_OPENSSL_VERSION}.tar.gz \
    && tar xzf openssl-${RESTY_OPENSSL_VERSION}.tar.gz \
    && curl -fSL https://ftp.pcre.org/pub/pcre/pcre-${RESTY_PCRE_VERSION}.tar.gz -o pcre-${RESTY_PCRE_VERSION}.tar.gz \
    && tar xzf pcre-${RESTY_PCRE_VERSION}.tar.gz \
    && curl -fSL https://openresty.org/download/openresty-${RESTY_VERSION}.tar.gz -o openresty-${RESTY_VERSION}.tar.gz \
    && tar xzf openresty-${RESTY_VERSION}.tar.gz \
    && cd /tmp/openresty-${RESTY_VERSION} \
    && ./configure -j${RESTY_J} ${_RESTY_CONFIG_DEPS} ${RESTY_CONFIG_OPTIONS} ${RESTY_CONFIG_OPTIONS_MORE} \
    && make -j${RESTY_J} \
    && make -j${RESTY_J} install \
    && cd /tmp \
    && if [ -n "${RESTY_EVAL_POST_MAKE}" ]; then eval $(echo ${RESTY_EVAL_POST_MAKE}); fi \
    && rm -rf \
        openssl-${RESTY_OPENSSL_VERSION} \
        openssl-${RESTY_OPENSSL_VERSION}.tar.gz \
        openresty-${RESTY_VERSION}.tar.gz openresty-${RESTY_VERSION} \
        pcre-${RESTY_PCRE_VERSION}.tar.gz pcre-${RESTY_PCRE_VERSION} \
    && apk del .build-deps \
    && rm -rf /opt/ngx_http_substitutions_filter_module-master \
    && rm -rf /opt/echo-nginx-module \
    && ln -sf /dev/stdout /usr/local/openresty/nginx/logs/access.log \
    && ln -sf /dev/stderr /usr/local/openresty/nginx/logs/error.log && \
        mkdir /www && \
        apk del tzdata curl && \
	adduser -D -g 'www' www && \
        chown -R www:www /www && \
	mkdir /run/nginx/ && \
	mv /opt/old/* /usr/local/openresty/nginx/conf/ && \
	apk add rsyslog && \
        apk add --no-cache --repository http://dl-4.alpinelinux.org/alpine/edge/testing gnu-libiconv && \
	rm -rf /var/cache/apk/*

# Add additional binaries into PATH for convenience
ENV LD_PRELOAD /usr/lib/preloadable_libiconv.so php
ENV PATH=$PATH:/usr/local/openresty/luajit/bin:/usr/local/openresty/nginx/sbin:/usr/local/openresty/bin

# Copy nginx configuration files
COPY nginx.conf /usr/local/openresty/nginx/conf/nginx.conf
COPY vhosts /usr/local/openresty/nginx/conf/vhosts	
#COPY key /usr/local/openresty/nginx/conf/key

WORKDIR /www
VOLUME ["/www"]

EXPOSE 80
CMD ["/root/service.sh"]
```
