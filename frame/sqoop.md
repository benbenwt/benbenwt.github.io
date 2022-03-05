### 安装

```
http://sqoop.apache.org
1.4.6版本下载:http://mirrors.hust.edu.cn/apache/sqoop/1.4.6/
#修改配置文件
mv sqoop-env-template.sh sqoop-env.sh
vim sqoop-env.sh 
export HADOOP_COMMON_HOME=/opt/module/hadoop-3.1.3
export HADOOP_MAPRED_HOME=/opt/module/hadoop-3.1.3
export HIVE_HOME=/opt/module/hive
export ZOOKEEPER_HOME=/opt/module/zookeeper-3.5.7
export ZOOCFGDIR=/opt/module/zookeeper-3.5.7/conf
#拷贝mysql驱动
cp mysql-connector-java-5.1.48.jar /opt/module/sqoop/lib/
#验证能否成功运行
 bin/sqoop help
#测试Sqoop是否能够成功连接数据库
 bin/sqoop list-databases --connect jdbc:mysql://hadoop102:3306/ --username root --password 000000
```

### 使用

```
bin/sqoop import \
--connect jdbc:mysql://hadoop102:3306/gmall \
--username root \
--password 000000 \
--table user_info \
--columns id,login_name \
--where "id>=10 and id<=30" \
--target-dir /test \
--delete-target-dir \
--fields-terminated-by '\t' \
--num-mappers 2 \
--split-by id

```

```
import_data(){
$sqoop import \
--connect jdbc:mysql://hbase:3306/$APP \
--username root \
--password root \
--target-dir /origin_data/$APP/db/$1/$do_date \
--delete-target-dir \
--query "$2 and  \$CONDITIONS" \
--num-mappers 1 \
--fields-terminated-by '\t' \
--compress \
--compression-codec lzop \
--null-string '\\N' \
--null-non-string '\\N'
```

### 理论知识

>读取MySQL的表数据，使用hive sql(mapreduce)放入hive的hdfs中。
>
>flume:读取file放入kafka，读取kafka放入hdfs
>
>json：json处理完成后在内存中，可放入的有file文件系统，hdfs，kafka，elasticsearch。
