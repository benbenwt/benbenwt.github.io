# HIVE SQL基础语法

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

##### 创建视图

```
create view dim_sku_info_view
as
select
    id,
    price,
    sku_name,
    sku_desc,
    weight,
    is_sale,
    spu_id,
    spu_name,
    category3_id,
    category3_name,
    category2_id,
    category2_name,
    category1_id,
    category1_name,
    tm_id,
    tm_name,
    create_time
from dim_sku_info;
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
#插入很慢，25s左右，和聚合统计一样慢。是调用mapreduce实现的，就算是一条也很慢。
insert into sample(architecture) values("test_arch");
```

```
#load data是直接迁移数据文件，所以比insert快。
LOAD DATA [LOCAL] INPATH 'filepath' [OVERWRITE] INTO TABLE tablename [PARTITION (partcol1=val1, partcol2=val2 ...)]
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

### 自定义函数

##### 创建函数

```
add jar /root/1.jar
create function test_explode_json_array_from_hdfs1 as 'com.atguigu.gmall.hive.udtf.ExplodeJSONArray';
```

```
create function explode_json_array  as 'com.atguigu.gmall.hive.udtf.ExplodeJSONArray' using jar 'hdfs://hbase:9000/user/hive/jars/gmall-udtf-1.0-SNAPSHOT.jar';

```

##### 删除函数

```
show functions like '*explode*'
desc function add_months;
desc function extended add_months;
drop [temporary] function [if exists] [dbname.]function_name;
```



# HIVE SQL 练习

>https://www.gairuo.com/p/hive-sql-tutorial
>
>更多练习：https://www.gairuo.com/p/hive-sql-case

### gmall项目

##### ads_order_spu_stats 主题

>已知所有表如下所示，编写脚本实现从ods层至ads的数据流动。具体表结构查看datagrip。ods层数据为原始数据，对应一条订单，最小粒度。dwd也为一条订单的粒度，但是其关联了支付、退款等信息。dws为一天的粒度，dwt为1，7，30等不同粒度的统计，ads为最终数据聚合。
>
>     ods_order_detail，ods_order_info，ods_order_detail_activity，ods_order_detail_coupon
>     dwd_order_detail，dwd_order_refund_info，dwd_payment_info，dwd_refund_payment
>             dws_sku_action_daycount，dim_sku_info   
>                   dwt_sku_topic，dim_sku_info
>                      ads_order_spu_stats
>#订单，订单明细，订单活动关联，订单优惠券关联         
>#交易的订单信息，退款订单信息，支付信息，退款顺序
>dwd层的工作量很大，需要聚合很多表，整合成单条记录形式，最小粒度。后边的dws，dwt基本基于dwd处理，不会连接太多表。
>#sku行为以天为粒度，列和DWS一样的。sku维度表
>#sku主题表，下单次数（是否参与活动，是否使用优惠券），下单件数（是否参与活动，是否使用优惠券），下单原始金额（活动优惠金额，优惠券优惠金额），下单最终金额，退款，评价（好评，差评，中评），购物车，收藏，以及1，7，30粒度的统计。sku维度表
>dwt这一层太宽了吧，这么多列。
>#spu的订单聚合信息，对于指定spu商品，其订单金额，订单数目，最近天数（1，7，30）

######  dwd_order_detail

```
#首先确定一下每一列来自那个ods表
id
order_id:四个表的连接列
user_id:来自ods_order_info表
sku_id:来自ods_order_detail
province_id:ods_order_info
activitity_id:ods_order_detail_activity
activity_rule_id:ods_order_detail_activity
coupon_id:ods_order_detail_coupon
create_time:ods_order_info
source_type:ods_order_detail
source_id:ods_order_detail
sku_num:ods_order_detail

original_amount:ods_order_detail
split_final_amount:ods_order_detail
split_activity_amount:ods_order_detail
split_coupon_amount:ods_order_detail
dt:日期，来自ods_order_detail
```

###### dwd_order_detail 清洗sql

>注意dwd_order_detail聚合的行粒度是一个订单明细，不是一个订单，所以left join用order_detail_id进行连接。一个订单包含多个订单明细。

```sql
INSERT OVERWRITE TABLE dwd_order_detail_my_practice
SELECT od.id,od.order_id,oi.user_id,od.sku_id,oi.province_id,oda.activity_id,oda.activity_rule_id,odc.coupon_id,oi.create_time,od.source_type,od.source_id,od.sku_num,od.order_price,od.split_final_amount,od.split_activity_amount,od.split_coupon_amount, date_format(create_time,'yyyy-MM-dd')
FROM
(SELECT id,order_id,sku_id,sku_num,source_type,source_id,order_price,split_final_amount,split_activity_amount,split_coupon_amount FROM ods_order_detail)od
LEFT JOIN
(SELECT id,user_id,province_id,create_time FROM ods_order_info )oi
 ON od.order_id=oi.id
LEFT JOIN
(SELECT order_detail_id,activity_id,activity_rule_id FROM ods_order_detail_activity)oda
ON od.id=oda.order_detail_id
LEFT JOIN
(SELECT order_detail_id,coupon_id FROM ods_order_detail_coupon)odc
ON od.id=odc.order_detail_id
```

###### dwd_order_refund_info清洗sql

```sql
#查看有哪些列，使用哪些表，哪些需要聚合。查两个表就行了，无聚合操作。主要是多表join。
INSERT OVERWRITE TABLE dwd_order_refund_info_my_pratice
SELECT ori.id,oi.user_id,ori.order_id,sku_id,oi.province_id,ori.refund_type,ori.refund_num,ori.refund_amount,ori.refund_reason_type,ori.create_time, date_format(ori.create_time,'yyyy-MM-dd')
FROM
    (SELECT id,order_id,sku_id,refund_type,refund_amount,refund_num,refund_reason_type,create_time FROM ods_order_refund_info)ori
     LEFT JOIN
     (SELECT id,user_id,province_id FROM ods_order_info)oi
    ON ori.order_id=oi.id
```

###### dwd_payment_info清洗sql

```sql
SELECT pi.id,order_id,pi.user_id,oi.province_id,pi.trade_no,pi.out_trade_no,pi.payment_type,pi.payment_amount,pi.payment_status,pi.create_time,pi.callback_time,nvl(date_format(pi.callback_time,"yyyy-MM-dd"),"9999-99-99") FROM
(SELECT * FROM ods_payment_info)pi
LEFT JOIN
(SELECT * FROM ods_order_info)oi
ON pi.order_id=oi.id
```

##### 商品ads 主题

###### dws_sku_action_daycount

>dwd进行了多表join，dws需要进行聚合统计。
>
>这张表是对sku的信息进行统计，粒度为一天，包括下单、支付、退单、退款、评价与收藏这几个板块。在下单板块内包括下单件数、下单次数、下单金额，以及参与活动和使用优惠券的情况下，这次数、件数指标的计算，还有活动优惠金额、优惠券优惠金额，被下单原始金额，被下单最终金额。在支付板块，有支付件数、支付次数、支付金额。在退单板块有退单次数、件数、金额。退款板块有退款次数、件数、金额。评价好、中、查次数、默认评价数，收藏次数，购物车次数。

拆开逐个看吧

>count代表次数，num代表该sku的件数，amount代表该sku的金额

```
#被下单次数，被下单件数，参与活动被下单件数，参与活动被下单件数，使用优惠券被下单次数，使用优惠券被下单件数，优惠金额（活动），优惠金额（优惠券）
select
        date_format(create_time,'yyyy-MM-dd') dt,
        sku_id,
        count(*) order_count,
        sum(sku_num) order_num,
        sum(if(split_activity_amount>0,1,0)) order_activity_count,
        sum(if(split_coupon_amount>0,1,0)) order_coupon_count,
        sum(split_activity_amount) order_activity_reduce_amount,
        sum(split_coupon_amount) order_coupon_reduce_amount,
        sum(original_amount) order_original_amount,
        sum(split_final_amount) order_final_amount
    from dwd_order_detail
    group by date_format(create_time,'yyyy-MM-dd'),sku_id

```

```
被支付金额，被支付件数，被支付件数
 select
        date_format(callback_time,'yyyy-MM-dd') dt,
        sku_id,
        count(*) payment_count,
        sum(sku_num) payment_num,
        sum(split_final_amount) payment_amount
    from dwd_order_detail od
    join
    (
        select
            order_id,
            callback_time
        from dwd_payment_info
        where callback_time is not null
    )pi on pi.order_id=od.order_id
    group by date_format(callback_time,'yyyy-MM-dd'),sku_id
