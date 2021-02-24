Elasticsearch In Action

------

```
倒排索引(反向索引):
下面三个文件:
Document0 = "it is what it is"
Document1 = "what is it"
Document2 = "it is a document"

每个元素对文件的索引:
 "a":      {2}
 "document": {2}
 "is":     {0, 1, 2}
 "it":     {0, 1, 2}
 "what":   {0, 1}
 
 查询 "what is it":
 {0,1} ∩ {0,1,2} ∩ {0,1,2} = {0,1}
```

```
// match & match_phrase
match会分词，对全文只要发现和搜索条件相关的doc都会出现在最终结果里
match_phrase会将给定的短语当成一个完整的查询条件

// keyword & text
keyword类型不会建立分词，“张三李四”只会以“张三李四”一起查询
text会建立分词，“张三李四”会查询到“张三”和“李四”
如果匹配一个字段既能全词搜索也能分词搜索，可以新添加一个field，同原来field维护相同的数据

//************************
term和match_phrase作用于数据上，指定数据对应的字段是否为完全匹配。
加不加keyword作用在查询的query上，指定这个query是否需要分词。
如:
  term+title.keyword搜小猪佩奇,加keyword就只搜小猪佩奇，而term表示数据的title必须完全匹配小猪佩奇。
  term+title搜小猪佩奇，这里title为text类型，会进行分词，也就是现在title必须完全匹配小猪或者佩奇。
  match_phrase+title.keyword搜小猪佩奇，title必须完全匹配小猪佩奇。
  match_phrase+title搜小猪佩奇，title必须包含小猪佩奇。
```

```
Mysql      vs      ES
Database           Index                     库
Table              Type                      表/分类
Row                Document                  数据
Column             Field                     字段
Schema             Mapping                   映射关系
```

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

<img src="C:\Users\yang10.gao\Desktop\MyBox\UploadForGit\doc\pics\ES\ES集群.png" style="zoom:67%;" />

2.标识Document

```
index name + type name + document ID = uniquely document
```

3.Document的特点（内在一致性，可嵌套，结构灵活）

<img src="C:\Users\yang10.gao\Desktop\MyBox\UploadForGit\doc\pics\ES\ES-document的特点.png" style="zoom:67%;" />

4.mapping

```
对每个type里的fields的定义
如：name定义成string,地理坐标定义成geo_point
```

5.shards,replica和node的关系

<img src="C:\Users\yang10.gao\Desktop\MyBox\UploadForGit\doc\pics\ES\ES-shards,replica和node的关系.png" style="zoom:67%;" />

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

<img src="C:\Users\yang10.gao\Desktop\MyBox\UploadForGit\doc\pics\ES\ES-返回的数据模板.png" style="zoom:67%;" />

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

<img src="C:\Users\yang10.gao\Desktop\MyBox\UploadForGit\doc\pics\ES\ES-analyzer.png" style="zoom:67%;" />

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

<img src="C:\Users\yang10.gao\Desktop\MyBox\UploadForGit\doc\pics\ES\ES-updatingExistingDocuments.png" style="zoom:67%;" />

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

<img src="C:\Users\yang10.gao\Desktop\MyBox\UploadForGit\doc\pics\ES\ES-_source自定义返回.png" style="zoom:67%;" />

2.match query

operator:

![](C:\Users\yang10.gao\Desktop\MyBox\UploadForGit\doc\pics\ES\ES-matchQueryWithOperatorAnd.png)

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

<img src="C:\Users\yang10.gao\Desktop\MyBox\UploadForGit\doc\pics\ES\ES-query类型的选择.png" style="zoom:67%;" />

#### Chapter 5 Analyzing your data

1.custom analyzer 定制分析器工作原理

<img src="C:\Users\yang10.gao\Desktop\MyBox\UploadForGit\doc\pics\ES\ES-customAnalyzer.png" style="zoom:67%;" />

character filtering -> breaking into tokens -> token filtering -> token indexing

2.创建index同时设置分析器

<img src="C:\Users\yang10.gao\Desktop\MyBox\UploadForGit\doc\pics\ES\ES-index创建时添加customAnalyzer1.png" style="zoom:67%;" />

<img src="C:\Users\yang10.gao\Desktop\MyBox\UploadForGit\doc\pics\ES\ES-index创建时添加customAnalyzer2.png" style="zoom:67%;" />

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

<img src="C:\Users\yang10.gao\Desktop\MyBox\UploadForGit\doc\pics\ES\ES-带有filter的aggs.png" style="zoom:67%;" />

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