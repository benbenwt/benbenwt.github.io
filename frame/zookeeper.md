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
zookeeper-client -server localhost:2181
#查看zookeeper结点
ls /
get /zk_test
set /zk_test name
delete /zk_test
deleteall 
rmr 
```



# 理论知识

>ZooKeeper是一个分布式的分布式应用程序协调服务，为分布式服务提供一致性服务。
>目的：ZooKeeper目标是封装好易出错且复杂的关键服务，直接提供给用户。它提供了分布式独享锁、选举、队列的接口。
>
>分布式服务cap原则包括三个方面，即一致性Consistency，可用性Availability，Partition tolerance（分区容错性）。分区容忍性是指，系统能否在时限内达成数据一致性，必须在C和A之间做出选择。

>zookeeper是由一个leader和多个follower组成的集群
>
>集群只要有半数以上结点存活，zookeeper就能正常服务，所以zk适合安装奇数台服务器。
>
>全局数据一致，每个server保存一份相同的数据副本，client连接哪个server，数据都是一致的。
>
>更新请求顺序执行，来自同一个Client的更新请求其发送顺序一次执行。
>
>数据更新原子性，一次数据更新要么成功，要么失败。
>
>实时性，在一定时间范围内，client能读到最新数据。

## 基本架构

### 数据结构

>zookeeper的数据模型与Unix文件系统类似，可以看作一棵树，每个结点称作ZNode。每个ZNode默认能存储1MB的数据，ZNode通过其路径唯一标识。

## 应用场景

>统一命名服务、统一·配置服务、统一集群管理、服务器结点动态上下线、软负载均衡

### 统一配置服务

>1要求整个集群的配置信息是一致的
>
>2对配置文件修改后，希望能快速同步到各个结点上
>
>可以将配置信息写入ZNode，各个客户端监听这个ZNode，一旦ZNode的数据被修改，Zookeeper将通知各个客户端服务器。

### 统一集群管理

>需要实时掌握集群各个结点的状态，并做出相应调整。
>
>zookeeper可以实时监控结点状态，将结点信息写入ZNode，监听ZNode并获取它的实时状态变化。

### 服务器结点动态上下线

>洞察服务器上下线的变化，服务端启动时去注册信息。

### 软负载均衡

>记录每个节点的访问数，将任务分配给访问数最少的服务器。

## 选举机制

### myid

>每台服务器的唯一标识，可通过zoo.cfg设置

### SID

>服务器IO，用来标识集群中的机器，每台机器不能重复，和myid一致。

### ZXID

>事务ID，标识一次服务状态的变更，每当客户端发起一次更新操作，增加1，单调递增。

### Epoch

>每当leader变更一次，增加1。

### Zookeeper选举机制-第一次启动

>起初每台机器都投自己，后来根据每台机器的myid判定，投给myid最大的。例如对于5个结点的集群，当第三台机器启动时，该机器获得3票，超过了集群节点的半数票，当选为leader，其他为follower。

### Zookeeper选举机制-非第一次启动

>当集群中有结点宕机或重启会触发选举。
>
>1如果集群中仍然存在leader，直接与leader建立连接。
>
>2如果不存在leader，按照EPOCH->ZXID->SID的顺序逐个比较，大的就胜出，成为leader。即EPOCH大的胜出，如果并列再比较ZXID，如果事务id也并列再比较SID.

## 结点类型

>持久:客户端与服务器端断开后，创建的结点不删除
>
>短暂:客户端和服务器断开连接后，创建的结点自己删除
>
>持久化目录节点、临时目录节点顾名思义
>
>持久化顺序编号目录节点、临时顺序编号目录节点在上面两者的基础上，只是该结点名称进行顺序编号。

## 监听器原理

>在客户端创建main线程，这时会创建两个线程，一个负责网络连接通信connect，一个负责监听listener。
>
>通过connect将注册的监听事件发送到zk，zk会把需要监听的事件进行记录，当有数据变化或路径变化时，发送消息到listener。
>
>常见的监听1get path  [watch]监听数据变化  2.ls path [path] 监听是否添加子结点.

## Paxos算法

>一种基于消息传递且具有高度容错特性的一致性算法
>
>目的：解决如何在分布式系统中对某个数值达成一致，保证无论发生任何异常，都不会破坏整个系统的一致性。
>
>在一个Paxos系统中，结点被划分为Proposer、Acceptor、Learner。每个节点可以身兼数职。

>主要分为三个阶段：Prepare阶段、Accept阶段、Learn阶段

### Prepare阶段

>Proposer向多个Acceptor发出Proposal请求，请求中携带全局唯一的Proposal ID。
>
>Acceptor针对收到的Propose请求进行Promise，Promise内容为：
>
>1不接受Proposal ID小于等于当前的Proposal请求
>
> 2不再接受Proposal ID小于当前请求的Accep请求
>
>不违背以前做出的承诺下，回复已经Accept中的提案中Proposal ID最大的哪个value的Proposal ID，没有则返回空值。

### Accept接受阶段

>Proposer接受到Acceptor承诺的Promise后，从应答中去除Proposal ID最大的提案，作为本次提案，向Acceptor发出Propose请求。如果应带内容value为空，可以随意决定提案，然后携带Proposal ID发送到Acceptor。
>
>Acceptor针对收到的Propose请求进行Accept处理。

### Learn阶段

>将Proposer形成的决议发送给所有Learners

## zk如何保证数据一致性

###### 集群结点信息

```
存储了集群结点信息,如hbase的regionserver结点存储在zk中的/hbase/meta-regionserver中，如果将其删除，则hbase-master就不知道自己有哪些regionserver在线。
```

```
https://www.cnblogs.com/tkzL/p/12916116.html
保证数据一致性的核心协议是ZAB(ZooKeeper Atomic Broadcast)协议
写请求发送到follower，follower发送请求到leader，leader发起Proposal，广播给所有follower，半数follower同意后，再提交写请求，follower执行成功并返回给客户端。
```

## 分布式独享锁

```
https://blog.csdn.net/qq_40378034/article/details/117014648
分布式独享锁的核心是如何保证当前有且仅有一个事务获取锁，并且锁被释放后，所有正在等待获取锁的事务都能够被通知到。
通过在ZooKeeper上创建一个子节点来表示一个锁，在需要获取排他锁时，所有的客户端都会试图通过调用create()接口，在/exclusive_lock节点下创建临时子节点/exclusive_lock/lock。
然后查看自己是不是当前节点下最小的点，如果不是，说明没有强到锁。所有没有获取到锁的客户端就需要到/exclusive_lock节点上注册一个子节点变更的watcher监听，以便实时监听到lock节点的变更情况。当业务完成后，delete结点释放锁，然后再判定最小的结点获得锁。
```

## 分布式队列

```
使用路径为/queue的znode下的节点表示队列中的元素。/queue下的节点都是顺序持久化znode。
offer方法在/queue下面创建一个顺序znode。
remove方法移除一个元素
```

## 相关链接

>尚硅谷官网：zookeeper学习笔记
>
>https://www.jianshu.com/p/3fec1f8bfc5f

# 用法
