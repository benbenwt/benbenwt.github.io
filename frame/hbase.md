[TOC]
# 安装与启动

>修改hbase默认ssh端口
>
>hbase-env.sh
>
>export HBASE_SSH_OPTS="-p 52544"

http://hbase.apache.org/book.html#quickstart

1下载安装包解压，在conf/hbase-env.sh中设置jdk路径

```
在hbase-site.xml中指定zk的请求地址，在regionserver中填写regionserver服务器ip。
```

2快捷启动：运行bin/start-hbase.sh,访问http://localhost:16010

## 伪分布式

hdfs-site.xml,core-site.xml

```
cp  hdfs-site.xml,core-site.xml   $HBASE_HOME/conf
```

hbase-env.sh

```
export JAVA_HOME=directory_name
```

hbase-site.xml

```
<property>
  <name>hbase.cluster.distributed</name>
  <value>true</value>
</property>
<property>
  <name>hbase.rootdir</name>
  <value>hdfs://hbase:9000/hbase</value>
</property>
```

```
#hbase自动在hdfs中创建hbase文件夹
start-hbase.sh
hadoop fs -ls /hbase
hbase shell
```



## 完全分布式

>要求:hadoop3.1.4配置好
>
>ssh无密码连接配置好
>
>http://hbase.apache.org/book.html#quickstart_fully_distributed

| Node Name | Master | ZooKeeper | RegionServer |
| :-------- | :----- | :-------- | :----------- |
| hbase     | yes    | yes       | no           |
| hbase1    | backup | yes       | yes          |
| hbase2    | no     | yes       | yes          |

regionservers

```
hbase1
hbase2
```

backup-master

```
hbase1
```

hbase-env.sh

```
export JAVA_HOME=directory
```

hbase-site.xml

```

<configuration>
  <property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
  </property>
  <property>
    <name>hbase.rootdir</name>
    <value>hdfs://hbase:9000/hbase</value>
  </property>
<property>
  <name>hbase.zookeeper.quorum</name>
  <value>hbase,hbase1,hbase2</value>
</property>
<property>
  <name>hbase.zookeeper.property.dataDir</name>
  <value>/usr/local/zookeeper</value>
</property>
</configuration>

```

### 启动

```
hbase-daemons.sh start master
hbase-daemons.sh start regionserver
```



```
start-hbase.sh
jps
```



## 关闭

```
hbase-daemons.sh stop master
stop-hbase.sh

hbase-daemons.sh stop regionserver
stop-hbase.sh

```


# HBase理论知识

>hbase本质是key-value类型数据库，所以它最适合的场景就是key值查询，表现比其他数据库优异。一般用于实时数仓中的临时数据存储，如当日是否登陆过、当日是否消费过。


## 常用端口

16010 for the Master

 16030 for the RegionServer.

16020 数据库连接

2181 zookeeper  

3888 zooo

## 存储结构

>HBase通过构建多个map键值对存储数据，一个列簇放置在一起作为同一个value，row_key作为key，value部分除了对应的列簇数据，还有一些固定的数据，如TimeStamp时间戳、Type数据操作类型（PUT,DELETE），列簇名称，列名称。一条HBase数据取出来类似如下，(row_key,column_family,column qualifier,timestamp,type,value)

## 数据类型

### Name Space

>命名空间，类似mysql的DataBase概念。HBase自带两个命名空间，分别是hbase和default。hbase存放的是hbase内置的表，default存放的是用户默认使用的命名空间。

### Region

>类似关系型数据库的表概念，不同的是，hbase只需要声明列簇即可，不需要具体的列。向hbase写入时，可以灵活的指定列，所以hbase可以应对字段的变更。因为当你增加一个列时，hbase只需要创建一条新的键值对记录，在其中指定列簇、列名、timestamp、type、value即可，不需要操作其他数据。

### Row

>一个RowKey和多个列组成，数据按照Rowkey的字典顺序存储。查询数据只能通过rowkey，rowkey如何设计很重要。

### Column

>通过Column family 列族和Column Qualifier列限定符指定列。

### TimeStamp

>用于标识数据的不同版本，每条数据写入时都会添加。

### Cell

>由rowkey，column family，column qualifier，timestamp唯一确定的单元。cell中的数据无类型，都是字节形式。

## 基础架构

