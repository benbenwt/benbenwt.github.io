

### 安装

```
#解压复制到对应位置
#创建logs文件夹
mkdir logs
#允许删除topic，填写logs文件夹，指定zk集群地址。
cd config
vim server.properties
#指定kafka的id，每台机器不一样。
broker.id=0
#删除topic功能使能
delete.topic.enable=true
#kafka运行日志存放的路径
log.dirs=/opt/module/kafka/data
#配置连接Zookeeper集群地址
zookeeper.connect=hadoop102:2181,hadoop103:2181,hadoop104:2181/kafka
```

### 启动

```
#启动独立的zookeeper
zkServer.sh start
zkCli.sh -server localhost：2181
#指定config启动kafka
kafka-server-start.sh     conf/server.properties
```

### 管理

##### 查看zookeeper结点

```
#查看zookeeper结点
ls /
get /zk_test
set /zk_test name
delete /zk_test
```

##### 创建topic

```
#创建topic
bin/kafka-topics.sh --create --bootstrap-server hbase:9092 --replication-factor 1 --partitions 1 --topic test
```

##### 查看所有topic

```
#查看所有topic
bin/kafka-topics.sh --list --bootstrap-server hbase:9092
test
bin/kafka-topics.sh --delete --zookeeper localhost:2181 --topic test
bin/kafka-console-producer.sh --broker-list hbase:9092 --topic 
bin/kafka-console-consumer.sh --bootstrap-server hbase:9092 --topic test --from-beginning
```



# 理论知识

>kafka是高吞吐量的开源流处理平台，可以作为一种分布式的基于发布与订阅的消息队列，但消息队列的部分功能需要自己编写，如失败重试动作，同步异步任务等，与rabbitmq有区别。

## 异步处理

>用户请求服务后需等待服务完成再得到结果信息，这是同步处理。对于异步处理，用户提交请求后，请求放入消息队列后，无需等待执行完成，用户即可结束进程。异步处理可以实现解耦，异步通信，缓冲缓解服务器压力，提高灵活性和峰值处理能力。

##  消息队列的两种模式

### 1点对点模式

一对一，消费者主动拉取数据，消息收到后清楚，即一个消息对应一个消费者。

### 2发布订阅模式

>此模式又分为消费者主动拉取和队列主动推送，推送速度有不同的角色决定。

生产者发布消息到topic中，一个消息可以对应多个消息订阅者。

kafka是基于发布订阅模式的消费者拉取。

#### 消费者主动拉取模式

>消费者需要主动询问是否有我的消息，执行了长轮询，消耗资源。

#### 队列主动推送模式

>消费者性能有差别，队列推送要满足不同的速度。

# kafka

## kafka基础架构

生产者生产消息  kafka集群管理消息  消费者消费消息   Zookeeper注册消息

### kafka集群管理消息

kafka由多个broker构成，每个broker由topic和partion构成，一个topic可以有多个partion，由partion编号进行区分。每个topic的partion有多个，分布在不同机器上，他们有一个leader，当leader无法服务后，由其他机器上的follower代理。

```
消费者通过指定topic 进行消费， partion可以将topic中的数据分开存储。
offset针对特定topic，特定partion，特定消费者group。Zookeerper中保存这每个topic下的每个partition在每个group中消费的offset 。
```

### 消费者消费消息

#### consumer group

同一个分区的数据只能被同一个cg里的一个消费者消费。

```
conda install --channel https://conda.anaconda.org/conda-forge kafka-python
```

##### 版本

>kafka 2.4.1 zookeeper 3.5.9

```
kibana 5601,es 9200
```



### problem

##### poll无反应

```
开启过多消费者，将限制的进程杀死即可恢复。
```

```
查看客户端是否reset offset，若显示了则一般正常。
```

