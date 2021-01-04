## Hadoop

​                                                                                  hadoop架构

![](https://pennywong.gitbooks.io/hadoop-notebook/content/pic/architecture.png)

### 1.Zookeeper

（1）开源的分布式协调服务，本身就是一个分布式程序，只要半数以上节点存活Zookeeper就能正常服务。

（2）将数据保存在内存中，保证了高吞吐和低延迟，但存储数量有限。

（3）Leader选举算法：zab协议（zookeeper atomic brodcast）。

- Zab协议

原理：主控制写同步从

四个阶段：选举、发现、同步、广播。

​					选举：当集群新建或主机死机，或主机与一半以上的从机失联后触发选择新的主机。有两种选举算法                       			                   fast paxos和basic paxos。

基本概念：leader、follower、observer

节点状态：looking、leading、following



### 2.HDFS（分布式文件系统  Hadoop Distributed File System）

- 分布式和集中式和集群

集中式：由位于系统中心的服务器统一管理全部共享资源并处理来自所有用户的请求。

集群：分布式将不同的业务分布在不同的地方，集群将几台服务器集中在一起实现同一业务。集群有一个组织性，一个服务器挂了其他的服务器可以顶上来，而分布式每一个节点都可能进行不同的业务，某一节点垮了其业务就不可访问。

分布式：（1）分布性：系统中多台计算机之间没有主从之分。

​               （2）透明性：系统资源被所有计算机共享，包括CPU，文件，打印机等。

​               （3）同一性：系统中若干台计算机可以相互协作来完成一个共同任务，或者说一个程序能够分布在几台计算机上并行运行。

​               （4）通信性：系统中任意两台计算机可以通过通信来交换信息。

- hadoop

一个开源框架，允许使用简单的编程模型在跨计算机集群的分布环境中存储和处理大数据。从单个服务器扩展到数千个机器，每个都提供本地计算和存储。

- Distributed

分布式计算是利用互联网上计算机的CPU共同处理能力来解决大型计算问题。

- HDFS

HDFS将巨大的数据变成大量数据的数据——元数据。

 元数据是用于描述要素、数据集或数据集系列的内容、覆盖范围、质量、管理方式、数据的所有者、数据的提供方式等有关的信息。 

<img src="https://pic4.zhimg.com/80/v2-78957dd1a3c86068730f36c109e2aca7_hd.jpg" style="zoom:67%;" />

模块：

Block数据块：（1）基本存储单位和读写单位，一般大小为64M，每次都是读写一个块。

​                           （2）配置大块为了减少搜寻时间（硬盘的传输速率大于寻道时间，为减少寻道时间）；为了减少                  		                             块管理开销，每个块都需要在namenode记录。

​                           （3）一个大文件会被拆分成多个块，存储于不同的机器；如果一个文件少于Block大小，实际占                         									用空间为其文件大小。

​						   （4）每个块都被分配到多个机器，默认复制3份。

NameNode：（1）存储元数据，运行时所有数据都保存到内存，整个HDFS可存储的文件数受限于NameNode的	                              内存大小。失效则整个HDFS失效。

​                        （2）与DataNode相关的信息并不保存到NameNode文件系统中，而是NameNode每次重启后动 						          态重建。

Secondary NameNode： 定时与NameNode进行同步（定期合并文件系统镜像和编辑日志，然后把合并后的传			                                 给NameNode，替换其镜像，并清空编辑日志，类似于CheckPoint机制），但NameNode失效后仍需要手工将其设置成主机 。

DataNode：（1）存放Block块，负责其读写和赋值操作。

​                      （2）DataNode之间会进行通信，赋值数据块保证数据的冗余性。

- HDFS写文件

![](https://atts.w3cschool.cn/attachments/image/wk/hadoop/hdfs-write.png)

1.客户端将文件写入本地磁盘的HDFS Client文件中

2.当临时文件大小达到一个block大小时，HDFS client通知NameNode，申请写入文件

3.NameNode在HDFS的文件系统中创建一个文件，并把该block id和要写入的DataNode的列表返回给客户端

4.客户端收到这些信息后，将临时文件写入DataNodes

- 4.1 客户端将文件内容写入第一个DataNode（一般以4kb为单位进行传输）
- 4.2 第一个DataNode接收后，将数据写入本地磁盘，同时也传输给第二个DataNode
- 4.3 依此类推到最后一个DataNode，数据在DataNode之间是通过pipeline的方式进行复制的
- 4.4 后面的DataNode接收完数据后，都会发送一个确认给前一个DataNode，最终第一个DataNode返回确认给客户端
- 4.5 当客户端接收到整个block的确认后，会向NameNode发送一个最终的确认信息
- 4.6 如果写入某个DataNode失败，数据会继续写入其他的DataNode。然后NameNode会找另外一个好的DataNode继续复制，以保证冗余性
- 4.7 每个block都会有一个校验码，并存放到独立的文件中，以便读的时候来验证其完整性

5.文件写完后（客户端关闭），NameNode提交文件（这时文件才可见，如果提交前，NameNode垮掉，那文件也就丢失了。fsync：只保证数据的信息写到NameNode上，但并不保证数据已经被写到DataNode中）

- HDFS读文件

![](https://atts.w3cschool.cn/attachments/image/wk/hadoop/hdfs-read.png)

1. 客户端向NameNode发送读取请求
2. NameNode返回文件的所有block和这些block所在的DataNodes（包括复制节点）
3. 客户端直接从DataNode中读取数据，如果该DataNode读取失败（DataNode失效或校验码不对），则从复制节点中读取（如果读取的数据就在本机，则直接读取，否则通过网络读取）

### 3.MapReduce

MapReduce是一种非常简单又非常强大的编程模型。

简单在于其编程模型只包含map和reduce两个过程，map的主要输入是一对<key , value>值，经过map计算后输出一对<key , value>值；然后将相同key合并，形成<key , value集合>；再将这个<key , value集合>输入reduce，经过计算输出零个或多个<key , value>对。

![](https://user-gold-cdn.xitu.io/2019/4/4/169e3f402d44ab55?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 4.YARN

将资源调度功能从MapReduce中脱离出来单独设计的模块。

![](https://pic2.zhimg.com/80/v2-b5804eb2bb1b3f2ff091c18bbf915819_hd.jpg)

模块：

Container:对资源的一层抽象。

三大组件：

**ResourceManager**:整个系统只有一个RM负责资源调度，包含了两个子组件：定时调用器(Scheduler)和应用管理器(ApplicationManager)。

定时调度器： 它只负责向应用程序分配资源，当 Client 提交一个任务的时候，它会根据所需要的资源以及当前集群的资源状况进行分配。

应用管理器：负责监控应用的工作，管理Client用户提交的应用。

**ApplicationMaster**： 每当 Client 提交一个 Application 时候，就会新建一个 ApplicationMaster 。由这个 ApplicationMaster 去与 ResourceManager 申请容器资源，获得资源后会将要运行的程序发送到容器上启动，然后进行分布式计算。 数据量太大时移动数据成本很高，所以选择移动应用发送到对应容器里启动。

**NodeManager**: ResourceManager 在每台机器的上代理，负责容器的管理，并监控他们的资源使用情况（cpu，内存，磁盘及网络等），以及向 ResourceManager/Scheduler 提供这些资源使用报告。 

### 5.部署单机模式和伪分布模式Hadoop

（1）准备：

```
配置：
ubuntu18.0
jdk1.8.0_231
hadoop-2.10.0
教程：
https://blog.csdn.net/hitwengqi/article/details/8008203
资源地址：
https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
https://www.apache.org/dyn/closer.cgi/hadoop/common/hadoop-2.10.0/hadoop-2.10.0.tar.gz
```

（2）安装ssh

```
# 下载
sudo apt-get install openssh-server
# 启动
sudo /etc/init.d/ssh start
# 添加用户
sudo vi /etc/ssh/sshd_config
AllowUsers 用户名
# 查看是否启动
ps -e | grep ssh
出现sshd和ssh-agent说明启动成功
# 设置免密登录ssh
ssh-keygen -t rsa -P ""
会在.ssh文件下生成两个密钥文件id_rsa和id_rsa.pub
# 将公钥追加到authorized_keys中
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
# 重启ssh
service sshd restart
# 登录本地
ssh localhost
```

(3) 安装java

(4)安装hadoop

```
# 将下载解压后的文件移到指定目录
sudo mv hadoop-2.10.0 /home/hadoop
# 确保所有操作都在指定用户下完成（可以跳过）
sudo chown -R groupId:User /home/hadoop
# 设定hadoop-env.sh
cd /home/hadoop/hadoop-2.10.0/etc/hadoop
# 打开hadoop-env.sh文件
sudo vi hadoop-env.sh
# 在Java路径处添加环境变量
export JAVA_HOME=
export HADOOP_HOME=/home/hadoop/hadoop-2.10.0
export PATH=$PATH:${HADOOP_HOME}/bin
# 让环境变量生效
source hadoop-env.sh
# 查看hadoop版本
hadoop version
若出现版本信息则说明单机模式部署成功
# 测试单机模式下的mapreduce自带例子wordcount
mkdir input
# 将任意文件加入input中，运行wordcount程序
bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.10.0.jar wordcount input output
# 查看统计结果
cat output/*
```

(5)单机模式基础上进行伪分布模式配置

```
# 在bin同级目录下创建以下文件夹
mkdir tmp
mkdir hdfs
mkdir hdfs/name
mkdir hdfs/data
# 将data权限改为755
sudo chmod 755 data
# 编译这三个文件将hadoop不同模块对应到上面文件夹core-site.xml,hdfs-site.xml,mapred-site.xml
具体内容见教程
# 再次应用hadoop-env.sh配置文件
source hadoop-env.sh
# 启动hadoop
格式化namenode: bin/hadoop namenode -format
启动all: sbin/start-all.sh
查看守护进程：jps
如果出现nodemanager,namenode,datanode,resourcemanager,secondnamenode则说明成功
# 再次测试wordcount
# 这次在dfs中创建input
bin/hadoop dfs -mkdir -p input
# 将文件放入Input并允许wordcount
bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.10.0.jar wordcount input output
# 查看结果
bin/hadoop dfs -cat output/*
# 关闭所有hadoop
sbin/stop-all.sh
```

# HDFS 命令

```
前置命令分：
hadoop fs 
hdfs dfs
fs可操作任何文件系统，dfs只能操作HDFS文件系统相关。如果Hadoop本地模式中fs是local file system只能用fs
```

```
1.列出HDFS上的文件
hadoop fs -ls
未带参数的-ls默认是home目录下的内容。在hdfs中没有当前目录这个概念，也没有cd这个命令
hadoop fs -ls -R 路径
递归列出文件目录
2.删除HDFS上的文件
hadoop fs -rm -r fileName
3.创建HDFS文件夹
hadoop fs -mkdir /gy
创建文件
hadoop fs -touch /gy/test.txt
4.查看HDFS上的文件
hadoop fs -cat fileName
5.上传文件到HDFS
hadoop fs -put 本地文件 目的文件
6.下载文件到本地
hadoop fs -get 源文件 目的文件
7.查看HDFS状态
hadoop dfsadmin -report
8.查看文件大小
hadoop fs -du -h /file
```