```

```
#退单次数，退单金额
select
        date_format(create_time,'yyyy-MM-dd') dt,
        sku_id,
        count(*) refund_order_count,
        sum(refund_num) refund_order_num,
        sum(refund_amount) refund_order_amount
    from dwd_order_refund_info
    group by date_format(create_time,'yyyy-MM-dd'),sku_id
#
```

```
#退款次数，退款金额
select
        date_format(callback_time,'yyyy-MM-dd') dt,
        rp.sku_id,
        count(*) refund_payment_count,
        sum(ri.refund_num) refund_payment_num,
        sum(refund_amount) refund_payment_amount
    from
    (
        select
            order_id,
            sku_id,
            refund_amount,
            callback_time
        from dwd_refund_payment
    )rp
    left join
    (
        select
            order_id,
            sku_id,
            refund_num
        from dwd_order_refund_info
    )ri
    on rp.order_id=ri.order_id
    and rp.sku_id=ri.sku_id
    group by date_format(callback_time,'yyyy-MM-dd'),rp.sku_id
```

```
# 购物车，收藏
select
        dt,
        item sku_id,
        sum(if(action_id='cart_add',1,0)) cart_count,
        sum(if(action_id='favor_add',1,0)) favor_count
    from dwd_action_log
    where action_id in ('cart_add','favor_add')
    group by dt,item
),

```

```
#评价次数
select
        date_format(create_time,'yyyy-MM-dd') dt,
        sku_id,
        sum(if(appraise='1201',1,0)) appraise_good_count,
        sum(if(appraise='1202',1,0)) appraise_mid_count,
        sum(if(appraise='1203',1,0)) appraise_bad_count,
        sum(if(appraise='1204',1,0)) appraise_default_count
    from dwd_comment_info
    group by date_format(create_time,'yyyy-MM-dd'),sku_id

```

```
将5大板块通过dt加sku_id分组查询出来后，没有使用join，而是使用union all合并，不存在的字段使用0，方便后续的sum求和。union的优势，
```

###### dwt_sku_topic

>确定列来源的表，连接粒度，分块。分块基本按照业务流程划分，如下单、支付、退单、退款，然后这些块内部的属性，如金额、次数、联系上优惠券、活动属性。另外，DWT表还要负责更高粒度的统计，如1粒度、7粒度、30粒度。维度组合多了，看的很混乱，脑袋疼。总结以下固定顺序吧，按照如下顺序逐个处理: 选择业务过程(确认维度)→声明粒度→确认事实,与dws相似，但是dwt的一个表的维度可以是多个业务，比如包括下单、退款等等。选中业务后，确认此业务关注的列，以下单为例子，其维度可以通过组合确认，即（金额、次数）（原始金额、使用优惠券优惠的、活动优惠的）（时间跨度），。粒度一般为主题的最小单位，如sku、user_id、coupon_id、activity_id。确认事实这一步多余，因为有多个事实，如前边提到的金额、次数。以此表为例，业务分为下单、支付、退款、退单，组合维度如上所示

```sql

```



### 聚合查询

##### 指定商品带来的复购

>运营人员上架了一种专门用来拉新的商品，这些商品不管从需求还是价格都具有吸引力，目标是刺激用户快速下单这些商品，完成拉新。接下来，就要分析这些用户成为新用户后是否再有购买，形成复购的情况。

数据表位于 `dwd.order_detail`：

| p_day    | uid    | order_id   | create_time             | sku_id |
| -------- | ------ | ---------- | ----------------------- | ------ |
| 20190410 | 23424  | 3325264322 | 2019-04-10 14:55:37.300 | 1111   |
| 20190513 | 454121 | 9725372353 | 2019-05-13 13:54:17.300 | 23421  |
| 20190511 | 4234   | 5345433525 | 2019-05-11 10:15:31.300 | 21322  |
| 20190315 | 32546  | 5354378679 | 2019-03-15 08:01:41.667 | 14325  |
| 20190515 | 2525   | 6436438692 | 2019-05-15 19:05:55.000 | 1111   |

```
各字段说明：

p_day：hive 库以日期分片，下单日期
uid：用户的注册 id
order_id：下单订单号
create_time：订单创建时间，不重复
sku_id：商品 ID，其中 1111 是拉新的活动商品
数据表是一个以订单-商品为粒度的流水表。
```



##### 按周订单数及用户数

#####  按 UTM 串统计访问情况

#####  用户留存数据

### sql优化

# 仓库规范

>ODS Operation Data Store
>
>DIM 维度层
>
>DWD Data Warehouse Detail
>
>DWS Data Warehouse Service
>
>DWT Data Warehouse Topic
>
>ADS Application Data Store



### 范式理论

```
第一范式，属性不可分割
第二范式，不可存在部分依赖。如（学号，班级）->姓名，其中姓名部分依赖于学号，班级是多余的主键。
第三范式，不可存在传递依赖。如学号->班级->班主任。
关系建模符合第三范式，能减少冗余数据，避免数据不一致。
维度建模使用事实表和维度表，可以不符合范式，容忍冗余数据，是为了提高查询效率，避免太多join操作。
```

### 维度建模模型分类

>星型模型、雪花模型、星座模型。
>
>雪花模型多级，而星型模型一般只有一级。
>
>星座模型基于多个事实表，他们之间是否共享维度表，星座不星座反映是否有多个事实表。



### 维度表

一般是对事实的**描述信息**。每一张维表对应现实世界中的一个对象或者概念。  例如：用户、商品、日期、地区等。

维表的范围很宽（具有多个属性、列比较多）

跟事实表相比，行数相对较小：通常< 10万条

内容相对固定：编码表

### 事实表

非常的大

内容相对的窄：列数较少（主要是外键id和度量值）

经常发生变化，每天会新增加很多。

##### **周期型快照事实表**

>周期型快照事实表中**不会保留所有数据**，**只保留固定时间间隔的数据**，例如每天或者每月的销售额，或每月的账户余额等。

##### **累积型快照事实表**

>**累计快照事实表用于跟踪业务事实的变化。**例如，数据仓库中可能需要累积或者存储订单从下订单开始，到订单商品被打包、运输、和签收的各个业务阶段的时间点数据来跟踪订单声明周期的进展情况。当这个业务过程进行时，事实表的记录也要不断更新。

### lzo压缩

>https://cwiki.apache.org/confluence/display/Hive/LanguageManual+LZO

```
#创建lzo索引
hadoop jar /path/to/jar/hadoop-lzo-cdh4-0.4.15-gplextras.jar com.hadoop.compression.lzo.LzoIndexer /path/to/HDFS/dir/ contains/lzo/files
```



### 分区

>https://zhuanlan.zhihu.com/p/65442409
>
>分区列（Partition columns）是虚拟列，使用分区列显著加快分析速度。常用的有DATE

```
分区是指将hive的hdfs文件进行切分，
#单值分区
导入数据时手动指定分区
#单值动态分区
导入数据时，系统可以动态判断目标分区
```

##### 分区语法

```
#分区语法，在PARTITI1ONED BY后面跟上分区键、类型即可（指定的分区键不能出现在定义列名中）
CREATE [EXTERNAL] TABLE <table_name>
    (<col_name> <data_type> [, <col_name> <data_type> ...])
    -- 指定分区键和数据类型
    PARTITIONED BY  (<partition_key> <data_type>, ...) 
    [CLUSTERED BY ...] 
    [ROW FORMAT <row_format>] 
    [STORED AS TEXTFILE|ORC|CSVFILE]
    [LOCATION '<file_path>']    
   [TBLPROPERTIES ('<property_name>'='<property_value>', ...)];
```

##### 静态分区写入

```
-- 覆盖写入
INSERT OVERWRITE TABLE <table_name> 
    PARTITION (<partition_key>=<partition_value>[, <partition_key>=<partition_value>, ...]) 
    SELECT <select_statement>;

-- 追加写入
INSERT INTO TABLE <table_name> 
    PARTITION (<partition_key>=<partition_value>[, <partition_key>=<partition_value>, ...])
    SELECT <select_statement>;
```

##### 动态分区写入



```
#dpk必须放在spk后边，如下spk即static partition key（静态分区键），dpk为dynamic partition key（动态分区键）
CREATE TABLE <table_name>
 PARTITIONED BY ([<spk> <data_type>, ... ,] <dpk> <data_type>, [<dpk>
<data_type>,...]);
```

```
-- 开启动态分区支持，并设置最大分区数
set hive.exec.dynamic.partition=true;
//set hive.exec.dynamic.partition.mode=nostrict;
set hive.exec.max.dynamic.partitions=2000;

