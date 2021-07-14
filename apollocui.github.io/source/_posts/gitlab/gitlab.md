---
layout: post
title: gitlab
date: 2019-06-17 16:14:25
tags:
   - [ gitlab ]
categories:
    - [gitlab]
password: cbb4693
---

## gitlab 迁移 故障 总结

##### gitlab 迁移
```
升级gitlab服务和服务器配置准备:
a、关闭gitlab服务中(sidekiq和unicorn)、nginx服务和docker服务
b、对gitlab服务备份命令:gitlab-rake gitlab:backup:create
c、关闭gitlab所有服务，在阿里云控制台对系统盘做快照(恢复方式有两种:对快照做成镜像恢复系统，按量付费ECS机器)
d、gitlab服务升级:备份gitlab相关配置文件目录:/etc/gitlab,/opt/gitlab/embedded/service/gitlab-shell/hooks
(hooks文件中:pre-receive是本地推送到远端仓库使用的，post-receive:是远端仓库使用runner调用的文件，新版本:11.8和之前就有所改变，不然不能触发runner和CI文件)
e、所有服务已关闭，对ECS服务器进行升级
   
#新服务器gitlab环境部署(迁移前和当前新gitlab-ce环境必须保持一致，不然会报错)
修改/etc/sysctl.conf
加上vm.overcommit_memory = 1, Linux内核会根据参数vm.overcommit_memory参数的设置决定是否放行。
修改完执行sysctl -p
vm.overcommit_memory = 1，直接放行
vm.overcommit_memory = 0：则比较 此次请求分配的虚拟内存大小和系统当前空闲的物理内存加上swap，决定是否放行。
vm.overcommit_memory = 2：则会比较进程所有已分配的虚拟内存加上此次请求分配的虚拟内

安装php客户端，用于php语法检测

备份hooks目录，存放钩子文件

1、安装openresty环境,同步相关数据,如下所示:
/kuaibao/server/nginx/conf/nginx.conf
/kuaibao/server/nginx/conf/vhosts
/kuaibao/server/nginx/conf/key

/etc/gitlab/gitlab.rb   #gitlab配置文件
/etc/gitlab/gitlab-secrets.json  #gitlab秘钥文件
/etc/gitlab/ssl   #同步https存放秘钥目录

2、安装gitlab-ce版本
yum -y install policycoreutils-python curl policycoreutils openssh-server openssh-clients postfix  patch zlib-devel perl perl-devel  #安装gitlab-ce依赖包

rpm -ivh gitlab-ce-11.7.0-ce.0.el7.x86_64.rpm   #安装gitlab-ce
 
cat /etc/postfix/main.cf  #修改postfix发邮件配置
inet_interfaces = localtion 修改成  inet_interfaces = all

systemctl start postfix && systemctl enable postfix  #开启发邮件

gitlab-ctl reconfigure  #初始化gitlab配置

3、部署docker环境(docker环境中runner可以和gitlab服务分开分开)
yum install -y yum-utils device-mapper-persistent-data lvm2   #安装yum依赖

yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo #配置yum源

yum makecache fast  #生成yum本地缓存

yum -y  install  docker-ce

systemctl start docker  && systemctl enable docker 

cat /etc/docker/daemon.json 
{
  "registry-mirrors": ["https://d5ma8gvx.mirror.aliyuncs.com"]
}

cat /root/.docker/config.json #修改服务器pull镜像认证信息

##导出镜像并导入到新服务器上
[root@izbp19hcysthc34tbzp53bz ~]# docker save -o code-push-cli.tar bbe405d4b00c
[root@izbp19hcysthc34tbzp53bz ~]# docker save -o php-ci.tar d25001fdfaed

[root@iZbp12yr7w4vdshygsnusjZ ~]# docker tag bbe405d4b00c cqingwang/code-push-cli:v5
[root@iZbp12yr7w4vdshygsnusjZ ~]# docker tag d25001fdfaed registry.cn-hangzhou.aliyuncs.com/kuaibao/php-ci:latest
[root@iZbp12yr7w4vdshygsnusjZ ~]# docker images
REPOSITORY                                         TAG                 IMAGE ID            CREATED             SIZE
cqingwang/code-push-cli                            v5                  bbe405d4b00c        2 months ago        118MB
registry.cn-hangzhou.aliyuncs.com/kuaibao/php-ci   latest              d25001fdfaed        12 months ago       261MB

systemctl daemon-reload &&  systemctl restart docker

4、备份恢复gitlab数据(备份代码需要关闭:sidekiq,unicorn和nginx服务)
备份老gitlab上数据:gitlab-rake gitlab:backup:create  >>/opt/gitlab_backup.log 2>&1  

在新gitlab服务器上恢复数据(恢复代码不需要关闭gitlab服务):
备份数据放到gitlab默认备份目录下:/var/opt/gitlab/backups  
修改文件名: mv 1550593467_2019_02_20_11.5.0_gitlab_backup.tar 1550593467_gitlab_backup.tar

gitlab-rake gitlab:backup:restore BACKUP=1550593467  #恢复数据目录(会出现两次交互，都选择yes)

gitlab-ctl stop #关闭所有服务

gitlab-ctl start #开启所有服务

systemctl enable gitlab-runsvdir  #设置gitlab开机启动

gitlab-rake gitlab:check SANITIZE=true  #检查数据恢复情况

#查看gitlab所有的日志
gitlab-ctl tail  

# 拉取/var/log/gitlab下子目录的日志
sudo gitlab-ctl tail gitlab-rails

# 拉取某个指定的日志文件
sudo gitlab-ctl tail nginx/gitlab_error.log

systemctl restart docker 

/kuaibao/server/nginx/conf/nginx/sbin/nginx -t #检查nginx配置
/kuaibao/server/nginx/conf/nginx/sbin/nginx  #开启nginx服务

# chrome访问gitlab新服务器，并测试

# 小版本升级注意事项(比如:11.5至11.8)
1、备份:/etc/gitlab/和/opt/gitlab/embedded/service/gitlab-shell/hooks/目录
2、升级小版本不需要关闭gitlab服务
```
##### gitlab配置和生产环境中遇到问题问题
```
git命令参考网址:http://rogerdudler.github.io/git-guide/index.zh.html

1、项目归档（项目只有读权限）

projetc->Setting->General->Advanced->archive project 

2、开启分支保护和tags保护（和每组设置变量相关）

选中某个项目->Settings->Repository->Protected Tags->创建维护者(20*)

3、关闭gitlab注册功能

Admin Area->Settings->Sign-up restrictions（勾去掉，就是取消注册）

4、开启二次验证

Admin Area->Settings->Sign-in restrictions(开启二次验证)

5、设置CICD秘钥设置

选择项目->Settings->CICD->Environment variables(这里面添加变量:公私钥变量名设置ID_RSA和ID_RSA_PUB)

6、浏览器进行合并时需要先执行:git pull 或git fetch

7、配置runner

单独项目使用runner
project(选择项目)->Settings->CICD->Runner（复制里面key）
设置share runner
Admin Area->Settings->CICD->Runner（复制里面key）

8、docker登录信息本地存储
[root@izbp19hcysthc34tbzp53bz ~]# cat /root/.docker/config.json 
{
	"auths": {
		"registry.cn-hangzhou.aliyuncs.com": {
			"auth": "xxxxxxx密码xxxxxxxxxx"
		}
	},
	"HttpHeaders": {
		"User-Agent": "Docker-Client/17.09.0-ce (linux)"
	}
}

9、以下信息是注册runner时生成信息

concurrent = 1
check_interval = 0

[session_server]
  session_timeout = 1800

[[runners]]
  name = "mansoncui-test"
  url = "https://gitlab.XXXXXX.com"
  token = "XXXXXX"
  executor = "docker"
  [runners.docker]
    pull_policy = "if-not-present"  #判断镜像是否从本地取
    tls_verify = false
    image = "php"
    privileged = false
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    volumes = ["/cache"]
    shm_size = 0
  [runners.cache]
    [runners.cache.s3]
    [runners.cache.gcs]

9、gitlab-ci文件(注意文件中域名修改)

$ cat .gitlab-ci.yml
image: ${IMAGE}   #在groups中设置组变量

stages:
  - deploy

variables:
  PROJECT_NAME: mansoncui
  CI_DEBUG_TRACE: "true"    #开启debug模式

  #prod env
  PROD_ADDRES_IP: "192.168.1.120"
  PROD_ADDRES_PORT: "22"
  PROD_DOMAIN: "http://gitlab.mansoncui.com"

  #gitlab
  GITLAB_ADDRES_PORT: "22"
  GITLAB_ADDRES_IP: "gitlab.mansoncui.com"

before_script:
  - echo "${DEBUG} ${IMAGE}"   #打印变量
  - if [ "${DEBUG}" == "TRUE" ];then sleep 600;fi   #设置项目变量，调试CI文件
  - if [ "${PROJECT_NAME}" == "" ]; then echo "${PROJECT_NAME} 没有定义， jobs异常退出。。。"; exit 1;fi #判断变量是否存在，不存直接退出  

  - mkdir -p ~/.ssh
  - echo "118.126.66.60 www.mansoncui.com" >> /etc/hosts
  - echo "$ID_RSA_PUB" > ~/.ssh/id_rsa.pub
  - echo "$ID_RSA" > ~/.ssh/id_rsa && chmod 0600 ~/.ssh/id_rsa
  - ssh-keyscan -H -t ecdsa -p $PROD_ADDRES_PORT $PROD_ADDRES_IP >> ~/.ssh/known_hosts

deploy_prod:
  stage: deploy
  script:
    - rsync -avztH -e "ssh -p $PROD_ADDRES_PORT" --exclude ".git" --delete ./ $PROD_ADDRES_IP:/xxxx/www/${PROJECT_NAME}/
  only:
    - tags
  except:
    - /(?i:rc)$/
  tags:
    - vpc-share-tags
  environment:
    name: prod
    url: $DOMAIN

sendmail:
  stage: deploy
  script:
    - if [ ! -f sendmail.py ]; then wget -O sendmail.py https://www.xxxx.com/gitlab/xxxxx.py; fi
    - python ./sendmail.py
  only:
    - tags
  except:
    - /(?i:rc)$/
  tags:
    - vpc-share-tags
  allow_failure: false
```
