# K8S In Action

标签（空格分隔）： 后端

[TOC]



---

### K8S组件
#### Master Components
  Master提供控制面板，负责集群的控制和管理。
  ![](https://ws1.sinaimg.cn/large/006tKfTcgy1g1bm644fl1j30rb0mgk0g.jpg)

- kube-apiserver: 暴露了K8S的API,提供前端对外交互。
- etcd: 一致且高可用的键值存储，用作K8S的所有集群数据的后备存储。
- kube-scheduler: 负责集群资源调度
- kube-controller-manger: 管理控制器的组件。这些控制器包括：
  - Node Controller: 负责在节点出现故障时响应
  - Replication Controller: 负责为系统中每个复制控制器对象维护正确数量的pod.
  - Endpoints Controller: 控制Endpoints对象
  - Service Account & Token Controllers: 为新的namespace创建默认的账号和API访问token
- cloud-controller-manager: 与底层云服务商交互。


#### Node Components
  Node组件运行在每个node节点上，维护运行的pods和提供K8S运行环境。
 ![](https://ws1.sinaimg.cn/large/006tKfTcgy1g1bnzb1la8j30qh0lrqej.jpg)

- kubelet: 运行在集群中每个节点的代理，确保了容器是运行在pod中。
- kube-proxy: 维护主机上的网络规则并执行连接转发来启用K8S服务。

#### Addons(插件）
  插件是实现了某些集群特性的pods和services。
- DNS: K8S必须要有一个集群DNS服务器，记录了K8S中的服务。
- Web UI(Dashboard): 管理集群和应用的UI界面
- Container Resource Monitoring: 记录容器资源
- Cluster-level Logging:记录存储容器log

---

### Chapter1 Introducing Kubernetes
#### Understanding the needs for a system like Kubernetes
- 从单片应用程序迁移到微服务
- 给应用程序提供一致的环境
- 持续交付

#### Introducing container technologies
The mechanisms that make container isolation possible:

- isolating process with Linux namespaces
- Limiting resources avilable to a process

#### Introducing Kubernetes
Running an application in Kubernetes
![](https://ws2.sinaimg.cn/large/006tKfTcgy1g1cvj0qs6wj30u00u97e6.jpg)

---
### Chapter2 First Step With Docker and Kubernetes
#### Setting up a Kubernetes cluster
- 安装minikube: brew cask install minikube
- 展示集群信息: kubectl cluster-info
- 启动Dashboard: minikube dashboard

### Chapter3 Pods: running containers in Kubernetes
#### Introducing pods
一个Pod中的所有容器都运行在同一个节点中，一个pod不会分割到两个节点上。
通过多容器可以一个容器只跑一个进程，但又需要将这些容器关联在一起，因此有了pod。通过将一个pod中的所有容器配置在一个命名空间，让这些容器使用同一个hostname和network interface。但文件系统是来自镜像文件，因此是彼此分离的，可以通过挂载volume来共享。

#### Creating pods from YAML or JSON descriptors
Pod定义的三个重要部分：

- Metedata: 包括name, namespace, labels...
- Spec: 包含该pod内容的实际描述，包括containers、volumes等
- Status: 说明当前pod的状态信息(这是运行时只读的数据，创建新pod时不需要提供这部分内容)
![](https://ws1.sinaimg.cn/large/006tKfTcgy1g1f2h00qp9j30ru0dgtai.jpg)

- kubectl create -f yaml/json file(通过文件创建资源)

#### Organizing pods with labels
labels是资源上的键值对，一个资源可以有多个标签。通过标签可以进行过滤和组织资源。
![](https://ws4.sinaimg.cn/large/006tKfTcgy1g1f32z890jj30sx0dp773.jpg)

- kubectl get 资源 --show-labels 查看资源的标签
- kubectl get 资源 -l label 查看带有某种标签的资源

#### Using labels and selectors to constrain pod secheduling
不应该指定某个pod必须调度到某个指定的node上，因为K8S集群的初衷便是封装底层硬件设施，将所有节点变为一个资源池。
可以通过node labels和node label selector去选择符合要求的节点。

- 设置节点标签: kubectl label node nodeName labelKey=labelValue
- 获得指定标签的节点: kubectl get nodes -l labelKey=labelValue
- 删除指定key的label: kubectl label node nodeName key-
![](https://ws3.sinaimg.cn/large/006tKfTcgy1g1f48av33wj31240eqdhx.jpg)

#### Annotating pods
Annotations也是键值对，不过不是用来区分资源进行选择的，而是注解信息，用来描述当前资源的。例如说明是谁建立的当前资源。

#### Using namespace to group resources
通过命名空间可以将资源划分为更小的组。两个命名空间中的资源可以同名。但节点是全局的。
![](https://ws3.sinaimg.cn/large/006tKfTcgy1g1f5ff10h1j30um07d759.jpg)

- 创建命名空间: kubectl create namespace spaceName
- 删除命名空间: kubectl delete namespace spaceName(同时删除了该命名空间下的所有资源)

#### Stopping and removing pods
- 通过名字删除: kubectl delete po poName
- 通过标签删除: kubectl delete po -l labelKey=labelValue
- 删除所有pods: kubeclt delete po --all
- 删除所有资源: kubeclt delete all --all

---
### Chapter 4 Replication and other controllers: deploying managed pods
#### Keeping pods healthy
K8S通过存活探针(liveness probes)去检查容器是否存活。K8S通过以下探针机制：

- HTTP GET 探针执行GET请求，如果没有返回或者返回错误则进行重启。
- TCP Socket建立TCP连接
- Exec执行二进制命令。状态码0则是成功，其他视为错误。
![](https://ws3.sinaimg.cn/large/006tKfTcgy1g1f8i8xp1ij30tq0i6wh2.jpg)
- 可以添加额外的参数如delay、timeout、period等明确重启的条件。记得添加initialDelaySeconds延迟第一次探测时间，因为需要等应用起来。
- kubectl describe po poName查看pods信息,重启时可以看到上次退出的code。其值为128+x,x是真实的退出code。

#### Introducing ReplicationControllers
ReplicationControllers确保运行的pod数和副本数一致。当node挂掉时能够进行转移。而没被管理的pod在node挂掉时无法转移。
ReplicationController的三个关键部分：

- A label selector: 进行pods筛选
- A replica count: 确定pods的个数
- A pod template: 创建新pod的模板
![](https://ws4.sinaimg.cn/large/006tKfTcgy1g1f9jrxf70j30wc0clabn.jpg)

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g1fav7ut5mj30vc0dh76j.jpg)
![](https://ws4.sinaimg.cn/large/006tKfTcgy1g1favy5bhkj30qb0b4jsg.jpg)

删除ReplocationController时，它创建的pods同时会被删除。可以使用 --cascade=false参数，让pods不被删除。

#### Using ReplicaSets instead of ReplicationControllers
ReplicationControllers会被ReplicaSets取代。ReplicaSet可以允许匹配缺少某个特定标签的pods或者包含某个标签而不管它的值的pods。
![](https://ws4.sinaimg.cn/large/006tKfTcgy1g1feew30uvj317a0u0tgn.jpg)

![](https://ws4.sinaimg.cn/large/006tKfTcgy1g1feon0zowj319e0aq40i.jpg)
相比matchLabels,matchExpressions属性功能更加强大。有四种合法的operator:

- In: Pods的标签的值必须匹配这些特定值中的一个
- Not In: Pods标签的值不能匹配这些特定值中的任何一个
- Exists: Pods标签的key必须匹配这些值中的一个，并且不能使用values字段。
- DoseNotExists: Pods标签的key不能匹配这些值，并且不能使用values字段。
当使用多个表达式时，所有的值为真才能匹配一个pod。matchLabels和matchExpressions同时使用时，所有的值为真才能匹配一个pod.

#### Running exactly one pod on each node with DaemonSets
每个节点运行一个pod，这种需求是作为辅助进程，例如作为日志收集器，收集每个节点上的日志。作为资源监视器，监视每个节点上的资源。也可以通过nodeSelector属性，选择部署在所有节点的一个子集上。
![](https://ws2.sinaimg.cn/large/006tKfTcgy1g1fy3qvwxjj31040nhwi7.jpg)

#### Runing pods that perform a single completeable task
Job是成功跑完后便再执行，一般用于某些特殊任务，例如数据迁移。可以添加completions和parallelism属性跑多少次和同时跑的任务个数。添加activeDeadlineSeconds属性去标记任务的结束时间，超过这个时间视为失败。
![](https://ws4.sinaimg.cn/large/006tKfTcgy1g1fztsokwej30on0fzacj.jpg)
![](https://ws2.sinaimg.cn/large/006tKfTcgy1g1g06lsaupj30vf0av0ub.jpg)

#### Scheduling Jobs to run periodically or once in the future
CronJob是周期定时任务。schedule属性为Minute、Hour、Day Of month、Month、Day of Week
![](https://ws3.sinaimg.cn/large/006tKfTcgy1g1g0dejwijj30q90h1dik.jpg)

---
### Chapter5 Services: enabling clients to discover and talk to pods
因为pods每个具有独立的IP，并且在集群中可以迁移和扩展，为了统一组织这些pods，并对外提供服务，因此有了Services。
#### Introducing servcies
所有的内部和外部客户端通常都通过服务连接到pods。
![](https://ws1.sinaimg.cn/large/006tKfTcgy1g1g0wg8ylij30st0l4acl.jpg)
![](https://ws4.sinaimg.cn/large/006tKfTcgy1g1g5av350sj30wj0bxtah.jpg)

添加sessionAffinity属性可以控制固定的IP访问的时同一个pod.
在使用多端口的服务时，必须给每个端口添加名字。
可以在pod的定义中给端口添加名字，明确端口意义。然后在服务中targetPort引用该名字。
![](https://ws2.sinaimg.cn/large/006tKfTcgy1g1g78rrouvj312k0hwmzq.jpg)
![](https://ws2.sinaimg.cn/large/006tKfTcgy1g1g7d3mmxbj316u0huwhe.jpg)

#### Exposing Service to external clients N
可以使用以下方式让服务对外部客户端暴露：
- 服务类型设置为NodePort
  
  > 通过所有节点上的同一个端口，服务能够彼此连接，外部客户端可以通过任意节点的IP和端口访问。
> ![](https://ws3.sinaimg.cn/large/006tKfTcgy1g1gbpqaxafj30nl0b7ta5.jpg)

- 服务类型设置为LoadBalancer: 可以提供负载均衡(服务需要公网IP)
![](https://ws1.sinaimg.cn/large/006tKfTcgy1g1hb7045m1j30yo0fp0v5.jpg)
- 创建一个Ingress resource

#### Exposing services externally through an Ingress resource
LoadBalancer类型的服务，每个服务都需要一个独立的外网IP。但Ingress可以让多个服务使用同一个外网IP。
![](https://ws1.sinaimg.cn/large/006tKfTcgy1g1hbo422ynj30y60de76w.jpg)

![](https://ws4.sinaimg.cn/large/006tKfTcgy1g1hc9u5bv3j30t20dhdhx.jpg)
![](https://ws3.sinaimg.cn/large/006tKfTcgy1g1hd66q711j30jw08mdgs.jpg)

#### Signling when a pod is ready to accept connections
readiness probes(就绪探针): 如果pod需要进行一些配置或者预热工作，不想一启动就接收请求，可以使用readiness probes去检查应用是否已经就绪。同存活探针一样，也有Exec、HTTP、TCP Socket这三种方式。
存活探针主要判断程序是否健康，如果不健康该pod则会被杀死，然后创建一个新的pod.就绪探针主要判断程序是否可以开始服务，如果没有就绪，该pod这会从服务的Endpoints中移除，准备就绪后再次被加入。
应当定义就绪探针在服务中，避免应用还在启动就开始接收请求。
![](https://ws2.sinaimg.cn/large/006tKfTcgy1g1he1m0740j30wm0ffgnw.jpg)

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g1hfcn110qj30r80dx0u5.jpg)

---
### Chapter6 Volumes: attaching disk storage to containers
#### Introducing volumes
卷的类型：

- empyDir: 简单的空目录用于存放临时数据
- hostPath: 挂载工作节点的文件系统进入Pod
- gitRepo: git仓库
- nfs: 挂载到pod中的NFS共享卷
- 云服务商特定存储类型: gcePersistentDisk(Google高效能型存储磁盘卷)、awsElasticBlockStore(AmazonWeb服务弹性块存储卷)、azureDisk(MicrosoftAzure磁盘卷）
- config、secret、downwardAPI: 用于将K8S部分资源和集群信息公开给pod的特殊类型的卷。
- persistentVolumeClaim: 一种使用预置或者动态配置的持久存储类型

#### Using volumes to share data between containers
emptyDir是挂载在磁盘上的空目录，可以用于容器存放临时数据。可以改变emptyDir的medium属性，将其指定为Memory，提高速率。
![](https://ws2.sinaimg.cn/large/006tKfTcly1g1iaxm0gznj31300abjsh.jpg)
![](https://ws2.sinaimg.cn/large/006tKfTcly1g1iaz1hdffj312n0j4dk6.jpg)
![](https://ws2.sinaimg.cn/large/006tKfTcly1g1ib4cujywj30nk054mxn.jpg)

#### Decoupling pods from the underlying storage technology
PersistentVolumes(PV)由集群管理员提供，然后用户创建PersistentVolumeClaim(PVC)去绑定到PV申请容量，Pod去引用这个PVC。
![](https://ws3.sinaimg.cn/large/006tKfTcgy1g1igodyko0j30up0imn14.jpg)

![](https://ws4.sinaimg.cn/large/006tKfTcly1g1ii9viy93j30tw0efdin.jpg)
```
# 创建PV
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
spec:
  capacity:
    storage: 2Gi
  accessModes:
  - ReadWriteOnce
  - ReadOnlyMany
  persistentVolumeReclaimPolicy: Retain
  local:
    path: /data/db/mysql
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - nlp-t-k8snode-03

#创建PVC
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
  resources:
    requests:
      storage: 1Gi
  accessModes:
  - ReadWriteOnce
  - ReadOnlyMany
  volumeMode: Filesystem
  volumeName: mysql-pv
  
# 引用该PVC
apiVersion: apps/v1beta2
kind: ReplicaSet
metadata:
  name: mysql-replicaset
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0.15
        ports:
        - containerPort: 3306
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: 123456qq
        volumeMounts:
        - name: mysql-data
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-data
        persistentVolumeClaim:
          claimName: mysql-pvc
```

![](https://ws3.sinaimg.cn/large/006tKfTcgy1g1kttf86ipj30zh0m379g.jpg)

---
### Chapter7 ConfigMaps and Secrets: configuring applications
#### Configuring containerized applications
给应用传递配置的方式：

- 传递命令行给容器
- 设置自定义环境变量
- 通过volume挂载配置文件

#### Passing command-line arguments to containers
K8S中的指令与Docker指令对应关系，通常只用args去覆盖默认值。
![](https://ws4.sinaimg.cn/large/006tKfTcgy1g1kv7u24cdj30uj06qdgn.jpg)

#### Setting environment variables for a container
Pod中定义的环境变量会成为容器中的系统环境变量，应用可以引用该变量。同一个命名空间中的服务环境变量也会被自动暴露给容器。
![](https://ws1.sinaimg.cn/large/006tKfTcgy1g1kviiu9sxj30kb06u74t.jpg)

#### Decoupling configuration with a ConfigMap
创建configMap方式：
![](https://ws1.sinaimg.cn/large/006tKfTcgy1g1kwhjuhjnj30u10710tu.jpg)
![](https://ws4.sinaimg.cn/large/006tKfTcgy1g1kwiivryoj30uo0sj77c.jpg)
```
# 创建configMap
kind: ConfigMap
apiVersion: v1
metadata:
  name: db-config
data:
  MONGO_HOST: mongo-service,
  MYSQL_HOST: mysql-service
  

# POD中引用该环境变量
env:
- name: MYSQL_HOST
  valueFrom:
    configMapKeyRef:
      name: db-config
      key: MYSQL_HOST
      
# 项目中引用该环境变量
url: jdbc:mysql://${MYSQL_HOST}:3306/speechcraft
```

可以通过envFrom获得configMap中的所有变量。如果命名不规范，这条记录会被省略。
![](https://ws4.sinaimg.cn/large/006tKfTcgy1g1lz71qzg1j30w00awjt8.jpg)

---
### Accessing pod metadata and other resoureces from applications
可以通过环境变量给应用暴露Pod的一些元信息。
![](https://ws3.sinaimg.cn/large/006tKfTcgy1g1qoe53pewj30xg0i40up.jpg)
![](https://ws4.sinaimg.cn/large/006tKfTcgy1g1qoel0m5dj30w80q3dkl.jpg)
也可以通过文件挂载到downwardAPI volume上。

---
### Deployments: updateing applications declaratively
在镜像使用的tag为latest时,进行拉取的默认策略为Always,避免了不同节点之间存在相同tag的镜像不同的情况。但其他tag的镜像默认拉取策略为IfNoPresent,因此在使用其他tag名时，更新版本需要更新tag。这样也利于版本回退。
![](https://ws4.sinaimg.cn/large/006tNc79gy1g1uyokmihxj30wb0ju0w4.jpg)
Deployment保留了部署版本，可以通过以下命令查看及回滚：
> kubectl rollout history deployment deploymentName
> kubectl rollout undo deployment deploymentName --to-revision=version
>
>   ![](https://ws1.sinaimg.cn/large/006tNc79gy1g1x9lodq57j30sg0awq45.jpg)

  ![](https://ws4.sinaimg.cn/large/006tNc79gy1g1x9scygdfj30z30r1wvu.jpg)
  ![](https://ws1.sinaimg.cn/large/006tNc79gy1g1x9vl5cjrj30xu0l6wi5.jpg)

  可以通过暂停升级进行金丝雀发布，让部分流量命中新版本，当检测无误后继续进行发布。
  ![](https://ws1.sinaimg.cn/large/006tNc79gy1g1xa5gtgclj30oy04laan.jpg)
  ![](https://ws4.sinaimg.cn/large/006tNc79gy1g1xa5rgo4hj30gu02iwel.jpg)

### StatefulSets: deploying replicated stateful applications
#### Replicating stateful pods
  同一个副本集创建的pod会使用同一个PVC和PV。
  ![](https://ws4.sinaimg.cn/large/006tNc79gy1g233fh2i18j30l909dq3u.jpg)

#### Understanding StatefulSets
    宠物与牛的类比：我们倾向于把应用看做宠物，给每个实例起个名字，细心照顾。但大多情况下，我们把应用看做牛，并不需要对单独的实例关心，可以方便的替换掉不健康的实例，就像替换掉一头生病的牛。
    RelicaSet或ReplicationController管理的pod副本比较像牛，因为它们是无状态的，任何时候都可以被一个全新的pod替换。而Statefulset管理的pod更像宠物，当pod挂掉后，会新建一个与其有相同的名称、网络标识、状态的pod。
![](https://ws4.sinaimg.cn/large/006tNc79gy1g234jmojftj30zc0endid.jpg)
    StatefulSets一次只能关掉一个实例，并且当存在不健康的实例时不允许关掉实例。
![](https://ws4.sinaimg.cn/large/006tNc79gy1g2481a9jxwj30zl0aljtd.jpg)
    当一个实例重新被调度的时候，新实例的存储与旧实例的存储相同，这是通过新实例复用就实例的PVC。
![](https://ws3.sinaimg.cn/large/006tNc79gy1g248not9g7j30mh09075n.jpg)
![](https://ws3.sinaimg.cn/large/006tNc79gy1g2ft5g6esoj30ua0mqjuv.jpg)

#### Using a StatefulSet
![](https://ws3.sinaimg.cn/large/006tNc79gy1g2ftq7qmycj30xd06vmym.jpg) 
  部署StatefulSet必须创建headless Service,即clusterIP为None。
  ![](https://ws2.sinaimg.cn/large/006tNc79gy1g2fub9uhs5j30r90psn0p.jpg)

#### Discovering peers in a StatefulSet
   StatefulSet中的成员需要能够容易的找到集群中的其他成员。同过SRV Record实现。
   ![](https://ws4.sinaimg.cn/large/006tNc79gy1g2g0m31anpj30km05hdgy.jpg)

---
### Chapter11 Understanding K8S internals
#### Understanding the architecture

k8s由两部分组成：The K8S Control Plane、The (Worker) nodes
Components of the control plane
- The etcd distributed persistent storage:
- The API Server
- The Scheduler
- The Controller Manager

Components running on the worker nodes:
- The Kubelet
- The Kubernetes Service Proxy(kube-proxy)
- THe Container Runtime(Docker,rkt,or others)

Add-on Components
- The Kubernetes DNS server
- The Dashboard
- An Ingress controller
- Heapster
- The Container Network Interface network plugin

![](https://ws3.sinaimg.cn/large/006tNc79gy1g2su2b6a7bj30u10dzq4h.jpg)

etcd:
etcd是k8s中存储数据的一个分布式，key-value系统。只有k8s API Server能与其交互。其他组件都通过API Server间接与etcd交互。etcd数量应该是奇数。
k8s中的元数据存储在前缀为/registry的key中。key对应的value便是对应资源的JSON数据。etcd通过乐观锁控制了一次只能有一个API server改写数据。

![](https://ws3.sinaimg.cn/large/006tNc79gy1g2sve3sr8lj30bj0730tc.jpg)

API Server: API server通过RESTful API提供了对集群数据的CRUD功能。客户端如kubectl通过http发送请求，经过API server的验证和授权，然后修改etcd中的数据。如果请求需要创建、修改、删除资源，请求会通过Admission control去验证资源，如果只是读取数据则不会通过Admission control。当资源改变时，API server会发送更新后的资源给所有监听了这个资源的客户端。

![](https://ws4.sinaimg.cn/large/006tNc79gy1g2svppbyb7j31220c7tb0.jpg)

![](https://ws3.sinaimg.cn/large/006tNc79gy1g2sw31haa0j30z00f2q4y.jpg)

Scheduler: scheduler通过API server更新pod的定义，然后API server通知kubelet这个pod被重新调度。然后目标node的kubelet看pod是否被带调度到这个节点，创建运行这个pod的容器。

Controller: Controller Manager由许多controller组成，controller确保部署的资源状态与定义的一样。

Kubelet: kubelet存在于每个work nodes，负责向API server注册node，已经监视被调度到该node的pod资源，并向API server报告容器的状态、事件等。

Kube-proxy: 代理服务，确保服务能够连接到每个pod。最初实现是利用userspace代理模式，将每个连接转发到代理服务器，然后通过iptables规则转发到实际的pod.现在使用iptables代理模式，直接发送到pod,不在使用代理服务器。

![](https://ws4.sinaimg.cn/large/006tNc79gy1g2zxgc5ukej30s608f75a.jpg)

![](https://ws4.sinaimg.cn/large/006tNc79gy1g2zxhxvdejj30sx08xmya.jpg)

Add-ons:插件，通过Development、ReolicationController、DaemonSet等形式部署。

DNS server:集群中的所有pod默认是使用集群内部的DNS服务器，这让pod可以通过名字查询服务，甚至在headless service（cluster IP is None，用于StatefulSet)中可以查到pod的IP地址。DNS server pod通过kube-dns service暴露。通过API serve监听机制去订阅Services和Endpoints的变化。并允许客户端几乎实时的获得DNS信息。在DNS pod接收Server或者Endpoints资源变更时，DNS记录可能失效。

Ingress Controller：运行了一个反向代理服务器如Nginx,然后通过Ingress配置转发规则。

#### How controllers cooperate

![](https://ws4.sinaimg.cn/large/006tNc79gy1g30mkr14hyj30vq0glwgs.jpg)

![](https://ws2.sinaimg.cn/large/006tNc79gy1g30ml2zhapj31380qrdkt.jpg)

K8S的组件通过监听API server中的变化的，然后通过事件链完成协作。Component和Kubelet执行动作都会发送事件给API server，并且创造Event resources. 可以通过kubectl get events —watch查看。

#### Understanding what a runing pod is 

当一个pod运行时，除了用户的容器还会有一个基础容器，该容器会在所有容器创建之前创建，它的作用是将pod中的所有容器放置在同一个命名空间。基础容器与pod的生命周期绑定。

#### Inter-pod networking(pod间网络)

![](https://ws4.sinaimg.cn/large/006tNc79gy1g30o1baehfj30x30cl402.jpg)

同一个node中的pod会通过虚拟以太网接口连接到同一个网桥。

![image-20190514112223948](/Users/yangshuai/Library/Application Support/typora-user-images/image-20190514112223948.png)

不同node之间的连接也需要一些网络技术进行连接。Container Network Interface(CNI)插件便是解决容器间连接问题的。

#### How services are implemented

服务会有虚拟IP和端口号，并且监听Pod和Endpoints的变化，当有发往该服务的数据包时，通过iptables规则，将数据包重定向到支持该服务的pod。

![](https://ws1.sinaimg.cn/large/006tNc79gy1g30oyptp1wj30se0r7787.jpg)

#### Running highly available clusters

App: 对于应用,可以通过多实例的方式确保应用的高可用。对于无法水平扩展的应用可以通过选举机制，让一个实例成为领导者，其他实例待命的方式确保高可用。

Etcd: etcd是分布式存储系统，可以通过etcd集群方式确保高可用。

API server: 多实例方式

Controller and Scheduler: 选举机制。

![](https://ws2.sinaimg.cn/large/006tNc79gy1g3127sonjfj316z0fdq6m.jpg)

----

### Chapter12 Securing the K8S API server

#### Understanding Authentication

认证插件返回被认证的用户的用户名和所属组。一个用户可以属于一个或多个组。通过组去赋予用户权限，而不是赋予给单个用户。

![](http://ww1.sinaimg.cn/large/006tNc79gy1g38oocgihfj30z80d4dj1.jpg)

每个pod都与一个ServiceAccount关联，token文件持有了ServiceAccount的认证token,认证插件认证了ServiceAccount后会把这个ServiceAccount的用户名返回给API server.然后API server再传递给授权插件，授权插件确认用户权限。

![](http://ww1.sinaimg.cn/large/006tNc79gy1g38ox6nu3kj30y004kdgk.jpg)

ServiceAccount资源属于命名空间，每个命名空间会自动创建一个默认的ServiceAccount.同一命名空间的pod可以使用同一个ServiceAccount，也可以使用不同的ServiceAccount，但不能夸命名空间使用其他空间的ServiceAccount，每个pod也只能使用一个ServiceAccount。

![](http://ww3.sinaimg.cn/large/006tNc79gy1g38ptjjxgoj30pd0adgna.jpg)

创建ServiceAccount: kubectl create serviceaccount saName

Image pull secrets用于从私有仓库拉取镜像时的验证

![](http://ww2.sinaimg.cn/large/006tNc79gy1g38qwfcxsmj30m207wq46.jpg)

![](http://ww1.sinaimg.cn/large/006tNc79gy1g38qz3m9rnj30mc09sab9.jpg)

如果没有使用基于角色认证的插件，新建的ServiceAccount与默认的没有什么区别，只有提供镜像拉取秘钥。在使用基于角色的访问控制插件后才能通过角色去确认权限。

#### Securing the cluster with role-based access control

![](http://ww3.sinaimg.cn/large/006tNc79gy1g38w58axrsj31360dsjt6.jpg)

![](http://ww1.sinaimg.cn/large/006tNc79gy1g38w9l15qkj30nt0doq50.jpg)

角色确定能做什么(权限)，角色绑定确定谁能做(用户、组、ServiceAccount)。RBAC资源分为角色、集群角色、角色绑定和集群角色绑定。区别是角色和角色绑定属于命名空间资源，而集群角色和集群角色绑定属于集群资源。

创建角色

![image-20190521140528896](/Users/yangshuai/Library/Application Support/typora-user-images/image-20190521140528896.png)

 角色绑定![image-20190521141117197](/Users/yangshuai/Library/Application Support/typora-user-images/image-20190521141117197.png)

资源-角色-角色绑定-ServiceAccount的关系：![image-20190521141234843](/Users/yangshuai/Library/Application Support/typora-user-images/image-20190521141234843.png)

角色绑定只能绑定一个角色，但可以绑定多个主体，甚至可以绑定不同命名空间的主体。

![image-20190521142223253](/Users/yangshuai/Library/Application Support/typora-user-images/image-20190521142223253.png)

角色只能获得同一个命名空间的资源，因此有了集群角色和集群角色绑定。集群角色可以获得集群资源或者非资源的URLs.

![image-20190521155833558](/Users/yangshuai/Library/Application Support/typora-user-images/image-20190521155833558.png)

对应非资源的URL必选显示的授权，否则API服务器会拒绝客户端的请求。通常由system:discovery这个ClusterRole和同名的ClusterRoleBinding完成。

![image-20190521160237423](/Users/yangshuai/Library/Application Support/typora-user-images/image-20190521160237423.png)

![image-20190521160254372](/Users/yangshuai/Library/Application Support/typora-user-images/image-20190521160254372.png)

ClusterRole和ClusterRoleBinding确保可以访问夸空间的命名资源。![image-20190521160948172](/Users/yangshuai/Library/Application Support/typora-user-images/image-20190521160948172.png)

而ClusterRole和RoleBinding只能看到同一个命名空间中的资源。![image-20190521161125034](/Users/yangshuai/Library/Application Support/typora-user-images/image-20190521161125034.png)

![image-20190521161259978](/Users/yangshuai/Library/Application Support/typora-user-images/image-20190521161259978.png)

K8S中已经有一些默认的ClusterRole和ClusterRoleBinding。

view ClusterRole允许对命名空间资源的只读访问，除了Role、RoleBinding、Secret。

edit ClusterRole允许对一个命名空间资源的修改，同时允许读取Secret，但不允许查看或修改Role、RoleBinding。

admin ClusterRole可以读取和修改命名空间中的任何资源，除了ResourceQuota和命名空间资源本身，edit和admin的主要区别是能否修改Role和RoleBinding。

Cluster-admin ClusterRole能得到完全的控制。

以system:为前缀的为给K8S组件使用的角色，如system:kube-scheduler是给调度器使用的。

---

### Chapter13 Securing cluster nodes and the network

#### Using the host node's namespaces in a pod

每个Pod都有自己的PID namespace和IPC namespace,使用hostNetWork:true可以使pod使用node的网络接口，不再有自己的IP地址，进程也被绑定到了node的端口。

hostPort是node直接转发到pod中，而且端口只绑定到运行了该pod的node，而NodePort Service是通过service随机转发到任意pod,且服务绑定了所有的node端口。![image-20190522102607232](/Users/yangshuai/Library/Application Support/typora-user-images/image-20190522102607232.png)

设置pod的hostPID和hostIPC为true,可以使Pod中的容器是用node的PID和IPC namespace,这样可以说使这项容器月node的进程进行交互。

#### Configuring the container's security context

安全上下文可以配置到pod或者容器中。这可以做到：

- 指定容器中运行进程的用户
- 阻止容器使用root用户运行
- 使用特权模式运行容器，使其对宿主节点的内核据用完全的访问权限
- 通过添加或禁用内核功能，配置细粒度的内核访问权限
- 设置SELinux(Security Enhaced Linux)选项，加强对容器的限制。

### Chapter 14 Managing pod's computational resources

#### Requesting resources for a pod's containers

当创建pod时可以指定每个容器的CPU、Memory的资源请求量(requests)和资源限制量(limits)。

![image-20190527202350671](/Users/yangshuai/Library/Application Support/typora-user-images/image-20190527202350671.png)

资源request会影响调度，调度器判断一个pod是否适合调度到某个节点根据资源请求量，而不是实际使用量，只有当资源请求量满足pod的需求才会被调度到这个节点。

![image-20190527203422277](/Users/yangshuai/Library/Application Support/typora-user-images/image-20190527203422277.png)

#### Limiting resources available to a container

资源limit限制了容器能够使用的最大资源。资源limit能够被超卖，即超过100%。当某个容器的使用的CPU超过限制，这个容器将不会在分配到更多的CPU限额；当如果使用比限额更高的内存时，这个容器可能因为OOM被杀掉。

![image-20190527205521492](/Users/yangshuai/Library/Application Support/typora-user-images/image-20190527205521492.png)

#### Understanding pod Qos classes

k8s的Qos(Quality of Service)优先级分为三类：

- BestEffort(最低优先级)：没有设置requests和limits，有可能占用所有资源，因此在资源不足情况下被最先杀掉

- Burstable：requests小于limits

- Guaranteed(最高优先级)：cpu和memory的requests和limits都要设置，并且相等，而且pod中的每个容器都要设置。

  ![image-20190528151812517](/Users/yangshuai/Library/Application Support/typora-user-images/image-20190528151812517.png)

![image-20190528152118270](/Users/yangshuai/Library/Application Support/typora-user-images/image-20190528152118270.png)

![image-20190528152221088](/Users/yangshuai/Library/Application Support/typora-user-images/image-20190528152221088.png)

当资源不足时，优先级更低的Pod会先被杀死，当两个优先级相同时，系统会杀掉内存实际使用量占内存申请量比例更高的pod。

#### Setting default requests and limits for pods per namespace

LimitRange可以为同一命名空间中的所有pod和容器设置默认资源requests和limits.LimitRange只能应用于单独的pod或容器，用户仍然可以创建大量的Pod吃掉集群所有可用资源。

![image-20190528154203591](/Users/yangshuai/Library/Application Support/typora-user-images/image-20190528154203591.png)

### Limiting the total resources available in namesapce

ResourceQuota限制命名空间的资源总量，仅作用于在其后创建的Pod,并不影响已经存在的pod。

![image-20190528160340643](/Users/yangshuai/Library/Application Support/typora-user-images/image-20190528160340643.png)