>由Master，Region Server构成，通过zookeeper提供高可用、regionserver的监控、集群配置的维护、元数据入口等。HDFS为HBase提供几层数据存储服务。

### Master

>Master是所有Region Server的管理者。
>
>功能：管理对于表的create，delete，alter
>
>对于RegionServer的管理，分配regions到server，负载均衡、故障转义、监控状态。

### Region Server

>一台分区服务器对应一个server，负责对数据的操作，如get ,put,delete。对分区的操作，如splitRegion,compactRegion。

>RegionServer上有多个Region分区、以及一个WAL(Write-Ahead logFile)、一个Block Cache。WAL和Block Cache在region server是唯一的，多个region分区共同使用。

### WAL(Write-Ahead logFile)

>WAL(Write-Ahead logFile)是写入日志，为了防止memStore数据丢失，在写入mem  Store前，先在WAL中进行记录，如果mem Store故障，可以通过WAL进行恢复。

### Block Cache

>每个region server都有一个读缓存。

### Region

>每个Region分区包含一个或多个store组成，至少是一个store。实际上，hbase为每个列族创建一个store，有多少列族就有多少store。
>
>可以理解为在行的方向上分为多个Region，在列的方向上划分为多个Store，在一个Region内是包含对应行的全部列族的。也就是说，每个Region内的store数目一致，就是每个列族。

### Store

>存储列族的基本单位，由mem Store、StoreFile构成。
>
>mem Store是内存缓存，当写入的数据达到阈值时，刷写到StoreFile。
>
>StoreFile是实际存储的文件系统，也就是HDFS文件，可以有多个StoreFile。每次mem Store的刷写都会生成一个新的StoreFile，底层以HFile格式保存。

### HFile

>HBase中KeyValue数据的存储格式，基于谷歌BigTable中的SSTable。
>
>大致包括以下File Info，Data Index，Meta Index，Trailer。

## 写流程

>设计的主要组件：regionserver、region、mem store、WAL、storeFile，怎么Master没有参与呢，看来Master只负责region的分区，表的管理、监控状态等。
>
>1Client先访问zookeeper，获取hbase：meta表位于哪个Region Server。
>
>2访问对应的Region Server，获取hbase：meta表，查询出位于哪个region分区，将该table的分区信息以及meta表的信息缓存在客户端的meta cache，方便下次访问。
>
>3.与目标Region server通讯
>
>4将数据顺序加入WAL
>
>5将数据写入memStore，数据会在memStore进行排序
>
>6向客户端发送ack
>
>7等待memStore的刷写时机后，将数据刷写到HFile。

## 读流程

>1 访问zk后去region server
>
>2访问region server获取region分区，获取table的meta信息到客户端并缓存。
>
>3与目标Region server进行通讯
>
>4分别在Block Cache，Mem Store和StoreFile中查询数据，并进行合并，得到同一条数据的不同版本，即timestamp或type不同。
>
>5将查询到的数据缓存到Block Cache
>
>6.返回结果给客户端

## StoreFile Compaction

>由于每次刷写都会生成新的HFile，当对数据查询时，其不同timestamp、type的数据可能分布在多个HFile中，因此需要查询遍历所有HFile，为了减少HFile的数量，会进行StoreFile Compaction.
>
>Compaction分为两种，Minor Compaction和Major Compaction。Minor Compaction将临近的若干个HFile合并成大的HFile。但不会清理过期和删除的数据。Major Compaction会将一个store下的所有HFile合并为一个大HFile，会清理过期和删除的数据。

## Region Split

>当1个region中的某个Store下的所有StoreFile总大小超过Min（R^2*"hbase.hregion.memestore.flush.size","hbase.hregion.max.filesize"）就会拆分，其中R为当前regionserver中属于该table的region个数。

## MemStore刷写时机

>1当某个分区的store的memstore达到了hbase.hregion.memstore.flush.size，该region全部刷写。
>
>2当region server中的memstore的总大小达到java_heapsize*hbase.regionserver.global.memstore.size*hbase.regionserver.global.memstore.size.lower.limit，region会将所有memstore按照从大到小的顺序依次进行刷写，直到memstore减小到lowerlimit以下。

## 相关链接

>尚硅谷官网：HBase学习笔记

## rowkey设计原则

