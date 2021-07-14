---
title: kubernetes
date: 2019-06-11 16:42:04
tags: 
    - [ kubernetes ]
categories:
    - [ kubernetes ]
---
## kubernetes 工作原理
```
1、Master节点包括API Server、Scheduler、Controller manager、etcd

API Server是整个系统的对外接口，供客户端和其它组件调用，相当于“营业厅”
Scheduler负责对集群内部的资源进行调度，相当于“调度室”
Controller manager负责管理控制器，相当于“大总管

2、Node节点包括Docker、kubelet、kube-proxy、Fluentd、kube-dns（可选）、pod

Pod是Kubernetes最基本的操作单元。一个Pod代表着集群中运行的一个进程，它内部封装了一个或多个紧密相关的容器。除了Pod之外，K8S还有一个Service的概念，一个Service可以看作一组提供相同服务的Pod的对外访问接口

Docker 创建容器的
Kubelet，主要负责监视指派到它所在Node上的Pod，包括创建、修改、监控、删除等
Kube-proxy，主要负责为Pod对象提供代理
Fluentd，主要负责日志收集、存储与查询
```
---
## kubernetes kubeadm 部署
```
[镜像地址](https://download.csdn.net/download/qq_33432713/10775173) :https://download.csdn.net/download/qq_33432713/10775173

系统环境:CentOS Linux release 7.3.1611 (Core)

1、关闭selinux，关闭firewalld，关闭swap
systemctl  stop   firewalld  `&&`   systemctl  disable   firewalld

swapoff  -a

2、配置主机映射和修改主机名

hostnamectl   set-hostname  k8s-master

3、yum  -y  install   docker   `&&`  systemctl  start  docker  `&&`  systemctl enable docker 

4、 yum  -y  install   kubectl kubelet kubernetes-cni kubeadm    (这几个软件需要去google的yum源下载，建议下本地vpnFQ，把软件包下载下来)

5、systemctl  start  kubelet  `&&`    systemctl  enable  kubelet

6、master端导入需要镜像（详细见软件包）

7、kubeadm进行初始化：

kubeadm init --kubernetes-version=v1.9.1 --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=0.0.0.0 --apiserver-cert-extra-sans=10.2.0.35,127.0.0.1,k8s-master 

 --apiserver-cert-extra-sans   #该参数指定自己内网地址和映射

在执行过程中，要等几分钟,然后执行以下命令：

然后执行如下命令：
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
成功以后可以指定一下命令进行查看：

kubectl  get  cs

kubectl  get nodes

8、注意：

记住该条命令：kubeadm join --token d506ef.63c05fa829529565 10.2.0.37:6443 --discovery-token-ca-cert-hash sha256:4cd1954bf2a1c0904f92328d33bc25471604abd918e019b3c1905289fb8130f2   #在node节点执行该条命令，--token是有有效期的，默认是24小时

 9、部署 flannel

mkdir -p ~/k8s/
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
kubectl apply -f kube-flannel.yml   #执行会输出一下内容：

10、kubectl  get pod -n kube-system   #注意红框内出现Running状态然后在node节点

在node节点部署

1、同master节点前五步一样部署

2、导入镜像：其实我导入多了，只需要导入一下几个就可以了:gcr.io/google_containers/kube-apiserver-amd64、gcr.io/google_containers/etcd-amd64、quay.io/coreos/flannel、gcr.io/google_containers/etcd-amd64

3、执行master的第8步，就可以了，然后在master上执行：kubectl  get  nodes,看到如下图：

注意：要想在node节点执行该条命令，需要再node上执行以下命令：

sudo cp /etc/kubernetes/admin.conf $HOME/

sudo chown $(id -u):$(id -g) $HOME/admin.conf

export KUBECONFIG=$HOME/admin.conf
不然会出现以下错误:The connection to the server localhost:8080 was refused - did you specify the right host or port?(单独开终端也会出现）
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
最后是版本对应:
master:
REPOSITORY                                                         TAG             IMAGE ID           CREATED      SIZE
gcr.io/google_containers/kube-apiserver-amd64     v1.9.1       e313a3e9d78d     7 days ago     210.4 MB
gcr.io/google_containers/kube-scheduler-amd64    v1.9.1       677911f7ae8f        7 days ago     62.7 MB
gcr.io/google_containers/kube-proxy-amd64           v1.9.1       e470f20528f9        7 days ago     109.1 MB
gcr.io/google_containers/kube-controller-manager-amd64     v1.9.1    4978f9a64966    7 days ago    137.8 MB
quay.io/coreos/flannel     v0.9.1-amd64     2b736d06ca4c     8 weeks ago     51.31 MB
gcr.io/google_containers/k8s-dns-sidecar-amd64   1.14.7       db76ee297b85      11 weeks ago   42.03 MB
gcr.io/google_containers/k8s-dns-kube-dns-amd64   1.14.7    5d049a8c4eec      11 weeks ago          50.27 MB
gcr.io/google_containers/k8s-dns-dnsmasq-nanny-amd64      1.14.7    5feec37454f4    11 weeks ago    40.95 MB
gcr.io/google_containers/etcd-amd64    3.1.10    1406502a6459          4 months ago    192.7 MB
gcr.io/google_containers/pause-amd64   3.0     99e59f495ffa     20 months ago     746.9 kB

node节点对应:

REPOSITORY                                               TAG                 IMAGE ID            CREATED             SIZE

gcr.io/google_containers/kube-apiserver-amd64            v1.9.1              e313a3e9d78d        7 days ago          210.4 MB

quay.io/coreos/flannel                                   v0.9.1-amd64        2b736d06ca4c        8 weeks ago         51.31 MB

gcr.io/google_containers/etcd-amd64                      3.1.10              1406502a6459        4 months ago        192.7 MB

gcr.io/google_containers/pause-amd64                     3.0                 99e59f495ffa        20 months ago       746.9 kB

#以下是另一种网络版本对应:

REPOSITORY                                               TAG                 IMAGE ID            CREATED             SIZE

quay.io/calico/node                                      v2.6.2              6763a667e3ba        12 weeks ago        281.6 MB

quay.io/calico/cni                                       v1.11.0             c3482541970f        3 months ago        70.88 MB

错误信息总结:

1、错误:Unable to connect to the server: x509: certificate signed by unknown authority (possibly because of "crypto/rsa: verification error" while trying to verify candidate authority certificate "kubernetes")

解决方案：设置环境变量：

sudo cp /etc/kubernetes/admin.conf $HOME/

sudo chown $(id -u):$(id -g) $HOME/admin.conf
export $KUBECONFIG=$HOME/.kube/admin.conf

2、错误：[ERROR FileContent--proc-sys-net-bridge-bridge-nf-call-iptables]: /proc/sys/net/bridge/bridge-nf-call-iptables contents are not set to 1

解决方案：cat /etc/sysctl.conf　　　　
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1

3、获取登录token
kubectl describe secrets -n kube-system $(kubectl -n kube-system get secret | awk '/dashboard-admin/{print $1}')
```
