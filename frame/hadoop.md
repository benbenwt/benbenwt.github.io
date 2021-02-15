# hadoop服务器搭建

>软件及系统版本：hadoop3.1.4,centos7.2，idea2019.3.3，maven为idea的bundle版本。
>
>参考官方文档3.1.4
>
>参考的视频https://www.bilibili.com/video/BV1cW411r7c5
>
>过程中碰到的问题总结在最后

### 初步安装

1进入http://hadoop.apache.org/下载hadoop3.1.4.tar.gz并解压缩。

2服务器终端输入vim  /etc/ profile 。在profile中添加如下内容,添加java和hadoop的环境变量。注意替换自己的JAVA_HOME和HADOOP_HOME目录：

```
#JAVA_HOME
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.181-7.b13.el7.x86_64
export PATH=$PATH:$JAVA_HOME/bin

#HADOOP_HOME
export HADOOP_HOME=/root/module/hadoop-3.1.4
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin

```

终端运行source /etc/profile，载入修改的配置。

3给hadoop配置java目录

```
vim  etc/hadoop/hadoop-env.sh
添加如下：
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.181-7.b13.el7.x86_64
```

source   etc/hadoop/hadoop-env.sh

4终端运行hadoop，显示为帮助信息，即配置成功。

帮助信息以如下文字结尾：

```
SUBCOMMAND may print help when invoked w/o parameters or with -h.
```

### 单机模式运行

官方文档：https://hadoop.apache.org/docs/r3.1.4/hadoop-project-dist/hadoop-common/SingleCluster.html

进入hadoop安装的根目录，执行如下命令。在output目录下生成结果文件，即为成功。注意output由hadoop自动创建，不要手动。

```
mkdir input
cp etc/hadoop/*.xml input
bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.4.jar grep input output 'dfs[a-z.]+'
cat output/*
```

如上命令的功能：将etc/hadoop下的以xml为后缀的文件，复制到input文件夹下面。

然后用hadoop-mapreduce-examples-3.1.4.jar中的java程序统计input下的所有文件，再对结果进行'dfs[a-z.]+'正则匹配，将结果输出到output文件夹。

### 伪分布式

1vim    etc/hadoop/core-site.xml，修改为如下内容，指定默认文件系统路径。:

```
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
```

2vim etc/hadoop/hdfs-site.xml，修改为如下内容:

```
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```

3执行如下命令，使得ssh localhost无需密码。

```
cd 用户根目录
cd .ssh
ssh-keygen -t rsa
ssh-copy-id localhost
输入密码即可
```

执行ssh localhost无需密码，即为成功。

4进入hadoop根目录，执行如下，格式化namenode，启动hdfs：

```
 $ bin/hdfs namenode -format
 $ sbin/start-dfs.sh
```

服务器本机访问 `http://localhost:9870/`即可查看hdfs系统

关闭防火墙后，外部也可访问。

5执行mapreduce作业

创建hdfs文件目录：

```
bin/hdfs dfs -mkdir /user
bin/hdfs dfs -mkdir /user/<username>
```

创建input文件夹，复制一些文件到input中

```
 bin/hdfs dfs -mkdir input
 bin/hdfs dfs -put etc/hadoop/*.xml input
```

执行mapreduce

```
bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.4.jar grep input output 'dfs[a-z.]+'
```

获取hdfs中结果文件存储到本地

```
bin/hdfs dfs -get output output
cat output/*
```

注意，必须使用hdfs命令创建，删除，修改文件目录。hdfs的目录和本地是相互独立的。

6关闭hdfs

```
 sbin/stop-dfs.sh
```

### 完全分布式搭建

>编写好scp分发脚本,快速同步etc配置文件到其他集群机器.
>
>编写好删除日志，data，tmp文件的脚本。当需要初始化时需要删除这些文件，然后格式化namenode再启动
>
>编写脚本参见bili视频
>
>以三台机器为例，同bili视频

我的三台机器对应如下

| ip主机号         | 190               | 189                         | 188                        |
| ---------------- | ----------------- | --------------------------- | -------------------------- |
| 我的hostname     | hbase             | hbase1                      | hbase2                     |
| bili的hostname : | hadoop102         | hadoop103                   | hadoop104                  |
| hdfs分工：       | namenode,datanode | datanode                    | secondarynamenode,datanode |
| hadoop分工:      | nodemanager       | resourcemanager,nodemanager | nodemanager                |

