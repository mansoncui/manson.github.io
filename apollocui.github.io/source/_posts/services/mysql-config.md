---
layout: post
title: mysql_config
date: 2019-06-20 10:53:04
tags:
    - [ mysql ]
categories:
    - [ services ]
password: cbb4693
---

## mysql5.6和5.7 my.cnf配置
```
[client]
port=3306
socket=/usr/local/mysql/mysql.sock
default-character-set=utf8
password="Thankyou123"

[mysqld]
 basedir = /usr/local/mysql
 datadir = /usr/local/mysql/data
 port = 3306
 server_id = 2
 socket = /usr/local/mysql/mysqld.sock
 pid-file = /usr/local/mysql/mysql.pid
 max_allowed_packet=16M
 log-error	 = /usr/local/mysql/logs/error.log

#sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
sql_mode=""

#bind-address=127.0.0.1
skip-name-resolve
interactive_timeout=300
open_files_limit=65535
skip-external-locking

#*** network ***
back_log = 512
#skip-networking #默认没有开启
max_connections = 3000
max_connect_errors = 30
table_open_cache = 8192
#external-locking #默认没有开启
max_allowed_packet = 32M
max_heap_table_size = 128M

# *** global cache ***
read_buffer_size = 8M
read_rnd_buffer_size = 64M
sort_buffer_size = 16M
join_buffer_size = 16M

# *** thread ***
thread_cache_size = 16
thread_concurrency = 8
thread_stack = 512K

# *** query  cache ***
#query_cache_type=1
#query_cache_size = 128M
#query_cache_limit = 4M

# *** index ***
ft_min_word_len = 8

#memlock #默认没有开启
default-storage-engine = MyISAM
transaction_isolation = REPEATABLE-READ
                                                                                                                                                                             
# *** tmp table ***
tmp_table_size = 128M

# *** bin log ***
#log-bin=/data/3306/logs/mysql-binlog
#binlog_cache_size = 4M
#binlog_format=mixed
#log_slave_updates #默认没有开启
#log #默认没有开启，此处是查询日志，开启会影响服务器性能
log_warnings#开启警告日志
slow_query_log = ON
slow_launch_time = 2
long_query_time = 2
slow-query-log
slow_query_log_file = /usr/local/mysql/logs/slow-log
expire_logs_days= 7
#max_binlog_size = 500M

#*** MyISAM Specific options
key_buffer_size = 56M
bulk_insert_buffer_size = 256M
myisam_sort_buffer_size = 256M
myisam_max_sort_file_size = 10G
myisam_repair_threads = 1
myisam_recover=BACKUP,FORCE
delay_key_write=1

# *** INNODB Specific options ***
#skip-innodb #默认没有开启
innodb_additional_mem_pool_size = 64M
innodb_buffer_pool_size = 4G #注意在32位系统上你每个进程可能被限制在 2-3.5G 用户层面内存限制, 所以不要设置的太高.
innodb_data_file_path = ibdata1:10M:autoextend
#innodb_data_home_dir = <directory>
innodb_write_io_threads = 8
innodb_read_io_threads = 8
#innodb_force_recovery=1
innodb_thread_concurrency = 16
innodb_flush_log_at_trx_commit = 2
#innodb_fast_shutdown
innodb_log_buffer_size = 16M
innodb_log_file_size = 4096M
innodb_log_files_in_group = 3
#innodb_log_group_home_dir
innodb_max_dirty_pages_pct = 90
#innodb_flush_method=O_DSYNC
innodb_lock_wait_timeout = 120

[mysqldump]
quick
max_allowed_packet = 32M
[mysql]
no-auto-rehash
[myisamchk]
sort_buffer_size = 2048M
read_buffer = 32M
write_buffer = 32M
[mysqlhotcopy]
interactive-timeout
[mysqld_safe]
open-files-limit = 102400
```