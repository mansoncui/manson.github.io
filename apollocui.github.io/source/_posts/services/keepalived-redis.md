---
layout: post
title: keepalived_redis
date: 2019-06-25 13:38:15
tags:
    - [ keepalived, redis ]
categories:
    - [ services ]
password: cbb4693
---
## 部署redis

##### 主redis配置
```
grep -v "^#\|^$" /kuaibao/server/redis/conf/redis.conf  
daemonize yes
bind 0.0.0.0
protected-mode yes
port 6379
tcp-backlog 511
timeout 0
tcp-keepalive 300
pidfile /kuaibao/server/redis/run/redis_6380.pid
loglevel notice
logfile "/kuaibao/server/redis/log/redis.log"
databases 16
save ""
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename dump.rdb
dir /kuaibao/server/redis/data/
slave-serve-stale-data yes
slave-read-only yes
repl-diskless-sync no
repl-diskless-sync-delay 5
repl-disable-tcp-nodelay no
slave-priority 100
maxmemory 6gb
maxmemory-policy volatile-ttl
appendonly no
appendfilename "appendonly.aof"
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 60
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
lua-time-limit 5000
slowlog-log-slower-than 10000
slowlog-max-len 128
latency-monitor-threshold 0
notify-keyspace-events ""
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-size -2
list-compress-depth 0
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
hll-sparse-max-bytes 3000
activerehashing yes
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
hz 10
aof-rewrite-incremental-fsync yes
```

##### 从redis 配置
```
grep -v "^#\|^$" /kuaibao/server/redis/conf/redis.conf 
daemonize yes
bind 0.0.0.0
protected-mode yes
port 6379
tcp-backlog 511
timeout 0
tcp-keepalive 300
pidfile /kuaibao/server/redis/run/redis_6380.pid
loglevel notice
logfile "/kuaibao/server/redis/log/redis.log"
databases 16
save ""
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename dump.rdb
dir /kuaibao/server/redis/data/
slaveof 192.168.1.118 6379
slave-serve-stale-data yes
slave-read-only yes
repl-diskless-sync no
repl-diskless-sync-delay 5
repl-disable-tcp-nodelay no
slave-priority 100
maxmemory 6gb
maxmemory-policy volatile-ttl
appendonly no
appendfilename "appendonly.aof"
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 60
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
lua-time-limit 5000
slowlog-log-slower-than 10000
slowlog-max-len 128
latency-monitor-threshold 0
notify-keyspace-events ""
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-size -2
list-compress-depth 0
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
hll-sparse-max-bytes 3000
activerehashing yes
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
hz 10
aof-rewrite-incremental-fsync yes
```
---

## 安装和配置keepalived

##### master配置
```
yum -y install keepalived
 
cat /etc/keepalived/keepalived.conf  
global_defs { 
	router_id redis 
} 
vrrp_script chk_redis { 
	script "/etc/keepalived/scripts/check_redis.sh" 
	interval 4 
	weight -5 
	fall 3 
	rise 2 
} 
vrrp_instance VI_REDIS { 
	state MASTER 
	interface ens192
	virtual_router_id 51
	priority 100
	advert_int 1
	nopreempt

	authentication { 
		auth_type PASS 
		auth_pass 1111 
	} 
	virtual_ipaddress { 
		192.168.1.111 
	} 
	track_script { 
		chk_redis 
	} 
	notify_master /etc/keepalived/scripts/redis_master.sh 
	notify_backup /etc/keepalived/scripts/redis_backup.sh 
	notify_fault /etc/keepalived/scripts/redis_fault.sh 
	notify_stop /etc/keepalived/scripts/redis_stop.sh 
}
```

##### slave配置
```
cat /etc/keepalived/keepalived.conf  
global_defs { 
	router_id redis 
} 
vrrp_script chk_redis { 
	script "/etc/keepalived/scripts/check_redis.sh" 
	interval 4 
	weight -5 
	fall 3 
	rise 2 
} 
vrrp_instance VI_REDIS { 
	state slave 
	interface eth0
	virtual_router_id 51
	priority 99
	advert_int 1
	nopreempt

	authentication { 
		auth_type PASS 
		auth_pass 1111 
	} 
	virtual_ipaddress { 
		192.168.1.111 
	} 
	track_script { 
		chk_redis 
	} 
	notify_master /etc/keepalived/scripts/redis_master.sh 
	notify_backup /etc/keepalived/scripts/redis_backup.sh 
	notify_fault /etc/keepalived/scripts/redis_fault.sh 
	notify_stop /etc/keepalived/scripts/redis_stop.sh 
}
```
#####  配置master服务器相应脚本
```
mkdir /etc/keepalived/scripts/
```
```
[root@k8s-slave scripts]# cat check_redis.sh 
#!/bin/bash 
CHECK=`/kuaibao/server/redis/bin/redis-cli PING` 
if [ "$CHECK" == "PONG" ] ;then 
	echo $CHECK exit 0 
else 
	echo $CHECK 
	service keepalived stop #可确保让出MASTER 
	exit 1 
fi
```
```
[root@k8s-slave scripts]# cat redis_backup.sh 

#!/bin/bash 
REDISCLI="/kuaibao/server/redis/bin/redis-cli" 
LOGFILE="/kuaibao/server/redis/log/keepalived-redis-state.log" 
echo "[backup]" >> $LOGFILE 
date >> $LOGFILE 
echo "Being slave...." >> $LOGFILE 2>&1 
sleep 15 #延迟15秒待数据被对方同步完成之后再切换主从角色 
echo "Run SLAVEOF cmd ..." >> $LOGFILE 
$REDISCLI SLAVEOF 192.168.1.118 6379 >> $LOGFILE 2>&1
```

