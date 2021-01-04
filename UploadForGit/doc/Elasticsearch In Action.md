Elasticsearch In Action

------

#### Chapter 1 Introducing Elasticsearch

1.json格式的string值可以是日期，ES能够自动识别

```
{
	"timestamp": "2020-02-01T15:03:00"
}
```

2.Lucene&Solr

```
Lucene:是一个用于信息检索的工具库
Solr:是Lucene更高一级的封装，类似ES
```

#### Chapter 2  Diving into the Functionality

1.ES集群

<img src="C:\Users\yang10.gao\Desktop\doc\pics\ES集群.png" alt="ES集群" style="zoom:67%;" />

2.标识Document

```
index name + type name + document ID = uniquely document
```

3.Document的特点（内在一致性，可嵌套，结构灵活）

<img src="C:\Users\yang10.gao\Desktop\doc\pics\ES-document的特点.png" alt="ES-document的特点" style="zoom:67%;" />

4.mapping

```
对每个type里的fields的定义
如：name定义成string,地理坐标定义成geo_point
```

5.shards,replica和node的关系

<img src="C:\Users\yang10.gao\Desktop\doc\pics\ES-shards,replica和node的关系.png" alt="ES-shards,replica和node的关系" style="zoom:67%;" />

```
A shard is a Lucene index: a directory of files containing an inverted index. An inverted index is a structure that enable ES to tell you which document contains a term(a word) without having to look at all the documents.

shard分primary和replica
replica shard 是 primary shard 的副本
```

6.search支持多type

```
curl "localhost:9200/index/type1,type2/_search"
不同type下的同名field应该具有相同的设置，都为string或者其他类型

curl "localhost:9200/+get-toge*,-get-together/_search"
查询前缀带有get-toge的index,排除get-together
```

7.ES返回的数据模板

<img src="C:\Users\yang10.gao\Desktop\doc\pics\ES-返回的数据模板.png" alt="ES-返回的数据模板" style="zoom:67%;" />

8.过滤模板

```
{
	"query": {
		"result": {
			"filter": {
				"term": {
					"name": "A"
				}
			}
		}
	}
}
```

9.聚合模板

```
{
	"aggs": {
		"result": {
			"terms": {
				"field": "A"
			}
		}
	}
}

terms可改为min,max,sum,avg
```

#### Chapter 3 Indexing,updating,and deleting data

1.定义一个mapping

```
curl -XPUT 'localhost:9200/index/_mapping/type' -d '{
	"type": {
		"properties": {
			"field": {
				"type": "string"
			}
		}
	}
}'

不能改变一个已经存在的mapping,可以清除对应type下所有的data，重新生效新mapping
```

2.核心field类型

string

numeric

date

boolean

3.Analyzer

<img src="C:\Users\yang10.gao\Desktop\doc\pics\ES-analyzer.png" alt="ES-analyzer" style="zoom:67%;" />

```
默认的，index会进行analyze，如果希望某field不被analyze，需要这样设置：
{
	"type": {
		"properties": {
			"field": {
				"type": "string",
				"index": "not_analyzed"
			}
		}
	}
}
```

4.multi-field for a string

```
{
	"type": {
		"properties": {
			"fieldName1": {
				"type": "string",
				"index": "analyzed",
				"fields": {
					"fieldName2": {
						"type": "string",
						"index": "not_analyzed"
					}
				}
			}
		}
	}
}
```

5.预定义域（predefined fields）

_source 存储索引到的原始json文件

_all 搜索所有fields

_timestamp 记录文档被索引的日期

_ttl time to live 文档的有效期

include_in_all 是否包含在_all中

6.更新已有的document

<img src="C:\Users\yang10.gao\Desktop\doc\pics\ES-updatingExistingDocuments.png" alt="ES-updatingExistingDocuments" style="zoom: 50%;" />

```
curl -XPOST 'localhost:9200/index/type/id/_update' -d '{
	"doc": {
		"field": "change"
	}
}'
```

7.更新文档并使用script

```
curl -XPUT 'localhost:9200/shop/shirts/1' -d '{
	"price": 15
}'

curl -XPOST 'localhost:9200/shop/shirts/1/_update' -d '{
	"script": "ctx._source.price += price_diff",
	"params": {
		"price_diff": 10
	}
}'
```

8.删除文档

```
删除单个文档：
curl -XDELETE 'localhost:9200/index/type/id'

删除一个mapping type:
curl -XDELETE 'localhost:9200/index/type'
```

9.开启/关闭一个index

```
curl -XPOST 'localhost:9200/index/_open'
                                  _close
```

#### Chapter 4 Searching your data

1.search request 各大组件

