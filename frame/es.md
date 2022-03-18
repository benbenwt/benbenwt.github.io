

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



### painless脚本

### es分析和分析器

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

### 映射

>映射用于控制特定字段域的分析器类型、存储类型

```
主要类型：text,keyword,float,double,boolean,date,byte,short,integer,long
#查看映射
GET /gb/_mapping/tweet
```





```
若构建三个表存储stix2，当需要聚合查询时效率可以接受。但如果需要分页，就要请求全量数据再汇总，效率受限于索引查询，网络传输和网络请求次数开销。
```



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

es-head插件

```
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

# 迁移数据

```
新建索引，迁移到新索引
```



# kibana

## 安装配置kibana

修改ip

```
vim $ES_HOME/config/elasticsearch.yml
hostname 0.0.0.0
vim KIBANA_HOME/config/kibana.yml
server.host: "this_host_id"
elasticsearch.hosts: ["http://hbase2:9200"]
bin/kibana --allow-rooot
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
	  					}
	  		“must":{
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
POST /myindex/user/_search
{
	"query":{
	  	"match":{#match是查询的一种，还有match_case等
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
{
"query": {
	"bool": {
		"must": [
			{``
				"match": {
					"objects.pattern": test"
							}
				}
					]
}
}
}
```

java api

# 主键查询大量数据

>默认的window_size=1000，不能超过此数值。否则应使用scroll。es不太适合分页操作，本质是不适合大量主键查询、和深页查询（基于主键查询，再排序）。因为其构建的是基于列的分片存储，对于主键没有像mysql一样的索引，只能靠排序来进行分页。其主要面临大量查询时的内存问题，以及分页时间问题。
>
>es的几个限制条件：
>
>1.单次不能查询超过10000条
>
>2.from、size不可以查询超过10000的深页
>
>3如果我想获取全部数据，
>
>4如果我想获得满足特定条件的全部数据，其大于10000条。
>
>

##### 关于es内存

>https://zhuanlan.zhihu.com/p/99718374

```
只将需要搜索的字段放入es，这样可以减少数据量，如果你的数据量比filesystem cache还小，那么相当于你的所有数据都在内存而不是磁盘，这种情况下速度最快，磁盘中放的数据越多，效果越差。
```



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

### 性能

##### 全量读写

##### 深度分页

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

# elasticsearch深度分页

>elasticsearch可以使用form，size进行分页操作，但是elasticsearch默认情况下，不允许使用from，size对10000条以后的数据进行分页。因为elasticsearch这种情况下，elasticsearch的查询效率很慢。

### 解决办法

##### scroll函数

### 分页原理

##### 主键索引

### mysql分页和elasticsearch分页

B+树索引
