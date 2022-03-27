


```
使用bulk批量api可以一次请求提交多个，缩短花费时间。

关于readtimeout，在规定timeout时间内，无法发送完所有数据，es无法处理完发送给他的数据并返回正确信息，客户端直接抛出超时异常。
花费时间有：1发送数据到服务端的花费，当数据很大时，占比也很大2服务器解析json，当json解析占用了很多内存，资源不足就会connection refused。
对于大量数据可能会readtimeout，措施如下：
1控制客户端发送批次的大小，每个批次指定文件数量或批次文件总大小。如100个文件或50M发送一次。实际上按照大小分批后就不会报错了，分批次保证了每次请求50M，在timeout时间内能传送完。
2扩充服务端内存，网络带宽等。
3针对可connection refused只能失败重传,分批次提交，并进行失败重传，注意重传覆盖数据是否影响功能。

```

api document:https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-get-mapping.html

# python client



##### 查询所有id

```
body = {
        "_source":"false",
        "query": {
            "match_all": {}
        }
    }
hits = es.search(index="cti", body=body, size=10000)['hits']['hits']
```

##### 删除

```
   # es.get(index="myindex", id=1)['_source']
es.delete(index='indexName', doc_type='typeName', id='idValue')
```



##### problem

###### 超时

```
  es.index(index="cti", id=md5, body=data, request_timeout=60)加上request_timeout
```



# curl指令

```
https://www.cnblogs.com/shanhua-fu/p/10429417.html
```

##### 插入数据

```
curl -XPOST ‘http://localhost:9200/{index}/{type}/{id}’ -d'{“a”:”avalue”,”b”:”bvalue”}’
```

##### curl GET

```
#_doc类型字段不影响查询结果
GET /customer/_doc/1

curl -X GET "hbase2:9200/customer/_doc/1?pretty"

curl -X PUT "localhost:9200/customer/_doc/1?pretty" -H 'Content-Type: application/json' -d'
{
  "name": "John Doe"
}

curl -X GET 10.44.99.102:9200/situation-event/_refresh


```

```
curl -X POST "localhost:9200/_bulk?pretty" -H "Content-Type: application/json;charset=UTF-8" --data-binary @5eb920947209f2ced78b15a4.json 
```

##### curl PUT

```
PUT /customer/_doc/1
{
  "name": "John Doe"
}
curl -X PUT "localhost:9200/customer/_doc/1?pretty" -H 'Content-Type: application/json' -d'
{
  "name": "John Doe"
}
'
```

插入数据

```
/cti/_mapping
```



```
curl -X GET "172.18.65.185:9200/cti/_count"
搜索id
curl -X GET "172.18.65.185:9200/cti/_search/232daf111111111111"
```



##### es bulk 错误

```
bulk通过换行符分割传输的多个数据，如果json数据中带有换行符就会导致报错，干扰正确的分割。
```

```
可能问题，result_window,sysctl_max_heap
```

```
#search_after
https://blog.csdn.net/zzh920625/article/details/84593590
https://www.elastic.co/guide/en/elasticsearch/reference/6.7/search-request-search-after.html
GET twitter/_search
{
    "size": 10,
    "query": {
        "match" : {
            "title" : "elasticsearch"
        }
    },
    "search_after": [1000],
    "sort": [
        {"date": "asc"},
        {"tie_breaker_id": "asc"}
    ]
}
```



```
分页,临时解决如下：完全解决使用scroll和scroll-scan
PUT cti/_settings
{
  "index":{
    "max_result_window":1000000
  }
}
```





# es

>es的确是非主键检索，全文检索，但是检索出来后呢，并不是所有数据都放在es中，因为es不支持复杂聚合操作、多表联查，所以不适合业务数据的关系建模方法，以及复杂聚合统计的维度建模方法，那么很多数据其实是放在其他数据库中的，如业务数据、聚合统计数据等。当全文搜索后，需要拼接其他数据时，再使用主键在其他数据库进行检索。

### 聚合查询

##### 统计malware组件没有malwaretypes的数量

```
GET /myindex/_search 
{
  "size":0,
  "query": {
    "term": {
      "objects.type.keyword": {
        "value": "malware"
      }
    }
  },
  "aggs": {
    "group_by_date": {
      "terms": {
        "script": {
          "lang":"painless",
          "inline": """if (doc['objects.malware_types.keyword'].length==0){return 'other'+"####"+doc["objects.created"].value.toString();} """
        },
        "size":240,
        "shard_size": 240
      }
    }
  }
}
```



