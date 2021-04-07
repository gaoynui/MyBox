## redis

特点：

- key-value存储系统

- 所有操作均在内存完成

- 去中心化分布集群

- 支持持久化

  ```
  Redis有两种持久化的方式：快照（RDB:redis db）和追加式文件（AOF）：
  
  RDB持久化方式会在一个特定的间隔保存那个时间点的一个数据快照。
  AOF持久化方式则会记录每一个服务器收到的写操作。在服务启动时，这些记录的操作会逐条执行从而重建出原来的数据。写操作命令记录的格式跟Redis协议一致，以追加的方式进行保存。
  Redis的持久化是可以禁用的，就是说你可以让数据的生命周期只存在于服务器的运行时间里。
  两种方式的持久化是可以同时存在的，但是当Redis重启时，AOF文件会被优先用于重建数据。
  ```

  

启动：

服务器：redis-server.exe

客户端：redis-cli.exe -h hostIP -p port

## 配置

配置文件：redis.conf/redis.windows.conf

```
通过客户端进入控制台
127.0.0.1:6379>config get * 
127.0.0.1:6379>config set
```



## 命令

```
# string: 设置键值对
>set key value
>get key
# 删除键值对
>del key

# Hash: 设置一个key对应多个value
>hmset key field1 value1 field2 value2
>hget key field1

# list: 以列表形式存储值
>lpush key value1
>lpush key value2
>lpush key value3

>lrange key 0 len

# set: 元素唯一
>sadd key value1
>sadd key value2

>smembers key

# sorted set
>zadd key score value1
>zadd key score value2

>zrangebyscore key 0 1000

# 查看服务是否运行
>ping
pong
# 获取服务器统计信息
>info
# 设置密码
>config set requirepass "password"
# 授权登录
>auth "password"
```

### 事务

打包的批量执行脚本，非原子性

### 备份和恢复

```
# 在redis安装目录中创建dump.rdb文件
>SAVE
# 恢复
将dump.rdb文件移动到redis安装目录并启动服务。
获取redis安装目录：
>config get dir
```

### 名词

1.cap

```
Consistency: 一致性
更新操作成功并返回客户端后，所有节点在同一时间的数据完全一致。
Availability: 可用性
服务一直可用，在正常响应时间
Partition Tolerance: 分区容错性
分布式系统在遇到某个节点或网络分区故障的时候，仍然能够对外提供满足一致性或可用性的服务。
```

# Spring Boot 配置redis

依赖:

org.springframework.boot:spring-boot-starter-data-redis

org.apache.commons.commons-pool2

yml配置:

```
spring:
  # REDIS (RedisProperties)
  redis:
    # Redis数据库索引（默认为0）
    database: 1
    # Redis服务器地址
    host: ${REDIS_HOST}
    # Redis服务器连接端口
    port: 6379
    # Redis服务器连接密码（默认为空）
    password: ${REDIS_PASSWORD}
    # 连接超时时间（毫秒）
    timeout: 50s
    # 连接池
    jedis:
      pool:
        # 连接池最大连接数（使用负值表示没有限制）
        maxActive: 1000
        # 连接池最大阻塞等待时间（使用负值表示没有限制）
        maxWait: -1s
        # 连接池中的最大空闲连接
        maxIdle: 8
        # 连接池中的最小空闲连接
        minIdle: 5
```

@Cacheable(value=cacheName)

表示当前方法的返回值是会被缓存到哪个cache上，对应cache的cacheName