##### 配置环境

>主要配置这些firewall,hosts,hostname,ssh.
>
>java_home,hadooop_home,hadoop-env.sh

进入190主机，即hbase。

用单机模式的方法配置好，java环境变量，hadoop环境变量，hadoop的java_home。

关闭firewall:

```
systemctl stop firewalld.service
```

手动分配服务器ip,可省略。

hosts修改ip和域名映射:

```
vim /etc/hosts
追加如下内容：
0.0.0.0 hbase  #此处是个坑，不要配置为localhost，按此配置。
111.111.111.189 hbase1
111.111.111.188 hbase2

```

hostname改为对应域名:

```
vim /etc/sysconfig/network
修改为ip对应的hostname，我的190对应为
hbase
```

ssh配置使用公私钥不用密码：

```
cd 用户根目录
cd .ssh
ssh-keygen -t rsa
ssh-copy-id hbase
输入密码即可
ssh-copy-id hbase1
输入密码即可
ssh-copy-id hbase2
输入密码即可
```

此时hbase访问hbase1和hbase2都不需要密码。这样，之后才能控制hbase1和hbase2，启动运行在他们之上的hdfs结点。

配置ssh公钥的例子：

希望机器a可以无密码访问机器b。进行如下操作，进入a机器用户根目录下的.ssh文件夹，执行ssh-keygen -t  rsa。随后执行ssh-copy-id   b的ip，然后输入账号密码即可。可以进入b机器的.ssh的authorized_hosts查看，发现已经添加成功。希望a访问某台机器，使用同上方法拷贝到对应机器即可。

##### 配置etc下的文件

>配置env和xml，workers等。

1hadoop-env.sh引入java_home:

```
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.181-7.b13.el7.x86_64
```

2core-site.xml:

```
<configuration>
        <property>
                <name>fs.defaultFS</name>
                <value>hdfs://hbase:9000</value>
        </property>
        <property>
                <name>hadoop.tmp.dir</name>
                <value>/root/hadoop/dfs/tmp</value>
        </property>
</configuration>
```

3hdfs-site.xml:

```
<configuration>
        <property>
                <name>dfs.replicatin</name>
                <value>3</value>
        </property>
        <property>
                <name>dfs.namenode.secondary.http-address</name>
                <value>hbase2:50090</value>
        </property>
        <property>
                <name>dfs.namenode.name.dir</name>
                <value>/root/hadoop/dfs/name</value>
        </property>
        <property>
                <name>dfs.datanode.data.dir</name>
                <value>/root/hadoop/dfs/data</value>
        </property>
</configuration>

```

4etc/workers,3.x由slaves更名为workers。

```
vim workers
改为：
hbase
hbase1
hbase2
```

##### 转发hadoop到其他机器

1转发hadoop-3.1.4到hbase1，hbase2（也可以直接克隆整个虚拟机，但注意要修改ip，hostname，hosts文件）

```
scp hadoop-3.1.4  root@hbase1:`pwd`
scp hadoop-3.1.4  root@hbase2:`pwd`
```

如果修改配置文件，需要再次同步：

```
cd    hadoop-3.1.4/etc/hadoop
scp core-site.xml root@hbase1:`pwd`
scp core-site.xml root@hbase2:`pwd`
```

2配置hbase1，hbase2

对于hbase1和hbase2我们只需修改/etc/hosts,/etc/sysconfg/network,其他部分保持和hbase一致。

以hbase1为例子，主要改一下hostname和ip与域名对应关系。

hbase1的hosts修改ip和域名映射:

```
vim /etc/hosts
追加如下内容：
111.111.111.190  hbase  #此 处是个坑，不要配置为localhost，按此配置。
0.0.0.0        hbase1
111.111.111.188  hbase2

```

hbase1的hostname改为对应域名:

```
vim /etc/sysconfig/network
修改为ip对应的hostname，我的189对应为
hbase1
```

hbase2原理同上

##### 格式化namenode并启动

注意：若之前启动过hdfs，格式化时必须停止所有模块，并删除data，logs，tmp文件夹。