##### 针对report组件进行统计

```
#从oname中统计malware_types分布
GET /myindex/_search 
{
  "size":0,
  "query": {
    "term": {
      "objects.type.keyword": {
        "value": "report"
      }
    }
  },
  "aggs": {
    "group_by_date": {
      "terms": {
        "script": {
          "lang":"painless",
          "inline": """def name=doc['objects.name.keyword'].value;def loc=/Type-\[(.*?)\]/.matcher(name);if (loc.find()){ def loc_str=loc.group(1).replace(" ","");ArrayList array=new ArrayList();String splitter=",";StringTokenizer tokenValue=new StringTokenizer(loc_str,splitter);def h_time=/<<([0-9-:])>>/.matcher(name);while(tokenValue.hasMoreTokens()){array.add(tokenValue.nextToken()+"####"+doc["objects.created"].value.toString().substring(0,10));}if (array.size()>0){return array}}"""
        },
        "size":240,
        "shard_size": 240
      }
    }
  }
}
```



```
#object name中统计country分布
GET /myindex/_search 
{
  "size":0,
  "query": {
    "term": {
      "objects.type.keyword": {
        "value": "report"
      }
    }
  },
  "aggs": {
    "group_by_date": {
      "terms": {
        "script": {
          "lang":"painless",
          "inline": """def name=doc['objects.name.keyword'].value;def loc=/LOC-\[(.*?)\]',T/.matcher(doc['objects.name.keyword'].value);if (loc.find()){ def loc_str=loc.group(1).replace(" ","");ArrayList array=new ArrayList();String splitter=",";StringTokenizer tokenValue=new StringTokenizer(loc_str,splitter);while(tokenValue.hasMoreTokens()){array.add(tokenValue.nextToken());}if (array.size()>0){return array}}"""
        },
        "size":240,
        "shard_size": 240
      }
    }
  }
}
```



```
searchRequest
	sourceBuilder:from,size
		queryBuilders
```

```
聚合多个字段，例如聚合类别和日期
GET /myindex/_search 
{
  "size":0,
  "aggs": {
    "group_by_date": {
      "terms": {
        "script": "if(doc['objects.malware_types.keyword'].size()>0){return doc['objects.malware_types.keyword'].value+'####'+doc['objects.created'].value.toString().substring(0,10)}",
        "size":240,
        "shard_size": 240
      }
    }
  }
}
```



```
elasticsearch painless
```

```
#查询indicator，控制最终size和shard_size来控制精度。
GET /myindex/_search 
{
  "size":0,
  "query": {
    "term": {
      "objects.type.keyword": {
        "value": "indicator"
      }
    }
  },
  "aggs": {
    "group_by_date": {
      "terms": {
        "script": {
          "source": "def date=doc['objects.created'].value;def real_date=date.toString().substring(0,10);return real_date"
        },
        "size":240,
        "shard_size": 240
      }
    }
  }
}
```



```
"source": " ((doc['close_date'].value.toInstant().toEpochMilli() - doc['start_date'].value.toInstant().toEpochMilli()) / (3600000.0*24)) < 3.0 "
#基于query查询，转换日期为字符串截取并聚合排序
GET /myindex/_search 
{
  "size":0,
  "query": {
    "term": {
      "objects.malware_types.keyword": {
        "value": "unknow"
      }
    }
  },
  "aggs": {
    "group_by_date": {
      "terms": {
        "script": {
          "source": "def date=doc['objects.created'].value;def real_date=date.toString().substring(0,10);return real_date"
        }
      }
    }
  }
}
#指定时间段
{
  "query": {
    "bool": {
      "must": [
        {
          "query_string": {
            "qeury": "date:[2019-05-12 TO 2019-05-14]"
          }
        }
      ]
    }
  },
  "aggs": {
    "products": {
      "terms": {
        "field": "category",
        "size": 5
      }
    }
  }
}
#在query基础上进行聚合,对日期排序，每个分片上传最近8个月数据。
GET /myindex/_search 
{
  "size":0,
  "query": {
    "term": {
      "objects.malware_types.keyword": {
        "value": "unknow"
      }
    }
  },
  "aggs": {
    "group_by_date": {
      "terms": {
        "script": {
          "source": "def date=doc['objects.type.keyword'].value;def real_date=date.substring(1,2);return real_date"
        }
      }
    }
  }
}
```



