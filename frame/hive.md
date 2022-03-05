

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
create function test_explode_json_array_from_hdfs1 as 'com.atguigu.gmall.hive.udtf.ExplodeJSONArray' using jar 'hdfs://hbase:8020/user/hive/jars/gmall-udtf-1.0-SNAPSHOT.jar';
```

##### 删除函数

```
show functions like '*month*'
desc function add_months;
desc function extended add_months;
drop [temporary] function [if exists] [dbname.]function_name;
```



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

### lzo压缩

>https://cwiki.apache.org/confluence/display/Hive/LanguageManual+LZO

```
#创建lzo索引
hadoop jar /path/to/jar/hadoop-lzo-cdh4-0.4.15-gplextras.jar com.hadoop.compression.lzo.LzoIndexer /path/to/HDFS/dir/ contains/lzo/files
```



### 分区

>https://zhuanlan.zhihu.com/p/65442409

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





### 外部表

>https://blog.csdn.net/qq_36743482/article/details/78393678
>
>使用external修饰的即为外部表，内部表数据由hive自身管理，外部表数据由hdfs管理。
>
>内部表数据存储的位置是hive.metastore.warehouse.dir。外部表数据的存储位置由自己制定（如果没有LOCATION，Hive将在HDFS上的/user/hive/warehouse文件夹下以外部表的表名创建一个文件夹，并将属于这个表的数据存放在这里）；
>
>删除内部表会直接删除元数据（metadata）及存储数据；删除外部表仅仅会删除元数据，HDFS上的文件并不会被删除；
>
>对内部表的修改会将修改直接同步给元数据，而对外部表的表结构和分区进行修改，则需要修复（MSCK REPAIR TABLE table_name;）

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

# 理论知识

### hive 环境变量

```
如何读取的hbase/conf/core-site.xml
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