```
1.query 使用query DSL & filter DSL // query_string无区别查找该字段
2.size 代表要返回的文件数
3.from 与size一起使用，用来进行分页 // from=10&size=10从第10个开始返回十个
4._source 返回的原始数据 // 自定义_search?_source=title,publishyear 
  或者在query里添加:"_source": ["title","publishyear"]
5.sort默认的规则以score排序
```

<img src="C:\Users\yang10.gao\Desktop\doc\pics\ES-_source自定义返回.png" alt="ES-_source自定义返回" style="zoom:67%;" />

2.match query

operator:

<img src="C:\Users\yang10.gao\Desktop\doc\pics\ES-matchQueryWithOperatorAnd.png" alt="ES-matchQueryWithOperatorAnd" style="zoom:67%;" />

type:

phrase

phrase_prefix

3.range

```
{
	"query": {
		"range": {
			"score": {
				"gte": "8",
				"lte": "9"
			}
		}
	}
}
```

4.wildcard 使用通配符

```
{
	"query": {
		"wildcard": {
			"fieldName": {
				"wildcard": "西*记"
				//"wildcard": "西?记"
			}
		}
	}
}

*不限制字数
？只占一位
```

5.query类型的选择

<img src="C:\Users\yang10.gao\Desktop\doc\pics\ES-query类型的选择.png" alt="ES-query类型的选择" style="zoom:67%;" />

#### Chapter 5 Analyzing your data

1.custom analyzer 定制分析器工作原理

<img src="C:\Users\yang10.gao\Desktop\doc\pics\ES-customAnalyzer.png" alt="ES-customAnalyzer" style="zoom:67%;" />

character filtering -> breaking into tokens -> token filtering -> token indexing

2.创建index同时设置分析器

<img src="C:\Users\yang10.gao\Desktop\doc\pics\ES-index创建时添加customAnalyzer1.png" alt="ES-index创建时添加customAnalyzer1" style="zoom:67%;" />

<img src="C:\Users\yang10.gao\Desktop\doc\pics\ES-index创建时添加customAnalyzer2.png" alt="ES-index创建时添加customAnalyzer2" style="zoom:67%;" />

3.specifying the analyzer for a field in the mapping

```
{
	"mappings": {
		"document": {
			"properties": {
				"description": {
					"type": "string",
					"analyzer": "myCustomAnalyzer"
				}
			}
		}
	}
}
```

curl -XPOST "localhost:9200/index/_analyze?analyzer=myCustomAnalyzer" -d "this is a string"

#### Chapter 6 Searching with relevancy

1.相关方法

BM25 改进版TF-IDF

TF-IDF（词频-逆文档频率）用于去除高频无用词-用于减少通用词的排名

2.boosting

提升算法

3.function_score

[官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-function-score-query.html)

#### Chapter 7 Exploring your data with aggregations

1.facets

类似于aggregations但已经被取代的技能。

2.格式

```
{
	"aggs": {
		"top_tags": {              // aggs的名字
			"terms": {             // aggs的类型
				"field": ""
			}
		}
	}
}

reply:
{
	"hits": {
		"total": 5,
		"max_score": 1.0,
		"hits": [{
			"name": ""
			},
			"aggregations": {         // aggs部分
				"top_tags": {         // aggs名字
					"buckets": [{     // 存放结果
						"key": "big data",
						"doc_count": 3
						}, {
						"key": "open source",
						"doc_count": 3
						}
					}]
				}
			}
		}]
	}
}
```

3.带有filter的aggs

<img src="C:\Users\yang10.gao\Desktop\doc\pics\ES-带有filter的aggs.png" alt="ES-带有filter的aggs" style="zoom:67%;" />

4.博客

[聚合](https://www.cnblogs.com/xiaobaozi-95/p/9197616.html)

#### Chapter 8 Relations among documents

1.子文档

```
{
	"bool" : {
		"must": [
		{
			"term": {
				"parentDoc.field": ""
			}
		},{
			"match_phrase": {
				"title": "熊出没"
			}
		}
		]
	}
}
```

2.query

has_parent

has_child

_parent

#### Chapter 9 Scaling out

1.索引别名

索引别名就像一个快捷方式或软连接，可以指向一个或多个索引。

_alias用于单个操作，   _aliases用于执行多个原子级操作。

```
curl -XPUT 'http://localhost:9200/tcl_album/_alias/tcl_all'
curl -XPUT 'http://localhost:9200/tcl_episodes/_alias/tcl_all'
```

检测这个别名指向哪些索引：

GET /*/_alias/tcl_all

查看哪些别名指向这个索引：

GET /tcl_all/_alias/*

#### Chapter 10 Improving performance