```
elasticsearch 2中采用String类型，其分为analyzed_string和not_analyzed_string。not_analyzed对应3版本的keyword，analyzed对应3版本的text。text会对字符串分词，如new york会被拆分为new 和work。如果对text进行聚合查询，new和york会被分开统计。
在es3中自动映射为text的字段，会被添加一个fidld域，其中有keyword字段可用于聚合查询。
```

##### 查询type类型排行

```
#查询object type类型排行
GET /myindex/_search
{
  "size": 0,
  "aggs": {
    "popular_type": {
      "terms": {
        "field": "objects.type.keyword"
      }
    }
  }
}
#统计恶意软件架构
GET /myindex/_search
{
  "size": 0,
  "aggs": {
    "popular_type": {
      "terms": {
        "field": "objects.architecture_execution_envs.keyword"
      }
    }
  }
}
#统计攻击活动类别及时间，malware_types，first_seen,last_seen,漏洞定义规则再抽取。

#统计攻击活动地区
GET /myindex/_search
{
  "size": 0,
  "aggs": {
    "popular_type": {
      "terms": {
        "field": "objects.country.keyword"
      }
    }
  }
}
```



```
#构建索引
PUT /myindex
{
    "mappings": {
        "properties": {
          "objects":{
            "properties": {
              "type":{
                "type":"keyword"
              }
            }
          }
        }
    }
}
```



```
POST /myindex/_doc
{
  "properties":{
    "objects.type":{
      "type":"text",
      "fielddata":true
    }
  }
}
```



```
#分页查询
GET /myindex/_search?size=5&from=10000
```

##### 过滤查询

```
#查询type为malware的
POST /myindex/_search
{
"query": {
	"bool": {
		"must": [
			{
				"match": {
					"objects.type": "malware"
							}
				}
					]
	        }
        }
}
#统计type各种类型的个数
GET /myindex/_search
{
  "aggs" : {
    "stixtype" : {
      "terms" : {
         "field" : "objects.type",
         "size" :  10
      }
    }
  }
}
```



```
curl -X GET localhost:9200/_cat/indices?v
```



```
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "my_index",
        "alias": "my_index_alias"
      }
    }
  ]
}
```

## 安装

```
#docker安装
https://www.elastic.co/guide/en/elasticsearch/reference/7.5/docker.html
在宿主机上执行sysctl -w vm.max_map_count=262144命令，避免虚拟内存报错。
```

```
#修改系统配置，如系统内存限制、打开文件数量限制。
vim config/elasticsearch.yml
http.cors.enabled: true 
http.cors.allow-origin："*"
network.host: 0.0.0.0
discovery.seed_hosts: ["hbase2"]

vim config/jvm.options
-Xms8g
-Xmx8g

vim /etc/sysctl.conf
vm.max_map_count=655360
sysctl -p

vim /etc/security/limits.conf
* soft nofile 65536
* hard nofile 131072
* soft nproc 2048
* hard nproc 4096
ulimit -Sn/-Hn

vim /etc/security/limits.d/90-nproc.cnf
* soft nproc 4096
```



es 7.11

9200

/etc/sysconfig/network-scripts/ifcfg-exxx，

ip addr查看网卡

单节点配置文件

elasticsearch.yml

```
http.cors.enabled: true
http.cors.allow-origin: "*"
network.host: 0.0.0.0
discovery.seed_hosts: ["hbase2"]
```

java.options

```
-Xms8g
-Xmx8g
```

# es-head插件

```
vim config/elasticsearch.yml
http.cors.enabled: true 
http.cors.allow-origin："*"
network.host: 0.0.0.0
discovery.seed_hosts: ["hbase2"]
```



# Kibana

## 安装配置kibana

修改ip

```
vim $ES_HOME/config/elasticsearch.yml
hostname 0.0.0.0
vim KIBANA_HOME/config/kibana.yml
server.host: "this_host_id"
elasticsearch.hosts: ["http://hbase2:9200"]
bin/kibana --allow-root
```



```sh
./bin/elasticsearch -d -p pid
pkill -F pid
```