```
[root@k8s-slave scripts]# cat redis_fault.sh 
# !/bin/bash 
LOGFILE=/kuaibao/server/redis/log/keepalived-redis-state.log
echo "[fault]" >> $LOGFILE
date >> $LOGFILE
```

```
[root@k8s-slave scripts]# cat redis_master.sh 
#!/bin/bash
REDISCLI="/kuaibao/server/redis/bin/redis-cli" 
LOGFILE="/kuaibao/server/redis/log/keepalived-redis-state.log" 
echo "[master]" >> $LOGFILE 
date >> $LOGFILE 
echo "Being master...." >> $LOGFILE 2>&1 
echo "Run SLAVEOF cmd ..." >> $LOGFILE 
$REDISCLI SLAVEOF 192.168.1.118 6379 >> $LOGFILE 2>&1 
sleep 10 #延迟10秒以后待数据同步完成后再取消同步状态 
echo "Run SLAVEOF NO ONE cmd ..." >> $LOGFILE 
$REDISCLI SLAVEOF NO ONE >> $LOGFILE 2>&1
```

```
[root@k8s-slave scripts]# cat redis_stop.sh 
# !/bin/bash
LOGFILE=/kuaibao/server/redis/log/keepalived-redis-state.log
echo "[stop]" >> $LOGFILE 
date >> $LOGFILE
```
#####  配置slave服务器相应脚本
```
mkdir /etc/keepalived/scripts/
```
```
[root@k8s-slave scripts]# cat check_redis.sh 
#!/bin/bash 
CHECK=`/kuaibao/server/redis/bin/redis-cli PING` 
if [ "$CHECK" == "PONG" ] ;then 
	echo $CHECK exit 0 
else 
	echo $CHECK 
	service keepalived stop #可确保让出MASTER 
	exit 1 
fi
```

```
[root@k8s-slave scripts]# cat redis_backup.sh 
#!/bin/bash 
REDISCLI="/kuaibao/server/redis/bin/redis-cli" 
LOGFILE="/kuaibao/server/redis/log/keepalived-redis-state.log" 
echo "[backup]" >> $LOGFILE 
date >> $LOGFILE 
echo "Being slave...." >> $LOGFILE 2>&1 
sleep 15 #延迟15秒待数据被对方同步完成之后再切换主从角色 
echo "Run SLAVEOF cmd ..." >> $LOGFILE 
$REDISCLI SLAVEOF 192.168.1.118 6379 >> $LOGFILE 2>&1
```

```
[root@k8s-slave scripts]# cat redis_fault.sh 
# !/bin/bash 
LOGFILE=/kuaibao/server/redis/log/keepalived-redis-state.log
echo "[fault]" >> $LOGFILE
date >> $LOGFILE
```

```
[root@k8s-slave scripts]# cat redis_master.sh 
#!/bin/bash
REDISCLI="/kuaibao/server/redis/bin/redis-cli" 
LOGFILE="/kuaibao/server/redis/log/keepalived-redis-state.log" 
echo "[master]" >> $LOGFILE 
date >> $LOGFILE 
echo "Being master...." >> $LOGFILE 2>&1 
echo "Run SLAVEOF cmd ..." >> $LOGFILE 
$REDISCLI SLAVEOF 192.168.1.118 6379 >> $LOGFILE 2>&1 
sleep 10 #延迟10秒以后待数据同步完成后再取消同步状态 
echo "Run SLAVEOF NO ONE cmd ..." >> $LOGFILE 
$REDISCLI SLAVEOF NO ONE >> $LOGFILE 2>&1
```
```
[root@k8s-slave scripts]# cat redis_stop.sh 
# !/bin/bash
LOGFILE=/kuaibao/server/redis/log/keepalived-redis-state.log
echo "[stop]" >> $LOGFILE 
date >> $LOGFILE
```
---
## 备注
```
在master脚本中配置IP是slave的IP地址和端口

在slave脚本中配置IP是master的IP地址和端口

当两台服务器上都出现VIP时，出现脑裂问题
```