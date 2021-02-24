# docker学习

## 完整教程
教程一：

[安装和使用](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-18-04#step-4-%E2%80%94-working-with-docker-images)

教程二：

[简单教程](https://docs.khadas.com/vim1/InstallDocker.html)

## docker基本操作
```

docker stop bfd094233f96   #停止一个容器，根据容器 id 进行删除

docker rm [container ID]   #删除一个容器
 
docker rmi [image ID]  #删除一个镜像，需要先删除其容器 

docker attach [container ID]  #进入一个正在运行的容器

docker save -o name os   #存储镜像

docker ps -a 显示所有容器ID

docker images 显示镜像
```

```
下载docker:
docker pull [repository]

上传docker:
docker push [repository]:[tag]

登录docker hub:
docker login -u [username]

修改仓库和标签信息：
docker tag [imageID] repository:tag

查询相关镜像：
docker search [repository]
```

## build

- 编写Dockerfile:


FROM [存在的docker镜像]

```
FROM [image]:[tag]
```

MAINTAINER [维护者信息]

```
MATINTAINER [name]
```

RUN [build时需要进行的操作]

```
# 调用/bin/sh -c [command]
RUN [COMMAND]
# 以避免有些时候shell方式执行时的传递参数问题，而且有些基础镜像可能不包含/bin/sh
RUN ["executable", "param1", "param2"]
```

ENV [配置自动设置的环境变量]

ADD[从指定路径拷贝一个文件或目录到容器的指定路径中]

```
ADD [src] [dest]
```

CMD [镜像在完成环境配置后要运行的最后命令]

ex1:

```
# 给docker/whalesay镜像添加fortunes
FROM docker/whalesay:latest
RUN apt-get -y update && apt-get install -y fortunes
CMD /usr/games/fortune -a | cowsay
```

ex2:

```
FROM busybox
ADD myTodoLists-0.0.4-SNAPSHOT.jar .
RUN mkdir -p /app
```

- 编译：


在存Dockerfile文件夹下运行：

```
docker build -t [生成的镜像名称] .
```

- 执行：


```
docker run [生成的镜像名]
```

### docker镜像生命轨迹

![](C:\Users\yang10.gao\Desktop\MyBox\UploadForGit\doc\pics\docker\docker镜像的生命轨迹.jpg)

### docker搭建单节点本地服务器集群架构

<img src="C:\Users\yang10.gao\Desktop\MyBox\UploadForGit\doc\pics\docker\docker搭建单节点本地服务器集群架构.jpg" style="zoom:80%;" />