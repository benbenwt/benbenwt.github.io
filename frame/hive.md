

# HIVE的语法

>hive(hdfs)+hive sql(mr)
>
>hive本质是hdfs文件，对于文件变动和查询性能很差，增删改。它唯一的作用就是存储，它的存储作用是由hdfs分布式文件体现的，与它无关。另一个作用就是将sql翻译为执行引擎的语言，所以说它不能称为存储组件，而是辅助分析插件，因为其没有添加存储架构，没有修改文件的增删改查性能，其性能与hdfs文件完全相同。

### DDL数据定义语言

```
show databases；
```



##### 建表语句

```
create table test(id int,name nchar(4));
```

##### 删表语句

```
drop table if exists test;
```

##### 清空表格数据，保留表格结构

```
truncate table sample;
```

##### 修改表结构

```
ALTER TABLE test_table ADD COLUMNS (new_col INT);
ALTER TABLE invites ADD COLUMNS (new_col2 INT COMMENT 'a comment');
ALTER TABLE test REPLACE COLUMNS(id BIGINT, name STRING);
#改表名称
ALTER TABLE events RENAME TO new_events;
#为表添加主键

```



### DML数据操作语言

>https://www.cnblogs.com/starzy/p/11441131.html

##### 设置主键

```
https://www.cnblogs.com/sabertobih/p/14031047.html#gallery-1
```

##### 查询

```
select * from   sample where architecture="x86";
```

##### 聚合统计

```
#加入count后，比只使用*慢了很多，用了22s。使用*查询所有列，0.3s查出了1045行数据。
select count(*)  from   sample where architecture="x86";
#改成architecture后仍然很慢，1045条统计了半天。
select count(architecture)  from   sample where architecture="x86";
```



##### 插入数据

```
#插入很慢，25s左右，和聚合统计一样慢。
insert into sample(architecture) values("test_arch");
```

##### 删除数据

```
#同插入与修改，hive本质是hdfs文件，对于文件变动和查询性能很差，增删改。它唯一的作用就是存储，它的存储作用是由hdfs分布式文件体现的，与它无关。另一个作用就是将sql翻译为执行引擎的语言，所以说它不能称为存储组件，而是辅助分析插件，因为其没有添加存储架构，没有修改文件的增删改查性能，其性能与hdfs文件完全相同。
delete
```



##### 修改数据

```
#速度极慢，因为hive本质是hdfs文件，当执行修改操作时，先读取原文件，在程序中修改后再写一份新文件，如果文件很大，则需要大量IO操作。
update
```



### DCL数据控制语言

>(GRANT，REVOKE，COMMIT，ROLLBACK)

### JAVA API

#hive java api:https://cwiki.apache.org/confluence/display/Hive/HiveServer2+Clients

### 

# 仓库规范

### 分区分桶外部表

```
```



### 分层

```
当仓库分为多层，有的达到5层，如果源头输入新的数据，从第一层到最后一层需要多久？其耗费的时间说明，这种结构只适用于离线。那么即席查询的意思就是开发离线模型的demo，用于演示，而不是“实时查询”，因为它的“即席”是建立在数据已经从源头流入最后一层的基础上的，也就是已经处理完成了，这并不实时。
```

### json格式

```
json格式要实现主键查询，非主键查询，所有键的聚合。实际上查询的功能应该由hbase来做。
```

# hive自动映射到hbase

```
https://blog.csdn.net/weixin_44694973/article/details/98845551
hadoop.hive.hbase.HBbaseStorageHandler
```

```
CREATE TABLE cctable (key int, value string) STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler' WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key,cf:val") TBLPROPERTIES ("hbase.table.name" = "cc");
```



# HIVE服务搭建

### 安装mysql

>安装mysql，重置密码。
>
>mysql版本和mysql-connector的java包版本，进入mysql官网下载：
>
>mysql-connector-java-5.1.37
>
>mysql-5.7.28.el7.x86_64rpm-bundle.tar

安装新版mysql前，需将系统自带的mariadb-lib卸载

```
rpm -qa|grep mariadb           #查看是否安装了mariadb
rpm -e --nodeps  software_name #卸载对应软件,e:erase
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
service mysqld start 
mysql
FLUSH PRIVILEGES;
update mysql.user set authentication_string=password('root'),host='%'   where user='root';
service mysql restart
mysql -uroot -proot
ALTER USER USER() IDENTIFIED BY 'root';#重置密码
```

### 相关操作

>配置失败时，如果需要重新从0开始安装，按照此步骤卸载清除mysql，再重新执行搭建步骤。

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

hive删了重来

```
rm -rf   /root/module/apache-hive-3.1.2-bin
cp -r /root/software/apache-hive-3.1.2-bin /root/module
```



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

### beeline2连接

> https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-RunningHiveServer2andBeeline.1
>
> user:root,password:root

jdbc端口:10000

web端口:10002

开启代理权限,hadoop的core-site.xml:

```
#xxx为服务器的登录的用户名
<property>
        <name>hadoop.proxyuser.xxx.hosts</name>
        <value>*</value>
    </property>
    <property>
        <name>hadoop.proxyuser.xxx.groups</name>
        <value>*</value>
    </property>
```

添加高可用，不然报一个tez相关的警告，影响启动.

```
vim hive-site.xml
<property>
    <name>hive.server2.active.passive.ha.enable</name>
    <value>true</value>
</property>
</configuration>

#开启hiveserver2服务,启动后jps可以看到hive的进程
nohup hiveserver2 1>/dev/null 2>/dev/null &
#beeline连接
beeline -u jdbc:hive2://localhost:10000/default -n root -w root
#或这样连接
beeline
!connect jdbc:hive2://hbase:10000/platform root root
#或这样测试beeline连接
$HIVE_HOME/bin/beeline -u jdbc:hive2://
#操作hive
show databases;
use test;
show tables;
```





### problem

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

##### Retrying connect to server: 0.0.0.0/0.0.0.0:8032. Already tried 0 time(s); retry policy is...

在yarn-master执行mr无错误，其他机器有错误

修改yanr-site.xml:

```

<property>
    <name>yarn.resourcemanager.address</name>
    <value>master:8032</value>
  </property>
  <property>
    <name>yarn.resourcemanager.scheduler.address</name>
    <value>master:8030</value>
  </property>
  <property>
    <name>yarn.resourcemanager.resource-tracker.address</name>
    <value>master:8031</value>
  </property>
```

##### Error: Could not find or load main class org.apache.hadoop.mapreduce.v2.app.MRAppMaster

##### Please check whether your etc/hadoop/mapred-site.xml contains the below configuration:

mapred-site.xml

```
<property>
  <name>yarn.app.mapreduce.am.env</name>
  <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
</property>
<property>
  <name>mapreduce.map.env</name>
  <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
</property>
<property>
  <name>mapreduce.reduce.env</name>
  <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
</property>
```

