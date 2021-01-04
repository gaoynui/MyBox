# 自定义部署
## 下载yaml文件：
```
wget http://mirror.faasx.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
该文件中的镜像已经替换为国内七牛云镜像源
```

## 创建部署：
```
kubectl create -f kubernetes-dashboard.yaml 
会提示apps/v1beta2不支持Deployment,进入yaml修改Deployment的apiVersion为apps/v1
```

## 创建TLS证书
```
$ mkdir -p /usr/local/src/kubernetes/certs  
$ cd /usr/local/src/kubernetes  
$ openssl genrsa -des3 -passout pass:123qwe -out certs/dashboard.pass.key 2048  
$ openssl rsa -passin pass:123qwe -in certs/dashboard.pass.key -out certs/dashboard.key  
$ openssl req -new -key certs/dashboard.key -out certs/dashboard.csr -subj '/CN=hostname'  
这里会提示can't load into RNG,进入提示的路径下执行：  
  $openssl rand -writerand .rnd  
$ cd /usr/local/src/kubernetes  
$ openssl x509 -req -sha256 -days 365 -in certs/dashboard.csr -signkey certs/dashboard.key -out certs/dashboard.crt  
$ rm certs/dashboard.pass.key  
$ kubectl create secret generic kubernetes-dashboard-certs --from-file=tls.crt=certs/dashboard.crt --from-file=tls.key=certs/dashboard.key --from-file=certs/dashboard.key --from-file=certs/dashboard.crt -n kube-system  
这里会提示已经存在secret,是因为dasboard的yaml文件中已经添加创建好了，去kubernetes删除secret/kubernetes-dashboard-certs再重新创建  
```

## 修改dashboard-ingress.yaml
在文件末spec分支下创建:  

```
  tls:  
  - hosts:  
    - hostname  
      secretName: kubernetes-dashboard-certs  
```

## hostport暴露nginx-ingress-controller服务
修改nginx-ingress-controller配置文件，将ingress controller改为Daemmon方式部署。  
然后在containers.ports部分做主机端口映射，添加 hostPort: 80和 hostPort: 443
## 在dashboard配置文件中修改args
在deployment.containers部分增加args:  

```
          - --tls-key-file=dashboard.key
          - --tls-cert-file=dashboard.crt
```

## 查看dashboard service
在本地hosts文件中将host域名加到本地IP上
## 查看token
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')  
# 官方部署本地
[Web UI](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/)  

## 应用yaml文件
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta4/aio/deploy/recommended.yaml
```

## 启动dashboard服务
```
kubectl proxy
```

## 从这里访问
```
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```