`git clone git://github.com/mobz/elasticsearch-head.git`

## 启动

```
elasticsearch -allow-root
```



## 语句

### 数据管理语句

创建空索引

```
PUT /haoke
{
	"settiongs":{
		"index":{
			"number_of_shards": "2",
			"number_of_replicas": "0"
		}
	}
}
```

删除索引

```
DELETE /haoke
```



### 数据**处理**语句

```
PUT /customer/_doc/1
{
  "name": "John Doe"
}
curl -X PUT "localhost:9200/customer/_doc/1?pretty" -H 'Content-Type: application/json' -d'
{
  "name": "John Doe"
}
'
```

#### 增

```导入数据
POST _reindex
{
  "source": {
    "index": "myindex"
  }
  , "dest": {
    "index": "cti"
  }
}

```

插入数据

POST /{索引}/{类型}/{id}

```
POST /haoke/user/1001
{
	"id":1001,
	"name":"bob",
	"age":20,
	"sex":"man"
}
```



#### 删

```
删除数据


DELETE /myindex/user/1001
DELETE  /cti_txt/_doc/999999999999999999



```

#### 改

更新数据

```
POST /myindex/user/1001/_update
{
	"doc":{
	   "age":23
	}
}
```

#### 查

##### **列查询**

###### 精确查询

GET

```
#_doc类型字段不影响查询结果
GET /customer/_doc/1

curl -X GET "hbase2:9200/customer/_doc/1?pretty"
```

```
#搜索数据
GET /myindex/user/1001
#所有数据数据,默认10条。需要分页
GET /myindex/user/_search
#
GET /myindex/user/_search?q=age:21
GET /myindex/user/_count
updated:13114  total:13156  myindex:13156
GET /cti/_search?q=_id=66d90772d57e5e72172186674fb79347
```

```
POST /cti/_search
{
    "size": "20",
	"query":{
	  	"match":{
	  		"objects.type": "indicator"
	  	}
	}
}
```



###### 范围搜索

```
POST /myindex/user/_search
{
	"query":{
		"bool":{
			"filter":{
				"range":{
					"age":{
						"gt": 30
							} 
						}
	  					},
	  		"must":{
	  			"match":{
	  				"sex":"man"
	  			}
	  		}
		}
	}
}
```

DSL搜索

```
#match是查询的一种，还有match_case等
POST /myindex/user/_search
{
	"query":{
	  	"match":{
	  		"age": 20
	  	}
	}
}
```



##### **聚合统计**

聚合

```
{
	"aggs": {
		"all_interests":{
			"terms": {
				"field": "age"
			}
		}
	}
}
```

查询objects中pattern为test的样本

两组bool-must-match,bool-filter-range

```
POST /myindex/user/_search
{
"query": {
	"bool": {
		"must": [
			{
				"match": {
					"objects.pattern": "test"
							}
				}
					]
}
}
}
```

java api

python scroll

```
https://kb.objectrocket.com/elasticsearch/elasticsearch-and-scroll-in-python-953#:~:text=There%20are%20three%20different%20ways%20to%20scroll%20Elasticsearch,scroll%20%28%29%20method.%20Scrolling%20Elasticsearch%20documents%20in%20Python
```

# 理论知识

### 基本架构

##### shard

```
相当于master
```

##### coordinate ndoe

```
相当于
```

### 基本数据类型

>主要类型：text,keyword,float,double,boolean,date,byte,short,integer,long
>#查看映射
>GET /gb/_mapping/tweet

### 性能

### 适用场景及优缺点

>适合全文检索，由给定字段查询所在文档，数据量大时需要特定api。适合灵活的节点水平扩展。
>不适合深度分页，因为其跳页实现繁琐。不适合数据频繁的修改，其修改本质是删除加插入操作形成的，高频率的数据增删容易触发段合并，即数据的重新组装。不适合事务操作，没有关系型数据库的事务和锁，难以应对一致性要求较高的场景。

### es分词器

```
#倒排索引
为了实现实时查询（1s以内），es使用倒排索引维护信息。信息包括一个列表，列表描述每个单词出现在哪些文档。当进行搜索时只需要搜索需要的文档。
例如：
文档1内容：hello dogs Quick foxes
文档2：good quick fox
为了检索相似单词，如fox,foxes,quick,Quick。在构建索引时，使用标准化规则，即统一为词根或小写。在查询时也要先标准化再进行查询。
```

