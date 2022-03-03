### 安装

```
#下载压缩包并解压
#创建zkData文件夹
mkdir zkData
cd zkData
#为每个结点指定不同id
vim myid
xsync myid
#修改zoo.cfg，指定之前创建的zkData
cp  zoo-sample.cfg  zoo.cfg
vim  zoo.cfg
dataDir=/opt/module/zookeeper-3.5.7/zkData
#并增加如下配置，指定zk的集群结点。
server.1=hadoop1:2888:3888
server.2=hadoop2:2888:3888
server.3=hadoop3:2888:3888
#安装xsync
yun install xsync
#分发配置
xsync  zoo.cfg
```

### 启动

```
bin/zkServer.sh start
bin/zkServer.sh status
```

### 管理

```
#查看zookeeper结点
ls /
get /zk_test
set /zk_test name
delete /zk_test
```



### 理论知识

```
ZooKeeper是一个分布式的，开放源码的分布式应用程序协调服务，为分布式服务提供一致性服务。分布式服务cap原则包括三个方面，即一致性Consistency，可用性Availability，Partition tolerance（分区容错性）。分区容忍性是指，系统能否在时限内达成数据一致性，必须在C和A之间做出选择。
ZooKeeper目标是封装好易出错且复杂的关键服务，直接提供给用户。它提供了分布式独享锁、选举、队列的接口。
```

##### zk如何保证数据一致性

```
https://www.cnblogs.com/tkzL/p/12916116.html
保证数据一致性的核心协议是ZAB(ZooKeeper Atomic Broadcast)协议
写请求发送到follower，follower发送请求到leader，leader发起Proposal，广播给所有follower，半数follower同意后，再提交写请求，follower执行成功并返回给客户端。
```



##### 分布式独享锁

```
https://blog.csdn.net/qq_40378034/article/details/117014648
分布式独享锁的核心是如何保证当前有且仅有一个事务获取锁，并且锁被释放后，所有正在等待获取锁的事务都能够被通知到。
通过在ZooKeeper上创建一个子节点来表示一个锁，在需要获取排他锁时，所有的客户端都会试图通过调用create()接口，在/exclusive_lock节点下创建临时子节点/exclusive_lock/lock。
同时，所有没有获取到锁的客户端就需要到/exclusive_lock节点上注册一个子节点变更的watcher监听，以便实时监听到lock节点的变更情况
```

##### 选举

```
使用临时顺序znode来表示选举请求，创建最小后缀数字znode的选举请求成功。在协同设计上和分布式锁是一样的，不同之处在于具体实现。不同于分布式锁，选举的具体实现对选举的各个阶段做了细致的监控
```

##### 队列

```
使用路径为/queue的znode下的节点表示队列中的元素。/queue下的节点都是顺序持久化znode。
offer方法在/queue下面创建一个顺序znode。
remove方法移除一个元素
```