insert into table1 select 普通字段 分区字段 from table2
```



### 分桶

>对Hive(Inceptor)表分桶可以将表中记录按分桶键的哈希值分散进多个文件中，这些小文件称为桶。
>
>每个分区中的数据又可以基于表的某一列的散列函数的值被划分为桶,如对id进行hash分桶。

##### 分桶语法

```
CREATE [EXTERNAL] TABLE <table_name>
    (<col_name> <data_type> [, <col_name> <data_type> ...])]
    [PARTITIONED BY ...] 
    CLUSTERED BY (<col_name>) 
        [SORTED BY (<col_name> [ASC|DESC] [, <col_name> [ASC|DESC]...])] 
        
        INTO <num_buckets> BUCKETS  
        
    [ROW FORMAT <row_format>] 
    [STORED AS TEXTFILE|ORC|CSVFILE]
    [LOCATION '<file_path>']    
    [TBLPROPERTIES ('<property_name>'='<property_value>', ...)];
```

### 分层

```
当仓库分为多层，有的达到5层，如果源头输入新的数据，从第一层到最后一层需要多久？其耗费的时间说明，这种结构只适用于离线。那么即席查询的意思就是开发离线模型的demo，用于演示，而不是“实时查询”，因为它的“即席”是建立在数据已经从源头流入最后一层的基础上的，也就是已经处理完成了，这并不实时。
```

### json格式

```
json格式要实现主键查询，非主键查询，所有键的聚合。实际上查询的功能应该由hbase来做。
```

# 常用端口

```
hive.metastore.uri=thrift://hadoop102:9083
web port 10002

```



# hive自动映射到hbase

```
https://blog.csdn.net/weixin_44694973/article/details/98845551
hadoop.hive.hbase.HBbaseStorageHandler
```

```
CREATE TABLE cctable (key int, value string) STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler' WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key,cf:val") TBLPROPERTIES ("hbase.table.name" = "cc");
```

# HIVE SQL理论知识

### 基本类型

###### 数值型

```
TINYINT—1 byte integer（1 个字节的整形型）
SMALLINT—2 byte integer（2 个字节的整形型）
INT—4 byte integer（4 个字节的整形型）
BIGINT—8 byte integer（8 个字节的整形型）

FLOAT—single precision（单精度）
DOUBLE—Double precision（双精度）
```

###### 字符型

```
STRING — 指定字符集中的字符序列
VARCHAR — 指定字符集中具有最大长度的字符序列
CHAR — 指定字符集中具有最大长度的字符序列
```

###### 布尔型

```
BOOLEAN—TRUE/FALSE（只有真/假两个种取值）
```

###### 日期时间型

```
TIMESTAMP — 没有时区的日期和时间 ("LocalDateTime" 语意)
TIMESTAMP WITH LOCAL TIME ZONE — 精度到纳秒的时间点 ("Instant" 语意)
DATE — 一个日期
```

###### 二进制型

```
BINARY — 字节序列
```



### 运算符

>https://www.gairuo.com/p/hive-sql-operators

##### 算术运算符

| 操作符 | 描述                                               | 示例         |
| ------ | -------------------------------------------------- | ------------ |
| +      | 相加：将符号两边的数值加起来。                     | a + b 得 30  |
| -      | 相减：从最边的操作数中减去右边的操作数。           | a - b 得 -10 |
| *      | 相乘：将两边的操作数相乘。                         | a * b 得 200 |
| /      | 相除：用右边的操作数除以左边的操作数。             | b / a 得 2   |
| %      | 取余：用右边的操作数除以左边的操作数，并返回余数。 | b % a 得 0   |

以下运算符支持对操作数的各种常见算术运算。 所有返回数字类型； 如果任何操作数为NULL，则结果也为 NULL。

| **Operator** | **Operand types** | **Description**                                              |
| ------------ | ----------------- | ------------------------------------------------------------ |
| A + B        | All number types  | Gives the result of adding A and B. The type of the result is the same as the common parent(in the type hierarchy) of the types of the operands. For example since every integer is a float, therefore float is a containing type of integer so the + operator on a float and an int will result in a float. |
| A - B        | All number types  | Gives the result of subtracting B from A. The type of the result is the same as the common parent(in the type hierarchy) of the types of the operands. |
| A * B        | All number types  | Gives the result of multiplying A and B. The type of the result is the same as the common parent(in the type hierarchy) of the types of the operands. Note that if the multiplication causing overflow, you will have to cast one of the operators to a type higher in the type hierarchy. |
| A / B        | All number types  | Gives the result of dividing A by B. The result is a double type in most cases. When A and B are both integers, the result is a double type except when the [hive.compat](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.compat) configuration parameter is set to "0.13" or "latest" in which case the result is a decimal type. |
| A DIV B      | Integer types     | Gives the integer part resulting from dividing A by B. E.g 17 div 3 results in 5. |
| A % B        | All number types  | Gives the reminder resulting from dividing A by B. The type of the result is the same as the common parent(in the type hierarchy) of the types of the operands. |
| A & B        | All number types  | Gives the result of bitwise AND of A and B. The type of the result is the same as the common parent(in the type hierarchy) of the types of the operands. |
| A \| B       | All number types  | Gives the result of bitwise OR of A and B. The type of the result is the same as the common parent(in the type hierarchy) of the types of the operands. |
| A ^ B        | All number types  | Gives the result of bitwise XOR of A and B. The type of the result is the same as the common parent(in the type hierarchy) of the types of the operands. |
| ~A           | All number types  | Gives the result of bitwise NOT of A. The type of the result is the same as the type of A. |

```
可以对列值进行计算
select chinese+math from students
```

##### 关系运算符

以下运算符比较传递的操作数，并根据操作数之间的比较是否成立来生成TRUE或FALSE值。

| **操作**                 | **操作数类型**                   | **说明**                                                     |
| ------------------------ | -------------------------------- | ------------------------------------------------------------ |
| A = B                    | All primitive types 所有原始类型 | 如果表达式A等于表达式B，则为TRUE，否则为FALSE。              |
| A == B                   | All primitive types              | = 运算符的同义词。                                           |
| A <=> B                  | All primitive types              | 对于非空操作数，使用EQUAL（=）运算符返回相同的结果，但如果两个均为NULL，则返回TRUE，如果其中之一为NULL，则返回FALSE。(As of version [0.9.0](https://issues.apache.org/jira/browse/HIVE-2810).) |
| A <> B                   | All primitive types              | NULL if A or B is NULL, TRUE if expression A is NOT equal to expression B, otherwise FALSE. |
| A != B                   | All primitive types              | Synonym for the <> operator.                                 |
| A < B                    | All primitive types              | NULL if A or B is NULL, TRUE if expression A is less than expression B, otherwise FALSE. |
| A <= B                   | All primitive types              | NULL if A or B is NULL, TRUE if expression A is less than or equal to expression B, otherwise FALSE. |
| A > B                    | All primitive types              | NULL if A or B is NULL, TRUE if expression A is greater than expression B, otherwise FALSE. |
| A >= B                   | All primitive types              | NULL if A or B is NULL, TRUE if expression A is greater than or equal to expression B, otherwise FALSE. |
| A [NOT] BETWEEN B AND C  | All primitive types              | NULL if A, B or C is NULL, TRUE if A is greater than or equal to B AND A less than or equal to C, otherwise FALSE. This can be inverted by using the NOT keyword. (As of version [0.9.0](https://issues.apache.org/jira/browse/HIVE-2005).) |
| A IS NULL                | All types                        | TRUE if expression A evaluates to NULL, otherwise FALSE.     |
| A IS NOT NULL            | All types                        | FALSE if expression A evaluates to NULL, otherwise TRUE.     |
| A IS [NOT] (TRUE\|FALSE) | Boolean types                    | Evaluates to TRUE only if A mets the condition. (since:[3.0.0](https://issues.apache.org/jira/browse/HIVE-13583) ) Note: NULL is UNKNOWN, and because of that (UNKNOWN IS TRUE) and (UNKNOWN IS FALSE) both evaluates to FALSE. |
| A [NOT] LIKE B           | strings                          | NULL if A or B is NULL, TRUE if string A matches the SQL simple regular expression B, otherwise FALSE. The comparison is done character by character. The _ character in B matches any character in A (similar to . in posix regular expressions) while the % character in B matches an arbitrary number of characters in A (similar to .* in posix regular expressions). For example, 'foobar' like 'foo' evaluates to FALSE whereas 'foobar' like 'foo_ _ _' evaluates to TRUE and so does 'foobar' like 'foo%'. |
| A RLIKE B                | strings                          | NULL if A or B is NULL, TRUE if any (possibly empty) substring of A matches the Java regular expression B, otherwise FALSE. For example, 'foobar' RLIKE 'foo' evaluates to TRUE and so does 'foobar' RLIKE '^f.*r$'. |
| A REGEXP B               | strings                          | Same as RLIKE.                                               |

##### 逻辑运算符

以下运算符为创建逻辑表达式提供支持。 它们都根据操作数的布尔值返回布尔值TRUE，FALSE或NULL。 NULL表现为“未知”标志，因此，如果结果取决于未知状态，则结果本身是未知的。

| **Operator**               | **Operand types** | **Description**                                              |
| -------------------------- | ----------------- | ------------------------------------------------------------ |
| A AND B                    | boolean           | TRUE if both A and B are TRUE, otherwise FALSE. NULL if A or B is NULL. |
| A OR B                     | boolean           | TRUE if either A or B or both are TRUE, FALSE OR NULL is NULL, otherwise FALSE. |
| NOT A                      | boolean           | TRUE if A is FALSE or NULL if A is NULL. Otherwise FALSE.    |
| ! A                        | boolean           | Same as NOT A.                                               |
| A IN (val1, val2, ...)     | boolean           | TRUE if A is equal to any of the values. As of Hive 0.13 [subqueries](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+SubQueries) are supported in IN statements. |
| A NOT IN (val1, val2, ...) | boolean           | TRUE if A is not equal to any of the values. As of Hive 0.13 [subqueries](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+SubQueries) are supported in NOT IN statements. |
| [NOT] EXISTS (subquery)    |                   | TRUE if the the subquery returns at least one row. Supported as of [Hive 0.13](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+SubQueries). |

##### 字符串运算符

| **Operator** | **Operand types** | **Description**                                              |
| ------------ | ----------------- | ------------------------------------------------------------ |
| A \|\| B     | strings           | Concatenates the operands - shorthand for `concat(A,B)` . Supported as of [Hive 2.2.0](https://issues.apache.org/jira/browse/HIVE-14580). |

##### 复杂类型构造函数

| Constructor Function | Operands                          | Description                                                  |
| -------------------- | --------------------------------- | ------------------------------------------------------------ |
| Constructor Function | Operands                          | Description                                                  |
| map                  | (key1, value1, key2, value2, ...) | Creates a map with the given key/value pairs.                |
| struct               | (val1, val2, val3, ...)           | Creates a struct with the given field values. Struct field names will be col1, col2, .... |
| named_struct         | (name1, val1, name2, val2, ...)   | Creates a struct with the given field names and values. (As of Hive [0.8.0](https://issues.apache.org/jira/browse/HIVE-1360).) |
| array                | (val1, val2, ...)                 | Creates an array with the given elements.                    |
| create_union         | (tag, val1, val2, ...)            | Creates a union type with the value that is being pointed to by the tag parameter. |

##### 复杂类型的运算符

| **Operator** | **Operand types**                   | **Description**                                              |
| ------------ | ----------------------------------- | ------------------------------------------------------------ |
| A[n]         | A is an Array and n is an int       | Returns the nth element in the array A. The first element has index 0. For example, if A is an array comprising of ['foo', 'bar'] then A[0] returns 'foo' and A[1] returns 'bar'. |
| M[key]       | M is a Map<K, V> and key has type K | Returns the value corresponding to the key in the map. For example, if M is a map comprising of {'f' -> 'foo', 'b' -> 'bar', 'all' -> 'foobar'} then M['all'] returns 'foobar'. |
| S.x          | S is a struct                       | Returns the x field of S. For example for the struct foobar {int foo, int bar}, foobar.foo returns the integer stored in the foo field of the struct. |







### 关键字

##### DISTINCT

```
我们取到某列数据后发现有重复的内容，但需求可能是需要知道有几个不重复的内容。Select 里 DISTINCT 可用于对数据进行去重。
1distinct 必须放在开头
2如果值中有 NULL，会保留一个 NULL
3如果有多个列，则对这几个列组合去重
```

##### CASE

```
select c_1,
       CASE
           WHEN condition1 THEN result1
           WHEN condition2 THEN result2
           WHEN conditionN THEN resultN
           ELSE result_else
           END as c_name