##### 分析器构成

```
官网文档:
https://www.elastic.co/guide/cn/elasticsearch/guide/current/analysis-intro.html
字符过滤器:字符串按序通过字符过滤器，字符过滤器用于分此前的整理，用于去除HTML、&转为and等。
分词器：字符串被分为单个词条
Token过滤器：词条按序通过token过滤器，这个过程会改变词条，如QUICK->quick，删除词条，如a，and，the等五用词，或增加词条，例如jump和leap这种同义词。
es提供自定义：https://www.elastic.co/guide/cn/elasticsearch/guide/current/custom-analyzers.html的分析器：https://www.elastic.co/guide/cn/elasticsearch/guide/current/custom-analyzers.html
```

##### 内置分析器

```
标准分析器:默认分词器，按照unicode的定义划分文本，删除标点，最后进行小写。
简单分析器:任何不是字母的地方分割文本，再小写。
空格分析器:空格划分
语言分析器:考虑语言特点，删除and和the等。构建近义词、词根等。
```

查看分析器如何工作

```
GET /_analyze
{
  "analyzer": "standard",
  "text": "Text to analyze"
}
```



# Elasticsearch用法

### 配置使用环境

>es对内存等要求较高，默认的linux系统配置无法满足，一般都会报错，需要进行如下更改才能使用。

```
#vim /etc/security/limits.conf，修改对打开文件数量的限制
* soft nofile 65536
* hard nofile 65536
* soft nproc 4096
* hard nproc 4096
es soft memlock unlimited
es hard memlock unlimited
#查看
ulimit -a
#修改虚拟内存大小
vim /etc/sysctl.conf
vm.max_map_count=262144
```

### 设计mapping

##### 数据类型的选择



### 配置分词引擎

>不同领域对分词有不同的需求，需要结合专业领域的知识提升分词效果，因为分词效果关系到倒排索引的构建，最终影响到搜索性能。es自有的分词器有standard分词器，对中文效果不好。

>将需要的分词引擎拷贝到plugins下，

```
#为es配置IK分词器，先到github下载IK分词器代码，然后使用maven编译获得一个zip包
#将zip包解压到es的plugins目录下，查询时使用如下语法即可
curl -XGET 'http://localhost:9200/_analyze?pretty&analyzer=ik_max_word' -d '联想是全球最大的笔记本厂商'
#使用IK分词器创建索引，通过setting analyzer指定分词器，ik分为ik_max_word和ik_smart,ik_max_word拆分的粒度最细，smart拆分的粒度最粗。
"settings" : {
        "analysis" : {
            "analyzer" : {
                "ik" : {
                    "tokenizer" : "ik_max_word"
                }
            }
        }
    },
```

### 基础语法

>包括叶子查询字句、复合查询字句、

##### 叶子查询字句

>match、term、range、wildcard，用于在特定字段中。代码中的wildcard用于指定如何匹配给定的字段和值
>
>wildcard:表示严格匹配，即给定的字段必须包含完整的value。
>
>match:表示匹配分词的结果，如果匹配了value分词的一部分，也会进行返回。如搜索“张三”，他也会返回“李三”
>
>fuzzy：表示模糊，它比match更加广泛，如果分词中有错误字符，它也能进行纠正，并将结果返回。如搜索”邓字棋“，他也能修正为”邓紫棋“。
>
>term:表示精确匹配，不对数据value进行分词，直接进行查询。
>
>terms：同term，传入一个数据，相当于多个term的组合。
>
>exist：存在对应字段
>
>ids：根据id返回
>
>range：表示范围，用于数值型数据。gte：大于，lte：小于

```
SearchSourceBuilder ssb1=new SearchSourceBuilder();
        ssb1.from((pageNum-1)*5);
        ssb1.size(5);
        QueryBuilder queryBuilder=QueryBuilders.boolQuery()
                .should(QueryBuilders.wildcardQuery("objects.malware_types",value))
                .should(QueryBuilders.wildcardQuery("objects.pattern",value))
                .should(QueryBuilders.wildcardQuery("objects.architecture_execution_envs",value))
                .should(QueryBuilders.wildcardQuery("objects.value",value))     .should(QueryBuilders.wildcardQuery("objects.external_references.external_id.keyword",value));
        ssb1.query(queryBuilder);
        SearchRequest searchRequest=new SearchRequest("myindex").source(ssb1);
        return searchRequest;
```

