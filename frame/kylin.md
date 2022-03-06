### 安装

>安装Kylin前需先部署好Hadoop、Hive、Zookeeper、HBase

```
tar -zxvf apache-kylin-3.0.2-bin.tar.gz -C /opt/module/
mv /opt/module/apache-kylin-3.0.2-bin /opt/module/kylin
#解决依赖冲突
vim /opt/module/kylin/bin/find-spark-dependency.sh
在 spark_dependency后追加! -name '*jackson*' ! -name '*metastore*'
```

### 启动

```
bin/kylin.sh start
进入http://hbase:7070/kylin访问
```

### 使用

##### 创建project

##### 创建datasource

##### 创建model

```
```

##### 创建cube

##### 执行cube

### 理论

```
Kylin不能处理Hive表中的复杂数据类型（Array,Map,Struct）,即便复杂类型的字段并未参与到计算之中。故在加载Hive数据源时，不能直接加载带有复杂数据类型字段的表。需要创建table view过滤掉复杂字段。
kylin将创建好的model和cube进行执行，让后将任务定时执行并放入hbase用于查询。它本质上是在做数据的聚合工作，并将结果存入可以快速访问的hbase中。kylin处理好数据后，可用于可视化bi等工具使用。presto主要提供sql查询，没有完整的存储结果方案。
```

![kylin](..\resources\images\kylin.png)
