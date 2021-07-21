---
title: kubernetes 创建 secret 
date: 2021-03-21 16:30:00
tags: 
    - [ kubernetes ]
categories:
    - [ kubernetes ]
---
## kubernetes secret 描述
```
secret 用于存储和管理一些敏感数据，比如密码，token，密钥等敏感信息。它把 Pod 想要访问的加密数据存放到 Etcd 中。然后用户就可以通过在 Pod 的容器里挂载 Volume 的方式或者 环境变量 的方式访问到这些 Secret 里保存的信息

Secret 有三种类型

Opaque：base64 编码格式的 Secret，用来存储密码、密钥等；但数据也可以通过base64 –decode解码得到原始数据，所有加密性很弱。

Service Account：用来访问Kubernetes API，由Kubernetes自动创建，并且会自动挂载到Pod的 /run/secrets/kubernetes.io/serviceaccount 目录中。Service Account 对象的作用，就是 Kubernetes 系统内置的一种“服务账户”，它是 Kubernetes 进行权限分配的对象。比如，Service Account A，可以只被允许对 Kubernetes API 进行 GET 操作，而 Service Account B，则可以有 Kubernetes API 的所有操作权限

kubernetes.io/dockerconfigjson ： 用来存储私有docker registry的认证信息
```
## Opaque Secret 使用 
```
[root@test1 ~]# echo -n "admin" | base64
YWRtaW4=
[root@test1 ~]# echo -n "123456" | base64
MTIzNDU2
```
创建secret
```
vim myapp-secret.yaml

# 内容
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  username: YWRtaW4=
  password: MTIzNDU2
```
将secret导入环境变量
```
apiVersion: apps/v1
kind: Deployment
metadata:
 name: myapp-deploy
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: 192.168.26.160:86/xielong/myapp:v1.0
        ports:
        - containerPort: 80
        env:
        - name: MYSQL_SERVICE_USER
          valueFrom:
            secretKeyRef:
              name: mysecret
              key: username
        - name: MYSQL_SERVICE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysecret
              key: password
```
查看环境变量
```
[root@master1 yaml]# kubectl exec myapp-deploy-5df5dfbb7f-75qql env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=myapp-deploy-5df5dfbb7f-75qql
 
# 成功导入
MYSQL_SERVICE_USER=admin
MYSQL_SERVICE_PASSWORD=123456
 
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
NGINX_VERSION=1.19.6
NJS_VERSION=0.5.0
PKG_RELEASE=1~buster
HOME=/root
```
## k8s通过secret拉取harbor镜像
### 创建secret
````
kubectl create secret docker-registry test-hub-puller --docker-server=http://test-hub.xxxxx.com --docker-username=puller --docker-password=W2xxxxxxIh  --docker-email=puller@xxxx.com -n shxxxxxxiot

备注:
test-hub-puller #是secret name
--docker-email #邮箱任意写 
--docker-username=puller  #是harbor仓库用户名和密码
--docker-server=http://test-hub.xxxxx.com  #harbor仓库地址
````
### 查看secret和yaml配置
````
kubectl get secret -n shixxxiot
````
<img src="/images/k8s/secret/k8s-secret.jpg" width=100% height=50% align=left/>
<img src="/images/k8s/secret/k8s-secret-containers.jpg" width=100% height=50% align=left/>


参考链接: 
````
https://kubernetes.io/zh/docs/tasks/configure-pod-container/pull-image-private-registry/
https://blog.csdn.net/mshxuyi/article/details/110942969
````
