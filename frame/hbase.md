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



# java连接hbase

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



# hbase shell 命令

>http://hbase.apache.org/book.html#quickstart

### 连接

```
./bin/hbase shell启动shell
```

### DDL

##### 建表

```
create 'test', 'cf'
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

### DML

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

# 常用端口

16010 for the Master

 16030 for the RegionServer.

16020 数据库连接

2181 zookeeper  

3888 zooo

# 理论知识

>hbase本质是key-value类型数据库，所以它最适合的场景就是key值查询，表现比其他数据库优异。一般用于实时数仓中的临时数据存储，如当日是否登陆过、当日是否消费过。

##### timestamp

```
同一条数据有多个版本，用时间戳后缀作为区分。
```

# HBase用法

### hbase shell语法

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

### Phoenix sql语法

```
#查看数据库里有哪些表
select distinct TABLE_NAME from SYSTEM."CATALOG"

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