>目的：根据查询方法设计rowkey，避免全表扫描，避免集中在单节点。
>
>1唯一原则
>
>2排序原则，范围查询
>
>3散列原则，并发节点
>
>热点问题
>
>1reverse反转
>
>2前缀拼接，salt加盐
>
>3hash散列
>
>长度问题：
>
>1越短越好
>
>

## 列族

>列族建议在1-3个。
>
>列族数对Flush的影响：
>
>每个列族有自己的memstore，越多的列族，创建的memstore越多，列族越多，flush一次生成一个HFile，那么小文件就很多。
>
>列族数对Split的影响：
>
>拆分小的列族，小文件变多
>
>列族数对Compaciton的影响：
>
>region级别，不必要io
>
>列族数对regionserver的影响：
>
>占用内存

# HBase用法

## hbase shell 命令

>http://hbase.apache.org/book.html#quickstart

### 连接

```
./bin/hbase shell启动shell
```

### DDL (Data Define Language)

##### 建表

```
create 'test', 'columnfamily1','columnfamily2'
#展示表
list 'test'

#展示表结构
describe 'test'
desc 'test
```

##### 改表

```
#删除表
disable 'student'
drop 'student'
#清空表
truncate 'stu'
#增加列
alter 'stu2',NAME=>'f1',VERSIONS=>4
#删除列
alter 'stu2', NAME => 'cf1',METHOD => 'delete'
alter 'stu2', 'delete' => 'f1'
```

##### 查看hbase状态

```
#查看hbase结点状态
status
#查看hbase版本
version
#查看当前登录用户
whoami
#查看数据库里有哪些表
list
```



### DML (Data Manage Language)

##### 增删改查

```
#插入数据
put 'test', 'row_key', 'cf:a', 'value1'
#查看所有数据
scan 'test'
#查询分页
scan  'stu2',{COLUMNS => 'cf1:age', LIMMIT 10, STARTROW => 'xx'}
#查询单个数据
get 'test', 'row_key'
get 'test', 'row_key'，‘info:age'
#删除数据
deleteall 'student','1001'
delete 'student','1001',‘info:age'
#清空表数据
truncate 'student'

list_namespace
create_namespace "test"
describe_namespace "test002""
list_namespace_tables "test"
```

##### 主键查询

```
get 'test', 'row_key'
#按照行键过滤
scan ‘表名’, FILTER=>“RowFilter(=,‘binary:行健值’)”
scan ‘test’, FILTER=>“RowFilter(=,‘binary:01’)”

#过滤行键前缀过滤
scan ‘表名’, FILTER => “PrefixFilter (‘行健前缀’)”

#按照索引范围过滤
scan 'tableName',{STARTROW=>'startRow',ENDROW=>'endRow'}

```

##### 非主键查询

```
https://blog.csdn.net/weixin_43980049/article/details/90377435
#查询出某个表内列值包含指定字符串的记录,相当于where查询
scan '表名', FILTER=>"ValueFilter(=,'substring:列值')"
scan 'test', FILTER=>"ValueFilter(=,'substring:bob')"
```

```
#查找列簇的名字等于此字符的，注意是列簇的名字，不是列簇对应的值，也不是列的名字。
scan 'test1',FILTER=>"FamilyFilter(=,'substring:c')"
#找到有name属性的记录
scan 'test',FILTER=>"FamilyFilter(=,'substring:name')"
```



主键聚合

非主键聚合

## Java API

>java连接hbase也需要hadoop环境，再通过System.setProperty("hadoop.home.dir", "C:\\Users\\asus\\Desktop\\hadoop-3.1.4\\hadoop-3.1.4");设定路径。
>
>直接在reducer或map中操作hbase，使用相同的代码进行连接就行了。

HbaseConnector