from tab_name
-由 CASE 开始 END 结束
-最好起一个别名（as c_name，c_name 为别名），不然此列没有可读性名称
-WHEN 和 THEN 成对出现，WHEN 后边为条件，可以使用表中的所有字段，THEN 后为最终输出的值
-ELSE 为兜底逻辑，直接给出值，ELSE 可以没有
-没有被条件覆盖的值为 null
```

##### order by

```
order by (CASE
              WHEN b_year < 1990 THEN math+20
              ELSE math
    END) DESC
ascending desending 
```

```
#用序号代表字段
SELECT item_id, uesr_id
         ^^^^        ^^^^
          1           2
FROM tab
ORDER BY 1;
```

#####  sort by

>hive支持,reduce局部排序，比order by性能消耗少。

##### distribute by

>distribute 意为分发、分配。
>
>特别的，因为distribute by 通常和sort by 一起用，所以当distribute by 遇上 sort by时，distribute by要放在前面，这个不难理解，因为要先通过 distribute by 将待处理的数据从map端做分发，这样，sort by 这个擅长局部排序的才能去放开的干活。

```
FROM records
SELECT year, temperature
DISTRIBUTE BY year
SORT BY year DESC, temperature DESC
```

##### cluster by

>- order by：全部数据进行全局排序，只会启动一个 reducer
>- sort by：局部排序，会根据数据量大小启动一到多个 reducer
>- distribute by：控制 map 结果的分发 
>- cluster by：如果 distribute by 和 sort by 的列相同可以用 cluster by 简写

```
# cluster by = distribute by + sort by 
#但是cluster by 指定的列只能是降序，不能指定 asc 和 desc,例如：
select * from students cluster by b_year
等价于：
select * from students distribute by b_year sort by b_year
```

##### sum

```
#可以将 case 语句用在 sum 函数中，实现分段统计：
select sum(case when math > 80 then 1 else 0 end)  as 大于90,
       sum(case when math < 60 then 1 else 0 end)  as 小于60,
       sum(case when math <= 80 then 1 else 0 end) as 小于等于80
from students
#将要统计的分类映射为 1，不统计的为 0，sum 相加就是最终要统计的分类。
select
    sum(case when gender = "男" then 1 else 0 end) as boy_qty,
    sum(case when gender = "女" then 1 else 0 end) as girl_qty
from students
```

##### where

```
SELECT column, another_column, etc
FROM mytable
WHERE condition
    AND/OR another_condition
    AND/OR another_condition;
-WHERE 子句仅用于提取满足指定条件的那些记录
-条件子句可以是单个逻辑也可以是由 and or 组成的复杂条件表达式
-WHERE 子句不仅在 SELECT 语句中使用，还在 UPDATE，DELETE 语句等中使用
```

以下 Where 子句中的逻辑操作符号,包括数值，列表，字符

| 操作                   | 条件说明           | SQL 样例                   |
| ---------------------- | ------------------ | -------------------------- |
| =, !=, <>, < <=, >, >= | 数字及逻辑         | class != 3; math > 80      |
| IS NULL                | 值为 NULL          | name IS NULL               |
| IS NOT NULL            | 值不为 NULL        | name IS NOT NULL           |
| BETWEEN … AND …        | 包含两端的数字范围 | math BETWEEN 60 AND 80     |
| NOT BETWEEN … AND …    | 上述的不包含       | math NOT BETWEEN 60 AND 80 |
| IN (…)                 | 内容在指定的列表中 | class IN (1,2)             |
| NOT IN (…)             | 不在指定的列表中   | class NOT IN (1,2)         |
| LIKE …                 | 按内容搜索匹配     | name LIKE "张%"            |
| NOT LIKE …             | 不匹配此规则       | name NOT LIKE "张%"        |

```
-!= 和 <> 都是不等于
-判断是空字符串为 a = ''
-LIKE 可以用 % 和 _ 通配符等进行匹配
-In 不能滥用，in 里面只能有几个（如枚举），不能有几千几万几十万个，容易卡死系统。更好的办法是不用 in ，使用 join 处理
```

##### AND, OR 和 NOT 逻辑连接

| 符号 | 逻辑             | 举例                           |
| ---- | ---------------- | ------------------------------ |
| AND  | 和，全部为真     | b_year > 2000 and math > 80    |
| OR   | 或，只要一个为真 | b_year = 2010 and chinese > 80 |
| NOT  | 非，与逻辑值相反 | not gender == '男'             |

##### LIMIT 和 OFFSET 限制结果数量

>- limit Y： 从头取 Y 个数据
>- limit X, Y ：跳过 X 个数据，读取 Y 个数据
>- limit Y offset X ：跳过 X 个数据，选取 Y 个数据

```
select name, math
from students
order by math desc
limit 3, 4
-- 以上从第 3 条（不含）开始取 4 条。
-- 取3条，从第 5 个开始
select name from students limit 3 offset 4
-- 取4条，从第 4 个开始
select name from students limit 3, 4
```

##### 常用聚合统计函数

>分组依据的列直接可以输出，非分组列需要聚合计算才能输出。

| 函数              | 功能描述 | 其他                                          |
| ----------------- | -------- | --------------------------------------------- |
| count()           | 条数     | 不计 null 值                                  |
| sum()             | 求和     | True 按 1 处理，False 按 0 处理，忽略 null 值 |
| max()             | 最大值   | 时间字段代表最近最晚的时间                    |
| min()             | 最小值   | 时间字段代表最早的时间                        |
| avg()             | 平均值   | 忽略 null 值, sum 除以非空值的计数 count      |
| collect_list(col) |          | 返回具有重复项的对象列表（array）             |
| collect_set(col)  |          | 返回去除了重复元素的一组对象（array）         |



```
count() 经常与 DISTINCT 组合使用，表示去重后的总数量：
-- 有几个班：3
select count(distinct class) as class_qty from students
-- 有几个性别：2
select count(distinct gender) as gender_qty from students
```

```
select (case
            when b_year >= 2000 then '00后'
            when b_year >= 1990 then '90后'
            else '其他'
    end)         as gap,
       avg(math) as avg_math
