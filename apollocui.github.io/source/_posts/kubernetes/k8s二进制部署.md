---
title: kubernetes 二进制部署
date: 2021-06-28 22:20
tags: 
    - [ kubernetes ]
categories:
    - [ kubernetes ]
---
## kubernetes二进制部署
```
说明:
使用ipvs需要安装相应的工具来处理”yum install ipset ipvsadm -y“
确保 ipvs已经加载内核模块， ip_vs、ip_vs_rr、ip_vs_wrr、ip_vs_sh、
nf_conntrack_ipv4。如果这些内核模块不加载，当kube-proxy启动后，会退回到iptables模式。

##基础环境初始化:
yum -y install vim telnet iotop openssh-clients openssh-server ntp net-tools.x86_64 wget

sed -i '/*          soft    nproc     4096/d' /etc/security/limits.d/20-nproc.conf
echo '*  -  nofile  65536' >> /etc/security/limits.conf
echo '*       soft    nofile  65535' >> /etc/security/limits.conf
echo '*       hard    nofile  65535' >> /etc/security/limits.conf
echo 'fs.file-max = 65536' >> /etc/sysctl.conf
ssh-keygen -t rsa
ssh-copy-id -p52000 -i /root/.ssh/id_rsa.pub root@192.168.3.136 
ssh-copy-id -p52000 -i /root/.ssh/id_rsa.pub root@192.168.3.137

1.部署etcd
cd /home/k8s_install/Deploy/ssl_etcd
chmod +x cfssl.sh
 ./cfssl.sh

cat cfssl.sh 
curl -L https://pkg.cfssl.org/R1.2/cfssl_linux-amd64 -o /usr/local/bin/cfssl
curl -L https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64 -o /usr/local/bin/cfssljson
curl -L https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64 -o /usr/local/bin/cfssl-certinfo
chmod +x /usr/local/bin/cfssl /usr/local/bin/cfssljson /usr/local/bin/cfssl-certinfo

mkdir /home/k8s_install/ssl_etcd/ && cd /home/k8s_install/ssl_etcd/


mkdir /home/k8s_install/etcd/{cfg,bin,ssl} -p
cp {ca,server-key,server}.pem /home/k8s_install/etcd/ssl/

wget https://github.com/etcd-io/etcd/releases/download/v3.2.12/etcd-v3.2.12-linux-amd64.tar.gz

tar -zxvf etcd-v3.2.12-linux-amd64.tar.gz

cp -p  etcd-v3.2.12-linux-amd64/etcd* /home/k8s_install/etcd/bin/

[root@k8s-dev-master bin]# cat /usr/lib/systemd/system/etcd.service   
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
Type=notify
EnvironmentFile=/home/k8s_install/etcd/cfg/etcd
ExecStart=/home/k8s_install/etcd/bin/etcd \
--name=${ETCD_NAME} \
--data-dir=${ETCD_DATA_DIR} \
--listen-peer-urls=${ETCD_LISTEN_PEER_URLS} \
--listen-client-urls=${ETCD_LISTEN_CLIENT_URLS},http://127.0.0.1:2379 \
--advertise-client-urls=${ETCD_ADVERTISE_CLIENT_URLS} \
--initial-advertise-peer-urls=${ETCD_INITIAL_ADVERTISE_PEER_URLS} \
--initial-cluster=${ETCD_INITIAL_CLUSTER} \
--initial-cluster-token=${ETCD_INITIAL_CLUSTER_TOKEN} \
--initial-cluster-state=new \
--cert-file=/home/k8s_install/etcd/ssl/server.pem \
--key-file=/home/k8s_install/etcd/ssl/server-key.pem \
--peer-cert-file=/home/k8s_install/etcd/ssl/server.pem \
--peer-key-file=/home/k8s_install/etcd/ssl/server-key.pem \
--trusted-ca-file=/home/k8s_install/etcd/ssl/ca.pem \
--peer-trusted-ca-file=/home/k8s_install/etcd/ssl/ca.pem
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target


[root@k8s-dev-master bin]# cat /home/k8s_install/
etcd/     ssl_etcd/ 
[root@k8s-dev-master bin]# cat /home/k8s_install/etcd/
bin/ cfg/ ssl/ 
[root@k8s-dev-master bin]# cat /home/k8s_install/etcd/cfg/etcd 
#[Member]
ETCD_NAME="etcd01"
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_PEER_URLS="https://192.168.3.135:2380"
ETCD_LISTEN_CLIENT_URLS="https://192.168.3.135:2379"

#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://192.168.3.135:2380"
ETCD_ADVERTISE_CLIENT_URLS="https://192.168.3.135:2379"
ETCD_INITIAL_CLUSTER="etcd01=https://192.168.3.135:2380,etcd02=https://192.168.3.136:2380,etcd03=https://192.168.3.137:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER_STATE="new"




##同步到其它etcd节点
scp -rp -P52000 /home/k8s_install/etcd/{bin,cfg,ssl} root@192.168.3.136:/home/k8s_install/etcd/
scp -rp -P52000 /home/k8s_install/etcd/{bin,cfg,ssl} root@192.168.3.137:/home/k8s_install/etcd/

scp -rp -P52000 /usr/lib/systemd/system/etcd.service root@192.168.3.136:/usr/lib/systemd/system/
scp -rp -P52000 /usr/lib/systemd/system/etcd.service root@192.168.3.137:/usr/lib/systemd/system/

```
