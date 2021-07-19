---
title: zabbix 监控SSL证书过期时间
date: 2021-07-19 14:37:00
tags:
    - [ zabbix ]
    - [zabbix监控ssl]
categories:
    - [ zabbix ]
---

## zabbix 监控ssl证书
1.编写获取域名和443端口脚本
```
cat /etc/zabbix/scripts/ssl_discovery.py    
#!/usr/bin/env python
#coding:utf-8

import os
import sys
import json

#这个函数主要是构造出一个特定格式的字典，用于zabbix
def ssl_cert_discovery():
    web_list=[]
    web_dict={"data":None}
    with open("/etc/zabbix/scripts/ssl_cert_list","r") as f:
        for sslcert in f:
            dict={}
            dict["{#DOMAINNAME}"]=sslcert.strip().split()[0]
            dict["{#PORT}"]=sslcert.strip().split()[1]
            web_list.append(dict)
    web_dict["data"]=web_list
    jsonStr = json.dumps(web_dict,indent=4)
    return jsonStr
if __name__ == "__main__":
    print ssl_cert_discovery()
```
2.检查证书过期时间
```
cat /etc/zabbix/scripts/check-ssl-expire.sh
#!/bin/bash
host=$1
port=$2
end_date=`/usr/bin/openssl s_client -servername $host -host $host -port $port -showcerts </dev/null 2>/dev/null |
  sed -n '/BEGIN CERTIFICATE/,/END CERT/p' |
  /usr/bin/openssl  x509 -text 2>/dev/null |
  sed -n 's/ *Not After : *//p'`
# openssl 检验和验证SSL证书。
# -servername $host 因一台主机存在多个证书，利用SNI特性检查
# </dev/null 定向标准输入，防止交互式程序。从/dev/null 读时，直接读出0 。
# sed -n 和p 一起使用，仅显示匹配到的部分。 //,// 区间匹配。
# openssl x509 -text 解码证书信息，包含证书的有效期。

if [ -n "$end_date" ]
then
    end_date_seconds=`date '+%s' --date "$end_date"`
    now_seconds=`date '+%s'`
    echo "($end_date_seconds-$now_seconds)/24/3600" | bc
fi
```
3.域名和端口存放文件
```
cat /etc/zabbix/scripts/ssl_cert_list
test.xxxxxx.com 443
test1.xxxxxx.com 443
```
4.配置zabbix
```
把脚本上传:/etc/zabbix/scripts/下
cat /etc/zabbix/zabbix_agentd.d/sslcert.conf 
UserParameter=sslcert_discovery,/usr/bin/python  /etc/zabbix/scripts/ssl_discovery.py
UserParameter=sslcert.info[*],/bin/bash /etc/zabbix/scripts/check-ssl-expire.sh $1 $2
```
```
重启agent和proxy
systemctl restart zabbix-agent 
systemctl restart zabbix-proxy
测试
/etc/zabbix/scripts/check-ssl-expire.sh api.xxxxxx.com 443
```

![](/images/zabbix/zabbix_agent_ssl.jpg)

5.导入模板(下载链接)
[https://www.aliyundrive.com/s/WFfnahYBeTC]