进入namenode所在的服务器，即hbase，执行如下命令。

```
hdfs namenode -format
sbin/start-dfs.sh
```

会看到他按照配置文件，在三台不同机器启动了不同node。

访问hbase的9870端口，可查看hdfs存储情况。

##### 启动yarn

进入resourcemanager所在的服务器，执行sbin/start-yarn.sh

访问resourcemanager所在机器的ip的8088端口,查看mapreduce执行情况。

##### 测试

```
上传文件，可在9870查看
hdfs dfs -put filename
执行mr，可在8088查看
bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.4.jar grep input output 'dfs[a-z.]+'
```

# MapReduce开发环境搭建

1官网下载hadoop3.1.4.tar.gz并解压缩

下载hadoop.dll及winexe，下载链接：

https://github.com/ordinaryload/Hadoop-tools

将hadoop.dl,winexe复制到hadoop-3.1.4/bin

2编写程序，按照bili视频编写map,reduce,driver类。

也可按照此博客，使用源码示例，博客地址：

https://www.cnblogs.com/xingluo/p/9512961.html

3添加NativeIO，由于win10摒弃此函数，需自己创建。

参考此博客处理：https://blog.csdn.net/weixin_42229056/article/details/82686172

4直接运行driver的main函数，直接将输入输出目录写死在程序中，不使用控制台输入参数。

##### 源码

driver:

```
package com.weitao.mr.wordcount;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.IOException;

public class WordCountDriver {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        System.setProperty("hadoop.home.dir", "C:\\Users\\asus\\Desktop\\hadoop-3.1.4\\hadoop-3.1.4");
        //获取job对象
        Configuration conf=new Configuration();
        Job job=Job.getInstance(conf);
        //设置jar位置
        job.setJarByClass(WordCountDriver.class);
        //关联map和reduce
        job.setMapperClass(WordCountMapper.class);
        job.setReducerClass(WordCountReducer.class);

        //设置mapper阶段输出数据key和value类型
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(IntWritable.class);
        //设置最终数据输出的key和value类型
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        //设置输入路径和输出路径
        FileInputFormat.setInputPaths(job, new Path("C:\\Users\\asus\\Desktop\\words.txt"));
        FileOutputFormat.setOutputPath(job, new Path("C:\\Users\\asus\\Desktop\\output"));
        //提交job
        boolean result=job.waitForCompletion(true);
        System.exit(result?0:1);
    }
}

```

reducer:

```
package com.weitao.mr.wordcount;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

public class WordCountReducer extends Reducer<Text, IntWritable,Text,IntWritable> {
    IntWritable v=new IntWritable();
    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
        //合并相同key
        int sum=0;
        for(IntWritable value:values)
        {
            sum+=value.get();
        }

        v.set(sum);
        context.write(key,v);
    }
}
```

mapper

```
package com.weitao.mr.wordcount;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

public class WordCountMapper extends Mapper<LongWritable, Text,Text, IntWritable> {
    Text k=new Text();
    IntWritable v=new IntWritable(1);
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        //1转为string并切分
        String line=value.toString();
        String[] words=line.split(" ");
        //循环写出
        for(String word:words)
        {
            k.set(word);
            context.write(k,v);
        }
    }
}
```

# HIVE服务器搭建

### 安装mysql

>安装mysql，重置密码。
>
>mysql版本和mysql-connector的java包版本，进入mysql官网下载：
>
>mysql-connector-java-5.1.37
>
>mysql-5.7.28.el7.x86_64rmp-bundle.tar

安装新版mysql前，需将系统自带的mariadb-lib卸载

```
rpm -qa|grep mariadb           #查看是否安装了mariadb
rpm -e --nodeps  software_name #卸载对应软件
```

安装mysql

```
mkdir mysql
chmod 777 ./mysql
#将bundle文件拷贝到mysql目录下
rpm -ivh file-name#安装mysql模块
安装顺序：common,libs,libs-compact,client,server
```

初始化

```
mysqld --initialize --user=mysql
#重置密码
vim /etc/my.cnf
在[mysqld]后面任意一行添加“skip-grant-tables”用来跳过密码验证的过程
mysql
FLUSH PRIVILEGES;
update mysql.user set authentication_string=password('root')   where user='root' and host='localhost';
mysql -uroot -proot
ALTER USER USER() IDENTIFIED BY 'root';
 update mysql.user set host='%' where user='root';
```

