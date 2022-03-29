# TI

>数仓的数据一般来源于以下三种，1.用户的业务数据，如购买、注册、登录等，这些数据一般在mysql、oracle等业务数据库中。2.用户的行为日志数据，这些数据不会与后端交互，所以需要在前端采集，如点击了某个前端元素，在某个页面停留了多久。3.爬虫数据，从其他同类型网站或相关网站爬取的信息，如竞品。本项目主要是爬虫数据，这里的爬虫是指源头是爬虫，但是是将文章、软件分析之后生成的爬虫信息。

>之前用mr写的基本流程：后端直接推到kafka（这一步可以用flume代替），然后java程序读取kafka写入hdfs（这一步可以用flume代替），然后mr任务读取hdfs清洗为列写入hive（这一步的清洗为列能放到后边嘛，还是用mr吧，调试方便些）。应该是清洗到多个表，而不是一个表。和es_provider中解析json的代码差不多。后边再用hive，那怎么导入的，正常情况下是insert overwrite  partition（.... ....），以前采用的load了语法批量载入，一天一个分区。
>
>没有用户就没有复杂的业务过程，数仓的数据就会很单薄，只有维度的罗列。

### 爬虫模块

>爬虫模块主要包括文章的爬虫、

### 分析模块

>分析模块用于生成最终的json数据

### HIVE数仓

>将分析模块和爬虫模块的数据采集至hdfs，然后dump到hive表，主要目的，统计category，即类别，统计Location，即地区，统计Architecture，即架构。其他topic，ThreatActor，

#### 建表语句

##### ods_category

```
```



##### ods_location

##### ods_architecture

##### dwd_category

##### dwd_location

##### dwd_architecture

##### dws_category

##### dws_location

##### dws_architecture

##### dwt_category

##### dwt_location

##### dwt_architecture

##### ads_category

##### ads_location

##### ads_architecture

### 数据库

>Mysql用于存储基本业务，如保存分析的记录、展示分页、
>
>Elasticsearch用于对json进行全文检索，如关键id、软件关键信息的检索。
>
>ClickHouse用于对清洗结束的数据进行快速聚合查询，实际上Mysql也能胜任，因为清洗后的数据量不大，单ch数据量大表现更好。
>
>Kafka用于接受来自分析模块的数据
>
>SparkStreaming用于对kafka的数据进行消费，并进行实时处理，将结果存入Mysql、Elasticsearch、Clickhouse等。

### 后端

>web后端负责对接分析模块，展现分析模块记录，提供分析模块提交功能、分页功能。并对SparkStreaming处理过的统计数据Mysql、Clickhouse进行展示，提供ES的全文搜索

### 前端

# 实时数仓规范

>实时数仓如果不使用持久化结果的组件，那么其计算结果都不可复现、不可纠正，像内存中的数据一样，停止了就会消失。所以一般是处理日活类型的指标，将结果存入mysql，clickhouse等，后续不会再用到这些数据。今日日活+历史数据，不就是实时的嘛。然后离线的仍然一天一次，实时就这样增量就好了。

### 埋点与业务数据

>埋点分为前端埋点和后端埋点
>
>前端埋点通过在客户端或web端编写代码监控web的点击、浏览行为，并发送到后端的日志服务器，再采集到kafka。
>
>后端使用AOP处理，将行为日志采集到kafka。
>
>业务数据批量迁移使用sqoop，增量实时迁移使用canal、maxwell等。

# 尚硅谷

>尚硅谷SparkStreaming实时数仓。使用canal、maxwell采集mysql数据库的数据到kafka，使用日志服务器后端将采集到的数据落盘并发送到kafka，使用sparkStreaming编写程序进行处理。统计以下几个指标，当日日活数量、当日首单用户数量。

### 日志采集后端

>使用springboot搭建后端日志服务，借助logback将日志落盘，利用spring-kafka将日志按照类型分发到不同的kafka topic，主要包括启动日志topic、事件topic，命名为gmall_start、gmall_event
>
>在logback.xml中配置好落盘目录，输出日志到文件。
>
>使用spring-kafka将日志发送到对应的日志topic，区分启动日志和事件日志。

### 业务数据采集

>主要有以下几个表，使用maxwell、canal采集到kafka对应topic，每个表对应一个topic。
>
>这些表按照业务大致划分为：事实表：活动、优惠券、收藏、加入购物车、下单、支付、退款 、  维度表：商品信息、地区信息、品类信息、地区信息、日期信息
>
>安装好canal、maxwell后，配置好mysql地址，并为canal、maxwell创建好mysql账户，给予权限，为两者配置kafka地址，及发送的topic。

### nginx反代

>使用nginx反向代理三台日志服务器，因为用户的客户端操作日志数据量很大，使用nginx创建upstream，负载均衡三台机器，这样可以向nginx动态的替换机器，在需要重启服务时，可以逐个重启服务，保证upstream中有一个可用的结点。让用户无法感知服务器已经进行了重启和更新。

### 日活统计

>日活统计要求统计当日登录的人数，通过采集的登录日志进行统计。需要借助redis进行去重，比如某个用户进行了多次登录，只能记作一个活跃用户。然后将去重的日活记录插入到ES数据库中，用于可视化和查询。如果不需要使用kibana可以写入MySQL，给业务使用，也可以用sql统计后提供给其他可视化工具，如DataV、QuickBI。

### 首单分析

>ods，dwd
>
>交易的每一条记录都在mysql中，使用canal、maxwell采集到kafka，然后使用sparkStreaming对数据分流，放入不同的topic。使用hbase过滤掉不是首单的用户，如果是首单还需要修改hbase的标记，最终将首单的信息写入数据库。由于是否首单需要统计历史数据，数据量比较大，如果使用redis内存压力比较大，而且是否首单需要保存很久，不是24小时过期的数据，需要修改hbase的标记，需要是key-value模式的查询，综合上边三点，适合保存在hbase中。
>
>具体流程：读取ods_order_info，从hbase查出所有ods_order_info中存在的user_id信息，然后将ods_order_info中已存在的标记为1，然后对ods_order_info中同一个用户后续提交的标记为0，最终稿将过滤好的首单结果与省份表、用户表拼接得到宽表，将首单信息写入es，将首单标记写入hbase。

### Exactly once

>为了实现精确一次消费，有两种方案。1.使用具有幂等性的操作，如es，redis等，在数据操作完成后，再提交offset。2使用mysql事务，将数据操作和索引操作进行绑定，达到原子性事务。

### kafka分层

>对输入的不同类别的原始日志进行处理

#### ods

#### dwd

#### dws

#### dwt



### problem

Stream.foreach的操作单位、对象是什么
