---
layout: post
title: zabbix 自动发现IP监控
date: 2021-08-18 17:36:16
tags:
    - [ zabbix ]
categories:
    - [ monitor ]
---
## zabbix和服务器相关配置
### 监控脚本
````
[root@localhost zabbix]# cat /etc/zabbix/scripts/ip_check.sh
#!/bin/bash
# function:monitor tcp connect status from zabbix


web_ip_discovery () {
WEB_IP=($(cat /etc/zabbix/scripts/ip_list.txt | grep -v "^#"))
        printf '{\n'
        printf '\t"data":[\n'
for((i=0;i<${#WEB_IP[@]};++i))
{
num=$(echo $((${#WEB_IP[@]}-1)))
        if [ "$i" != ${num} ];
                then
        printf "\t\t{ \n"
        printf "\t\t\t\"{#SITENAME}\":\"${WEB_IP[$i]}\"},\n"
                else
                        printf  "\t\t{ \n"
                        printf  "\t\t\t\"{#SITENAME}\":\"${WEB_IP[$num]}\"}]}\n"
        fi
}
}

web_site_code () {
ip=`echo $1|awk -F ':' '{print $1}'`
#echo $ip
/usr/sbin/fping ${ip} 2> /dev/null |grep -c 'alive'
}

case "$1" in
web_ip_discovery)
web_ip_discovery
;;
web_site_code)
web_site_code $2
;;
*)

echo "Usage:$0 {web_ip_discovery|web_site_code [URL]}"
;;
esac
````
### zabbix conf配置
````
cat /etc/zabbix/zabbix_agentd.d/ip_check.conf
UserParameter=web.ip.discovery,/etc/zabbix/scripts/ip_check.sh web_ip_discovery
UserParameter=web.ip.code[*],/etc/zabbix/scripts/ip_check.sh web_site_code $1
````
### zabbix 存储IP文件
````
[root@localhost zabbix]# cat /etc/zabbix/scripts/ip_list.txt          
10.144.24.1:6_gateway
10.144.24.2:9_route
````

### zabbix 自动发现模板(导入模板并关联主机)
````
<?xml version="1.0" encoding="UTF-8"?>
<zabbix_export>
    <version>5.0</version>
    <date>2021-08-18T08:15:04Z</date>
    <groups>
        <group>
            <name>Ping</name>
        </group>
    </groups>
    <templates>
        <template>
            <template>Template ICMP Ping</template>
            <name>Template ICMP Ping</name>
            <groups>
                <group>
                    <name>Ping</name>
                </group>
            </groups>
            <applications>
                <application>
                    <name>IP_PING</name>
                </application>
            </applications>
            <discovery_rules>
                <discovery_rule>
                    <name>web.ip.discovery</name>
                    <key>web.ip.discovery</key>
                    <delay>30s</delay>
                    <lifetime>0</lifetime>
                    <item_prototypes>
                        <item_prototype>
                            <name>IP Code ON $1 Alive</name>
                            <key>web.ip.code[{#SITENAME},]</key>
                            <applications>
                                <application>
                                    <name>IP_PING</name>
                                </application>
                            </applications>
                            <trigger_prototypes>
                                <trigger_prototype>
                                    <expression>{max(#3)}&lt;&gt;1</expression>
                                    <name>IP {#SITENAME} host is not alive</name>
                                    <priority>HIGH</priority>
                                </trigger_prototype>
                            </trigger_prototypes>
                        </item_prototype>
                    </item_prototypes>
                    <graph_prototypes>
                        <graph_prototype>
                            <name>IP PING</name>
                            <graph_items>
                                <graph_item>
                                    <sortorder>1</sortorder>
                                    <color>1A7C11</color>
                                    <item>
                                        <host>Template ICMP Ping</host>
                                        <key>web.ip.code[{#SITENAME},]</key>
                                    </item>
                                </graph_item>
                            </graph_items>
                        </graph_prototype>
                    </graph_prototypes>
                </discovery_rule>
            </discovery_rules>
        </template>
    </templates>
</zabbix_export>
````
## 告警相关配置
### 钉钉告警脚本(python3运行)
````
[root@zabbix-server ~]# cat /usr/lib/zabbix/alertscripts/dingding.py
#!/usr/bin/python
# _*_coding: utf-8 _*_
import requests
import json
import sys

url = "https://oapi.dingtalk.com/robot/send?access_token=825207be6f4d5818d78bc192fde043d0a339a5a6ae5e025b13bdef175dc4793b"

def send_msg(msg):
    """
    发送消息的函数，这里使用阿里的钉钉
    :param msg: 要发送的消息
    :return: 200 or False
    """
    program = {"msgtype": "text", "text": {"content": msg}, }
    headers = {'Content-Type': 'application/json'}
    try:
        f = requests.post(url, data=json.dumps(program), headers=headers,verify=False)
    except Exception as e:
        return False
    return f.status_code

def main():
    msg = sys.argv[1]
    send_msg(msg)

if __name__ == '__main__':
    main()
````
### zabbix创建用户和用户组
<img src="/images/monitor/zabbix-group.jpg" width=100% height=10% align=left/>
<img src="/images/monitor/zabbix-user.jpg" width=100% height=10% align=left/>
<img src="/images/monitor/zabbix-user-1.jpg" width=100% height=10% align=left/>
<img src="/images/monitor/zabbix-user-2.jpg" width=100% height=10% align=left/>

### 报警媒介设置
<img src="/images/monitor/zabbix-报警媒介.jpg" width=100% height=10% align=left/>
<img src="/images/monitor/zabbix-报警.jpg" width=100% height=10% align=left/>
### 配置动作设置
<img src="/images/monitor/zabbix-动作.jpg" width=100% height=10% align=left/>
<img src="/images/monitor/zabbix-操作.jpg" width=100% height=10% align=left/>
<img src="/images/monitor/zabbix-操作细节.jpg" width=100% height=10% align=left/>
<img src="/images/monitor/zabbix-恢复操作.jpg" width=100% height=10% align=left/>
