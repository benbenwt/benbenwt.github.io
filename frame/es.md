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

vim /etc/security/limits.d/90-nproc.conf
* soft nproc 4096

```



无法访问9200，用nmap扫一下就好了。

查询url,hash,architecture,catetory.

```
使用should查询并行的多个结果
```



GET

```
GET /customer/_doc/1
curl -X GET "hbase2t:9200/customer/_doc/1?pretty"
```

PUT

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

更新数据

```
POST /myindex/user/1001/_update
{
	"doc":{
	   "age":23
	}
}
```

删除数据

```
DELETE /myindex/user/1001
```





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



```
 curl -X POST "localhost:9200/_bulk?pretty" -H "Content-Type: application/json;charset=UTF-8" --data-binary @5eb920947209f2ced78b15a4.json 
```



修改ip

```
vim $ES_HOME/config/elasticsearch.yml
hostname 0.0.0.0
vim #KIBANA_HOME/config/kibana.yml
server.host: "host2"
elasticsearch.hosts: ["http://hbase2:9200"]
```



```sh
./bin/elasticsearch -d -p pid
pkill -F pid
```

`git clone git://github.com/mobz/elasticsearch-head.git`

### search

```
#搜索数据
GET /myindex/user/1001
#所有数据数据,默认10条。需要分页
GET /myindex/user/_search
#
GET /myindex/user/_search?q=age:21
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

范围搜索

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
			{
				"match": {
					"objects.pattern": test"
							}
				}
					]
}
}
}
```



index/type/document,field

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

```
curl -X GET 10.44.99.102:9200/situation-event/_refresh
```

curl -X PUT "localhost:9200/customer/_doc/1?pretty" -H 'Content-Type: application/json' -d'
{
  "name": "John Doe"
}
'