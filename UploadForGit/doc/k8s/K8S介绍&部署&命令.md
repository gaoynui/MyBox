# Kubernetes

## 介绍

- 简称K8s，用8代替中间的8个字符得名。开源。
- 用于管理云平台中多个主机上容器化的应用。
- 提供了云平台中应用部署、规划、维护、更新的一种机制。

## 核心组件

![](https://dpzbhybb2pdcj.cloudfront.net/luksa/HighResolutionFigures/figure_1-9.png)

- #### etcd

集群的`主数据库`，保存了整个集群的状态。

使每台minion上运行的docker拥有不同的Ip段。

- #### apiserver 

提供了`资源操作的唯一入口`，并提供认证、授权、访问控制、API注册和发现等机制。

- #### Controller manager

负责`维护集群的状态`，比如故障检测、自动扩展、滚动更新等。

- #### scheduler

`资源调度`，按照预定的调度策略将Pod调度到相应的机器上。

- #### kubelet

负责维护容器的生命周期，负责`管理pods和它们上面的容器`，images镜像、volumes、etc。

同时也负责Volume（*CVI, Container Volume Interface*）和网络（*CNI, Container Network Interface*）的管理。

- #### Container runtime

负责镜像管理以及Pod和容器的真正运行（*CRI，Container Runtime Interface*）。

- #### kube-proxy

为Service提供cluster内部的`服务发现和负载均衡`。

## 其他重要组件

- #### kube-dns

为整个集群提供dns服务。

- #### Ingress Controller

提供外网入口。

- #### Dashboard

GUI。

## 相关概念

#### 容器

每个容器之间互相隔离，每个容器有自己的文件系统 ，容器之间进程不会相互影响，能区分计算资源。

容器与底层设施、机器文件系统解耦的，所以能在不同云、不同版本操作系统间进行迁移。

容器占用资源少、部署快，每个应用可以被打包成一个容器镜像。

#### master

k8s集群的管理节点，负责管理集群，提供集群的资源数据访问入口。

#### node

node是k8s集群操作的单元，用来承载被分配pod的运行，是Pod运行的宿主机。

#### pod

pod是可以创建和管理K8s计算的最小可部署单元。一个Pod代表集群中的一个进程。

由一个或多个容器组成，它们共享容器存储、网络和容器运行配置，共享IP地址和端口号，它们之间可以通过localhost互相发现。

同一个Pod中的容器会自动分配到同一个node上。

每个Pod都会被分配唯一一个IP地址。

<img src="https://github.com/gaoynui/MyTodoList/blob/master/doc/pics/pod%E5%86%85%E9%83%A8.png?raw=true" alt="pod内部" style="zoom:50%;" />

每个pod都有一个特殊的被称为根容器的pause容器(initContainer)，pause容器对应的镜像数据K8S平台的一部分。

创建pod经历阶段：

 apiservice  -> etcd -> statefulset(调度) -> node节点

#### Replication Controller（RC，副本控制器）

用来管理Pod的副本，保证集群中存在指定数量的pod副本。是实现弹性伸缩、动态扩容、滚动升级的核心。

#### ReplicaSet(RS, 副本集)

新一代RC，一般不单独使用，而是作为Deployment的理想状态参数使用。

#### StatefulSet（有状态副本集）

是为了解决有状态服务的问题 (对应 Deployments 和 ReplicaSets 是为无状态服务而设计)，可以保证资源创建和伸缩时候的顺序。应用场景包括：

- 稳定的持久化存储，即 Pod 重新调度后还是能访问到相同的持久化数据，基于 PVC 来实现。
- 稳定的网络标志，即 Pod 重新调度后其 PodName 和 HostName 不变，基于 Headless Service（即没有 Cluster IP 的 Service ）来实现。
- 有序部署、有序扩展，即 Pod 是有顺序的，在部署或者扩展的时候要依据定义的顺序依次进行 (即从 0 到 N-1，在下一个 Pod 运行之前所有之前的 Pod 必须都是 Running 和 Ready 状态)，基于 init containers 来实现。
- 有序收缩、有序删除 (即从 N-1 到 0)。

#### DaemonSet（守护进程集）

保证在每个 Node 上都运行一个容器副本，常用来部署一些集群的日志、监控或者其他系统管理应用。

#### deployment

k8s是通过各种controller来管理pod的生命周期。为了满足不同业务场景，k8s开发了Deployment、ReplicaSet、DaemonSet、StatefuleSet、Job 等多种 Controller。

注：RC、RS和Deployment只是保证了支撑服务的微服务Pod的数量。

#### Service

Ingress type:

NodePort

通过所有节点上的同一个端口，服务能够彼此连接，外部客户端可以通过任意节点的IP和端口访问。

LoadBalancer

提供负载均衡。每个服务需要一个独立的外网IP。但Ingress可以让多个服务使用同一个外网IP。

<img src="https://ws1.sinaimg.cn/large/006tKfTcgy1g1hbo422ynj30y60de76w.jpg" style="zoom: 50%;" />

Ingress

#### Volume

持久化存储卷

access mode:

ReadWriteOnce: 该卷可以被单个节点以读/写模式挂载

ReadOnlyMany: 该卷可以被多个节点以只读模式挂载

ReadWriteMany: 该卷可以被多个节点以读/写模式挂载

## Minikube

（环境，VirtualBox虚拟机）

（系统， ubuntu 18.04）

（基于主机起minikube服务）

- #### 准备

1.kubectl

```
下载并赋予kubectl可执行权限：
sudo curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.16.0/bin/linux/amd64/kubectl / && chmod +x ./kubectl
将下载的可执行文件放于用户bin目录下以便通过bash执行：
sudo mv ./kubectl /usr/local/bin/kubectl
验证：
sudo kubectl version
出现Client Version:...GitVersion...则安装成功
```

2.docker

```
更新apt包索引：
sudo apt-get update
安装apt依赖，用于从HTTPS获取仓库：
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
添加docker的官方密钥：
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
添加仓库：
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
再次更新索引：
sudo apt-get update
下载：
sudo apt-get install docker-ce
验证：
sudo docker -v
出现Docker version...则安装成功
###########或者
sudo apt-get install docker.io #但不是最新版本
```

3.minikube

```
下载阿里改版国内可用,同时赋予可执行权限并移到usr/bin目录下：
sudo curl -Lo minikube http://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/releases/v1.4.0/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
启动,直接以ubuntu虚拟系统作为主机，所以虚拟机驱动选none,再添加国内阿里镜像：
sudo minikube start --vm-driver=none --registry-mirror=https://docker.mirror.aliyuncs.com
等待minikube部署，看到Done! kubectl is now...则部署成功
打开GUI管理：
sudo minikube dashboard
复制出现的url用浏览器打开即可。
```

## kubeadm

- #### 准备

1.安装docker,见minikube的安装

2.禁用swap:

```
sudo swapoff -a
```

3.安装kubeadm,kubectl,kubelet

```
# 使用阿里镜像
sudo apt-get update && apt-get install -y apt-transport-https
curl -s https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
# 将它们设置为保留
apt-mark hold kubelet kubeadm kubectl
```

4.重启kubelet

```
systemctl daemon-reload
systemctl restart kubelet
```

5.初始化

```
# 先获取需要的镜像信息
kubeadm config images list
# 创建一个pull脚本拉取国内对应镜像 【其他文件\ubuntu下拉取kubeadm镜像脚本】
-------------------------------------------
#!/bin/bash
images=(
  kube-apiserver:v1.16.2
  kube-controller-manager:v1.16.2
  kube-scheduler:v1.16.2
  kube-proxy:v1.16.2
  pause:3.1
  etcd:3.3.15-0
  coredns:1.6.2
)

for imageName in ${images[@]};do
    docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName
    docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName k8s.gcr.io/$imageName
    docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName
done
--------------------------------------------
# 初始化k8s
kubeadm init
```

6.设置配置文件，给普通用户授予kubectl权限

```
sudo cp /etc/kubernetes/admin.conf $HOME/
sudo chown $(id -u):$(id -g) $HOME/admin.conf
export KUBECONFIG=$HOME/admin.conf
```

7.安装网络插件weave-net

```
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

8.检查节点运行状态

```
kubectl get node
```

9.单节点部署

```
kubectl taint nodes --all node-role.kubernetes.io/master-
```

*10.重置

```
kubeadm reset
删除$HOME/.kube/下config文件
```

# kubectl命令

```
展示集群信息：
kubectl cluster-info
通过文件创建YAML/JSON:
kubectl create -f yaml/json [fileName]
应用编辑好的yaml文件：
kubectl apply -f name.yaml
查看资源的标签：
kubectl get pods --show-labels -n [namespace]
查看带有某种标签的资源：
kubectl get pods -l [label] -n [namespace]
查看所有的Pod资源：
kubectl get pods -A
查看指定资源的日志：
kubectl logs [podName] -n [namespace]
查看指定pod中容器的日志：
kubectl logs [podname] -c [container] -n [namespace]
创建命名空间：
kubectl create namespace [spaceName]
删除命名空间（同时删除该空间下所有资源）：
kubectl delete namespace [spaceName]
删除deployment:
kubectl delete -n [namespace] pod [deployment]
查看Ingress信息：
kubectl get ing -all-namespaces
查看所有命名空间的Pods：
kubectl get pod --all-namespaces
查看对应命名空间的pods:
kubectl get pods -n [namespace]
进入指定命名空间：
kubectl get deployment -n [namespace]
编辑指定命名空间下的yaml:
kubectl edit deployment -n [namespace] 
获取对应namespace下message信息：
kubectl get events -n [namespace]
进入容器bash:
kubectl exec -it [podname] bash -n [namespace]
获取deployment信息：
kubectl describe deploy [deployment] -n [namespace]
编辑deployment信息：
kubectl edit deploy [deployment] -n [namespace]
编辑pod信息:
kubectl edit pod [pod] -n [namespace]
启动GUI代理：
kubectl proxy
导入数据库：
kubectl exec -i [mysqlpod名称] -n [namespace] -- mysql -uroot -p123456qq aisearchconfig < aisearchconfig.sql
```

