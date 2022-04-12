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

#### 数据采集

##### flume

>对于只能采集单行的问题，可以将json保存为单行。因为json保存时本来就应该剔除制表符和换行符，这样方便数据的处理。显示时如果需要，直接添加indent即可。flume采集后，直接load原生字符串到hive，再编写udf函数解析。

>hdfs sink:hdfs.fileType=DataStream

>flume Line legnth exceeds max
>
>这说明 截取 之后才会进入intecpter，所以intercepter报错了，其不是完整的json。

>taildir,spooldir

#### hive仓库

##### 自定义函数

```
#使用自定义函数，查看信息。
select get_id(json) id,get_arch(json) arch,get_location(json) location,get_category(json) category,get_time(json) created_time,json from ods_json;
```

```
#创建自定义函数
#getArch
create function ti.get_arch as 'com.atguigu.gmall.hive.udtf.GetArch' using jar 'hdfs://hbase:9000/user/root/hiveudf/gmall-udtf-1.0-SNAPSHOT.jar';
#getLocation
create function ti.get_location as 'com.atguigu.gmall.hive.udtf.GetLocation' using jar 'hdfs://hbase:9000/user/root/hiveudf/gmall-udtf-1.0-SNAPSHOT.jar';
#getCategory
create function ti.get_category as 'com.atguigu.gmall.hive.udtf.GetCategory' using jar 'hdfs://hbase:9000/user/root/hiveudf/gmall-udtf-1.0-SNAPSHOT.jar';
#getTime
create function ti.get_time as 'com.atguigu.gmall.hive.udtf.GetTime' using jar 'hdfs://hbase:9000/user/root/hiveudf/gmall-udtf-1.0-SNAPSHOT.jar';
#getId
create function ti.get_id as 'com.atguigu.gmall.hive.udtf.GetId' using jar 'hdfs://hbase:9000/user/root/hiveudf/gmall-udtf-1.0-SNAPSHOT.jar';
```

##### ods_json

```sql
create external table ods_json(
    `json` string
)partitioned by (`dt` string)
location '/warehouse/ti/ods/ods_json'
```

```
#将数据加载到ods_json
load data inpath '/origin_data/threat/json/stix_json/2022-03-30'
into table ti.ods_json partition(dt='2022-03-30');
```

##### dwd_category

```sql
create external table dwd_category(
    `id` string,
    `created_time` string,
    `category` string
)partitioned by (`dt` string)
location '/warehouse/ti/dwd/dwd_category'
```

```
insert overwrite table  dwd_category partition(dt='2022-03-30')
select get_id(json) id,get_time(json) created_time,category  from ods_json lateral view explode(get_category(json)) tmp as category;
```

###### 大量小文件合并

>在hive表下，有许多load进来的数据，它们的数量影响了mapTask的效率，故想使用合并的方式减少MapTask数量。

```
1.输入合并
set mapred.max.split.size=256000000;  #每个Map最大输入大小
set mapred.min.split.size.per.node=100000000; #一个节点上split的至少的大小 
set mapred.min.split.size.per.rack=100000000; #一个交换机下split的至少的大小
set hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat;  #执行Map前进行小文件合并

在开启了org.apache.hadoop.hive.ql.io.CombineHiveInputFormat后，一个data node节点上多个小文件会进行合并，合并文件数由mapred.max.split.size限制的大小决定。
mapred.min.split.size.per.node决定了多个data node上的文件是否需要合并~
mapred.min.split.size.per.rack决定了多个交换机上的文件是否需要合并~

2.输出合并
set hive.merge.mapfiles = true #在Map-only的任务结束时合并小文件
set hive.merge.mapredfiles = true #在Map-Reduce的任务结束时合并小文件
set hive.merge.size.per.task = 256*1000*1000 #合并文件的大小
set hive.merge.smallfiles.avgsize=16000000 #当输出文件的平均大小小于该值时，启动一个独立的map-reduce任务进行文件merge
```

##### dwd_location

```
create external table dwd_location(
    `id` string,
    `created_time` string,
    `location` string
)partitioned by (`dt` string)
location '/warehouse/ti/dwd/dwd_location'
```

```
insert overwrite table  dwd_location partition(dt='2022-03-30')
select get_id(json) id,get_time(json) created_time,location  from ods_json lateral view explode(get_location(json)) tmp as location;
```

##### dwd_architecture

```
create external table dwd_architecture(
    `id` string,
    `created_time` string,
    `location` string
)partitioned by (`dt` string)
location '/warehouse/ti/dwd/dwd_architecture'
```

```
insert overwrite table  dwd_architecture partition(dt='2022-03-30')
select get_id(json) id,get_time(json) created_time,architecture  from ods_json lateral view explode(get_arch(json)) tmp as architecture;
```