from students
group by (case
              when b_year >= 2000 then '00后'
              when b_year >= 1990 then '90后'
              else '其他'
    end)
'''
gap|avg_math|
---+--------+
00后|    73.0|
90后|    55.0|
其他|    77.4|
'''
以上按年龄分组再进行聚合。
```

##### HAVING

```
#HAVING 可以在聚合后对聚合的结果进行条件筛选，因为聚合后的字段不是数据表里的真实字段。
select class as class, -- 班级
       avg(2020 - b_year) as avg_age -- 平均年龄
from students
group by class
having avg_age > 30
order by avg_age
'''
class|avg_age|
-----+-------+
    3|   31.5|
    1|   39.5|
'''
```

### 语句的执行顺序

```
在 hive 和 mysql 中都可以通过 explain+sql 语句，来查看执行顺序。对于一条标准 sql 语句，它的书写顺序是这样的：
select … from … where … group by … having … order by … limit …
（1）mysql 语句执行顺序：
from... where...group by... having.... select ... order by... limit …
（2）hive 语句执行顺序：
FROM —> WHERE —> GROUP BY—> 聚合函数 —> HAVING—> SELECT —> ORDER BY —> LIMIT
```

```
hive 基于 MapReduce 程序，它的执行顺序决定了 hive 语句的执行顺序，Map 阶段：

执行 from 加载，进行表的查找与加载
执行 where 过滤，进行条件过滤与筛选
执行 select 查询：进行输出项的筛选
执行 group by 分组：描述了分组后需要计算的函数
map 端文件合并：map 端本地溢出写文件的合并操作，每个 map 最终形成一个临时文件。
然后按列映射到对应的 reduceReduce 阶段：

group by：对map端发送过来的数据进行分组并进行计算。
select：最后过滤列用于输出结果
limit：排序后进行结果输出到HDFS文件
```

###### 优化要点

```
根据执行顺序，我们平时编写时需要记住以下几点：

使用分区剪裁、列剪裁，分区一定要加
少用 COUNT DISTINCT，group by 代替 distinct
是否存在多对多的关联
连接表时使用相同的关键词，这样只会产生一个 job
减少每个阶段的数据量，只选出需要的，在 join 表前就进行过滤
大表放后面
谓词下推：where 谓词逻辑都尽可能提前执行，减少下游处理的数据量
sort by 代替 order by
```



### 多表查询

##### where

```
select students.name, class.teacher, students.math
from students,
     class
where students.class = class.class
```

##### join

| 连接方式   | 逻辑说明                                         |
| ---------- | ------------------------------------------------ |
| JOIN       | 即 INNER JOIN                                    |
| INNER JOIN | 将两个表公共都有的部分组成新表                   |
| FULL JOIN  | 包含左右两表的所有行， 对应左右表没有的都为 Null |
| LEFT JOIN  | 左表的全集及右表有的值，无值则为 Null            |
| RIGHT JOIN | 与 LEFT JOIN 相反                                |

![sql-join](../resources/images/sql-join.png)

```
JOIN
LEFT OUTER JOIN
RIGHT OUTER JOIN
FULL OUTER JOIN
```

###### semi join

>Semi Join，也叫半连接。Semi-join从一个表中返回的行与另一个表中数据行进行不完全联接查询（查找到匹配的数据行就返回，不再继续查找）。

###### cross join

>CROSS JOIN 会让左右表排列组合，产生笛卡尔积，效果如图示：

![cross-join](../resources/images/cross-join.jpg)

##### UNION 数据拼接

```
将学生和老师名单拼接在一起：
select class, teacher as name from class
union
select class,name as name from students
```

###### UNION ALL

UNION ALL 允许重复内容，会如实保留。

##### with as 临时中间表

```
-- with table_name as(子查询语句) 其他sql
with temp as (
    select * from xxx
)
select * from temp;
```

### 窗口函数

>窗口函数和 Group By 聚合函数区别在于：窗口函数仅仅只会将结果附加到当前的结果上，它不会对已有的行或列做任何修改。而 Group By 的做法完全不同：对于各个 Group 它仅仅会保留一行聚合结果。

```
SELECT
    id,
    avg(chinese) over()
FROM
    students
    id|avg(chinese) over()|
--|-------------------|
 1|  79.33333333333333|
 2|  79.33333333333333|
 3|  79.33333333333333|
 4|  79.33333333333333|
 5|  79.33333333333333|
 6|  79.33333333333333|
 7|  79.33333333333333|
 8|  79.33333333333333|
 9|  79.33333333333333|
'''
```

###### partition by 子句

```
如果我们按班级对语文求平均数呢？这就要使用 partition by 语句，partition by 的作用和 group by 是类似，用于分组，在 over() 中使用。

SELECT
    id,
    avg(chinese) over(PARTITION by class) as avg_class
FROM
    students
'''
id|avg_class|
--|---------|
 1|     82.5|
 3|     82.5|
 6|     82.5|
 8|     82.5|
 2|     84.0|
 4|     84.0|
 7|     84.0|
 5|     66.0|
 9|     66.0|
'''
就是按班级分组，计算出每班的平均语文成绩，最后显示在对应同学的后边。avg_class 列的意义为当前行同学 ID 所在班的语文平均成绩。
```

```
在每个窗口（分组）内，如果我们想按每个人的语文成绩排序，可以使用 order by 子句，这里我们用 RANK() 指定序号，同样的分数是相同的序号：

SELECT
    id,
    class,
    chinese,
    RANK() over(PARTITION by class ORDER by chinese DESC) as odr
FROM
    students
'''
id|class|chinese|odr|
--|-----|-------|---|
 6|    1|     99|  1|
 8|    1|     99|  1|
 1|    1|     77|  3|
 3|    1|     55|  4|
 2|    2|     99|  1|
 4|    2|     87|  2|
 7|    2|     66|  3|
 5|    3|     66|  1|
 9|    3|     66|  1|
'''
```

```
当 order by 与聚合函数一起使用时，会形成顺序聚合，如 sum 聚合与 order by 结合使用时，就实现类似于累计和的效果：

SELECT
    id,
    class,
    sum(id) over(PARTITION by class ORDER by id DESC) as sums
FROM
    students
'''
id|class|sums|
--|-----|----|
 8|    1|   8|
 6|    1|  14|
 3|    1|  17|
 1|    1|  18|
 7|    2|   7|
 4|    2|  11|
 2|    2|  13|
 9|    3|   9|
 5|    3|  14|
