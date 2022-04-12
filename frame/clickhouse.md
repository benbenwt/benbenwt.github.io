# 安装

```
#修改打开文件数和进程数
vim /etc/security/limits.conf
vim /etc/security/limits.d/20-nproc.conf
* soft nofile 65536 
* hard nofile 65536 
* soft nproc 131072 
* hard nproc 131072

#取消selinux
vim /etc/selinux/config

#安装依赖
yum install -y libtool
yum install -y *unixODBC*

#下载clickhouse的rpm安装包
rpm -ivh *.rpm
#修改配置文件
vim /etc/clickhouse-server/config.xml
取消注释<listen_host>::</listen_host>

#启动clickserver
systemctl start clickhouse-server
clickhouse-client -m
```



# 理论知识

>https://blog.csdn.net/breakout_alex/article/details/111305177

## Clickhouse特点

### 列式存储

>列式存储将同一列的数据存储在一起，方便进行聚合统计。
>
>1对于列的聚合、计数，求和等统计操作优于列式存储
>
>2对于某一列的数据类型是相同的，针对于数据存储更容易进行数据压缩，每一列选择更优的数据压缩方法，大大提高了数据的压缩比重。
>
>3对于数据压缩，一方面节省了磁盘空间，另一方面对于cache也有了更大的发挥空间。

### DBMS功能

#### 多样化引擎

>与mysql类似，把表级别的存储引擎插件化，根据表的不同需求设定不同的存储引擎。目标包括合并树、日志、接口和其他四大类20多种引擎。

#### 高吞吐写入能力

>clickhouse采用LSM Tree的结构，数据写入后定期在后台Compaction。通过类LSM tree的结构，ClickHouse在数据导入时全是顺序append写，写入后数据段不可更改，在后台compaction时也是多个段merge sort后顺序写会磁盘。顺序写的特性，充分利用了磁盘的吞吐能力。benchmark表示有50MN-200MB/s的写入吞吐能力。

#### 数据分区与线程级并行

>ClickHouse将数据划分为多个partition，每个partition进一步划分为多个index granularity，然后通过多个cpu核心分别处理一部分数据实现数据并行。这种情况，单条query就能利用整机的cpu，降低查询延迟。
>
>但是由于对单条查询使用了多个cpu，不利于同时并发多条查询，所以对于高qps的查询业务，clickhouse并不是强项。

## 数据类型

### 整型

>固定长度整型，包括有符号整型和无符号整型。
>
>Int8,Int16,Int32,Int64
>
>UInt8,UInt16,UInt32,UInt64

### 浮点型

>Float32
>
>Float64

### 布尔型

>没有单独的boolean，可以使用UInt8，限制为0,1。

### Decimal型

>有符号的浮点数，可在加、减和乘法运算过程中保持精度。对于除法，最低有效数字会被丢弃。
>
>有三种声明：
>
>Decimal32
>
>Decimal64
>
>Decimal128

### 字符串

#### String

>可以是任意长度

#### FixedString

>固定长度为N的字符串，N必须是正自然数。

### 枚举类型

>Enum保存‘string’=integer的对应关系
>
>Enum8
>
>Enum16
>
>建表时指定Enum的string和integer关系后，只能存储指定的string。

### 时间类型

>Date
>
>Datetime
>
>Datetime64:携带亚秒

### 数组

>由T类型元素组成的数组
>
>T可以是任意类型，包括数组类型。
>
>1使用array创建数组
>
>2使用[]创建数组

## 表引擎

### TinyLog

>没有索引、并发控制，主要用于测试环境

### Memory

>内存引擎，数据以未压缩形式存储在内存中，服务器重启数据消失。
>
>读写操作不会相互阻塞，不支持索引。

### MergeTree

>支持索引和分区，地位想当与innodb之于Mysql。

#### partition by

>分区的主要目的是降低扫描范围，优化查询速度。
>
>不填写此参数，只会使用一个分区
>
>MergerTree是以列文件+索引文件+表定义文件组成的，如果设定了分区，这些文件就会保存到不同分区。
>
>分区之间会并行处理
>
>写入的数据会放入临时分区，10-15分钟后，把临时分区的数据，合并到已有分区中。可以通过optimize手动执行