##### dws_category

>一天为粒度

```
select * from dwd_category;
```



##### dws_location

##### dws_architecture

##### dwt_category

>1，7，30天

##### dwt_location

##### dwt_architecture

##### ads_category

>存储到业务数据库，用于查询

##### ads_location

##### ads_architecture

##### 



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

## 埋点与业务数据

>埋点分为前端埋点和后端埋点
>
>前端埋点通过在客户端或web端编写代码监控web的点击、浏览行为，并发送到后端的日志服务器，再采集到kafka。
>
>后端使用AOP处理，将行为日志采集到kafka。
>
>业务数据批量迁移使用sqoop，增量实时迁移使用canal、maxwell等。

# 尚硅谷spark-streaming实时数仓

>尚硅谷SparkStreaming实时数仓。使用canal、maxwell采集mysql数据库的数据到kafka，使用日志服务器后端将采集到的数据落盘并发送到kafka，使用sparkStreaming编写程序进行处理。统计以下几个指标，当日日活数量、当日首单用户数量。

## 日志采集后端

>为了模拟生产中的场景，开启了一个sprinboot后端服务作为日志服务器，负责接收来自前段的日志，并存储到logfile，并发送到对应的kafka的topic，topic包括日志topic，事件topic。

### 日志模拟生成

>这里前端用一个java程序模拟代替，其源源不断的向后端日志服务器发送模拟的数据。发送的速度为每秒100条，因为设置的每条日志发送的延迟为10ms。

```
#使用okhttp将数据发送到日志服务器的后端接口。
#日志格式如下，第一层的key有common和start，start下是启动的信息，common是其他信息，如浏览的品牌等。
#启动信息：loading_time,open_ad_id,open_ad_ms,open_ad_skip_ms
{"common":{"ar":"110000","ba":"Redmi","ch":"oppo","md":"Redmi k30","mid":"mid_168","os":"Android 11.0","uid":"337","vc":"v2.1.132"},"start":{"entry":"notice","loading_time":17874,"open_ad_id":2,"open_ad_ms":2467,"open_ad_skip_ms":0},"ts":1594652955000}
```

### Logback

>1创建logback.xml，在其中控制日志的输出级别的输出节点root，logger，以及输出源appender。
>
>2为类添加@Slf4j注解

```
<appender name="console" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
    	<pattern>%msg%n</pattern>
    </encoder>
</appender>

<!-- 将某一个包下日志单独打印日志 -->
<logger name="com.atguigu.logger.controller.LoggerController" level="INFO" additivity="false">
    <appender-ref ref="rollingFile" />
    <appender-ref ref="console" />
</logger>

#当没有子节点logger，会调用root
 <root level="error" additivity="false">
 	<appender-ref ref="console" />
 </root>
```

### Kafka发送

>使用springframework提供的spring-kafka链接kafka。
>
>1首先配置好springframework-kafka的pom依赖，然后编写application.properties填写bootstrap-server,key-serializer,value-serializer。
>
>2然后在代码中创建KafkaTemplate对象，使用Autowired注入对象。
>
>3使用：kafkaTemplate.send(topic,jsonObject);

## 业务数据采集

>主要有以下几个表，使用maxwell、canal采集到kafka对应topic，每个表对应一个topic。
>
>这些表按照业务大致划分为：事实表：活动、优惠券、收藏、加入购物车、下单、支付、退款 、  维度表：商品信息、地区信息、品类信息、地区信息、日期信息
>
>安装好canal、maxwell后，配置好mysql地址，并为canal、maxwell创建好mysql账户，给予权限，为两者配置kafka地址，及发送的topic。

### binlog格式

>binlog支持以下格式：
>
>1row格式，一行为单位保存记录，仅仅保存那条记录被修改，可以不记录sql语句的信息。
>
>2statement，将每一条会修改数据的sql记录在binlog中。不需要记录每一行的变化，减少了日志量。还需要记录每条语句执行时候的一些信息，确保在slave节点上执行时候获得相同的结果。
>
>3Mixed格式
>
>该格式是以上两种level的混合使用

### maxwell的格式

```
#指明数据库，表名，data字段中携带表中的数据，old字段中携带旧的字段值，时间戳，xid（对应的sql的id）。
#maxwell以影响的数据为单位发送日志，每一条数据产生一条日志，如果想知道是否来自同一条sql，可以通过xid判断。
{
"database":"gmall-2020-04",
"table":"z_user_info",
"type":"insert",
"ts":1589385314,
"xid":82982,
"xoffset":0,
"data":{"id":30,
"user_name":"zhang3",
"tel":"1381000101
0"
}
}
```

