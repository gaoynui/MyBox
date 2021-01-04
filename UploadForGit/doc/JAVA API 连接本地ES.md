## JAVA API 连接本地ES

### 启动ES

1.下载时确定好版本，7.5.1版的ES对应API最低要求6.8.0

2.设置环境变量$ES_HOME/bin

3.配置文件elasticsearch.yml

```
cluster.name: "elasticsearch"
node.name: "node-1"
network.host: 0.0.0.0
discovery.seed_hosts: ["127.0.0.1", "[::1]"]
cluster.initial_master_nodes: ["node-1"]  // 与前面的node.name对应

还可以设置允许跨域：
http.cors.enabled: true
http.cors.allow-origin: /.*/
```

4.启动

```
elasticsearch.bat
最后提示：
[node-1] publish_address {}, bound_addresses {}
[node-1] started
启动成功
```

5.测试

```
新开cmd
curl http://localhost:9200/?pretty //?pretty用于规范输出格式
出现：
{
  "name" : "node-1",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "3vNuy8IJR0eaiHfx6Y2CJQ",
  "version" : {
    "number" : "7.5.1",
    "build_flavor" : "default",
    "build_type" : "zip",
    "build_hash" : "3ae9ac9a93c95bd0cdc054951cf95d88e1e18d96",
    "build_date" : "2019-12-16T22:57:37.835892Z",
    "build_snapshot" : false,
    "lucene_version" : "8.3.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

### 使用JAVA API

1.创建gradle项目

```
配置文件添加以下依赖：
compile('org.elasticsearch.client:transport:6.8.0') //这里的版本时刻注意与ES版本对应，会看到上面的测试结果minimum_wire_compatibility_version : 6.8.0，所以最低API版本6.8.0
```

2.创建client

```
关键步骤：
// 配置集群名称和监听
Settings settings = Settings.builder()
						.put("cluster.name", [cluster.name])
						.put("client.transport.sniff", true).build();
// 创建client host为本地，port为9300
Client client = new PreBuiltTransportClient(settings).addTransportAddress(
                                new TransportAddress(InetAddress.getByName(host), port));
```

注意：

9200作用HTTP协议，主要用于外部通讯，但JAVA,SCALA 通过API访问时使用9300

9300作用TCP协议，jar之间就是通过TCP协议通讯，ES集群之间通过9300通讯

3.之后就是一系列增删改查API

https://blog.csdn.net/l1028386804/article/details/78758691

### 注意事项

1.ES版本和JAVA API版本的对应

2.端口的选择，是9200还是9300

3.配置文件中cluster.name,node.name对应，network.host必须为0.0.0.0

### JAVA后台访问ES架构

> dto-确定数据对象实体

```
@Document(indexName = "", type = "", shards = , replicas = )
public class EntityBean {
	@Id
	private Long id;
	
	@Field(type = FieldType.keyword)
	private String key;
}
```

> dao-对ES的查询语句接口

```
@Query("{\"bool\" : {\"must\" : {\"field\" : {\"FieldType.keyword\" : \"?\"}}}}")
Page<EntityBean> findByFieldType(String key, Pageable pageable);
```

> service-调用dao接口实现暴露服务

```
@Service("elasticService")
public Page<EntityBean> findByFieldType(String key) {
	return repository.findByFieldType(key, pageable);
}
```

> controller