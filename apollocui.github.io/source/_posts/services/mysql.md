---
layout: post
title: mysql
date: 2019-06-19 15:40:15
tags:
    - [ mysql ]
categories:
    - [ services ]
password: cbb4693
---
## mysql delete和truncate区别
```
1、truncate删除数据比delete快，而且使用的系统和事务日志资源少。

    相同点:两个不加关键字都可以删除表中所有表记录

2、DELETE 语句每次删除一行，并在事务日志中为所删除的每行记录一项。TRUNCATE TABLE 通过释放存储表数据所用的数据页来删除数据，并且只在事务日志中记录页的释放。

TRUNCATE TABLE 删除表中的所有行，但表结构及其列、约束、索引等保持不变。新行标识所用的计数值重置为该列的种子。如果想保留标识计数值，请改用 DELETE。如果要删除表定义及其数据，请使用 DROP TABLE 语句。

对于由 FOREIGN KEY 约束引用的表，不能使用 TRUNCATE TABLE，而应使用不带 WHERE 子句的 DELETE 语句。由于 TRUNCATE TABLE 不记录在日志中，所以它不能激活触发器。

TRUNCATE TABLE 不能用于参与了索引视图的表。

示例
下例删除 authors 表中的所有数据。

TRUNCATE TABLE authors

权限
TRUNCATE TABLE 权限默认授予表所有者、sysadmin 固定服务器角色成员、db_owner 和 db_ddladmin 固定数据库角色成员且不可转让。
```