```
optimize table xxx final;
```

#### primary key

>clickhouse中的主键只提供了数据的一级索引，但是不是唯一约束，这意味着可以存在primary key一样的数据。根据where条件设置主键，定位正确的index granularity，避免全表扫描。
>
>index granularity：索引粒度，是指在稀疏索引的两个相邻索引对应数据的间隔。稀疏索引的好处是可以用很少的索引，定位更多的数据，代价是只能定位到索引粒度的第一行，然后再进行一点扫描。这样写入的工作变少了，读取的工作变多了。

#### order by

>设置了分区内数据按照那些字段排序
>
>是必填项

#### 数据TTL

>time to live，MergeTree提供了可以管理数据或者列的生命周期的功能。

##### 列级别TTL

```
create table t_order_mt3(
 id UInt32,
 sku_id String,
 total_amount Decimal(16,2) TTL create_time+interval 10 SECOND,
 create_time Datetime 
) engine =MergeTree
partition by toYYYYMMDD(create_time)
 primary key (id)
 order by (id, sku_id);

insert into t_order_mt3 values
(106,'sku_001',1000.00,'2020-06-12 22:52:30'),
(107,'sku_002',2000.00,'2020-06-12 22:52:30'),
(110,'sku_003',600.00,'2020-06-13 12:00:00');

```

##### 表级别TTL

```
alter table t_order_mt3 MODIFY TTL create_time + INTERVAL 10 SECOND
```

### ReplacingMergeTree

>相比于MergerTree，多了去重功能。
>
>去重时机：当数据合并时，触发去重，所以去重是在未知的时间进行的。
>
>去重范围：如果分区了，只在分区内去重。ReplacingMergeTree只能清除重复数据节省空间，但是无法保证不出现重复的。

```
create table t_order_rmt(
 id UInt32,
 sku_id String,
 total_amount Decimal(16,2) ,
 create_time Datetime 
) engine =ReplacingMergeTree(create_time)
 partition by toYYYYMMDD(create_time)
 primary key (id)
 order by (id, sku_id);

```

### SummingMergeTree

>用于关心汇总聚合结果的场景
>
>设计聚合表时可以去除唯一键值、流水号，所有字段都是维度、度量和时间戳

## SQL操作

>与mysql基本一直，有部分语法不同

### Insert

```
insert into [table_name] values(...),(...)
insert into [table_name]  select a,b,c from  [table_name_2]
```

### Update和Delete

>clickhouse提供delete和update的操作，这类操作被称为Mutation查询，可以看做Alter的一种。
>
>虽然可以实现修改和删除，但是和一般的OLTP数据库不一样，Mutation语句是一种很重的操作，而且不支持事务。
>
>重的原因是每次修改或删除都会导致房企目标数据的原有分区，重建新分区。所以尽量做批量的变更，不要进行频繁小数据的操作。

```
#删除操作
alter table t_order——smt  delete  where sku_id='sku_001';
#修改操作
alter table t_order_smt update total_amount=toDecimal32(2000.00,2) where id=102;
```

>由于操作比较重，所以Mutation分两步执行，同步执行的部分其实只是进行新增数据分区和并把分区打上逻辑上的实效标记。直到触发分区合并的时候，才会删除旧数据释放磁盘空间，一般不会开放这样的功能给用户，由管理员完成。

### 查询操作

>不支持窗口函数
>
>不支持自定义函数
>
>groupby 增加了 with rollup\with cube\with total 用来计算小计和总计

```
#with rollup:从右至左去掉维度进行小计
select id , sku_id,sum(total_amount) from t_order_mt group by 
id,sku_id with rollup;
#with cube：从右至左去掉维度小计，再从左至右去掉维度进行小计。
select id , sku_id,sum(total_amount) from t_order_mt group by 
id,sku_id with cube;
#只计算合计多少
select id , sku_id,sum(total_amount) from t_order_mt group by 
id,sku_id with totals;

```

### alter操作

>alter table tableName add column newcolname String after col1;

>alter table tableName modify column newcolname String;

>alter table tableName drop column newcolname;

## 导出数据

>clickhouse-client --query "select * from t_order_mt where create_time='2020-06-01  12:00:00'" --format CSVWithNames> /opt/module/data/rs1.csv













