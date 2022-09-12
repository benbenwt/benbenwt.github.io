# Flink kappa架构
>kappa是指实时和离线使用同一套代码，减少维护的量。比如，统计每日活跃用户数，如果使用lambda架构需要两套代码，但是使用kappa就只用流处理完成按日统计，这个日结果是实时的，所以完成了实时处理的功能。当一日结束后，当前日的数据就计算完了，将结果放入数据库，方便后续使用。就是说将离线数仓各层之间的流动，变为实时流处理了，一旦hive表发生变化，那么就会传递到下一层。至于这些层，可以采用kafka也可以采用数据库等，这取决于是否要复用这些数据。

>FLINK_ti:http://192.168.31.177:5602/goto/441f74056609ece736f5ad78a04335cd
## 分块
### flume采集
>从磁盘采集到kafka

### Flink实时处理
#### ods
>原始数据
>json发送到kafka队列中后就是ods数据了。
#### dwd
>解析后的基本粒度，如一条记录。
>解析json为基本粒度
>日期，地区，样本类别，架构类别,攻击者，ip，漏洞编号，地区iso编号。

##### 映射iso编码
##### 反射填充对象
>json解析后的信息需要填充到对应的实体类，便于后边使用Flink SQL进行指标统计。

#### dws
>轻度聚合，如一天。
>聚合后写入kafka。
#### dwt
>一个月聚合
>聚合后写入存储elasticsearch。
#### ads
>关注的主题：location，安全信息类别，安全信息架构分布，其他（攻击者，漏洞，ip地址）
>当新增一条记录时，按规则聚合后，sink的数据是覆盖还是增加，这取决于使用的sink端，和编写的sink逻辑。当使用elasticsearch时，会直接插入一条新数据，如果我们希望其覆盖之前的，可以通过指定唯一id_来实现。

### Elasticsearch
#### 索引设计
>必须有一个date类型的时间字段，因为es的dashboard需要此字段确认数据时间范围。
>如果需要使用地图功能，需要存储地区对应的iso code编码。
>注意需要进行计算字段，指定为需要的类型，如float等。

```
PUT /threatactormonth
{
  "settings": {
    "number_of_shards": 3
  },
 "mappings": {
    "properties": {
      "nums": {
          "type": "long"
        },
        "threatactor": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "year_month": {
          "type": "date"
        }
    }
  }
}
```

#### 国家iso编码可视化
>elasticsearch iso-3166介绍：https://maps.elastic.co/v7.11/index.html?locale=en#file/world_countries
>点击visualize后，创建Map，然后add layer 选择 EMS Boundaries类型指定数据源等设置。
## problem
### flume spooldir监控递归子目录
>1.19版本将此选项开启即可
>agent1.sources.file1.recursiveDirectorySearch=true
### flume读取json一行只读了一半
>flume只读了一行json的一半，导致flink读取到了一个无界的字符串。
>尝试了不同文件发现，一行如果超过了2048，flume就只采集2048个字符到kafka。且flume日志报了exceed max 2048的错误
>Flink的spooldir有一个参数，deserializer.maxLineLength，此参数设置单行数据byte上限，默认为2kb。

### Flink读取kafka中flume采集数据乱码
>agent1.channels.channel1.parseAsFlumeEvent = false
>将此项关闭，如果开启表示按照FlumeEvent的方式对其序列化并写入kafka，这意味着后续你会使用Flume Source反序列化该字节流。当我们存入kafka提供给其他应用使用时，应关闭此选项。

### flume采集单个文件大小限制
>flume采集的文件大小也有限制，event的大小需要在kafka的服务端设置，内容如下：
>server.properties加上的message.max.bytes、max.request.size配置，max.request.size要比message.max.bytes小。
>另外，还要给kafka的producer设置max.request.size参数，现在使用的是flume，所以在flume的配置文件中修改kafka
配置,添加producer.max.request.size。

### Flink connector，java对象传输到kafka序列化与反序列化
>当dwd层完成计算后，需要将数据写入kafka，然后再由dws从kafka读出该数据，进一步统计。在使用java编写flink程序时，这些数据就是用java对象表示和操作的。所以Flink如何将java对象数据写入kafka，以及如何读出，需要我们自己实现序列化和反序列化方法。通过实现org.apache.flink.api.common.serialization.SerializationSchema接口，我们可以定义自己的序列化方法;
>主要修改addSink和addSource指定的Schema即可。