```
package com.weitao.mr.wordcount;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.util.Bytes;

import java.io.IOException;

public class HbaseConnect {
    Connection conn;
    Admin admin;
    public void init() throws IOException {
        System.out.println("hbase--------------------------------------------");
        conn=null;
        System.setProperty("hadoop.home.dir", "C:\\Users\\asus\\Desktop\\hadoop-3.1.4\\hadoop-3.1.4");
        Configuration conf = HBaseConfiguration.create();
        conf.set("hbase.zookeeper.quorum","hbase");
        conf.set("hbase.zookeeper.property.clientPort","2181");
        try {
            conn = ConnectionFactory.createConnection(conf);
            System.out.println("连接hbase成功");
        } catch (IOException e){
            System.out.println("连接hbase失败");
            e.printStackTrace();
        }
        admin=conn.getAdmin();
    }
    
     public  void insert(String row,String value)
    {
        TableName tableName=TableName.valueOf("test");
        try {
            Table table=conn.getTable(tableName);
            Put put=new Put(Bytes.toBytes(row));
            put.addColumn("id".getBytes(),"pattern".getBytes(),value.getBytes());
            table.put(put);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        HbaseConnect hbaseConnect=new HbaseConnect();
        try {
            hbaseConnect.init();
        } catch (IOException e) {
            e.printStackTrace();
        }
        hbaseConnect.insert("123","123");
    }
//    private void createTable(String name)
//    {
//        Admin admin=conn.getAdmin();
//        TableName tableName=TableName.valueOf(name);
//        TableDescriptor tableDescriptor=new TableDescriptor(tableName);
//        admin.createTable(TableDescriptor);
//    }
    public void testScan(String name) throws IOException {
        TableName tableName=TableName.valueOf(name);
        Table table=conn.getTable(tableName);
        if(admin.tableExists(tableName))
        {
            System.out.println("exist");
        }
        Scan scan = new Scan();
        ResultScanner resultScanner = table.getScanner(scan);
        for(Result result: resultScanner){
            System.out.println("scan:  " + result);
        }
    }
    public void retrieveByRowKey(String name,String rowKey)
    {
    }
    public void deleteByRowKey(String name,String rowKey)
    {
        TableName tableName=TableName.valueOf(name);
        Table table=null;
        try {
             table=conn.getTable(tableName);
        } catch (IOException e) {
            e.printStackTrace();
        }
        Delete delete=new Delete(Bytes.toBytes(rowKey));
        try {
            table.delete(delete);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    public  void testDelete(String name){
        TableName tableName=TableName.valueOf(name);
        try {
            admin.disableTable(tableName);
            admin.deleteTable(tableName);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
   
}

```



pom

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.weitao.mr</groupId>
    <artifactId>mapreduce_start</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
            <version>2.8.2</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-common</artifactId>
            <version>3.1.4</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-client</artifactId>
            <version>3.1.4</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-hdfs</artifactId>
            <version>3.1.4</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-mapreduce-client-core</artifactId>
            <version>3.1.4</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-mapreduce-client-jobclient</artifactId>
            <version>3.1.4</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-mapreduce-client-common</artifactId>
            <version>3.1.4</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase-client</artifactId>
            <version>2.3.3</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase-common</artifactId>
            <version>2.3.3</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase-server</artifactId>
            <version>2.3.3</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase</artifactId>
            <version>2.3.3</version>
            <type>pom</type>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>8</source>
                    <target>8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```




## Phoenix

### 安装

>官网quick start:https://phoenix.apache.org/Phoenix-in-15-minutes-or-less.htm

```
#解压压缩包
#将其中的phoenix-server-hbase-2.3-5.1.2.jar拷贝到hbase/lib下，并重启hbase
#启动交互shell
bin/sqlline.py <your_zookeeper_quorum>
#从csv导入数据到hbase，sql文件为建表语句，csv为数据。
./psql.py <your_zookeeper_quorum> us_population.sql us_population.csv
./psql.py 172.18.65.187 us_population.sql us_population.csv
#登入shell
bin/sqlline.py 172.18.65.187
#查看从csv插入的数据
SELECT state as "State",count(city) as "City Count",sum(population) as "Population Sum"
FROM us_population
GROUP BY state
ORDER BY sum(population) DESC;
#如果需要在其他客户端访问phoenix，引入phoenix-client-hbase-2.3-5.1.2.jar 即可。
```

### DDL

#### 建表

```
create table test(host varchar not null primary key, description  varchar)salt_buckets=16;
#查看表
!table
!describe tablename

#查看数据库里有哪些表
select distinct TABLE_NAME from SYSTEM."CATALOG"

#查看sql执行历史
!history
!dbinfo
!index tb
help

#删除表
drop table if exists test;
```

```
create table gmall2020_province_info 
       (id varchar primary key,info.name 
        varchar,info.area_code varchar,info.iso_code varchar
        )SALT_BUCKETS = 3
        