## mysql 5.6 source install
```
源码下载地址:https://dev.mysql.com/get/Downloads/MySQL-5.6/mysql-5.6.42.tar.gz

第一步:安装依赖
yum -y install ncurses-devel cmake make ncurses gcc gcc-c++ ncurses ncurses-devel bison libgcrypt perl #编译需要软件包
yum -y remove mariadb-libs  #卸载默认依赖包，不卸载安装报错

第二部:创建用户和组
groupadd -g 500 mysql;useradd -g mysql -u 500 mysql

第三部:解压并安装(源码包自己下载)
tar -zxvf mysql-5.6.42.tar.gz

cd  mysql-5.6.42

cmake  -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_DATADIR=/usr/local/mysql/data -DMYSQL_TCP_PORT=3306 -DMYSQL_UNIX_ADDR=/usr/local/mysql/mysqld.sock -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci -DWITH_EXTRA_CHARSETS=all -DWITH_MYISAM_STORAGE_ENGINE=1 -DWITH_INNOBASE_STORAGE_ENGINE=1 -DENABLED_LOCAL_INFILE=1 -DEXTRA_CHARSETS=all -DWITH_EMBEDDED_SERVER=1 -DWITH_SSL=bundled -DWITH_DEBUG=0 -DENABLE_DOWNLOADS=1  #配置

make  && make install   #编译安装

mkdir /usr/local/mysql/{data,logs}  #创建日志目录

chown -R mysql.mysql  /usr/local/mysql/   #递归修改所属组和所有者

/usr/local/mysql/scripts/mysql_install_db --defaults-file=/usr/local/mysql/my.cnf --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --pid-file=/usr/local/mysql/data/mysql.pid  --user=mysql  #初始化

启动脚本下载下面压缩文件:
my.cnf  #放在/usr/local/mysql/下

mysqld  #放在/etc/init.d/  执行:chmod +x /etc/init.d/mysqld

chkconfig  mysqld on #设置开机启动

/etc/init.d/mysqld  start #启动mysql
```
---
## mysql 5.7 source install
```
源码下载地址:https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.24.tar.gz
            https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-boost-5.7.24.tar.gz

第一步:安装依赖
yum -y install ncurses-devel cmake make ncurses gcc gcc-c++ ncurses ncurses-devel bison libgcrypt perl #编译需要软件包
yum -y remove mariadb-libs  #卸载默认依赖包，不卸载安装报错

第二步:创建用户和指定组
groupadd -g 500 mysql;useradd -g mysql -u 500 mysql

第三步:创建目录和配置权限
mkdir -p  /usr/local/mysql/{data,logs} && touch  /usr/local/mysql/logs/error.log  &&  chown -R mysql.mysql  /usr/local/mysql/ 

第四步:下载tar包和解压
[boot链接](https://sourceforge.net/projects/boost/files/boost/1.59.0/boost_1_59_0.tar.gz/download)
[mysql5.7]()
boost_1_59_0.tar.gz  和  mysql-5.7.21.tar.gz
tar  -zxvf   boost_1_59_0.tar.gz -C  /usr/local/boost_1_59_0
mv  /usr/local/boost_1_59_0  /usr/local/boost
tar  -zxvf   mysql-5.7.21.tar.gz

第五步:配置
cd  mysql-5.7.24

cmake  -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_DATADIR=/usr/local/mysql/data -DMYSQL_TCP_PORT=3306 -DMYSQL_UNIX_ADDR=/usr/local/mysql/mysqld.sock -DWITH_BOOST=/usr/local/boost -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci -DWITH_EXTRA_CHARSETS=all -DWITH_MYISAM_STORAGE_ENGINE=1 -DWITH_INNOBASE_STORAGE_ENGINE=1 -DENABLED_LOCAL_INFILE=1 -DEXTRA_CHARSETS=all -DWITH_EMBEDDED_SERVER=1 -DWITH_SSL=bundled -DWITH_DEBUG=0 -DENABLE_DOWNLOADS=1 -DDOWNLOAD_BOOST=1 -DWITH_BOOST=/usr/local/boost

#拷贝自己my.cnf文件到指定目录(我这里指定到/usr/local/mysql/)
chmod 644 /usr/local/mysql/my.cnf

第六步:编译和安装
make && make  install

第七步:初始化mysql
/usr/local/mysql/bin/mysqld --defaults-file=/usr/local/mysql/my.cnf --initialize  --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --pid-file=/usr/local/mysql/data/mysql.pid  --user=mysql

cp -rp  /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql  #拷贝mysql启动文件

#添加mysql环境变量
echo "export PATH=$PATH:/usr/local/mysql/bin">>/etc/profile
source /etc/profile

/etc/init.d/mysql start  #启动mysql
说明:mysql初始化密码在error.log里面查找，首次登录以后修改密码
mysql>set password=password("新密码"); #修改数据库初始密码
mysql>flush privileges; #刷新数据库
```
---
## mysql 命令
```
##mysql排序
SELECT * FROM tbl_area_new  ORDER  BY id DESC, pid DESC   #倒叙（可以排序多个字段）
SELECT * FROM tbl_area_new  ORDER  BY id ASC    #正向排序 

##mysql导出命令（csv格式）
mysql> SELECT * FROM tbl_area_new_20180929 into outfile '/tmp/test15.csv' character set utf8  fields terminated by ',' optionally enclosed by '"'  escaped by '"' lines terminated by '\r\n';  #导出数据

mysql>load data infile '/tmp/12.csv' into table tbl_area_new_20180929 character set utf8 fields terminated by ',' optionally enclosed by '"'  escaped by '"' lines terminated by '\n';   #导入数据

##mysql 模糊查询
SELECT * FROM tbl_area_new_20180930 WHERE NAMES LIKE "台湾省%" ；   #模糊查询

##mysql or，and和MAX，MIN使用
SELECT * FROM tbl_area_new_20180928 WHERE NAMES="香港特别行政区" OR NAMES="台湾省" OR NAMES="澳门特别行政区"；

SELECT MAX(id) FROM tbl_area_new  #差某个字段最大值

#在脚本中判断表存在删除
$MySQL_CLI -h${mysql_host} -u $mysql_user -p$mysql_pass -D$mysql_db  -e"drop table if exists ${table}_${project}_${dt}"

#登录mysql数据库删除表
mysql>drop table if exists `表名`；

#表不存在创建表
mysql>create table if not exists order_goods (like ebsig_crm.order_goods);  #跨库复制表结构

#查看表中字段的字符集
mysql>show full columns from 表名;
     
#查看表的字符集
mysql>show table status  from  库名  like "表名";

#查看库的字符集
mysql>show variables like "%char%";

mysql>set names utf8;  #设置字符集，包括（client,connection,results）

#修改数据库字符编码
mysql> alter database mydb character set utf8 ;

#创建数据库时，指定数据库的字符编码
mysql> create database mydb character set utf8 ;

#获取数据表的完整结构:
mysql>show create table 表名\G;

#服务器版本信息
mysql>SELECT VERSION();

#当前数据库名
mysql>SELECT DATABASE(); (或者返回空)

#当前用户名
mysql>SELECT USER();

#服务器状态
mysql>SHOW STATUS;

#服务器配置变量
mysql>SHOW VARIABLES;

[client]下面，加上default-character-set=utf8，或者character_set_client=utf8
[mysqld]下面，加上character_set_server = utf8 ；
因为以上配置，mysql默认是latin1，如果仅仅是通过命令行客户端，mysql重启之后就不起作用了。
如下是客户端命令行修改方式，不推荐使用
mysql> set character_set_client=utf8 ;
mysql> set character_set_connection=utf8 ;
mysql> set character_set_database=utf8 ;
mysql> set character_set_database=utf8 ;
mysql> set character_set_results=utf8 ;
mysql> set character_set_server=utf8 ;
mysql> set character_set_system=utf8 ;
mysql> show variables like 'character%';

#复制表结构及数据到新表
CREATE TABLE 新表 SELECT * FROM 旧表 
这种方法会将oldtable中所有的内容都拷贝过来，当然我们可以用delete from newtable;来删除。 
不过这种方法的一个最不好的地方就是新表中没有了旧表的primary key、Extra（auto_increment）等属性。需要自己用&quot;alter&quot;添加，而且容易搞错。 

#只复制表结构到新表 
CREATE TABLE 新表 SELECT * FROM 旧表 WHERE #=# 
或CREATE TABLE 新表  LIKE 旧表 

#复制旧表的数据到新表(假设两个表结构一样) 
INSERT INTO 新表 SELECT * FROM 旧表 

#复制旧表的数据到新表(假设两个表结构不一样) 
INSERT INTO 新表(字段#,字段#,.......) SELECT 字段#,字段#,...... FROM 旧表 

#可以将表#结构复制到表# 
SELECT * INTO 表# FROM 表# WHERE #=# 

#可以将表#内容全部复制到表# 
SELECT * INTO 表# FROM 表# 

#show create table 旧表; 
这样会将旧表的创建命令列出。我们只需要将该命令拷贝出来，更改table的名字，就可以建立一个完全一样的表 

#mysqldump 
用mysqldump将表dump出来，改名字后再导回去或者直接在命令行中运行
```
####更新插入命令
```
INSERT INTO cust_master SELECT * FROM tmp_cust_master ON DUPLICATE KEY UPDATE updated_at=VALUES(updated_at),creator=VALUES(creator),created_at=VALUES(created_at),cust_id=VALUES(cust_id),cust_name=VALUES(cust_name),email=VALUES(email),rank_name=VALUES(rank_name),sum_amt=VALUES(sum_amt),IP_port=VALUES(IP_port),last_login=VALUES(last_login),upgrade_date=VALUES(upgrade_date),sexy=VALUES(sexy),pwd=VALUES(pwd),rank_id=VALUES(rank_id),birthday=VALUES(birthday),phone=VALUES(phone),mobile=VALUES(mobile),nick_name=VALUES(nick_name),use_fly=VALUES(use_fly),head_pic=VALUES(head_pic),cust_type=VALUES(cust_type),provinceid=VALUES(provinceid),province_name=VALUES(province_name),cityid=VALUES(cityid),city_name=VALUES(city_name),countyid=VALUES(countyid),county_name=VALUES(county_name),address=VALUES(address),QQ=VALUES(QQ),identity_card=VALUES(identity_card),marital_status=VALUES(marital_status),monthlyIncome=VALUES(monthlyIncome),pay_pwd=VALUES(pay_pwd),referee=VALUES(referee),card_no=VALUES(card_no),offline_pcustID=VALUES(offline_pcustID),mall_id=VALUES(mall_id),mall_code=VALUES(mall_code),mall_name=VALUES(mall_name),marry_day=VALUES(marry_day),open_id=VALUES(open_id)

#cust_master 是正式表
#tmp_cust_master #是临时表导入数据，利用上边的命令更新和插入数据

truncate table  表名；   #删除表中的数据，但是不能单独删除某个字段的值

delete from 表名；  #删除表中的数据，可以删除单独某条数据，要用where条件才行
```
---
## mysql安全规范操作
```
1.账号
以普通帐户安全运行mysqld，禁止mysql以root帐号权限运行，攻击者可能通过mysql获得系统root超级用户权限，完全控制系统。
配置/etc/my.cnf
[mysql.server]
user=mysql

补充操作说明
直接通过本地网络之外的计算机改变生产环境中的数据库是异常危险的。有时，管理员会打开主机对数据库的访问：
> GRANT ALL ON *.* TO 'root'@'%';
这其实是完全放开了对root的访问。所以，把重要的操作限制给特定主机非常重要：
> GRANT ALL ON *.* TO 'root'@'localhost';
> GRANT ALL ON *.* TO 'root'@'myip.athome' ;
> FLUSH PRIVILEGES;

判定条件
禁止以root账号运行mysqld;

检测操作
检查进程属主和运行参数是否包含--user=mysql类似语句：

# ps –ef | grep mysqld
#grep -i user /etc/my.cnf

用户权限
应按照用户分配账号，避免不同用户间共享账号

创建用户 设定指定ip地址登陆数据库
create user vvera@'指定ip地址'   identified by 'vv@122'；
这样就创建了一个名为：vvera 密码为：vv@122 的用户。
然后登录一下。

检测方法
判定条件
不用名称的用户可以连接数据库；使用不同用户连接数据库

应删除或锁定与数据库运行、维护等工作无关的账号
移除匿名账户和废弃的账户
DROP USER语句用于删除一个或多个MySQL账户。要使用DROP USER，必须拥有mysql数据库的全局CREATE USER权限或DELETE权限。账户名称的用户和主机部分与用户表记录的User和Host列值相对应。
使用DROP USER，您可以取消一个账户和其权限，操作如下：

DROP USER user;
该语句可以删除来自所有授权表的帐户权限记录。红色标识的无用账户都可以删除。

使用操作命令之后的结果
drop user ''@'mysql',''@'localhost','root'@'::1','root'@'mysql';

补充操作说明
要点：
DROP USER不能自动关闭任何打开的用户对话。而且，如果用户有打开的对话，此时取消用户，则命令不会生效，直到用户对话被关闭后才生效。一旦对话被关闭，用户也被取消，此用户再次试图登录时将会失败。
检侧操作：
mysql 查看所有用户的语句
输入指令

select user();
select user ,host ,password from mysql.user;
tool-manager

依次检查所列出的账户是否为必要账户，删除无用户或过期账户。

2.口令
检查帐户默认密码和弱密码
修改帐户弱密码
如要修改密码，执行如下命令：
检查本地密码：(注意，管理帐号root默认是空密码)

mysql> update user set password=password('vv@122') where user='root';
mysql> flush privileges;
检测方法
mysql> use mysql;
mysql> select Host,User,Password,Select_priv,Grant_priv from user;

3.权限设置
在数据库权限配置能力内，根据用户的业务需要，配置其所需的最小权限。

合理设置用户权限
补充操作说明
有些应用程序是通过一个特定数据库表的用户名和口令连接到MySQL的，安全人员不应当给予这个用户完全的访问权。
如果攻击者获得了这个拥有完全访问权的用户，他也就拥有了所有的数据库。查看一个用户许可的方法是在MySQL控制台中使用命令SHOW GRANT

>SHOW GRANTS FOR ; 'vvera'@'localhost'
为定义用户的访问权，使用GRANT命令。在下面的例子中，vvera仅能从tanggula数据库的mserver表中选择：
> GRANT SELECT ON tanggula. mserver TO 'vvera'@'localhost';
> FLUSH PRIVILEGES;
vvera用户就无法改变数据库中这个表和其它表的任何数据。
如果你要从一个用户移除访问权，就应使用一个与GRANT命令类似的REVOKE命令：
> REVOKE SELECT ON tanggula. mserver FROM 'vvera'@'localhost';
> FLUSH PRIVILEGES;
权限	权限范围	给谁授权	权限范围
grant all	ON .	to vvera	授权vvera全库权限
grant select	ON tanggula.*	to vvera	授权vvera唐古拉数据库查看权限
grant create	ON tanggula.*	to vvera	授权vvera唐古拉数据库添加权限
授权并创建用户，并指定密码

grant 权限 on 权限范围  to 用户 identified by '密码'
回收权限
revoke 权限 on 范围 from 用户

4.日志审计
数据库应配置日志功能，
show variables like 'log_%';查看所有的log命令 
show variables like 'log_bin';查看具体的log命令

5.禁用或限制远程访问
禁止网络连接，防止猜解密码攻击，溢出攻击和嗅探攻击。（仅限于应用和数据库在同一台主机）

参考配置操作
如果数据库不需远程访问，可以禁止远程tcp/ip连接, 通过在mysqld服务器中参数中添加 --skip-networking 启动参数来使mysql不监听任何TCP/IP连接，增加安全性。强迫MySQL仅监听本机，方法是在my.cnf的[mysqld]部分增加下面一行：bind-address=127.0.0.1

6.移除测试（test）数据库和禁用LOCAL INFILE
删除可以匿名访问的test数据库和防止非授权用户访问本地文件

移除测试（test）数据库
在默认安装的MySQL中，匿名用户可以访问test数据库。我们可以移除任何无用的数据库，以避免在不可预料的情况下访问了数据库。因而，在MySQL控制台中，执行：
> DROP DATABASE test;

禁用LOCAL INFILE
另一项改变是禁用”LOAD DATA LOCAL INFILE”命令，这有助于防止非授权用户访问本地文件。在PHP应用程序中发现有新的SQL注入漏洞时，这样做尤其重要。此外，在某些情况下，LOCAL INFILE命令可被用于访问操作系统上的其它文件(如/etc/passwd)，应使用下现的命令：

mysql> SELECT load_file("/etc/passwd")

为禁用LOCAL INFILE命令，应当在MySQL配置文件的[mysqld]部分增加下面的参数：
set-variable=local-infile=0

检查操作
Mysql>show databases;
```
---
## mysql数据库安全方面
```
1、避免从互联网访问MySQL数据库，确保特定主机才拥有访问特权
直接通过本地网络之外的计算机改变生产环境中的数据库是异常危险的。有时，管理员会打开主机对数据库的访问：

> GRANT ALL ON *.* TO ';root'@'%'
这其实是完全放开了对root的访问。所以，把重要的操作限制给特定主机非常重要：

> GRANT ALL ON *.* TO 'root'@'localhost';
> GRANT ALL ON *.* TO 'root'@'myip.athome'
> FLUSH PRIVILEGES
此时，你仍有完全的访问，但只有指定的IP(不管其是否静态)可以访问。

2、定期备份数据库
任何系统都有可能发生灾难。服务器、MySQL也会崩溃，也有可能遭受入侵，数据有可能被删除。只有为最糟糕的情况做好了充分的准备，才能够在事后快速地从灾难中恢复。企业最好把备份过程作为服务器的一项日常工作。

3、禁用或限制远程访问
前面说过，如果使用了远程访问，要确保只有定义的主机才可以访问服务器。这一般是通过TCP wrappers、iptables或任何其它的防火墙软件或硬件实现的。
为限制打开网络socket，管理员应当在my.cnf或my.ini的[mysqld]部分增加下面的参数：

skip-networking
这些文件位于windows的C:\Program Files\MySQL\MySQL Server 5.1文件夹中，或在Linux中，my.cnf位于/etc/，或位于/etc/mysql/。这行命令在MySQL启动期间，禁用了网络连接的初始化。请注意，在这里仍可以建立与MySQL服务器的本地连接。
另一个可行的方案是，强迫MySQL仅监听本机，方法是在my.cnf的[mysqld]部分增加下面一行：

bind-address=127.0.0.1
如果企业的用户从自己的机器连接到服务器或安装到另一台机器上的web服务器，你可能不太愿意禁用网络访问。此时，不妨考虑下面的有限许可访问：

mysql> GRANT SELECT, INSERT ON mydb.* TO ';someuser'@'somehost'
这里，你要把someuser换成用户名，把somehost换成相应的主机。

4、设置root用户的口令并改变其登录名
在linux中，root用户拥有对所有数据库的完全访问权。因而，在Linux的安装过程中，一定要设置root口令。当然，要改变默认的空口令，其方法如下：
Access MySQL控制台：

$ mysql -u root -p
在MySQL控制台中执行：

> SET PASSWORD FOR 'root'@'localhost' = PASSWORD('new_password');
在实际操作中，只需将上面一行的new_password换成实际的口令即可。
在Linux控制台中更改root口令的另一种方法是使用mysqladmin工具：

$ mysqladmin -u root password new_password
此时，也是将上面一行的new_password换成实际的口令即可。
当然，这是需要使用强口令来避免强力攻击。
为了更有效地改进root用户的安全性，另一种好方法是为其改名。为此，你必须更新表用户中的mySQL数据库。在MySQL控制台中进行操作：

> USE mysql;
> UPDATE user SET user="another_username" WHERE user="root";
> FLUSH PRIVILEGES;
然后，通过Linux访问MySQL控制台就要使用新用户名了：

$ mysql -u another_username -p
5、移除测试(test)数据库
在默认安装的MySQL中，匿名用户可以访问test数据库。我们可以移除任何无用的数据库，以避免在不可预料的情况下访问了数据库。因而，在MySQL控制台中，执行：

> DROP DATABASE test;
6、禁用LOCAL INFILE
另一项改变是禁用”LOAD DATA LOCAL INFILE”命令，这有助于防止非授权用户访问本地文件。在PHP应用程序中发现有新的SQL注入漏洞时，这样做尤其重要。
此外，在某些情况下，LOCAL INFILE命令可被用于访问操作系统上的其它文件(如/etc/passwd)，应使用下现的命令：

mysql> LOAD DATA LOCAL INFILE '/etc/passwd' INTO TABLE table1
更简单的方法是：

mysql> SELECT load_file("/etc/passwd")
为禁用LOCAL INFILE命令，应当在MySQL配置文件的[mysqld]部分增加下面的参数：

set-variable=local-infile=0
7、移除匿名账户和废弃的账户
有些MySQL数据库的匿名用户的口令为空。因而，任何人都可以连接到这些数据库。可以用下面的命令进行检查：

mysql> select * from mysql.user where user="";
在安全的系统中，不会返回什么信息。另一种方法是：

mysql> SHOW GRANTS FOR ''@'localhost';
mysql> SHOW GRANTS FOR ';'@'myhost'
如果grants存在，那么任何人都可以访问数据库，至少可以使用默认的数据库“test”。其检查方法如下：

shell> mysql -u blablabla
如果要移除账户，则执行命令：

mysql> DROP USER "";
从MySQL的5.0版开始支持DROP USER命令。如果你使用的老版本的MySQL，你可以像下面这样移除账户：

mysql> use mysql;
mysql> DELETE FROM user WHERE user="";
mysql> flush privileges;
8、降低系统特权
常见的数据库安全建议都有“降低给各方的特权”这一说法。对于MySQL也是如此。一般情况下，开发人员会使用最大的许可，不像安全管理一样考虑许可原则，而这样做会将数据库暴露在巨大的风险中。
为保护数据库，务必保证真正存储MySQL数据库的文件目录是由”mysql” 用户和” mysql”组所拥有的。

shell>ls -l /var/lib/mysql
此外，还要确保仅有用户”mysql”和root用户可以访问/var/lib/mysql目录。
Mysql的二进制文件存在于/usr/bin/目录中，它应当由root用户或特定的”mysql”用户所拥有。对这些文件，其它用户不应当拥有“写”的访问权：

shell>ls -l /usr/bin/my*
9、降低用户的数据库特权
有些应用程序是通过一个特定数据库表的用户名和口令连接到MySQL的，安全人员不应当给予这个用户完全的访问权。
如果攻击者获得了这个拥有完全访问权的用户，他也就拥有了所有的数据库。查看一个用户许可的方法是在MySQL控制台中使用命令SHOW GRANT

>SHOW GRANTS FOR ';user'@'localhost'
为定义用户的访问权，使用GRANT命令。在下面的例子中，user1仅能从dianshang数据库的billing表中选择：

> GRANT SELECT ON billing.dianshang TO 'user1'@'localhost';
> FLUSH PRIVILEGES;
如此一来，user1用户就无法改变数据库中这个表和其它表的任何数据。
另一方面，如果你要从一个用户移除访问权，就应使用一个与GRANT命令类似的REVOKE命令：

> REVOKE SELECT ON billing.ecommerce FROM 'user1'@'localhost';
> FLUSH PRIVILEGES;
10、移除和禁用.mysql_history文件
在用户访问MySQL控制台时，所有的命令历史都被记录在~/.mysql_history中。如果攻击者访问这个文件，他就可以知道数据库的结构。

$ cat ~/.mysql_history
为了移除和禁用这个文件，应将日志发送到/dev/null。

$export MYSQL_HISTFILE=/dev/null
上述命令使所有的日志文件都定向到/dev/null，你应当从home文件夹移除.mysql_history：$ rm ~/.mysql_history，并创建一个到/dev/null的符号链接。

11、安全补丁
务必保持数据库为最新版本。因为攻击者可以利用上一个版本的已知漏洞来访问企业的数据库。

12、启用日志
如果你的数据库服务器并不执行任何查询，建议你启用跟踪记录，你可以通过在/etc/my.cnf文件的[Mysql]部分添加：log =/var/log/mylogfile。
对于生产环境中任务繁重的MySQL数据库，因为这会引起服务器的高昂成本。
此外，还要保证只有root和mysql可以访问这些日志文件。
错误日志
务必确保只有root和mysql可以访问hostname.err日志文件。该文件存放在mysql数据历史中。该文件包含着非常敏感的信息，如口令、地址、表名、存储过程名、代码等，它可被用于信息收集，并且在某些情况下，还可以向攻击者提供利用数据库漏洞的信息。攻击者还可以知道安装数据库的机器或内部的数据。
MySQL日志
确保只有root和mysql可以访问logfileXY日志文件，此文件存放在mysql的历史目录中。

13、改变root目录
Unix操作系统中的chroot可以改变当前正在运行的进程及其子进程的root目录。重新获得另一个目录root权限的程序无法访问或命名此目录之外的文件，此目录被称为“chroot监狱”。
通过利用chroot环境，你可以限制MySQL进程及其子进程的写操作，增加服务器的安全性。
你要保证chroot环境的一个专用目录，如/chroot/mysql。此外，为了方便利用数据库的管理工具，你可以在MySQL配置文件的[client]部分改变下面的参数：

[client]
socket = /chroot/mysql/tmp/mysql.sock
14、禁用LOCAL INFILE命令
LOAD DATA LOCAL INFILE可以从文件系统中读取文件，并显示在屏幕中或保存在数据库中。如果攻击者能够从应用程序找到SQL注入漏洞，这个命令就相当危险了。下面的命令可以从MySQL控制台进行操作：

> SELECT LOAD_FILE("/etc/passwd");
该命令列示了所有的用户。解决此问题的最佳方法是在MySQL配置中禁用它，在CentOS中找到/etc/my.cnf或在Ubuntu中找到/etc/mysql/my.cnf，在[mysqld]部分增加下面一行：set-variable=local-infile=0。搞定。
当然，唇亡齿寒，保护服务器的安全对于保障MySQL数据库的安全也是至关重要的。服务器的安全对于数据库来说可谓生死攸关
```