##### 复合查询字句

>bool query、constant_score query、dis_max query
>
>bool query：用于连接多个查询，多个查询之间可用修饰词连接，如：should、must、must_not、filter

```
SearchSourceBuilder ssb1=new SearchSourceBuilder();
        ssb1.from((pageNum-1)*5);
        ssb1.size(5);
        QueryBuilder queryBuilder=QueryBuilders.boolQuery()
                .should(QueryBuilders.wildcardQuery("objects.malware_types",value))
                .should(QueryBuilders.wildcardQuery("objects.pattern",value))
                .should(QueryBuilders.wildcardQuery("objects.architecture_execution_envs",value))
                .should(QueryBuilders.wildcardQuery("objects.value",value))     	.should(QueryBuilders.wildcardQuery("objects.external_references.external_id.keyword",value));

        ssb1.query(queryBuilder);
        SearchRequest searchRequest=new SearchRequest("myindex").source(ssb1);
        return searchRequest;
```



### 聚合查询

>复杂的聚合查询需要使用painless脚本完成，可以使用类似mr的思想完成聚合统计，规避es的聚合统计弱点。

### 迁移数据

>由于之前的索引设计不合理，或者有新的需求,需要新建索引，并在索引之间迁移数据。
>
>另外，还可能需要跨机器和跨数据库的迁移，将一台机器的数据迁移到另一个es服务的表中。

##### 索引之间迁移

```
#kibana语法,5601端口
#curl语法
#新建索引
PUT /test
{
......
}
#使用reindex迁移到新索引
POST _reindex { "source": { "index": "my_index_name" }, "dest": { "index": "my_index_name_new" } } 
```

##### 数据库之间迁移

###### 使用logstash进行迁移

```
input {
  elasticsearch {
  hosts => [ "100.200.10.54:9200" ]
  index => "doc"
  size => 1000
  scroll => "5m"
  docinfo => true
  scan => true
  }
}

filter {
json {
  source => "message"
  remove_field => ["message"]
  }
  mutate {
  # rename field from 'name' to 'browser_name'
  rename => { "_id" => "wid" }
}
}

output {
  elasticsearch {
  hosts => [ "100.20.32.45:9200" ]
  document_type => "docxinfo"
  index => "docx"
  }

  stdout {
  codec => "dots"
  }

}
```

### es结点扩展

>随着业务增长和需求变化，需要加入新的结点，扩展es的内存容量。

```
#配置：16G，500G磁盘

```

### es-head插件使用

>es-head是一个前端插件，解压就能使用，它使用web前端请求es数据库并进行展示。

```
将压缩包解压到新的机器，修改配置文件添加对应host，拷贝es/data的数据到新的机器
#注意要禁用rebalance，不然会花费很久时间
PUT /_cluster/settings
{
    "transient" : {
        "cluster.routing.allocation.enable" : "none"      //取消分片权衡
    }
}
#指定inde想到特定机器
PUT index_name/_settings
{
  "index.routing.allocation.include._ip": "10.124.105.5,10.124.105.6,10.124.105.7"
}
```

### es-kibana使用

##### dashboard使用

>创建dashboard，create panel,左侧拖拽需要的键值，底部栏选择需要的什么类型的图形，如饼状等，右侧选择聚合参数，如平均，最大，求和等。

# 其他

index/type/document,**field**

```
elasticsearch -Des.insecure.allow.root=true
curl http://127.0.0.1:9200
http://127.0.0.1:5601.
```

```text
useradd testuser  //创建用户testuser
passwd testuser   //给已创建的用户testuser设置密码
说明：新创建的用户会在/home下创建一个用户目录testuser
usermod --help    //修改用户这个命令的相关参数
userdel testuser  //删除用户testuser
rm -rf testuser   //删除用户testuser所在目录
```

```text
chown -R newuser 目录  //改目录所有人
chmod -R 777 目录      //修改目录权限
```

rm -rf data

_shards_percent_as_number" : 100.0





# Elasticsearch大量查询和深度分页