```
#dwd层，即需要序列化方法将对象写入kafka的应用

package com.ti.dwd;

import com.ti.domain.SecurityInfo;
import com.ti.functions.map.ParseJsonMapFunction;
import com.ti.utils.serializer.FlinkSecurityInfoSerializationSchema;
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.api.common.serialization.SimpleStringSchema;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaProducer;

import java.util.Properties;

public class ParseJson {
    public static void main(String[] args) throws Exception {
//        从kafka创建流，元素为json str。
        StreamExecutionEnvironment env=StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        Properties properties=new Properties();
        properties.setProperty("bootstrap.servers","localhost:9092");
        properties.setProperty("group.id","dwd_ParseJson");
        properties.setProperty("key.deserializer","org.apache.kafka.common.serialization.StringDeserializer");
        properties.setProperty("value.deserializer","org.apache.kafka.common.serialization.StringDeserializer");
        properties.setProperty("auto.offset.reset","latest");

        DataStream<String> kafkaStream=env.addSource(new FlinkKafkaConsumer<String>("ods_json",new SimpleStringSchema(),properties));
        kafkaStream.print("2");

        DataStream<SecurityInfo> infoStream=kafkaStream.map(new ParseJsonMapFunction());

        Properties properties1=new Properties();
        properties1.setProperty("bootstrap.servers","localhost:9092");

        infoStream.addSink(new FlinkKafkaProducer<SecurityInfo>("dwd_ParseJson",new FlinkSecurityInfoSerializationSchema<SecurityInfo>(),properties1));
        env.execute("ParseJson");
    }
}

#dws层，即需要反序列化方法将kafka的字节流转为对象的应用
package com.ti.dws;

import com.ti.domain.SecurityInfo;
import com.ti.functions.map.ParseJsonMapFunction;
import com.ti.utils.serializer.FlinkSecurityInfoDeserializationSchema;
import com.ti.utils.serializer.FlinkSecurityInfoSerializationSchema;
import org.apache.flink.api.common.functions.ReduceFunction;
import org.apache.flink.api.common.serialization.SimpleStringSchema;
import org.apache.flink.api.java.functions.KeySelector;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer;
import org.apache.flink.table.api.Table;
import org.apache.flink.table.api.bridge.java.StreamTableEnvironment;

import java.util.Properties;

public class LocationDayPeriod {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env=StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        Properties properties=new Properties();
        properties.setProperty("bootstrap.servers","localhost:9092");
        properties.setProperty("groupid","dws_LocationDayPeriod");
        properties.setProperty("key.deserializer","org.apache.kafka.common.serialization.StringDeserializer");
        properties.setProperty("value.deserializer","org.apache.kafka.common.serialization.StringDeserializer");
        properties.setProperty("auto.offset.reset","latest");

        DataStream<SecurityInfo> infoStream=env.addSource(new FlinkKafkaConsumer<SecurityInfo>("dwd_ParseJson",new FlinkSecurityInfoDeserializationSchema<SecurityInfo>(SecurityInfo.class), properties));
        StreamTableEnvironment tableEnv=StreamTableEnvironment.create(env);
        Table infoTable=tableEnv.fromDataStream(infoStream);
        infoTable.printSchema();
        tableEnv.toChangelogStream(infoTable).print("infoTable:");
        Table countLocationTable=tableEnv.sqlQuery("select location,sampletime,count(location) from "+infoTable+" group by location,sampletime");
        Table countTypeTable=tableEnv.sqlQuery("select type,sampletime,count(location) from "+infoTable+" group by type,sampletime");
        Table countArchTable=tableEnv.sqlQuery("select architecture,sampletime,count(location) from "+infoTable+" group by architecture,sampletime");
        tableEnv.toChangelogStream(countLocationTable).print("countLocationTable: ");
        tableEnv.toChangelogStream(countTypeTable).print("countTypeTable:");
        tableEnv.toChangelogStream(countArchTable).print("countArchTable:");
//  所有数据中，每天分布了多少个。
        env.execute("locationDayPeriod");
    }
}

```

```
#序列化 Flink schema
package com.ti.utils.serializer;

import org.apache.flink.api.common.serialization.DeserializationSchema;
import org.apache.flink.api.common.serialization.SerializationSchema;

public class FlinkSecurityInfoSerializationSchema<T> implements SerializationSchema<T> {

    @Override
    public byte[] serialize(T element) {
        return SerializeUtil.serialize(element);
    }
}

```
```
#反序列化  Flink schema
package com.ti.utils.serializer;

import org.apache.flink.api.common.serialization.DeserializationSchema;
import org.apache.flink.api.common.typeinfo.TypeInformation;
import org.apache.flink.api.java.typeutils.TypeExtractor;

import java.io.IOException;

public class FlinkSecurityInfoDeserializationSchema<T> implements DeserializationSchema<T> {
    private Class<T> clazz;
    public FlinkSecurityInfoDeserializationSchema(Class<T> clazz){
        this.clazz=clazz;
    }

    @Override
    public T deserialize(byte[] message) throws IOException {
        return (T)SerializeUtil.deserialize(message,clazz);
    }

    @Override
    public boolean isEndOfStream(T nextElement) {
        return false;
    }

    @Override
    public TypeInformation<T> getProducedType() {
        return TypeExtractor.getForClass(clazz);
    }
}

```

```
#实现具体功能的util

package com.ti.utils.serializer;

import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;

public class SerializeUtil {

    public static byte[] serialize(Object object) {
        ObjectOutputStream oos = null;
        ByteArrayOutputStream baos = null;
        try {
            // 序列化
            baos = new ByteArrayOutputStream();
            oos = new ObjectOutputStream(baos);
            oos.writeObject(object);
            byte[] bytes = baos.toByteArray();
            return bytes;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

    @SuppressWarnings("unchecked")
    public static <T> Object deserialize(byte[] bytes,Class<T> className) {
        ByteArrayInputStream bais = null;
        T tmpObject = null;
        try {
            // 反序列化
            bais = new ByteArrayInputStream(bytes);
            ObjectInputStream ois = new ObjectInputStream(bais);
            tmpObject = (T)ois.readObject();
            return tmpObject;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }
}
```


# TI离线demo

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


## problem