## nginx反代

>使用nginx反向代理三台日志服务器，因为用户的客户端操作日志数据量很大，使用nginx创建upstream，负载均衡三台机器，这样可以向nginx动态的替换机器，在需要重启服务时，可以逐个重启服务，保证upstream中有一个可用的结点。让用户无法感知服务器已经进行了重启和更新。

## 需求1日活统计

>日活统计要求统计当日登录的人数，通过采集的登录日志进行统计。需要借助redis进行去重，比如某个用户进行了多次登录，只能记作一个活跃用户。然后将去重的日活记录插入到ES数据库中，用于可视化和查询。如果不需要使用kibana可以写入MySQL，给业务使用，也可以用sql统计后提供给其他可视化工具，如DataV、QuickBI。这一部分代码在DauApp类中完成，即day active user。

### 读取kafka的启动日志topic

#### 精确一次消费

>精确一次消费可以通过两种形式达到：
>
>1使用事务，让两步操作必须同时失败或成功。
>
>2手动提交+幂等性，在存储好数据后在提交offset，避免offset成功但数据没存储的情况。然后，确保数据存储是幂等性的操作，也就是说offset提交失败时，虽然会重试数据存储的操作，但是无论重试多少次都是结果一样的，会进行覆盖，例如es相同的id会进行覆盖。

>为了达到精确一次消费，需要进行手动提交及读取offset，避免出现offset提交了，但是数据还没有存储的情况。然后借助es的幂等性达到精确一次消费。

>kafka默认5秒提交一次偏移。
>
>enable.auto.commit 的默认值是 true；就是默认采用自动提交的机制。 
>
>auto.commit.interval.ms 的默认值是 5000，单位是毫秒。

```
#首先,从redis取出kafka偏移量
#获取jedis客户端
val prop = MyPropertiesUtil.load("config.properties")
val host = prop.getProperty("redis.host")
val port = prop.getProperty("redis.port")

val jedisPoolConfig: JedisPoolConfig = new JedisPoolConfig
jedisPoolConfig.setMaxTotal(100)  //最大连接数
jedisPoolConfig.setMaxIdle(20)   //最大空闲
jedisPoolConfig.setMinIdle(20)     //最小空闲
jedisPoolConfig.setBlockWhenExhausted(true)  //忙碌时是否等待
jedisPoolConfig.setMaxWaitMillis(5000)//忙碌时等待时长 毫秒
jedisPoolConfig.setTestOnBorrow(true) //每次获得连接的进行测试

jedisPool=new JedisPool(jedisPoolConfig,host,port.toInt)
jedis=jedisPool.getResource
val offsetMap: util.Map[String, String]=jedis.hgetAll(offsetKey)
```

```
#从kafka创建sparkstreaming输入流
kafkaParam("group.id")=groupId
val dStream = KafkaUtils.createDirectStream[String,String](
      ssc,
      LocationStrategies.PreferConsistent,
      //这句话将整个topic,groupid传递进去,也就是所有partition的offset都传递进去了,这表示,consumer(topic,groupid)接受所有传递进去的分区的消息.
      ConsumerStrategies.Subscribe[String,String](Array(topic),kafkaParam,offsets))
      dStream
```

```
#从kafka读取偏移量。
var offsetRanges: Array[OffsetRange] = Array.empty[OffsetRange]

    val offsetDStream: DStream[ConsumerRecord[String, String]] = recordDStream.transform {
      rdd => {
        //因为recodeDStream底层封装的是KafkaRDD，混入了HasOffsetRanges特质，这个特质中提供了可以获取偏移量范围的方法
        offsetRanges = rdd.asInstanceOf[HasOffsetRanges].offsetRanges
        rdd
      }
    }
```

### 借助redis去重

```
#使用sadd函数，如果存在插入元素，返回0，否则返回1并插入。
val isNew:lang.Long=jedisClient.sadd(dauKey,mid)
去重这一部分使用了mapPartitions函数，对于每个分区创建了一次jedisClient，对于分区内每个记录请求了一次redis服务。
#然后得到首次登录的用户列表。
#将首次登录的用户信息封装好之后，插入ES。
#使用了foreachPartition方法，将整个分区的数据封装好后，借助ES bulk insert批量插入。
```

#### foreachPartition mapPartition

>1foreachPartition是action算子，用于数据处理的结尾，获取处理的结果，并将结果插入es，hbase等数据库中，它不会返回一个新的数据流rdd。mapPartition是转换操作，返回一个可以继续操作的rdd

### 写入结果数据

#### 精确一次消费

## 需求2首单分析

