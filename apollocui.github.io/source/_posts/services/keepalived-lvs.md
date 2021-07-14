---
layout: post
title: keepalived-lvs
date: 2019-07-08 13:17:45
tags:
    - [ keepalived, lvs ]
categories:
    - [ services ]
---

## keepalived + lvs 
##### 主和从上安装keepalived
```
yum -y install keepalived 
```
##### keepalived.conf 配置(主)
```
global_defs {                       
   notification_email {             
   }
      
    smtp_connect_timeout 30
        router_id LVS_DEVEL             
}
vrrp_instance VI_1 {            
        state MASTER     #配置LVS是主机的状态        
        interface eth0     #配置LVS机器对外开放的IP       
        virtual_router_id 51        
        priority 100                  
        advert_int 1           
        authentication {        
                auth_type PASS
                auth_pass 1111
        }
        virtual_ipaddress {         
                192.168.1.121    #LVS的对内IP
        }
}
virtual_server 192.168.1.121 80 {
        delay_loop 6           
        lb_algo wrr            
        lb_kind DR         #使用LVSDR模式                 
        nat_mask 255.255.255.0   
        persistence_timeout 0    
        protocol TCP                          
        real_server 192.168.1.101 80 {    #真实服务的IP 
                weight 1        #配置加权轮询的权重             
                TCP_CHECK {                     
                        connect_timeout 10   
                        nb_get_retry 3
                        delay_before_retry 3
                        connect_port 80
                }
        }
        real_server 192.168.1.89 80 {
                weight 2
                TCP_CHECK {
                        connect_timeout 10
                        nb_get_retry 3
                        delay_before_retry 3
                        connect_port 80
                }
        }
}
```

##### keepalived.conf 配置(从)
```
global_defs {                       
#   notification_email {             
#   }
#   smtp_connect_timeout 30
        router_id LVS_DEVEL             
}
vrrp_instance VI_1 {            
        state BACKUP     #配置LVS是主机的状态        
        interface ens192     #配置LVS机器对外开放的IP       
        virtual_router_id 51        
        priority 100                  
        advert_int 1           
        authentication {        
                auth_type PASS
                auth_pass 1111
        }
        virtual_ipaddress {         
                192.168.1.121    #LVS的对内IP
        }
}
virtual_server 192.168.1.121 80 {
        delay_loop 6           
        lb_algo wrr            
        lb_kind DR         #使用LVSDR模式                 
        nat_mask 255.255.255.0   
        persistence_timeout 0    
        protocol TCP                          
        real_server 192.168.1.101 80 {    #真实服务的IP 
                weight 1        #配置加权轮询的权重             
                TCP_CHECK {                     
                        connect_timeout 10   
                        nb_get_retry 3
                        delay_before_retry 3
                        connect_port 80
                }
        }
        real_server 192.168.1.89 80 {
                weight 2
                TCP_CHECK {
                        connect_timeout 10
                        nb_get_retry 3
                        delay_before_retry 3
                        connect_port 80
                }
        }
}
```
```
service keepalived start 

观察系统日志
```
##### 两台 real server 部署一下脚本(放到/etc/init.d/下 )
```
#!/bin/bash  
#description : start realserver  
SNS_VIP=192.168.1.121 #定义了一个VIP变量，必须跟真是服务在一个网段
source /etc/rc.d/init.d/functions  
case "$1" in  
start)  
echo " start LVS of REALServer"  
/sbin/ifconfig lo:0 $SNS_VIP broadcast $SNS_VIP netmask 255.255.255.255 up  #增加一个本地路由 lo:0
echo "1" >/proc/sys/net/ipv4/conf/lo/arp_ignore  
echo "2" >/proc/sys/net/ipv4/conf/lo/arp_announce  
echo "1" >/proc/sys/net/ipv4/conf/all/arp_ignore  
echo "2" >/proc/sys/net/ipv4/conf/all/arp_announce  
;;  
stop)  
/sbin/ifconfig lo:0 down  
echo "close LVS Directorserver"  
echo "0" >/proc/sys/net/ipv4/conf/lo/arp_ignore  
echo "0" >/proc/sys/net/ipv4/conf/lo/arp_announce  
echo "0" >/proc/sys/net/ipv4/conf/all/arp_ignore  
echo "0" >/proc/sys/net/ipv4/conf/all/arp_announce  
;;  
*)  
echo "Usage: $0 {start|stop}"  
exit 1  
esac
```
```
chmod 777 /etc/init.d/realserver

/etc/init.d/realserver start 
```

##### 两台真是服务器安装nginx
```
yum -y install nginx 

把两台相应IP写到index.html 文件中(有标识行)
```

##### 测试
```
关闭一台keepalived 

浏览器访问
https://192.168.1.121/?a=Math.random()
```