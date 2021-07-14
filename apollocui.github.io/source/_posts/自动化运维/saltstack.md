---
layout: post
title: saltstack
date: 2019-06-25 15:23:16
tags:
    - [ saltstack ]
categories:
    - [ 自动化运维 ]
---
## 部署saltstack

##### saltstack 环境配置
```
环境:关闭firewalld，iptables，selinux,设置dns或者hosts解析
     python 版本: 2.7 <= python < 3
     master和slave设置hosts解析:
     192.168.1.117 saltstack-master
     192.168.1.118 saltstack-slave
```
##### 安装saltstack
```
服务端安装:
yum  -y  install  epel-release
yum  -y  install  salt-master salt-minion 

服务端配置修改:
# vim /etc/salt/minion                   //在第16行添加，冒号后有一个空格
master: saltstack-master

客户端安装:
yum  -y  install  epel-release
yum  -y  install  salt-minion

客户端配置修改:
# vim /etc/salt/minion                   //在第16行添加，冒号后有一个空格
master: saltstack-master

客户端和服务端启动服务:
service salt-master start #只有服务端安装了该服务
service salt-minion start #服务端和客户端都安装此服务

配置认证:
服务端操作:
salt-key -a  saltstack-master
salt-key -a  saltstack-slave
salt-key

说明：-a ：accept ，-A：accept-all，-d：delete，-D：delete-all。可以使用 salt-key 命令查看到已经签名的客户端。此时我们在客户端的 /etc/salt/pki/minion 目录下面会多出一个minion_master.pub 文件

测试验证:
示例1： salt '*' test.ping                  //检测通讯是否正常，也可以指定其中一个 'saltstack-master'

示例2:  salt '*' cmd.run   'df -h'        //远程执行命令
```
---
##### saltstack 常用模块
```
salt "*" sys.list_modules  #查看支持的模块

1.Archive模块
#采用gunzip解压sourcefile.txt.gz包
salt '*' archive.gunzip sourcefile.txt.gz

#采用gzip压缩sourcefile.txt文件
salt '*' archive.gzip sourcefile.txt
API调用：

client.cmd('*','archive.gunzip',['sourcefile.txt.gz'])

2.cmd模块
#获取被控主机的内存使用情况
salt '*' cmd.run 'free -m'

#同步test.py到被控端，再执行
salt '*' cmd.script salt://script/test.py
API调用：

client.cmd('*','cmd.run',['free -m'])

3.cp模块
#将被控主机的/etc/hosts文件复制到被控主机本地的salt cache目录（/var/cache/salt/minion/localfiles/）
salt '*' cp.cache_local_file /etc/hosts

#将主控端file_roots指定位置下的目录复制到被控主机/minion/目录下
salt '*' cp.get_dir salt://script/ /minion/

#将主控端file_roots指定位置下的文件复制到被控主机/minion/test.py文件(file为文件名)
salt '*' cp.get_dir salt://script/test.py /minion/test.py

#下载URL内容到被控主机指定位置(/tmp/index.html)
salt '*' cp.get_url http://www.slashdot.ort /tmp/index.html
复制代码
API调用：

client.cmd('*','cp.get_file',['salt://script/test.py','/minion/test.py'])

4.cron模块
#查看指定被控主机、root用户的crontab操作
salt '*' cron.raw_cron root

#为指定被控主机、root用户添加任务
salt '*' cron.set_job root '*' '*' '*' '*' 1 /usr/sbin/ntpdate

#删除指定被控主机、root用户crontab的任务
salt '*' cron.rm_job root /usr/sbin/ntpdate 
复制代码
API调用：

client.cmd('*','cron.set_job',['root','*','*','*','*',1,'/usr/sbin/ntpdate'])

5.file模块
#校验所有被控主机/etc/fstab文件的md5值是否为xxxxxxxxxxxxx,一致则返回True值
salt '*' file.check_hash /etc/fstab md5=xxxxxxxxxxxxxxxxxxxxx

#校验所有被控主机文件的加密信息，支持md5、sha1、sha224、shs256、sha384、sha512加密算法
salt '*' file.get_sum /etc/passwd md5

#修改所有被控主机/etc/passwd文件的属组、用户权限、等价于chown root:root /etc/passwd
salt '*' file.chown /etc/passwd root root

#复制所有被控主机/path/to/src文件到本地的/path/to/dst文件
salt '*' file.copy /path/to/src /path/to/dst

#检查所有被控主机/etc目录是否存在，存在则返回True,检查文件是否存在使用file.file_exists方法
salt '*' file.directory_exists /etc

#获取所有被控主机/etc/passwd的stats信息
salt '*' file.stats /etc/passwd

#获取所有被控主机/etc/passwd的权限mode，如755，644
salt '*' file.get_mode /etc/passwd

#修改所有被控主机/etc/passwd的权限mode为0644
salt '*' file.set_mode /etc/passwd 0644

#在所有被控主机创建/opt/test目录
salt '*' file.mkdir /opt/test

#将所有被控主机/etc/httpd/httpd.conf文件的LogLevel参数的warn值修改为info
salt '*' file.sed /etc/httpd/httpd.conf 'LogLevel warn' 'LogLevel info'

#给所有被控主机的/tmp/test/test.conf文件追加内容‘maxclient 100’
salt '*' file.append /tmp/test/test.conf 'maxclient 100'

#删除所有被控主机的/tmp/foo文件
salt '*' file.remove /tmp/foo

API调用：

client.cmd('*','file.remove',['/tmp/foo'])

6.network 模块
salt '*' network.dig www.qq.com
salt '*' network.ping www.qq.com
salt '*' network.ip_addrs

7.pkg包管理模块
管理yum， apt-get等
salt '*' pkg.install php   # 安装php
salt '*' pkg.remove php    # 移除php
salt '*' pkg.upgrade       # 升级所有的软件包

8.service模块
salt '*' service.enable nginx    # 使nginx可用
salt '*' service.disable nginx   # 使nginx不可用
salt '*' service.restart nginx   # 重启nginx

9.pillar模块
记录所有minion通用的属性，如cpu、内存、磁盘等信息，然后同步到master端
salt-call saltutil.refresh_pillar # minion端
salt '*' saltutil.refresh_pillar  # master端

pillar（在master上定义）（yaml语法）
在配置文件中找pillar的文件路径：

找到以后，mkdir /export/salt/pillar
vim top.sls
base:
"*":
- test
vim test.sls
conf: xiang
然后刷新pillar： salt '*' saltutil.refresh_pillar
验证：salt '*' pillar.items conf
或者： salt -I 'conf:xiang' test.ping

10. grains模块
记录minion的属性key：value
[root@k8s-master ~]# salt "*" grains.ls
k8s-node1:
    - SSDs
    - biosreleasedate
    - biosversion
    - cpu_flags
    - cpu_model
    - cpuarch
    - domain
    ......

自定义grians（在minion上定义的）
grains是在minion启动时搜集一些信息，如操作系统类型，网卡，内核版本，cpu架构等
salt "*" grains.ls    列出所有grains项目名字
salt "*app.*" grains.items  列出所有grains项目以及值
grains的信息并不是动态的，并不会实时变化，它只是在minion启动时收集到的
我们可以根据grains收集到的一些信息，做一些配置管理工作
在minion上：vim /etc/salt/grains
role: nginx
env: test
重启service salt-minion restart
获取grians：
salt "*" grains.item role env
或者：
salt -G "*" role:nginx cmd.run "hostname“
salt "*" grains.items
```
---
##### 错误收集
```
1、错误信息 Minion did not return. [Not connected]
删除客户端公钥:rm -rf /etc/salt/pki/minion/minion_master.pub
service salt-minion restart #重启client服务  

2、错误信息:Failed building wheel for uwsgi
yum -y install python-devel（python2.7）
```