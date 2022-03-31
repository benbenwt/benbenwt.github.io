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
$sqoop import \
--connect jdbc:mysql://hadoop102:3306/$APP \
--username root \
--password 000000 \
--target-dir /origin_data/$APP/db/$1/$do_date \
--delete-target-dir \
--query "$2 where \$CONDITIONS" \
--num-mappers 1 \
--fields-terminated-by '\t' \
--compress \
--compression-codec lzop \
--null-string '\\N' \
--null-non-string '\\N'
```

# 理论知识

>读取MySQL的表数据，使用hive sql(mapreduce)放入hive的hdfs中。

# sqoop2用法

# sqoop1用法

## Import

>将mysql的数据迁移到hdfs

| Argument                     | Description                       |
| ---------------------------- | --------------------------------- |
| --connect  jdbc-uri          |                                   |
| --username  username         |                                   |
| --password  password         |                                   |
| --query  "sql"               |                                   |
| --target-dir  hdfs_dir       | hdfs_dir                          |
| --fields-terminated-by  '\t' | 列之间使用制表符分隔              |
| --null-string  '\\\\N'       | 字符串内容为null的，存储为‘\\\\N‘ |
| --null-non-string '\\\\N'    | 数据库空值null的，存储为‘\\\\N‘   |
| --num-mappers   1            |                                   |
| --as-textfile                | 与load表存储格式对应              |
| --as-parquetfile             | 与load表存储格式对应              |

```
#必须参数示例
$sqoop import \
--connect jdbc:mysql://hadoop102:3306/$APP \
--username root \
--password 000000 \
--target-dir /origin_data/$APP/db/$1/$do_date \
--query "$2 where \$CONDITIONS" \
--fields-terminated-by '\t' \
```

## Export

| Argument                      | Description            |
| ----------------------------- | ---------------------- |
| --connect  jdbc-uri           | 数据库连接             |
| --username  username          | username               |
| --password  password          | password               |
| --table                       |                        |
| --num-mapper                  | mapper数目             |
| --export-dir                  | 导出的hive表路径       |
| --input-null-string '\\N'     |                        |
| --input-null-non-string '\\N' |                        |
| --input-fields-terminated-by  |                        |
| --update-mode allowinsert     | 允许插入               |
| --update-key  $2              | 使用哪一列作为主键插入 |