### 相关操作

删除mysql设置及文件重新初始化

>主要删除/var/lib/mysql,/etc/my.cnf,/var/log/mysqld.log

```
cd /var/lib/mysql
rm -rf ./*
rm /etc/my.cnf
rm -rf /var/log/mysqld.log
mysqld --initialize --user=mysql
 #查看初始密码，或使用重置密码的方法。
cat /var/log/mysqld.log
chmod 777 /var/lib/mysql
systemctl start mysqld
mysql -uroot -ppassword
```

 jgtW6NmSjO:(

版本

 21e.9LxZ4a4

mysql-connector-java-5.1.37

mysql-5.7.28.el7.x86_64rmp-bundle.tar

### 安装hive

1下载apache-hive-3.1.2-bin.tar.gz并解压

配置环境变量

删除冲突jar包，slf4j及guava

```
rm -rf $HIVE_HOME/lib/log4j-slf4j-impl-2.10.0.jar
rm -rf $HIVE_HOME/lib/guava-19.0.jar
cp -r $HADOOP_HOME/share/hadoop/common/lib/guava-27.0-jre.jar  $HIVE_HOME/lib
```

2启动hadoop，hive要使用hdfs。

创建metasotre

```
mysql -uroot -proot
CREATE DATABASE metastore;
```

将mysql-connector-java-5.1.37拷贝到$HIVE_HOME/lib，并解压将jar包放在lib下的一级目录。

修改配置文件，hive-site.xml

```

<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
<property>
    <name>javax.jdo.option.ConnectionURL</name>
    <value>jdbc:mysql://localhost/metastore?useSSL=false</value>
</property>
<property>
    <name>javax.jdo.option.ConnectionDriverName</name>
    <value>com.mysql.jdbc.Driver</value>
</property>
<property>
    <name>javax.jdo.option.ConnectionUserName</name>
    <value>root</value>
</property>
<property>
    <name>javax.jdo.option.ConnectionPassword</name>
    <value>root</value>
</property>
<property>
<name>hive.metastore.schema.verification</name>
<value>false</value>
</property>
<property>
<name>hive.metastore.event.db.notification.api.auth</name>
<value>false</value>
</property>
<property>
<name>hive.metastore.warehouse.dir</name>
<value>/user/hive/warehouse</value>
</property>
</configuration>

```

hive-env.sh配置hadoop_home和conf_dir

```
# Set HADOOP_HOME to point to a specific hadoop install directory
HADOOP_HOME=/root/module/hadoop-3.1.4

# Hive Configuration Directory can be controlled by:
export HIVE_CONF_DIR=/root/module/apache-hive-3.1.2-bin/conf
```

3删除/root, $HIVE_HOME 及$HIVE_HOME/bin下的derby.log metastore_db

初始化hive数据库

```
$HIVE_HOME/bin/schematool -dbType mysql -initSchema
$HIVE_HOME/bin/hive
show databases;
#测试
CRATE DATABASE test_db;
use test_db;
CREATE TABLE test(id string);
```

### 相关操作

删了重来

```
rm -rf   /root/module/apache-hive-3.1.2-bin
cp -r /root/software/apache-hive-3.1.2-bin /root/module
```

# 其他

### 大数据之源

Google的GFS、MapReduce、BigTable

### 常用配置文件

##### 文件夹

etc/hadoop:环境配置sh文件，各组件配置xml文件。

sbin：启动，停止各组件控制sh文件。

bin：执行mapreduce任务的sh文件。

share：基础配置模板等。

##### core-site.xml

配置hdfs的ip和端口

hdfs的存储文件夹

##### hdfs-site.xml

配置hdfs的详细信息，例如hdfs的replication，namenode位置和datanode的位置。

##### hadoop-env.sh,mapred-env.sh,yarn-env.sh

环境配置，至少配置JAVA_HOME绝对路径。

##### mapred-site.xml

指定管理框架，如yarn

##### yarn-site.xml

配置resourceManagers

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

### 基本分工

1，平台。搭建平台，集群调优

2，数据仓库。sqlboy

3，数据挖掘。算法

4，开发。java开发



# problem

### source core-site.xml第一行未预期的错误,newline。

不必理会

### sbin/strat-dfs.sh报错

解决：对于start-dfs.sh和stop-dfs.sh文件，添加下列参数：

```
#!/usr/bin/env bash
HDFS_DATANODE_USER=root
HADOOP_SECURE_DN_USER=hdfs
HDFS_NAMENODE_USER=root
HDFS_SECONDARYNAMENODE_USER=root
```

对于start-yarn.sh和stop-yarn.sh文件，添加下列参数：

```
#!/usr/bin/env bash
YARN_RESOURCEMANAGER_USER=root
HADOOP_SECURE_DN_USER=yarn
YARN_NODEMANAGER_USER=root
```

##### 9000端口未启动

端口也没有被占用就是没有启动服务。

解决:在hosts中添加如下设置:0.0.0.0 hbase

相应的core-site.xml应设置如下:fs.defaultFS的value为hdfs://hbase:9000。随后，使用sbin/stop-all.sh停止服务，使用sbin/start-all.sh开启即可。

相关，查看hadoop官方configuration，其默认的设置ip的确为0。但使用其他教程修改了

##### outputfile already exists

在本地用rm -rf这种方式删除output文件夹是错误的，在hdfs系统中仍能查看到output文件存在。应使用hadoop指令删除相关文件。如下 bin/hdfs -rm -r -f  /user/root/output.

##### 重启dfs

停止进程，删除dfs的data。删除hadoop的logs和tmp。再重启

##### 无法访问9870

1关闭selinux和防火墙

2使用http访问，不支持https

##### no such file

 hadoop fs -mkdir -p /user/root/

##### Exception in thread "main"java.lang.UnsatisfiedLinkError:org.apache.hadoop.io.nativeio.NativeIO$Windows.access0(Ljava/lang/String;I)Z

解决：参考此博客处理：https://blog.csdn.net/weixin_42229056/article/details/82686172

也可能：缺少hadoop.dll和winexe组件，下载winexe和hadoop.dll放到hadoop3.1.4的bin目录下。下载链接：https://github.com/ordinaryload/Hadoop-tools

##### 执行bin/hive提示no hbase in......

官网下载hbase2.3.3并解压

配置环境变量

```
#HBASE_HOME
export HBASE_HOME=/root/module/hbase-2.3.3
export PATH=$PATH:$HBASE_HOME/bin

```

##### Exception in thread "main" java.lang.NoSuchMethodError: com.google.common.base.Preconditions.checkArgument(ZLjava/lang/String;Ljava/lang/Object;)V

hadoop/share/hadoop/common/lib与hive/lib下的guava冲突，删除低版本的，将高版本的复制到另一个目录。

##### javax.jdo.JDODataStoreException: Error executing SQL query "select "DB_ID" from "DBS"".

hive初始化表失败

```
$HIVE_HOME/bin/schematool -dbType <db type> -initSchema
```

##### ConnectionRefuesd

>java.lang.RuntimeException: Error applying authorization policy on hive configuration: java.net.ConnectException: Call From hbase/0.0.0.0 to hbase:9000 failed on connection exception: java.net.ConnectException: Connection refused; For more details see:  http://wiki.apache.org/hadoop/ConnectionRefused

官方

https://cwiki.apache.org/confluence/display/HADOOP2/ConnectionRefused

将hosts中的0.0.0.0改为真实ip，不要写127.0.0.1。写服务器之前的通信ip。

##### java.sql.SQLException: Failed to start database 'metastore_db' with class loader sun.misc.Launcher$AppClassLoader@3930015a, see the next exception for details.

 rm  -rf  metastore_db

##### mysql Access denied

密码复制的也说错，干脆设置为不需要密码。

```
vim /etc/my.cnf
在[mysqld]后面任意一行添加“skip-grant-tables”用来跳过密码验证的过程
mysql

```

##### MySQL5.7 启动报错:initialize specified but the data directory has files in it. Aborting.

删除/var/lib/mysql

##### timestamp错误

修改/etc/my.cnf中explicit_defaults_for_timestamp=true

##### You must reset your password using ALTER USER statement before executing this statement报错处理

ALTER USER USER() IDENTIFIED BY 'root';