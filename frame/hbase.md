### 安装

http://hbase.apache.org/book.html#quickstart

1下载安装包解压，在conf/hbase-env.sh中设置jdk路径

2快捷启动：运行bin/start-hbase.sh,访问http://localhost:16010

##### 伪分布式

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



##### 完全分布式

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

启动

```
start-hbase.sh
jps
```



### hbase shell

>http://hbase.apache.org/book.html#quickstart

./bin/hbase shell启动shell

基本操作

```
create 'test', 'cf'
list 'test'
describe 'test'
put 'test', 'row1', 'cf:a', 'value1'
scan 'test'
get 'test', 'row1'
disable 'test'
drop 'test'
```

##### 常用端口

16010 for the Master and

 16030 for the RegionServer.

### 与关系型数据库基本结构对比

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