## kafka消息队列

### 1.多范式编程语言

-  泛型编程

泛型编程是一种编程风格，其中算法以尽可能抽象的方式编写，而不依赖于将在其上执行这些算法的数据形式。泛型如C++的Template，Java的Generics；泛型编程的典型例子如C++的STL。

- Java中的泛型编程

Java中分泛型类和泛型方法。具有代表性的就是集合类。

java的泛型只可以表示类，不能表示单个对象。

```
List<Integer> list = new ArrayList<Integer>();
```

- 多范式编程语言

可以支持超过一种编程泛型的编程语言。

```
编程泛型：典型的编程风格，如函数式编程、过程式编程、面向对象编程、指令式编程、泛型编程等
```

### 2.Scala

运行于Java平台并兼容现有的Java程序。

多范式编程语言，设计初衷：面向对象编程+函数式编程

### 3.Kafka

 Kafka是由Apache软件基金会开发的一个开源流处理平台，由[Scala](https://zh.wikipedia.org/wiki/Scala)和[Java](https://zh.wikipedia.org/wiki/Java)编写。该项目的目标是为处理实时数据提供一个统一、高吞吐、低延迟的平台。其持久化层本质上是一个“按照分布式事务日志架构的大规模发布/订阅消息队列”，这使它作为企业级基础设施来处理流式数据非常有价值。此外，Kafka可以通过Kafka Connect连接到外部系统（用于数据输入/输出），并提供了Kafka Streams——一个Java流式处理库。 

<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/6/64/Overview_of_Apache_Kafka.svg/600px-Overview_of_Apache_Kafka.svg.png" alt="Kafka架构" style="zoom:67%;" />

（Topic： 用来对消息进行分类，每个进入到Kafka的信息都会被放到一个Topic下。

 Partition： 每个Topic中的消息会被分为若干个Partition，以提高消息的处理效率 ）

- 四个主要API

生产者API： 支持应用程序发布Record流。Topic由Record组成，Record持有不同的信息。

消费者API：支持应用程序订阅Topic和处理Record流。

Stream API：将输入流转换为输出流，并产生结果。

Connector API：执行可重用的生产者和消费者API，可将Topic链接到现有的应用程序。

### 4.安装并启动Kafka

(1)安装zookeeper

a. [下载地址](http://apache.communilink.net/zookeeper/) （下载bin压缩文件）

b.配置环境变量ZOOKEEPER_HOME为安装路径，PATH添加%ZOOKEEPER_HOME%\bin

c.修改conf下zoo_sample.cfg为zoo.cfg并修改以下参数：

```
# dataDir数据存放位置，dataLogDir日志存放位置
dataDir=D:\\zookeeper\\dataDir
dataLogDir=D:\\zookeeper\\dataLogDir
```

d.命令行运行zkServer.cmd

看到binding to port 0.0.0.0/0.0.0.0:2181说明启动成功

（2）安装Kafka

a.[下载地址]( https://www.apache.org/dyn/closer.cgi?path=/kafka/2.3.1/kafka_2.11-2.3.1.tgz )

b.修改conf/server.properties配置

```
# 日志存放位置
log.dirs=D:\\kafka\\logDir
# 连接zookeeper配置
zookeeper.connect=localhost:2181
# 监听端口
listeners=PLAINTEXT://localhost:9092
```

c.启动

在安装路径下打开命令行运行：

```
.\bin\windows\kafka-server-start.bat .\config\server.properties
```

看到INFO [KafkaServer id=0] started说明启动成功

d.创建topic,producer,consumer

topic:

```
# bin/windows目录下，test为topic名称
.\kafka-topics.bat --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
# 查看topic信息
.\kafka-topics.bat --zookeeper localhost:2181 --topic test --describe
参数：
Partition:
Leader:
```

producer:

```
# bin/windows目录下，test为生产者名称
 .\kafka-console-producer.bat --broker-list localhost:9092 --topic test
```

consumer:

```
# bin/windows目录下，test为消费者名称，必须与生产者位于同一端口，否则监听不到消息
 .\kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic test
```

在生产者处输入消息能在消费者接收到即显示成功。

### 5.Java实操

配置文件

```
    <dependencies>
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka_2.12</artifactId>
            <version>1.0.0</version>
            <scope>provided</scope>
        </dependency>

        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-clients</artifactId>
            <version>1.0.0</version>
        </dependency>

        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-streams</artifactId>
            <version>1.0.0</version>
        </dependency>
    </dependencies>
```

Producer.java

```
1.重写run函数创建线程
2.构造函数进行生产者参数配置
3.run函数发布消息
4.main创建线程
#--------------------------------
生产者参数（加*为必须）：
*bootstrap.servers: localhost:9062  // kafka交互地址
*acks: {0/1/all} // 为0，生产者不会等待kafka响应；为1，kafka会把消息写到本地日志中，但不会等待机器中                     其他集群的成功响应；为all，所有消息响应完成，确保消息不会丢失。
*retries: 0 // value大于0的话，当消息发送失败时会重新发送。
batch.size: 16384 // 当多条消息需要发送到同一个分区时，生产者会尝试合并网络请求。
*key.serializer: StringSerializer.class.getName() // 键序列化
*value.serializer: StringSerializer.class.getName() // 值序列化
```

Consumer.java

```
消费者参数（加*为必须）：
*bootstrap.servers: localhost:9062
*group.id: 组名 // 不同组名的消费者可以重复消费
enable.auto.commit: true // 是否自动提交，默认为true
auto.commit.interval.ms: 1000 // 从poll的回话处理时常
max.poll.records: 1000 // 一次最大拉取的条数
auto.offset.reset: {earliest/latest/none} // 从offset开始消费，否则从头开始/从offset开始消费，否                                              则从新产生的消息开始/从offset开始消费，如果有一个分                                              区不存在offset，抛出异常。
*key.deserializer: StringDeserializer.class.getName() // 键值序列化
*value.deserializer: StringDeserializer.class.getName() // 键值序列化
```