>大量查询问题：elasticsearch在默认情况下，不允许单词请求窗口大于10000，因为在这种情况下，elasticsearch的查询效率很慢，占用内存很大。
>
>分页问题：elasticsearch可以使用form，size进行分页操作，但是elasticsearch默认情况下，不允许使用from，size对10000条以后的数据进行分页。因为在这种情况下，elasticsearch的查询效率很慢，占用内存很大。

### 解决办法

##### 解决大量查询问题

>相比于深度分页问题，此问题虽然导致大量数据的请求，但是不对结果做顺序要求，一般使用sroll_scan。注意scan返回的是一个迭代器，可以进行迭代取值。

###### sroll_scan

```python
#scroll_scan,其中size是指每个shard上size为50，最终为shard_num*size。scroll_scan不保证结果顺序。
POST ip:port/my_index/my_type/_search?search_type=scan&scroll=1m&size=50
{
    "query": { "match_all": {}}
}
```

```python
#python
es = Elasticsearch(['ip:9200'])
def scroll_scan():
    search_body = {
        "query": {
            "ids": {
                "values": ["f7a5dcd376be777c6593a29b8ebd411a","494d9a5e25b9e1d3eedb7a2341aa49ad"]
            }
        }
    }

    page = helpers.scan(
        client=es,
        query=search_body,
        scroll='5m',
        index="cti",
        request_timeout=5
        # search_type ='scan',
    )

    result_list=[item["_source"]for item in page]

    # i=0
    # for item in page:
    #     print(i)
    #     i+=1
    #     result_list.append(item["_source"])
    print(len(result_list))
    print(result_list)
    # print("page_len:", len(page))
```



##### 解决深度分页问题

>深度分页要求结果的有序性，要达到和from，size一样的效果，一般使用scroll函数或search_after函数。scroll函数用于非实时场景，如批量导出，因为它是一份历史快照，无法得到最新的数据变化。而search_after是根据上一页的最后一条数据确定下一页的位置，并且在这个过程中，数据的变化会反映到游标上，但是其无法进行跳页请求。

###### scroll函数

```java
#第一次请求，获取到scroll_id和第一批数据
BoolQueryBuilder mustQuery = QueryBuilders.boolQuery();
        //设置查询条件
        mustQuery.must(QueryBuilders.matchQuery("sex","男"));
        mustQuery.must(QueryBuilders.matchQuery("city","杭州市"));
        SearchResponse rep =  client.prepareSearch()
                .setIndices(index) // 索引
                .setTypes(type)  //类型
                .setQuery(mustQuery)
                .setScroll(TimeValue.timeValueMinutes(2))  //设置游标有效期
                .setSize(100)  //每页的大小
                .execute()
                .actionGet();
                
#后续循环使用scroll_id进行请求，直到返回地hits为空。           
SearchResponse rep1 = client.prepareSearchScroll(scrollId)  //设置游标
                    .setScroll(TimeValue.timeValueMinutes(2))  //设置游标有效期
                    .execute()
                    .actionGet();

```

```python
#python
es = Elasticsearch(['ip:9200'])
def scroll():
    search_body={
        "size":10000,
        "query":{
            "match_all":{}
        }
    }

    page = es.search(
        index="cti1",
        # search_type ='scan',
        body=search_body,
        scroll='5m',
    )
    print("page_len:",len(page["hits"]["hits"]))
    scroll_id=page['_scroll_id']
    print("scroll_id:"+scroll_id)
    page["hits"] = "empty"
    print(json.dumps(page,indent=4))

    while(True):
        print("---------------------------")
        page=es.scroll(
            scroll_id=scroll_id,
            scroll='5m'
        )
        hits_len=len(page["hits"]["hits"])
        print("scroll lenght:",hits_len)
        if hits_len<10000:
            print("hits_len ==0,break")
            break
        scroll_id=page['_scroll_id']
        print("scroll_id:"+scroll_id)
        page["hits"]="empty"
        print(json.dumps(page, indent=4))

```

###### search_after函数

>search_after是根据上一页的最后一条数据确定下一页的位置，并且在这个过程中，数据的变化会反映到游标上，但是其无法进行跳页请求。

```curl
GET test_dev/_search
{
  "query": {
    "bool": {
      "filter": [
        {
          "term": {
            "age": 28
          }
        }
      ]
    }
  },
  "size": 10,
  "from": 0,
  "search_after": [
    1541495312521,
    "d0xH6GYBBtbwbQSP0j1A"
  ],
  "sort": [
    {
      "timestamp": {
        "order": "desc"
      },
      "_id": {
        "order": "desc"
      }
    }
  ]
}
```