>ods，dwd
>
>交易的每一条记录都在mysql中，使用canal、maxwell采集到kafka，然后使用sparkStreaming对数据分流，放入不同的topic。这一部分分流的代码在BaseDBMaxwellApp。
>
>基础的dim维度信息，由dim层的流处理app处理。
>
>使用hbase过滤掉不是首单的用户，如果是首单还需要修改hbase的标记，最终将首单的信息写入数据库。由于是否首单需要统计历史数据，数据量比较大，如果使用redis内存压力比较大，而且是否首单需要保存很久，不是24小时过期的数据，需要修改hbase的标记，需要是key-value模式的查询，综合上边三点，适合保存在hbase中。这一部分代码在orderInfoApp中。
>
>具体流程：读取ods_order_info，从hbase查出所有ods_order_info中存在的user_id信息，然后将ods_order_info中斗胆用户的标记为1，然后对ods_order_info中同一个用户后续提交的标记为0，最终稿将过滤好的首单结果与省份表、用户表拼接得到宽表，将首单信息写入es，将首单标记写入hbase。

```
#用maxWell采集数据到kafka
#取table字段，根据表名称发送到不同的kafka topic，作为ods层
if(
                ("order_info".equals(tableName)&&"insert".equals(opType))
                  || (tableName.equals("order_detail") && "insert".equals(opType))
                  ||  tableName.equals("base_province")
                  ||  tableName.equals("user_info")
                  ||  tableName.equals("sku_info")
                  ||  tableName.equals("base_trademark")
                  ||  tableName.equals("base_category3")
                  ||  tableName.equals("spu_info")
              ){
                //拼接要发送到的主题
                var sendTopic = "ods_" + tableName
                MyKafkaSink.send(sendTopic,dataJsonObj.toString)
              }
              
#从ods_order_info的kafka主题中创建directStream

#借助phoenix从hbase读取首单信息
Class.forName("org.apache.phoenix.jdbc.PhoenixDriver")
    //建立连接
    val conn: Connection = DriverManager.getConnection("jdbc:phoenix:hbase,hbase1,hbase2:2181")
    //创建数据库操作对象
val ps: PreparedStatement = conn.prepareStatement(sql)
    //执行SQL语句
val rs: ResultSet = ps.executeQuery()

#后去es链接
jestFactory = new JestClientFactory
    jestFactory.setHttpClientConfig(new HttpClientConfig
        .Builder("http://172.18.65.187:9200")
        .multiThreaded(true)
        .maxTotalConnection(20)
        .connTimeout(10000)
        .readTimeout(1000).build())
jestFactory.getObject
```

## 需求3实付分摊金额及交易额统计

>购买商品有时会有优惠，但优惠一般是以订单为单位，现在需要以商品为单位，计算每个商品优惠了多少，进一步计算出每个商品的实付金额。对于除法除不尽的情况，不是最后一件商品的分摊金额使用乘除法比例计算，最后一件商品的分摊金额使用减法计算，用实付金额减去前边计算的其他商品的金额得到。

### 双流join

>由于两个流的数据是独立保存的，独立消费，很有可能同一业务的数据，分布在不同的批次。因为join算子只join同一批次的数据，如果只用简单的join流方式，会丢掉不同批次的数据。
>
>可以通过缓存的形式处理这种情况，将没有匹配join成功的数据放入该流的缓存中，当其需要匹配的数据到来时，再进行join。但是其编写逻辑负责。
>
>也可以通过滑动窗口+数据去重，这种方式处理代码简单，但是会造成数据重复，滑动窗口每次会覆盖一些重复数据，如果使用滚动窗口，每次窗口的数据不重复，无法解决join问题。

## 需求4ADS聚合及可视化

>将dws的数据进行聚合，插入mysql。编写发布接口，让阿里云DataV使用发布的接口。

## Exactly once

>为了实现精确一次消费，有两种方案。1.使用具有幂等性的操作，如es，redis等，在数据操作完成后，再提交offset。2使用mysql事务，将数据操作和索引操作进行绑定，达到原子性事务。

## kafka分层

>对输入的不同类别的原始日志进行处理

## ods

>将maxwell采集到kafka的数据取出，再根据表名称写入对应的ods消息队列。

## dwd

>读取对应的ods消息队列，将订单、订单明细写入dwd消息队列

## dws

>拼接维度形成宽表

## ads

>将数据从dws取出并处理，结果写入mysql、es等。

## 使用的DStream算子

>按照无状态，有状态，窗口，输出区分。

### 无状态

```
transform
mapPartitions
map
join
```

### 窗口

```
window(windowDuration,slideDuration)
```

### 输出DStream

```
foreachRDD
```



## problem

Stream.foreach的操作单位、对象是什么
