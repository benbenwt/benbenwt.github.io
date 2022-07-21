[TOC]
# maxwell
## 安装使用
>https://maxwells-daemon.io/quickstart/
### mysql配置和授权
>步骤可以跟随上边链接的quickstart。
>首先开启msyql的binlog
>然后创建一个maxwell数据库，并创建一个maxwell账户并授予权限，设置密码。
```
CREATE DATABASE maxwell;
GRANT ALL ON maxwell.* TO 'maxwell'@'%' IDENTIFIED BY '123456';
GRANT SELECT ,REPLICATION SLAVE , REPLICATION CLIENT ON *.* TO maxwell@'%';
```

### 修改maxwell配置文件
```
cp config.properties.example config.properties
vim config.properties

#mysql 配置
host= m1
user= maxwell
password= 123456
client_id=maxwell_1

producer=kafka
kafka.bootstrap.servers= m1:9092,m2:9092,m3:9092
kafka_topic=gmall2020_db_m

bin/maxwell --config config.properties > /dev/null 2>&1 &
```
## 工作原理
> maxwell 也是读取binlog发送到kafka等
### 与cannal差异
>canal只能抓取新数据，对于已经存在的数据没有办法。而maxwell提供了bootstrap功能，可以直接引导出整个完整的历史数据用于初始化，很好用。maxwell更轻量级。
## 日志格式
>当binlog_format= row时，canal会将同一次sql影响的row放到同一个json中，而maxwell则是将row分开到多个json中。cannal会带入表的结构，maxwell更简洁。
```
INSERT INTO z_user_info VALUES(30,'zhang3','13810001010'),(31,'li4','1389999999');

{"database":"gmall-2020-0
4","table":"z_user_info","ty
pe":"insert","ts":158938531
4,"xid":82982,"xoffset":0,"d
ata":{"id":30,"user_name":"
zhang3","tel":"1381000101
0"}}

{"database":"gmall-2020-0
4","table":"z_user_info","ty
pe":"insert","ts":158938531
4,"xid":82982,"commit":tru
e,"data":{"id":31,"user_nam
e":"li4","tel":"1389999999"}
}
```
# cannal
## 安装使用
### 开启mysql binlog
```
vim /etc/my.cnf
可以通过locate my.cnf查找配置文件位置
在[mysqld]添加 log-bin=mysql-bin这表示生成的binlog文件前缀是mysql-bin.XXXXX类型。
添加binlog_format=row指定binlog类别
添加binlog-do-db=gmall指定需要binlog的数据库
#重启生效
sudo systemctl restart mysqld
#该目录查看binlog文件
cd /var/lib/mysql 
```

### mysql赋予canal权限
```
set global validate_password_length=4;
set global validate_password_policy=0;

#其中*.*为 数据库.表名称,'canal'@'%' 为 用户名@地址，后边的IDENTIFIED BY 'canal'是指密码为 canal。授予在什么库什么表的权限给谁（名称，地址），密码是什么。
GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 
'canal'@'%' IDENTIFIED BY 'canal' 
```

### canal安装
```
#目前使用1.1.4版本
https://github.com/alibaba/canal/releases
解压到目录
#canal分为单机版和高可用版本，但是高可用配置复杂，且只是提供了高可用，同一时间只有一个canal节点工作，所以一般直接用单机模式。

#修改配置
vim canal.properties
#配置kafka地址
canal.serverMode=kafka
canal.msq.servers= m1:9092,m2:9092,m3:9092
canan.msq.retries=0
canal.destinations= instance1，instance2 ....

#canal可以配置多个instance，每个instance写一个instance.perperties然后在canal.propeties中的canal.destinations启用即可。
vim instance.perperties
canal.instance.master.address=m1:3306
#前边的步骤在mysql中创建了此账户，并给予了权限。
canal.instance.dbUsername=canal
canal.instance.dbPassword=cananl

canal.mq.topic=gmall2020_db_c
#如果要开启多个kafka分区，要在kafka中创建对应分区数的主题。多个分区只能保证分区内的有序性。
#cananl.mq.partition=0
canal.mq.partitionNum=4

#启动canal
bin/startup.sh
#停止
bin/stop.sh
```
## 工作原理
### MySQL主从复制
>Master主库改变记录，写入到二进制日志binary log中。
>Slave从库向mysql master发送dump 协议，将master主库的binary log events拷贝到它的中继日志relay log
>Slave从库读取并重做中继日志的事件，把数据同步到自己的数据库
#### binlog
>MySQL的二进制日志记录了所有的DDL和DML语句，以事件形式记录，是事务安全型的。其主要使用场景1是主从复制，保证数据一致性2数据恢复，借助mysqlbinlog来恢复数据。binlog共有两类文件，1是日志索引(.index)，记录有哪些二进制文件2二进制日志文件，记录DDL和DML的语句事件
>binlog 分为三种，STATEMENT,MIXED,ROW。可以通过binlog_format= statement|mixed|row设置。
>STATEMENT:语句级，binlog 会记录每次一执行写操作的语句。相对 row 模式节省空间，但是可能产生不一致性.
>ROW: 行级， binlog 会记录每次操作后每行记录的变化。占用空间，保持了数据绝对一致性，记录执行语句的行结果。
>MIXED:混合，正常用STATEMENT，特殊情况用ROW，例如UUID(),AUTO_INCREMENT字段。

### canal 工作原理
>canal伪装成Slave，从master读取binlog，复制数据。
## 日志格式
```
INSERT INTO z_user_info VALUES(16,'zhang3','13810001010'),(17,'zhang3','13810001010');
#其日志分为data，database，table，字段信息，等其他信息。
{"data":[{"id":"16","user_name":"zhang3","tel":"13810001010"},{"id":"17","user_nam
e":"zhang3","tel":"13810001010"}],"database":"gmall-2020-04","es":158919650200
0,"id":4,"isDdl":false,"mysqlType":{"id":"bigint(20)","user_name":"varchar(20)","tel":"
varchar(20)"},"old":null,"pkNames":["id"],"sql":"","sqlType":{"id":-5,"user_name":12,"t
el":12},"table":"z_user_info","ts":1589196502433,"type":"INSERT"}

```