### 查询和分页原理

##### ES的基本结构

>ES集群中的节点主要分为Coordinate、DataNode，Cordinate负责接受来自用户Client的操作请求，并分发任务给DataNode，汇总DataNode的结果并进行排序等操作，最后返回给Client。在DataNode上分布着ES分片，分片是ES的最小级别工作单元，它只保存了索引中所有数据的一部分。

##### ES的查询过程

>​		ES的查询过程分为两阶段，称为Query Then Fetch。Query就是通过倒排索引查询满足条件的doc_id,Fetch就是根据得到的doc_id从各个节点获取对应文档。
>
>#Query阶段
>	Query阶段，当Coordinate接收到来自Client的查询请求时，它将查询广播到每一个DataNode上的分片。每个DataNode的分片对本地的Filesystem Cache、磁盘上的数据执行搜索，并得到一个结果列表。
>#Query 查询
>	通常，分片对Filesystem Cache中的数据进行搜索，如果所需要的数据不在Filesystem Cache中，就需要从磁盘中加载到Filesystem Cache，所以Filesystem Cache能否覆盖需要查询的数据，极大的影响查询效率。
>#Query结果列表
>	Query阶段的结果列表包括文档的ID和排序值，排序值即分数，是在本地分片上计算的TF-IDF分数。结果列表的大小由筛选条件决定，例如使用{from 100 ，size 50}，那么每个分片要的结果列表150条数据，如果使用{size 50},那么每个节点结果列表包含50条数据。这些数据会传递给Coordinate,完成Query阶段。
>#Fetch阶段
>	Fetch阶段，协调节点对收到的各个节点信息进行排序，选择需要查询的文档。并使用doc_id、shard标识到对应的DataNode的shard取回信息，最终经过汇总排序，返回给Client。

>​		可见，from size查询方法对服务器的内存要求较高，比较容易OOM，特别是深度分页的情况，from+size决定了查询的数据量。当然，直接查询大量数据也会造成同样的压力，所以需要使用其他方式避免es的分片上的大量数据查询。

##### scroll原理

>​		scroll通过为当前ES的数据和索引创建一份快照，然后使用scroll_id来对这个快照进行持续性的遍历，避免每来一个请求都要去各个节点查询和排序。

##### ES适用场景

>适合全文检索，由给定字段查询所在文档，数据量大时需要特定api。适合灵活的节点水平扩展。
>不适合深度分页，因为其跳页实现繁琐。不适合数据频繁的修改，其修改本质是删除加插入操作形成的，高频率的数据增删容易触发段合并，即数据的重新组装。不适合事务操作，没有关系型数据库的事务和锁，难以应对一致性要求较高的场景。

##### 关于es内存

>https://zhuanlan.zhihu.com/p/99718374

```
只将需要搜索的字段放入es，这样可以减少数据量，如果你的数据量比filesystem cache还小，那么相当于你的所有数据都在内存而不是磁盘，这种情况下速度最快，磁盘中放的数据越多，效果越差。
```

### mysql分页和elasticsearch分页

>​	为什么mysql处理深度分页的能力比es强，不需要非常大的内存支持。虽然mysql随着深度加深，查询时间也会上身，但是没有es的剧烈，对于10000数据量级的很容易处理，而es的深度分页被限制在10000条数据。

>​	ES的设计是为了方便集群节点的水平扩展，所以以shard为基本工作单元存储和管理数据，但是这增加了Coordinate和DataNode中shard沟通的成本，以及shard本地计算的成本，如上文所示。所以，对于同样的查询语句，如上文的from，size，ES相比mysql工作量上升了节点个数的倍数，而且ES为了加速查询适用内存索引，所以对内存提出了更高的要求。

### 相关连接

>elasticserach详解长文：https://www.cnblogs.com/Leo_wl/p/16006513.html#_label0
>深入分片：https://www.jianshu.com/p/cc06f9adbe82
>mysql与es对比：https://blog.csdn.net/adparking/article/details/109773492
>ES如何正确深度分页：https://www.cnblogs.com/you-you-111/p/5849945.html
>es线程池:https://www.iteye.com/blog/rockelixir-1890867
