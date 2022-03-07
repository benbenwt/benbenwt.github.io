# 项目

### 目的

```
搭建采集通道，将json格式的用户行为日志文件采集到hdfs上，并进行过滤处理，然后存入hive数据仓库。搭建采集通道，将mysql业务数据采集到hdfs上，并进行过滤处理，然后存入hive数据仓库。搭建hive数据仓库，存储存储用户行为日志和电商平台的业务数据，并合理设计和搭建分层数据仓库，包括ods,dwd,dim,dws,dwt,ads层。实现分析目的，包括活动的分析信息、优惠券的分析信息、订单在省份维度上的分析、订单在spu（商品）上的分析、订单的总体分析、用户的点击路径分析、商品的回购力度分析、用户行为在1、7、30天的分析（）、用户变动信息（回归、离开）、用户停留时间（不同创建日期）、用户1、7、30天总信息分析（下单、上限、）、用户浏览商品信息。并精心设计仓库结构，优化sql和函数，提高处理速度。
```



### 仓库设计

>设计原因，设计技巧

##### ODS

##### DIM

##### DWD

##### DWS

##### DWT

##### ADS

### hivesql脚本

>sql含义，使用的sql技巧，扩展场景

##### ODS

##### DIM

##### DWD

##### DWS

##### DWT

##### ADS

##### azkaban

### ketttle

##### DIM

### superset图表

>活动的分析信息、优惠券的分析信息、订单在省份维度上的分析、订单在spu（商品）上的分析、订单的总体分析、用户的点击路径分析、商品的回购力度分析、用户行为在1、7、30天的分析（）、用户变动信息（回归、离开）、用户停留时间（不同创建日期）、用户1、7、30天总信息分析（下单、上限、）、用户浏览商品信息

### 采集脚本

##### sqoop

##### flume

# summary

```
zookeeper,kafka,flume,sqoop,superset
支持组件(zookeeper，kafka)，数据采集迁移框架(flume，logtash，sqoop)，数仓(hadoop,hive)，数据分析转换引擎（etl）(kettle,hivesql，spark,kylin,presto)，可视化框架(superset)、调度组件(azkaban)、sql封装工具（phoenix）
```

# 服务分布

| 机器       | 服务                                       | 教程机器 |
| ---------- | ------------------------------------------ | -------- |
| 187,hbase  | hive,mysql_hive,flume_file_kafka           | 102      |
| 186,hbase1 | flume_file_kafka                           | 103      |
| 185,hbase2 | mysql_platform,mysql_lisa,flume_kafka_hdfs | 104      |

# 软件版本

![软件版本](..\resources\images\image-20220103163534955.png)

##### 
