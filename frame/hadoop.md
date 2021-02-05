### 配置步骤

java,hadoop 环境变量，env.sh.

防火墙，hosts，hostname，ip

ssh

sudo systemctl stop firewalld.service关闭防火墙

sudo systemctl disable firewalld.service取消自启

### 常用配置文件

##### 文件夹

etc/hadoop:环境配置sh文件，各部件配置xml文件。

sbin：启动，停止各部件控制sh文件。

bin：执行mapreduce任务sh文件。

share：基础配置等。

##### core-site.xml

配置hdfs的ip和端口

hdfs的存储文件夹

##### hdfs-site.xml

配置hdfs的详细信息，例如hdfs的replication，namenode位置和datanode的位置。

##### hadoop-env.sh,mapred-env.sh,yarn-env.sh

环境配置，至少配置JAVA_HOME绝对路径。

##### mapred-site.xml

##### yarn-site.xml

### 常用端口

9870  用户查看hdfs节点信息，hdfs的http端口

  8088  用户查看YARN的http端口，显示任务执行信息，查找错误等。

| 端口  | 用途                                                         |
| ----- | ------------------------------------------------------------ |
| 9000  | fs.defaultFS，如：hdfs://172.25.40.171:9000                  |
| 9001  | dfs.namenode.rpc-address，DataNode会连接这个端口             |
| 50070 | dfs.namenode.http-address                                    |
| 50470 | dfs.namenode.https-address                                   |
| 50100 | dfs.namenode.backup.address                                  |
| 50105 | dfs.namenode.backup.http-address                             |
| 50090 | dfs.namenode.secondary.http-address，如：172.25.39.166:50090 |
| 50091 | dfs.namenode.secondary.https-address，如：172.25.39.166:50091 |
| 50020 | dfs.datanode.ipc.address                                     |
| 50075 | dfs.datanode.http.address                                    |
| 50475 | dfs.datanode.https.address                                   |
| 50010 | dfs.datanode.address，DataNode的数据传输端口                 |
| 8480  | dfs.journalnode.rpc-address                                  |
| 8481  | dfs.journalnode.https-address                                |
| 8032  | yarn.resourcemanager.address                                 |
| 8090  | yarn.resourcemanager.webapp.https.address                    |
| 8030  | yarn.resourcemanager.scheduler.address                       |
| 8031  | yarn.resourcemanager.resource-tracker.address                |
| 8033  | yarn.resourcemanager.admin.address                           |
| 8042  | yarn.nodemanager.webapp.address                              |
| 8040  | yarn.nodemanager.localizer.address                           |
| 8188  | yarn.timeline-service.webapp.address                         |
| 10020 | mapreduce.jobhistory.address                                 |
| 19888 | mapreduce.jobhistory.webapp.address                          |
| 2888  | ZooKeeper，如果是Leader，用来监听Follower的连接              |
| 3888  | ZooKeeper，用于Leader选举                                    |
| 2181  | ZooKeeper，用来监听客户端的连接                              |
| 60010 | hbase.master.info.port，HMaster的http端口                    |
| 60000 | hbase.master.port，HMaster的RPC端口                          |
| 60030 | hbase.regionserver.info.port，HRegionServer的http端口        |
| 60020 | hbase.regionserver.port，HRegionServer的RPC端口              |
| 8080  | hbase.rest.port，HBase REST server的端口                     |
| 10000 | hive.server2.thrift.port                                     |
| 9083  | hive.metastore.uris                                          |

### 安装

下载hadoop到机器，解压缩后在/etc/profile配置java环境变量，hadoop需要使用java。随后，在profile中配置hadoop环境变量,其多了一个参数，为export PATH=$PATH:$HADOOP_HOME/sbin。source /etc/profile载入配置检查正确性，终端运行hadoop检查是否成功。

### 单机运行

https://hadoop.apache.org/docs/r3.1.4/hadoop-project-dist/hadoop-common/SingleCluster.html

配置好java及hadoop环境变量后，来到bin的同级目录。执行如下命令测试是否搭建完成。功能为使用examples的jar包编写的程序，处理input中数据，输出到output中。input中数据为在etc中拷贝的xml文件。

```
 bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.4.jar grep input output 'dfs[a-z.]+'
```

### 伪分布式

##### 创建无密码ssh

```
 $ ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
  $ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
  $ chmod 0600 ~/.ssh/authorized_keys
```

### 完全分布式搭建

>编写好分发脚本方便快速同步文件到其他机器
>
>编写好删除日志，data，tmp文件的脚本。当需要初始化时需要删除这些文件，然后格式化namenode再启动

##### 配置环境

java环境变量,hadoop 环境变量，env.sh.

防火墙，hosts，hostname，ip

ssh使用公私钥不用密码

##### 配置etc下的文件，env和xml，workers等。

env引入环境变量。xml配置好namenode，datanode，存储位置等。

配置workers，在其中加上集群机器的ip或域名，方便在一台机器上使用ssh调用启动其他机器。

##### 格式化namenode并启动

进入namenode所在的服务器，执行如下命令

hdfs namenode -format

sbin/start-dfs.sh

会看到他按照配置文件，在三台不同机器启动了不同文件。

访问namenode的ip的9870端口

##### 启动yarn

进入resourcemanager所在的服务器，执行sbin/start-yarn.sh

访问resourcemanager的ip的8088端口

##### 上传文件

hdfs dfs -put filename

1，平台。搭建平台，集群调优

2，数据仓库。sqlboy

3，数据挖掘。算法

4，开发。java开发

### 大数据之源

Google的GFS、MapReduce、BigTable

### problem

##### source core-site.xml第一行未预期的错误,newline。

##### sbin/strat-dfs.sh报错

error： but there is no HDFS_NAMENODE_USER defined. Aborting operation

解决：对于start-dfs.sh和stop-dfs.sh文件，添加下列参数：

```
#!/usr/bin/env bash
HDFS_DATANODE_USER=root
HADOOP_SECURE_DN_USER=hdfs
HDFS_NAMENODE_USER=root
HDFS_SECONDARYNAMENODE_USER=root
```


2、对于start-yarn.sh和stop-yarn.sh文件，添加下列参数：

```
#!/usr/bin/env bash
YARN_RESOURCEMANAGER_USER=root
HADOOP_SECURE_DN_USER=yarn
YARN_NODEMANAGER_USER=root
```

##### 9000端口未启动

端口也没有别占用就是没有启动服务。

解决:在hosts中添加如下设置:0.0.0.0 hbase

相应的core-site.xml应设置如下:fs.defaultFS的value为hdfs://hbase:9000。随后，使用sbin/stop-all.sh停止服务，使用sbin/start-all.sh开启即可。

相关，查看hadoop官方configuration，其默认的设置ip的确为0。但使用其他教程修改了

##### outputfile already exists

在本地用rm -rf这种方式删除output文件夹是错误的，在hdfs系统中仍能查看到output文件存在。应使用hadoop指令删除相关文件。如下 bin/hdfs -rm -r -f  /user/root/output.

##### 重启dfs

停止进程，删除dfs的data。删除hadoop的logs和tmp。

##### 无法访问9870

1关闭selinux和防火墙

2使用http访问，不支持https

##### no such file

 hadoop fs -mkdir -p /user/root/