'''
```

### hive专用窗口函数

### Hive SQL 函数介绍

>日期函数，类型转换函数，集合函数，数学函数，表生成函数（UDTF）

### sql优化

查看执行计划

```
explain select * from dwd_action_log;
#加入extended展示计划更详细
explain extended select * from dwd_action_log;
```

```
Fetch Operator
TableScan
Select Operator
limit: -1
ListSink
```

### hive UDF函数

##### get_json_object

```
```

##### explode_json

```
```



### 触发器

```
一种特殊的存储过程，存储过程一般通过定义的名字直接调用，而触发器是通过增、删、改进行触发执行的。会在事件发生时自动强制执行。
常见触发器：after（for）或 instead of 用于 insert、update、delete事件。
```

```
create trigger 触发器的名字   on 操作表
　　for|after      instead of
　　update|insert|delete
　　as

　　SQL语句
#举例，
create trigger tr_delete on work
    for
    insert
    as
delete * from work where id=(select id from inserted);
创建了这个触发器，当我对表work进行insert操作完后，会自动执行delete * from work where id=(select id from inserted);将刚插入的数据删除（inserted这个是临时表并且只会存储最后一次操作的数据）；
```



### 存储过程

>hive没有存储过程
>
>存储过程（Stored Procedure）是在大型[数据库系统](https://baike.baidu.com/item/数据库系统/215176)中，一组为了完成特定功能的SQL 语句集，它存储在数据库中，一次[编译](https://baike.baidu.com/item/编译/1258343)后永久有效，用户通过指定存储过程的名字并给出参数（如果该存储过程带有参数）来执行它。存储过程是数据库中的一个重要对象。在数据量特别庞大的情况下利用存储过程能达到倍速的效率提升。

```
CREATE PROCEDURE order_tot_amt
@o_id int,
@p_tot int output
AS
SELECT @p_tot = sum(Unitprice*Quantity)
FROM orderdetails
WHERE orderid=@o_id
GO
该例子是建立一个简单的存储过程order_tot_amt,这个存储过程根据用户输入的订单ID号码(@o_id),由订单明细表 (orderdetails)中计算该订单销售总额[单价(Unitprice)*数量(Quantity)],这一金额通过@p_tot这一参数输出给调用这一存储过程的程序。
```



# HIVE 理论知识

### 外部表内部表

>https://blog.csdn.net/qq_36743482/article/details/78393678
>
>使用external修饰的即为外部表，内部表数据由hive自身管理，外部表数据由hdfs管理。
>
>内部表数据存储的位置是hive.metastore.warehouse.dir。外部表数据的存储位置由自己制定（如果没有LOCATION，Hive将在HDFS上的/user/hive/warehouse文件夹下以外部表的表名创建一个文件夹，并将属于这个表的数据存放在这里）；
>
>删除内部表会直接删除元数据（metadata）及存储数据；删除外部表仅仅会删除元数据，HDFS上的文件并不会被删除；
>
>对内部表的修改会将修改直接同步给元数据，而对外部表的表结构和分区进行修改，则需要修复（MSCK REPAIR TABLE table_name;）

### hive四种排序方式的区别

```
order by order by 是要对输出的结果进行全局排序，这就意味着只有一个reducer才能实现（多个reducer无法保证全局有序）但是当数据量过大的时候，效率就很低。如果在严格模式下（hive.mapred.mode=strict）,则必须配合limit使用

sort by sort by 不是全局排序，只是在进入到reducer之前完成排序，只保证了每个reducer中数据按照指定字段的有序性，是局部排序。配置mapred.reduce.tasks=[nums]可以对输出的数据执行归并排序。可以配合limit使用，提高性能

distribute by distribute by 指的是按照指定的字段划分到不同的输出reduce文件中，和sort by一起使用时需要注意， distribute by必须放在前面

cluster by

cluster by 可以看做是一个特殊的distribute by+sort by，它具备二者的功能，但是只能实现倒序排序的方式,不能指定排序规则为asc 或者desc
```

### metastore服务

>https://www.itcast.cn/news/20190829/12032894477.shtml

##### 内嵌模式

>使用hive自带的Derby数据库存储元数据

#####  本地模式

>配置了mysql，但是依赖于hive的服务无法远程访问metastore

##### **远程模式**

>配置了mysql，可通过hive.metastore.uris远程访问metastore,例如使用kylin进行访问。

```
#修改配置文件
<property>
    <name>hive.metastore.uris</name>
    <value>thrift://node-1:9083</value>
</property>
```

```
#启动metastore服务
nohup /export/servers/hive/bin/hive --service metastore &
nohup /export/servers/hive/bin/hive --service hiveserver2 &
```

### hive中join都有哪些

见HIVE SQL理论 join一章中

### Impala 和 hive 的查询有哪些区别

>

```
Impala是基于Hive的大数据实时分析查询引擎，直接使用Hive的元数据库Metadata,意味着impala元数据都存储在Hive的metastore中。并且impala兼容Hive的sql解析，实现了Hive的SQL语义的子集，功能还在不断的完善中。
```

### HIVE UDF

```
一般分为UDAF（用户自定义聚合函数）和UDTF（用户自定义表生成函数）
Hive有两个不同的接口编写UDF程序。一个是基础的UDF接口，一个是复杂的GenericUDF接口。
org.apache.hadoop.hive.ql. exec.UDF 基础UDF的函数读取和返回基本类型，即Hadoop和Hive的基本类型。如，Text、IntWritable、LongWritable、DoubleWritable等。
org.apache.hadoop.hive.ql.udf.generic.GenericUDF 复杂的GenericUDF可以处理Map、List、Set类型。
```

### Hive Sql 是怎样解析成MR job的

##### group by

```
将group by的字段作为map的输出key和reduce的key，实现聚合操作。
```

##### SQL转化为MapReduce的过程

>https://www.cnblogs.com/Dhouse/p/7132476.html

```
1Antlr定义SQL的语法规则，完成SQL词法，语法解析，将SQL转化为抽象语法树AST Tree
2遍历AST Tree，抽象出查询的基本组成单元QueryBlock
3遍历QueryBlock，翻译为执行操作树OperatorTree
4逻辑层优化器进行OperatorTree变换，合并不必要的ReduceSinkOperator，减少shuffle数据量
5遍历OperatorTree，翻译为MapReduce任务
6物理层优化器进行MapReduce任务的变换，生成最终的执行计划
SQL转化为MapReduce的过程
抽象语法树AST Tree:具有层级目录的树，不依赖于语言细节，抽象表示源代码。借助AST可以进行优化等功能。
QueryBlock：三元组：输入源，计算过程，输出.
OperatorTree：...
ReduceSinkOperator：...
```



# SGG_DW教程

### 建表语句

```
#日志ods表
drop table if exists ods_log;
CREATE EXTERNAL TABLE ods_log (`line` string)
PARTITIONED BY (`dt` string) -- 按照时间创建分区
STORED AS -- 指定存储方式，读数据采用LzoTextInputFormat；
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_log'  -- 指定数据在hdfs上的存储位置
;
#活动信息表
DROP TABLE IF EXISTS ods_activity_info;
CREATE EXTERNAL TABLE ods_activity_info(
    `id` STRING COMMENT '编号',
    `activity_name` STRING  COMMENT '活动名称',
    `activity_type` STRING  COMMENT '活动类型',
    `start_time` STRING  COMMENT '开始时间',
    `end_time` STRING  COMMENT '结束时间',
    `create_time` STRING  COMMENT '创建时间'
) COMMENT '活动信息表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_activity_info/';

#活动规则表
DROP TABLE IF EXISTS ods_activity_rule;
CREATE EXTERNAL TABLE ods_activity_rule(
    `id` STRING COMMENT '编号',
    `activity_id` STRING  COMMENT '活动ID',
    `activity_type` STRING COMMENT '活动类型',
    `condition_amount` DECIMAL(16,2) COMMENT '满减金额',
    `condition_num` BIGINT COMMENT '满减件数',
    `benefit_amount` DECIMAL(16,2) COMMENT '优惠金额',
    `benefit_discount` DECIMAL(16,2) COMMENT '优惠折扣',
    `benefit_level` STRING COMMENT '优惠级别'
) COMMENT '活动规则表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_activity_rule/';

#一级品类表
DROP TABLE IF EXISTS ods_base_category1;
CREATE EXTERNAL TABLE ods_base_category1(
    `id` STRING COMMENT 'id',
    `name` STRING COMMENT '名称'
) COMMENT '商品一级分类表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_base_category1/';

#二级品类表
DROP TABLE IF EXISTS ods_base_category2;
CREATE EXTERNAL TABLE ods_base_category2(
    `id` STRING COMMENT ' id',
    `name` STRING COMMENT '名称',
    `category1_id` STRING COMMENT '一级品类id'
) COMMENT '商品二级分类表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_base_category2/';

#三级品类表
DROP TABLE IF EXISTS ods_base_category3;
CREATE EXTERNAL TABLE ods_base_category3(
    `id` STRING COMMENT ' id',
    `name` STRING COMMENT '名称',
    `category2_id` STRING COMMENT '二级品类id'
) COMMENT '商品三级分类表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_base_category3/';

#编码字典表
DROP TABLE IF EXISTS ods_base_dic;
CREATE EXTERNAL TABLE ods_base_dic(
    `dic_code` STRING COMMENT '编号',
    `dic_name` STRING COMMENT '编码名称',
    `parent_code` STRING COMMENT '父编码',
    `create_time` STRING COMMENT '创建日期',
    `operate_time` STRING COMMENT '操作日期'
) COMMENT '编码字典表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_base_dic/';

#省份表
DROP TABLE IF EXISTS ods_base_province;
CREATE EXTERNAL TABLE ods_base_province (
    `id` STRING COMMENT '编号',
    `name` STRING COMMENT '省份名称',
    `region_id` STRING COMMENT '地区ID',
    `area_code` STRING COMMENT '地区编码',
    `iso_code` STRING COMMENT 'ISO-3166编码，供可视化使用',
    `iso_3166_2` STRING COMMENT 'IOS-3166-2编码，供可视化使用'
)  COMMENT '省份表'
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_base_province/';

#地区表
DROP TABLE IF EXISTS ods_base_region;
CREATE EXTERNAL TABLE ods_base_region (
    `id` STRING COMMENT '编号',
    `region_name` STRING COMMENT '地区名称'
)  COMMENT '地区表'
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_base_region/';

#品牌表
DROP TABLE IF EXISTS ods_base_trademark;
CREATE EXTERNAL TABLE ods_base_trademark (
    `id` STRING COMMENT '编号',
    `tm_name` STRING COMMENT '品牌名称'
)  COMMENT '品牌表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_base_trademark/';

#购物车表
DROP TABLE IF EXISTS ods_cart_info;
CREATE EXTERNAL TABLE ods_cart_info(
    `id` STRING COMMENT '编号',
    `user_id` STRING COMMENT '用户id',
    `sku_id` STRING COMMENT 'skuid',
    `cart_price` DECIMAL(16,2)  COMMENT '放入购物车时价格',
    `sku_num` BIGINT COMMENT '数量',
    `sku_name` STRING COMMENT 'sku名称 (冗余)',
    `create_time` STRING COMMENT '创建时间',
    `operate_time` STRING COMMENT '修改时间',
    `is_ordered` STRING COMMENT '是否已经下单',
    `order_time` STRING COMMENT '下单时间',
    `source_type` STRING COMMENT '来源类型',
    `source_id` STRING COMMENT '来源编号'
) COMMENT '加购表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_cart_info/';

#评论表
DROP TABLE IF EXISTS ods_comment_info;
CREATE EXTERNAL TABLE ods_comment_info(
    `id` STRING COMMENT '编号',
    `user_id` STRING COMMENT '用户ID',
    `sku_id` STRING COMMENT '商品sku',
    `spu_id` STRING COMMENT '商品spu',
    `order_id` STRING COMMENT '订单ID',
    `appraise` STRING COMMENT '评价',
    `create_time` STRING COMMENT '评价时间'
) COMMENT '商品评论表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_comment_info/';

#优惠券信息表
DROP TABLE IF EXISTS ods_coupon_info;
CREATE EXTERNAL TABLE ods_coupon_info(
    `id` STRING COMMENT '购物券编号',
    `coupon_name` STRING COMMENT '购物券名称',
    `coupon_type` STRING COMMENT '购物券类型 1 现金券 2 折扣券 3 满减券 4 满件打折券',
    `condition_amount` DECIMAL(16,2) COMMENT '满额数',
    `condition_num` BIGINT COMMENT '满件数',
    `activity_id` STRING COMMENT '活动编号',
    `benefit_amount` DECIMAL(16,2) COMMENT '减金额',
    `benefit_discount` DECIMAL(16,2) COMMENT '折扣',
    `create_time` STRING COMMENT '创建时间',
    `range_type` STRING COMMENT '范围类型 1、商品 2、品类 3、品牌',
    `limit_num` BIGINT COMMENT '最多领用次数',
    `taken_count` BIGINT COMMENT '已领用次数',
    `start_time` STRING COMMENT '开始领取时间',
    `end_time` STRING COMMENT '结束领取时间',
    `operate_time` STRING COMMENT '修改时间',
    `expire_time` STRING COMMENT '过期时间'
) COMMENT '优惠券表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_coupon_info/';

#优惠券领用表
DROP TABLE IF EXISTS ods_coupon_use;
CREATE EXTERNAL TABLE ods_coupon_use(
    `id` STRING COMMENT '编号',
    `coupon_id` STRING  COMMENT '优惠券ID',
    `user_id` STRING  COMMENT 'skuid',
    `order_id` STRING  COMMENT 'spuid',
    `coupon_status` STRING  COMMENT '优惠券状态',
    `get_time` STRING  COMMENT '领取时间',
    `using_time` STRING  COMMENT '使用时间(下单)',
    `used_time` STRING  COMMENT '使用时间(支付)',
    `expire_time` STRING COMMENT '过期时间'
) COMMENT '优惠券领用表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_coupon_use/';

#收藏表
DROP TABLE IF EXISTS ods_favor_info;
CREATE EXTERNAL TABLE ods_favor_info(
    `id` STRING COMMENT '编号',
    `user_id` STRING COMMENT '用户id',
    `sku_id` STRING COMMENT 'skuid',
    `spu_id` STRING COMMENT 'spuid',
    `is_cancel` STRING COMMENT '是否取消',
    `create_time` STRING COMMENT '收藏时间',
    `cancel_time` STRING COMMENT '取消时间'
) COMMENT '商品收藏表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_favor_info/';

#订单明细表
DROP TABLE IF EXISTS ods_order_detail;
CREATE EXTERNAL TABLE ods_order_detail(
    `id` STRING COMMENT '编号',
    `order_id` STRING  COMMENT '订单号',
    `sku_id` STRING COMMENT '商品id',
    `sku_name` STRING COMMENT '商品名称',
    `order_price` DECIMAL(16,2) COMMENT '商品价格',
    `sku_num` BIGINT COMMENT '商品数量',
    `create_time` STRING COMMENT '创建时间',
    `source_type` STRING COMMENT '来源类型',
    `source_id` STRING COMMENT '来源编号',
    `split_final_amount` DECIMAL(16,2) COMMENT '分摊最终金额',
    `split_activity_amount` DECIMAL(16,2) COMMENT '分摊活动优惠',
    `split_coupon_amount` DECIMAL(16,2) COMMENT '分摊优惠券优惠'
) COMMENT '订单详情表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_order_detail/';

#订单明细活动关联表
DROP TABLE IF EXISTS ods_order_detail_activity;
CREATE EXTERNAL TABLE ods_order_detail_activity(
    `id` STRING COMMENT '编号',
    `order_id` STRING  COMMENT '订单号',
    `order_detail_id` STRING COMMENT '订单明细id',
    `activity_id` STRING COMMENT '活动id',
    `activity_rule_id` STRING COMMENT '活动规则id',
    `sku_id` BIGINT COMMENT '商品id',
    `create_time` STRING COMMENT '创建时间'
) COMMENT '订单详情活动关联表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_order_detail_activity/';

#订单明细优惠券关联表
DROP TABLE IF EXISTS ods_order_detail_coupon;
CREATE EXTERNAL TABLE ods_order_detail_coupon(
    `id` STRING COMMENT '编号',
    `order_id` STRING  COMMENT '订单号',
    `order_detail_id` STRING COMMENT '订单明细id',
    `coupon_id` STRING COMMENT '优惠券id',
    `coupon_use_id` STRING COMMENT '优惠券领用记录id',
    `sku_id` STRING COMMENT '商品id',
    `create_time` STRING COMMENT '创建时间'
) COMMENT '订单详情活动关联表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_order_detail_coupon/';

#订单表
DROP TABLE IF EXISTS ods_order_info;
CREATE EXTERNAL TABLE ods_order_info (
    `id` STRING COMMENT '订单号',
    `final_amount` DECIMAL(16,2) COMMENT '订单最终金额',
    `order_status` STRING COMMENT '订单状态',
    `user_id` STRING COMMENT '用户id',
    `payment_way` STRING COMMENT '支付方式',
    `delivery_address` STRING COMMENT '送货地址',
    `out_trade_no` STRING COMMENT '支付流水号',
    `create_time` STRING COMMENT '创建时间',
    `operate_time` STRING COMMENT '操作时间',
    `expire_time` STRING COMMENT '过期时间',
    `tracking_no` STRING COMMENT '物流单编号',
    `province_id` STRING COMMENT '省份ID',
    `activity_reduce_amount` DECIMAL(16,2) COMMENT '活动减免金额',
    `coupon_reduce_amount` DECIMAL(16,2) COMMENT '优惠券减免金额',
    `original_amount` DECIMAL(16,2)  COMMENT '订单原价金额',
    `feight_fee` DECIMAL(16,2)  COMMENT '运费',
    `feight_fee_reduce` DECIMAL(16,2)  COMMENT '运费减免'
) COMMENT '订单表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_order_info/';

#退单表
DROP TABLE IF EXISTS ods_order_refund_info;
CREATE EXTERNAL TABLE ods_order_refund_info(
    `id` STRING COMMENT '编号',
    `user_id` STRING COMMENT '用户ID',
    `order_id` STRING COMMENT '订单ID',
    `sku_id` STRING COMMENT '商品ID',
    `refund_type` STRING COMMENT '退单类型',
    `refund_num` BIGINT COMMENT '退单件数',
    `refund_amount` DECIMAL(16,2) COMMENT '退单金额',
    `refund_reason_type` STRING COMMENT '退单原因类型',
    `refund_status` STRING COMMENT '退单状态',--退单状态应包含买家申请、卖家审核、卖家收货、退款完成等状态。此处未涉及到，故该表按增量处理
    `create_time` STRING COMMENT '退单时间'
) COMMENT '退单表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_order_refund_info/';

#订单状态日志表
DROP TABLE IF EXISTS ods_order_status_log;
CREATE EXTERNAL TABLE ods_order_status_log (
    `id` STRING COMMENT '编号',
    `order_id` STRING COMMENT '订单ID',
    `order_status` STRING COMMENT '订单状态',
    `operate_time` STRING COMMENT '修改时间'
)  COMMENT '订单状态表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_order_status_log/';

#支付表
DROP TABLE IF EXISTS ods_payment_info;
CREATE EXTERNAL TABLE ods_payment_info(
    `id` STRING COMMENT '编号',
    `out_trade_no` STRING COMMENT '对外业务编号',
    `order_id` STRING COMMENT '订单编号',
    `user_id` STRING COMMENT '用户编号',
    `payment_type` STRING COMMENT '支付类型',
    `trade_no` STRING COMMENT '交易编号',
    `payment_amount` DECIMAL(16,2) COMMENT '支付金额',
    `subject` STRING COMMENT '交易内容',
    `payment_status` STRING COMMENT '支付状态',
    `create_time` STRING COMMENT '创建时间',
    `callback_time` STRING COMMENT '回调时间'
)  COMMENT '支付流水表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_payment_info/';

#退款表
DROP TABLE IF EXISTS ods_refund_payment;
CREATE EXTERNAL TABLE ods_refund_payment(
    `id` STRING COMMENT '编号',
    `out_trade_no` STRING COMMENT '对外业务编号',
    `order_id` STRING COMMENT '订单编号',
    `sku_id` STRING COMMENT 'SKU编号',
    `payment_type` STRING COMMENT '支付类型',
    `trade_no` STRING COMMENT '交易编号',
    `refund_amount` DECIMAL(16,2) COMMENT '支付金额',
    `subject` STRING COMMENT '交易内容',
    `refund_status` STRING COMMENT '支付状态',
    `create_time` STRING COMMENT '创建时间',
    `callback_time` STRING COMMENT '回调时间'
)  COMMENT '支付流水表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_refund_payment/';

#商品平台属性表
DROP TABLE IF EXISTS ods_sku_attr_value;
CREATE EXTERNAL TABLE ods_sku_attr_value(
    `id` STRING COMMENT '编号',
    `attr_id` STRING COMMENT '平台属性ID',
    `value_id` STRING COMMENT '平台属性值ID',
    `sku_id` STRING COMMENT '商品ID',
    `attr_name` STRING COMMENT '平台属性名称',
    `value_name` STRING COMMENT '平台属性值名称'
) COMMENT 'sku平台属性表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_sku_attr_value/';

#商品（SKU）表
DROP TABLE IF EXISTS ods_sku_info;
CREATE EXTERNAL TABLE ods_sku_info(
    `id` STRING COMMENT 'skuId',
    `spu_id` STRING COMMENT 'spuid',
    `price` DECIMAL(16,2) COMMENT '价格',
    `sku_name` STRING COMMENT '商品名称',
    `sku_desc` STRING COMMENT '商品描述',
    `weight` DECIMAL(16,2) COMMENT '重量',
    `tm_id` STRING COMMENT '品牌id',
    `category3_id` STRING COMMENT '品类id',
    `is_sale` STRING COMMENT '是否在售',
    `create_time` STRING COMMENT '创建时间'
) COMMENT 'SKU商品表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_sku_info/';

#商品销售属性表
DROP TABLE IF EXISTS ods_sku_sale_attr_value;
CREATE EXTERNAL TABLE ods_sku_sale_attr_value(
    `id` STRING COMMENT '编号',
    `sku_id` STRING COMMENT 'sku_id',
    `spu_id` STRING COMMENT 'spu_id',
    `sale_attr_value_id` STRING COMMENT '销售属性值id',
    `sale_attr_id` STRING COMMENT '销售属性id',
    `sale_attr_name` STRING COMMENT '销售属性名称',
    `sale_attr_value_name` STRING COMMENT '销售属性值名称'
) COMMENT 'sku销售属性名称'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_sku_sale_attr_value/';

#商品（SPU）表
DROP TABLE IF EXISTS ods_spu_info;
CREATE EXTERNAL TABLE ods_spu_info(
    `id` STRING COMMENT 'spuid',
    `spu_name` STRING COMMENT 'spu名称',
    `category3_id` STRING COMMENT '品类id',
    `tm_id` STRING COMMENT '品牌id'
) COMMENT 'SPU商品表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_spu_info/';

#用户表
DROP TABLE IF EXISTS ods_user_info;
CREATE EXTERNAL TABLE ods_user_info(
    `id` STRING COMMENT '用户id',
    `login_name` STRING COMMENT '用户名称',
    `nick_name` STRING COMMENT '用户昵称',
    `name` STRING COMMENT '用户姓名',
    `phone_num` STRING COMMENT '手机号码',
    `email` STRING COMMENT '邮箱',
    `user_level` STRING COMMENT '用户等级',
    `birthday` STRING COMMENT '生日',
    `gender` STRING COMMENT '性别',
    `create_time` STRING COMMENT '创建时间',
    `operate_time` STRING COMMENT '操作时间'
) COMMENT '用户表'
PARTITIONED BY (`dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_user_info/';

```

### 加载数据

```
#将2020-06-14加载进入对应分区
load data inpath '/origin_data/gmall/log/topic_log/2020-06-14' into table ods_log partition(dt='2020-06-14');
#创建lzo压缩文件索引，这是干啥的？为每个lzo文件都创建了同名的index文件，加快搜索吗？不，是为了提供可切片特性，mr处理时的split.
hadoop jar /opt/module/hadoop-3.1.3/share/hadoop/common/hadoop-lzo-0.4.20.jar com.hadoop.compression.lzo.DistributedLzoIndexer /warehouse/gmall/ods/ods_log/dt=2020-06-14
```

##### 设置InputFormat

```
#让hive正确加载lzo文件
set hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
```



##### 

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









### metastore服务

>https://www.itcast.cn/news/20190829/12032894477.shtml

##### 内嵌模式

>使用hive自带的Derby数据库存储元数据

#####  本地模式

>配置了mysql，但是依赖于hive的服务无法远程访问metastore

##### **远程模式**

>配置了mysql，可通过hive.metastore.uris远程访问metastore

```
#修改配置文件
<property>
    <name>hive.metastore.uris</name>
    <value>thrift://node-1:9083</value>
</property>
```

```
#启动metastore服务
nohup bin/hive --service metastore >>./metastore.log 2>&1 &
nohup bin/hive --service hiveserver2  >>./hiveserver2.log 2>&1 &
```



### problem

##### Hive not found lzo file codec

```
https://stackoverflow.com/questions/67559129/hive-cannot-find-lzo-codec
已经按照教程将依赖jar包放入hadoop/share/hadoop/common中，也填写了core-site.xml,但是hive执行时找不到这个包，很明显hive没在这个路径上找，这是由于其读取hbase/conf下的core-site.xml，这是一个旧版拷贝，没有配置lzo信息的。
```



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

