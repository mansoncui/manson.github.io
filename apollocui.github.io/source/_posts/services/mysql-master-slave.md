---
layout: post
title: mysql_master_slave
date: 2019-06-25 16:38:43
tags:
---

## 主从搭建
```
环境:两台服务器(主:192.168.1.10,从:192.168.1.20)，两个数据库服务器有的库和表必须一样，不一样用mysqldump备份导入从数据库服务器上。
mysql主服务器:
1、启用binlog日志
2、service mysql stop
vim  /etc/my.cnf
[mysqld]
log-bin   #启用binlog日志（默认是主机名和数据库目录，也可以自定义目录:/root/dsn） 
server_id=10    主机位标识(1-255) #标识自己的身份
server mysql start

3、授权一个连接用户可以从192.168.1.20来连接自己，连接后有拷贝数据的权限
grant  replication slave  on *.*   to   用户名@“从服务器地址”   identified  by  “密码”；#给从服务器授权，到从服务器上测试是否能连接

二、从服务器设置:
1、server  mysql  stop

vim  /etc/my.cnf
[mysqld]
log-bin=slave  #可有可无，不做硬性规定
server-id=20    #主机位标识(1-255) #标识自己的身份

server mysql start

2、连接主数据库服务器，看连接是否正常(命令行下)
#mysql  -h192.168.1.20 -u授权用户  -p密码

3、从本机登录数据库
mysql  -uroot   -p密码
mysql>change master to master_host="192.168.1.10",master_user="授权用户"，master_password="授权密码"，master_log_file="binlog日志文件",
master_log_pos=时间节点；
master_log_file=""   #主服务器上日志文件名
mysql>change master to master_host="10.2.0.29",master_user="scrm",master_password="thankyou",master_log_file="mysql-bin.000003",master_log_pos=154;

/ebsig/mysqldata/mysql/auto.cnf  #如果是克隆从数据库，需要重新生成uuid，先把老的UUID备份，重启mysql服务就可以了

mysql>set global server_id=2;  #配置文件server_id不能生效要自己设置；

用命令mysql>show  master status;   #查看时间节点和binlog日志文件名（在主服务器上操作的）

mysql>show  slave  status\G；   #在从服务器上操作的，看状态:Slave_IO_Running: No 　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　Slave_SQL_Running: No

mysql>start slave; #开启从服务器Slave_IO_Running: Yes 　　　　　　　　　　　　　　　　  Slave_SQL_Running: Yes

三、测试:在主数据库服务器上创建表、库和更新数据，到从数据库服务器上查看是否同步

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
数据库主从同步原理:

如果有NO没有成功：原因:从服务器是克隆，修改uuid，存放在数据库目录下/var/lib/mysql/auto.cnf,随便修改一下就可以了

错误信息看一下四项:
IO进程原理:IO进程负责拷贝主数据库服务器binlog里的SQL语句到本机中继binlog日志里 IO进程报错信息 Last_IO_Errno: 0   #出错的个数
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　             Last_IO_Error:      #出错的信息
SQL进程原理:SQL进程负责本机中继binlog日志里sql语句把数据写进数据库中 SQL进程报错信息 Last_SQL_Errno: 0   #出错的个数
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　                   Last_SQL_Error:     #出错的信息
IO进程什么时候出错?  连接不上主数据库服务器时会出错    ping  (线路问题   安全)    授权
SQL进程什么时候会出错? sql语句操作的库、表和字段在自己本机不存在，就会出错
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
从数据库目录下有几个文件
验证从数据库服务器是否能自动备份主数据库服务器上的数据
master.info    #记录主数据文件的信息
web-relay-bin.index   #记录已有的中继binlog日志
relay-log.info #记录中继日志信息
web-relay-bin.00000X   #中继binlog日志：是从主数据库服务器上拷贝到本机的binlog日志
```

## DDL 和 DML含义
```
DDL：数据定义语言(创建表和修改表、truncate 删除表)
DML(Data Manipulation Language)：数据操纵语言命令使用户能够查询数据库以及操作已有数据库中的数据。
truncate不是可以删除表中数据，这就算可以操作数据库中的数据了，为什么还是DDL而不是DML呢？谢谢！
那像drop table这些命令是手工提交的？
```

## 主从延迟
##### MySQL数据库主从同步延迟原理
```
mysql主从同步原理：
主库针对写操作，顺序写binlog，从库单线程去主库顺序读”写操作的binlog”，从库取到binlog在本地原样执行(随机写)，来保证主从数据逻辑上一致。
mysql的主从复制都是单线程的操作，主库对所有DDL和DML产生binlog，binlog是顺序写，所以效率很高，slave的Slave_IO_Running线程到主库取日志，效率比较高，下一步，问题来了，slave的Slave_SQL_Running线程将主库的DDL和DML操作在slave实施。DML和DDL的IO操作是随即的，不是顺序的，成本高很多，还可能可slave上的其他查询产生lock争用，由于Slave_SQL_Running也是单线程的，所以一个DDL卡主了，需要执行10分钟，那么所有之后的DDL会等待这个DDL执行完才会继续执行，这就导致了延时。
```

##### MySQL数据库主从同步延迟是怎么产生的?
```
有朋友会问：“主库上那个相同的DDL也需要执行10分，为什么slave会延时?”，答案是master可以并发，Slave_SQL_Running线程却不可以。
当主库的TPS并发较高时，产生的DDL数量超过slave一个sql线程所能承受的范围，那么延时就产生了，当然还有就是可能与slave的大型query语句产生了锁等待。
首要原因：数据库在业务上读写压力太大，CPU计算负荷大，网卡负荷大，硬盘随机IO太高
次要原因：读写binlog带来的性能影响，网络传输延迟。
```

