### 安装

```
https://www.jianshu.com/p/77bd458cee6f
#解压安装包
#运行
bin/logstash -e "input { stdin {}} output { stdout {}}"
命令里的-e表示后面跟着的是一个配置信息字符串，字符串的功能就是让 Logstash
接受控制台里的输入，并输出到控制台。
#用最简化的方式执行
bin/logstash -e ""
会看到 Logstash 采用了默认的配置，从控制台输入，并以 json 格式输出到控制台。
#指定配置文件
bin/logstash -f config/demo.conf
```

##### 配置文件示例

```
input { stdin {}}
output { stdout {}}
```

##### output-elasticsearch插件

```
#如果要将数据输出到，则可以用 Elasticsearch 插件：
output {
        elasticsearch {
                action => "index"
                hosts  => "localhost:9200"
                index  => "log-example"
         }

详细文档说明https://www.elastic.co/guide/en/logstash/current/plugins-outputs-elasticsearch.html
```

```
PUT /gmall2020_dau_info_20220418
{
 "settings": { 
 "number_of_shards": 3
 },
 "aliases" : { 
 "{index}-query": {},
"gmall2020_dau_info-query":{}a
 },
 "mappings": {
 "properties":{
 "mid":{
 "type":"keyword"
 },
 "uid":{
 "type":"keyword"
 },
 "ar":{
 "type":"keyword"
 },
 "ch":{
 "type":"keyword"
 },
 "vc":{
 "type":"keyword"
 },
 "dt":{
 "type":"keyword"
 },
 "hr":{
 "type":"keyword"
 },
 "mi":{
 "type":"keyword"
 },
 "ts":{
 "type":"date"
 }
 }
 }
}

```



### 理论知识

>输入->过滤器（grok 表达式）->输出

```
```

