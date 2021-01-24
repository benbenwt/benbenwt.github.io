##### 安装

下载hadoop到机器，解压缩后在/etc/profile配置java环境变量，hadoop需要使用java。随后，在profile中配置hadoop环境变量其多了一个参数，为export PATH=$PATH:$HADOOP_HOME/sbin。source /etc/profile载入配置检查正确性，终端运行hadoop检查是否成功。

##### 单机运行

https://hadoop.apache.org/docs/r3.1.4/hadoop-project-dist/hadoop-common/SingleCluster.html

配置好java及hadoop环境变量后，来到bin的同级目录。执行如下命令测试是否搭建完成。功能为使用examples的jar包编写的程序，处理input中数据，输出到output中。input中数据为在etc中拷贝的xml文件。

```
 bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.4.jar grep input output 'dfs[a-z.]+'
```

##### 伪分布式

##### 创建无密码ssh

```
 $ ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
  $ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
  $ chmod 0600 ~/.ssh/authorized_keys
```



##### 基本部门

1，平台。搭建平台，集群调优

2，数据仓库。sqlboy

3，数据挖掘。算法

4，开发。java开发

##### 大数据之源

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