create table gmall2020_user_info 
        (id varchar primary key ,user_level varchar, birthday varchar, 
        gender varchar, age_group varchar , gender_name varchar
        )SALT_BUCKETS = 3

```



### DML

>语法介绍：https://blog.csdn.net/chenfeng_sky/article/details/103248398

#### 插入数据

```
upsert into tb values('ak','hhh',222)
```

#### 删除数据

```
delete from tb; 清空表中所有记录，Phoenix中不能使用truncate table tb；
delete from tb where city = 'kenai';
```

#### 查询数据

```
select * from test limit 1000;
select * from test limit 1000 offset 100;

#查看表
show tables;

```



### JAVA API

>https://blog.csdn.net/xwd127429/article/details/110483966

```
#添加依赖
#phoenix依赖需要将phoenix-client-hbase加入到类路径中
<dependency>
    <groupId>org.apache.hbase</groupId>
    <artifactId>hbase-client</artifactId>
    <version>1.2.12</version>
</dependency>

<dependency>
    <groupId>org.apache.hbase</groupId>
    <artifactId>hbase-server</artifactId>
    <version>1.2.12</version>
</dependency>
```



```
public static void main(String[] args) throws Throwable {

	try {

		Class.forName("org.apache.phoenix.jdbc.PhoenixDriver");

		//这里配置zookeeper的地址，可单个，多个(用","分隔)可以是域名或者ip

		String url = "jdbc:phoenix:hbase:2181";

		Connection conn = DriverManager.getConnection(url);

		Statement statement = conn.createStatement();

		long time = System.currentTimeMillis();

		ResultSet rs = statement.executeQuery("select * from test");

		while (rs.next()) {
			String myKey = rs.getString("MYKEY");
			String myColumn = rs.getString("MYCOLUMN");

			System.out.println("myKey=" + myKey + "myColumn=" + myColumn);
		}

		long timeUsed = System.currentTimeMillis() - time;

		System.out.println("time " + timeUsed + "mm");

		// 关闭连接
		rs.close();
		statement.close();
		conn.close();

	} catch (Exception e) {
		e.printStackTrace();
	}
}
```



### sql练习

```
select * from us_population where city like 'Los%';
select * from us_population where population>=1;
select * from  test where TO_DATE(ttime,'yyyyMMddHHmmss')=TO_DATE('20141125','yyyyMMdd')
```



# 与关系型数据库基本结构对比

传统数据库一个表的结构如下：

| 姓名       | 年龄 | 性别       | 成绩 |
| :--------- | :--- | :--------- | :--- |
| liujinghui | 18   | strong man | 100  |
| zhangsan   | 99   | Niangs     | -1   |

转换成HBase数据库的表结构就如下所示

|             | Info      |          |          | Score      |             |
| ----------- | --------- | -------- | -------- | ---------- | ----------- |
| **Row_key** | Info:name | Info:age | Info:sex | Score:name | Score:score |
|             |           |          |          |            |             |

通过行key，列族：列确定一个单一元素

[org.apache.hadoop.hbase.client.RpcRetryingCallerImpl] - Call exception, tries=6, retries=16, started=14301 ms ago, cancelled=false, msg=Call to hbase2/172.18.65.188:16020 failed on connection exception: org.apache.hbase.thirdparty.io.netty.channel.AbstractChannel$AnnotatedConnectException: Connection refused: no further information: hbase2/172.18.65.188:16020, details=row 'test' on table 'hbase:meta' at region=hbase:meta,,1.1588230740, hostname=hbase2,16020,1613512234805, seqNum=-1, see https://s.apache.org/timeout

# problem

##### org.apache.hbase.thirdparty.io.netty.channel.AbstractChannel$AnnotatedConnectException

```
hadoop dfsadmin -safemode leave
```

##### org.apache.hadoop.hbase.PleaseHoldException: Master is initializing

>**如果是个人的测试环境!!!**数据不重要，可以把zk中的数据全删了
>deleteall /hbase
>再把hbase在hdfs的文件全删了。

##### address already in use

```
在hbase-env.sh里面设置HBASE_MANAGES_ZK 改成false 默认是true，先手动启动zookeeper，然后再启动hbase
```

##### 无法根据pid关闭hbase进程

```
#指定一个合适的目录，tmp文件夹会丢失pid，因为系统定期对此文件夹清理。
vim hbase-env.sh
PID_DIRECOTYR: /var/hbase/pid
```

