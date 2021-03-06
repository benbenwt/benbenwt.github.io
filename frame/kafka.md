

# 背景

>kafka是一种分布式的基于发布与订阅的消息队列。

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



##### standalone

```
#启动独立的zookeeper
zkServer.sh start
zkCli.sh -server localhost：2181
```

```
#查看zookeeper结点
ls /
get /zk_test
set /zk_test name
delete /zk_test

C:\Users\guo\Desktop\lab\Result\stix2\VirusShare_ELF_20200405_part8\598bbee58a81eec6d30326d207a260b9.json
C:\Users\guo\Desktop\lab\Result\stix2\VirusShare_ELF_20200405_part8\0598aea3e4e081d8ce1cda8649f8b0ec.json
```

```
#使用默认的zookeeper，若已用独立的zookeeper，不要启动此选项。
bin/zookeeper-server-start.sh config/zookeeper.properties
#设置kafka下的zookeeper.properties和server.properties
bin/kafka-server-start.sh config/server.properties
bin/kafka-server-stop.sh
bin/kafka-topics.sh --create --bootstrap-server hbase:9092 --replication-factor 1 --partitions 1 --topic test
#查看所有topic
bin/kafka-topics.sh --list --bootstrap-server hbase:9092
test
bin/kafka-topics.sh --delete --zookeeper localhost:2181 --topic test
bin/kafka-console-producer.sh --broker-list hbase:9092 --topic 
bin/kafka-console-consumer.sh --bootstrap-server hbase:9092 --topic test --from-beginning
```

### problem

##### poll无反应

```
开启过多消费者，将限制的进程杀死即可恢复。
```

```
查看客户端是否reset offset，若显示了则一般正常。
```

