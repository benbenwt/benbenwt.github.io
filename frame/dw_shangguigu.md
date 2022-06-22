# 项目

## 组件版本

```
hadoop 3.1.4
hive 3.1.2
hbase-2.3.3
spark 3.1.2-bin-hadoop3.2
flink 1.13.6

zookeeper3.5.7
kafka2.11-2.4.1
sqoop 1.4.6
flume  1.9.0
azkaban 3.84.4
```

## 基本

```
spu，standard product unit，标准化产品单元，描述一个产品的特性，一般是品牌+型号+关键属性确定。
sku,stock keep unit ,库存量单位，不可分割的最小单位，涉及到具体的属性，如颜色、尺寸等。
实时数仓可以快速步骤当前的行为，有时候不需要获得大量的数据，关注当前这一天、一小时的行为，也有很大的价值，越即时越有效。
```

## 目的

```
搭建采集通道，将json格式的用户行为日志文件采集到hdfs上，并进行过滤处理，然后存入hive数据仓库。搭建采集通道，将mysql业务数据采集到hdfs上，并进行过滤处理，然后存入hive数据仓库。搭建hive数据仓库，存储存储用户行为日志和电商平台的业务数据，并合理设计和搭建分层数据仓库，包括ods,dwd,dim,dws,dwt,ads层。实现分析目的，包括活动的分析信息、优惠券的分析信息、订单在省份维度上的分析、订单在spu（商品）上的分析、订单的总体分析、用户的点击路径分析、商品的回购力度分析、用户行为在1、7、30天的分析（）、用户变动信息（回归、离开）、用户停留时间（不同创建日期）、用户1、7、30天总信息分析（下单、上限、）、用户浏览商品信息。并精心设计仓库结构，优化sql和函数，提高处理速度。
```

## 采集

##### flume

###### flume配置

```
a1.source
a1.channel
a1.sink
```

###### flume过滤器

```
时间戳分文件夹放好
```

### sqoop

```
```

## hdfs

>无法使用hivesql、sparksql等完成的功能，用java mapreduce（spark）代码操作hdfs文件。

##### 解析json文件

###### 版本1

>多个json小文件，每个启动一个maptask，效率极慢，可使用combineInputformat，或在文件系统存储时就保存为一个json文件，一行就是一个完整的json。

```
#读取hdfs文件
#解析json文件
#写入hive文件
```





## spark

## 仓库设计

>设计原则，设计技巧
>
>最重要的几个主题：用户，商品，活动，优惠，时间，地区
>
>画一个数据流向图，会比较清除，也就是atlas框架所说的仓库血缘关系，它通过执行的sql解析仓库依赖。
>
>学习一个数仓项目的基本目标：
>
>知道数据采集，数据仓库设计（整体的血缘关系（构思），每层这么设计的原因，每层设计成什么样了，每个表这么设计的原因，每个表设计成什么样了），etl（sql怎么写的，kettle怎么用），superset（分析了哪些指标，使用了什么图形）
>
>数据仓库获取聚合统计的指标，从整体上对用户、订单、活动、优惠券进行了解，如用户增加与流式、活动的收益、订单的分布情况等，更像是给人（上级、企业）的一份报告，不会作为算法的输入。所以说数仓本质上还是在做报表，少数指标需要数仓统计好之后给算法用。但有些指标不需要聚合统计，如单个用户的行为数据，本来就要以个体为单位。
>
>ods原始数据，dim维度表固定信息，dwd维度表建模join多表，粒度为一条记录，dws_daycount以天为粒度，聚合并join成宽表。dwt以dws为基础聚合1、7、30天数据。ads从各层汇总数据用于分析。

|                | **时间** | **用户** | **地区** | **商品** | **优惠券** | **活动** | **度量值**                      |
| -------------- | -------- | -------- | -------- | -------- | ---------- | -------- | ------------------------------- |
| **订单**       | √        | √        | √        |          |            |          | 运费/优惠金额/原始金额/最终金额 |
| **订单详情**   | √        | √        | √        | √        | √          | √        | 件数/优惠金额/原始金额/最终金额 |
| **支付**       | √        | √        | √        |          |            |          | 支付金额                        |
| **加购**       | √        | √        |          | √        |            |          | 件数/金额                       |
| **收藏**       | √        | √        |          | √        |            |          | 次数                            |
| **评价**       | √        | √        |          | √        |            |          | 次数                            |
| **退单**       | √        | √        | √        | √        |            |          | 件数/金额                       |
| **退款**       | √        | √        | √        | √        |            |          | 件数/金额                       |
| **优惠券领用** | √        | √        |          |          | √          |          | 次数                            |

```
业务DWD如下：
可选的业务过程有：收藏，加购，支付，订单，订单详情，退单，退款，评价，优惠券领用
可选的粒度有：一次交易，一天，一个商品，一次领用
可选的维度有：时间，用户，地区，商品，优惠券，活动。
可选的事实度量值：次数，交易金额。
```

##### 血缘关系

###### ads_order_spu_stats

```
     ods_order_detail，ods_order_info，ods_order_detail_activity，ods_order_detail_coupon
     dwd_order_detail，dwd_order_refund_info，dwd_payment_info，dwd_refund_payment
             dws_sku_action_daycount，dim_sku_info   
                   dwt_sku_topic，dim_sku_info
                      ads_order_spu_stats
#订单，订单明细，订单活动关联，订单优惠券关联         
#交易的订单信息，退款订单信息，支付信息，退款顺序
dwd层的工作量很大，需要聚合很多表，整合成单条记录形式，最小粒度。后边的dws，dwt基本基于dwd处理，不会连接太多表。
#sku行为以天为粒度，列和DWS一样的。sku维度表
#sku主题表，下单次数（是否参与活动，是否使用优惠券），下单件数（是否参与活动，是否使用优惠券），下单原始金额（活动优惠金额，优惠券优惠金额），下单最终金额，退款，评价（好评，差评，中评），购物车，收藏，以及1，7，30粒度的统计。sku维度表
dwt这一层太宽了吧，这么多列。
#spu的订单聚合信息，对于指定spu商品，其订单金额，订单数目，最近天数（1，7，30）
```



##### ODS

>使用sqoop和flume采集到hdfs目录，在导入到ods表中，ods表的设计不用维度建模，参考采集的数据即可。
>
>ods是原生的数据，没有所谓的维度。可以通过维度建模的维度来考虑和设计这些表，并进行一定扩展。如维度包括优惠券，订单，商品，可扩展为优惠券信息表、优惠券领用表，订单表、订单明细优惠券关联表，商品表、商品销售属性表（如白色，4英寸等属性）。
>
>时间是常用的列，如创建，使用，取消，支付等

表清单：行为记录表（ods_log），其他业务表（每个对应一个mysql数据库表）

###### ods_log

```
行为日志DWD如下：
CREATE EXTERNAL TABLE ods_log (`line` string)
PARTITIONED BY (`dt` string) -- 按照时间创建分区
STORED AS -- 指定存储方式，读数据采用LzoTextInputFormat；
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_log'  -- 指定数据在hdfs上的存储位置
;
原则：直接一条json进去，没啥好说的。
技巧：使用时间分区，使用lzo文件压缩。
```

###### ods_activity_info

```
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
原则：活动该维度的基本属性，如时间、类型、名称。
技巧：使用时间分区。
```

###### ods_activity_rule

```
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
原则：活动规则的要素有，满减规则（满减件数，满减金额），优惠多少（优惠金额，优惠折扣）。
技巧：较为固定，不使用时间分区。
```

###### ods_base_category1

```
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
原则：品类名称。
技巧：商品上新，变动大数量多，使用时间分区。
```

###### ods_base_category2

###### ods_base_category3

```
这三个ods_base_category类似，连接到上一级ods_base_category即可，只有两个基本属性，id和name。
```

###### ods_base_dic

```
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
原则：编号与编码名称的对应，父编码。
技巧：使用时间分区。
```

###### ods_base_province

```
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
省份信息包括：身份名称、编号、地区编码、地区id、可视化编码
```

###### ods_base_region

```
CREATE EXTERNAL TABLE ods_base_region (
    `id` STRING COMMENT '编号',
    `region_name` STRING COMMENT '地区名称'
)  COMMENT '地区表'
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_base_region/';
地区编号、地区名称
```

###### ods_base_trademark 

```
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
品牌名称，品牌编号
```

###### ods_cart_info

```
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
用户id，价格，sku名称，  创建时间，修改时间，是否下单，下单时间
```

###### ods_comment_info

```
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
编号，用户id，商品sku，商品spu，订单id，   评价，评价时间
```

###### ods_coupon_info

```
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
购物券编号，购物券名称，类型（满减，满件打折，现金，折扣）， 满减数，满减件，折扣，减金额，范围类型（品牌，品类，商品），时间（创建，开始领取，结束领取，修改，过期）
```

###### ods_coupon_use

```
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
编号，商品spu，商品sku，  时间（领取，过期，下单，支付）
```

###### ods_favor_info

```
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
用户id，sku，spu   时间（收藏，取消）
```

###### ods_order_detail

>一项订单记录是一件商品，维护一个用户与多件同类商品的映射。
>
>来源类型是指收藏、购物车之类的？
>
>将此表与订单表联系起来，此表包含了 商品的详细信息，如名称，id，数量，金额，所以称为明细。

```
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
编号，用户id，商品id，商品名称，商品价格，商品数量    时间（创建）、分摊（金额，活动优惠，优惠券优惠）
```

###### ods_order_detail_activity

>对应订单明细中的活动优惠、优惠券优惠
>
>没有使用外键链接，专门创了一个关联表？为什么
>
>活动 与活动规则表（满减、满件、金额、折扣等详细信息）

```
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
订单号、活动那个id、商品id、活动规则id   时间（创建）
```

######  ods_order_detail_coupon

```
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
将订单与优惠券领用表联系起来
```

###### ods_order_info

>订单明细：订单，用户，商品，活动，优惠的关联
>
>订单表：订单（状态（已取消，已购买），金额），用户，商品，活动，支付，物流 （运费，运费减免） 地区（省份ID），    时间（创建，）
>
>这两个表分开有什么意义？

```
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

```

###### ods_order_refund_info

>退单是支付之后发生的行为，未支付就推出是订单表的订单状态为取消。

```
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
订单，金额，商品（件数，id），原因类型  退单状态  时间（退单）
```

###### ods_order_status_log

```
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
订单（id，状态）   时间
```

###### ods_payment_info

```
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
订单（id），业务（id），用户（id），支付（编号，金额，类型，状态），时间（创建，支付）
```

###### ods_refund_payment

```
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
订单（），商品（sku），支付（金额，类型，），时间（创建）
```

######  ods_sku_attr_value

```
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
将商品和属性及属性值连接
```

###### ods_sku_info

>库存量单位
>
>大小关系：类目 >（品牌）品类>  型号（spu） > sku

```
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
sku,spu,价格，商品属性（重量，描述，名称），品牌，品类（如华为有耳机、手机等品类）    时间（创建）
```

###### ods_sku_sale_attr_value

>销售属性是指品类、型号确认后的相关属性，如颜色等，以此确定库存。

```
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
将商品（sku，spu）和销售属性及销售属性值连接
```

###### ods_spu_info

```
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
spuid,品牌，品类
```

###### ods_user_info

```
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
用户的信息（昵称，手机号，姓名，等级，性别）   时间（创建，操作）
```



##### DIM

>描述几个topic的维度表：用户，商品，活动，优惠，时间，地区。和ads基本一致

###### dim_sku_info

```
CREATE EXTERNAL TABLE dim_sku_info (
    `id` STRING COMMENT '商品id',
    `price` DECIMAL(16,2) COMMENT '商品价格',
    `sku_name` STRING COMMENT '商品名称',
    `sku_desc` STRING COMMENT '商品描述',
    `weight` DECIMAL(16,2) COMMENT '重量',
    `is_sale` BOOLEAN COMMENT '是否在售',
    `spu_id` STRING COMMENT 'spu编号',
    `spu_name` STRING COMMENT 'spu名称',
    `category3_id` STRING COMMENT '三级分类id',
    `category3_name` STRING COMMENT '三级分类名称',
    `category2_id` STRING COMMENT '二级分类id',
    `category2_name` STRING COMMENT '二级分类名称',
    `category1_id` STRING COMMENT '一级分类id',
    `category1_name` STRING COMMENT '一级分类名称',
    `tm_id` STRING COMMENT '品牌id',
    `tm_name` STRING COMMENT '品牌名称',
    `sku_attr_values` ARRAY<STRUCT<attr_id:STRING,value_id:STRING,attr_name:STRING,value_name:STRING>> COMMENT '平台属性',
    `sku_sale_attr_values` ARRAY<STRUCT<sale_attr_id:STRING,sale_attr_value_id:STRING,sale_attr_name:STRING,sale_attr_value_name:STRING>> COMMENT '销售属性',
    `create_time` STRING COMMENT '创建时间'
) COMMENT '商品维度表'
PARTITIONED BY (`dt` STRING)
STORED AS PARQUET
LOCATION '/warehouse/gmall/dim/dim_sku_info/'
TBLPROPERTIES ("parquet.compression"="lzo");
品牌，123级分类，（名称，重量，描述是否在售），spu，sku
```

###### dim_coupon_info

```
CREATE EXTERNAL TABLE dim_coupon_info(
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
    `limit_num` BIGINT COMMENT '最多领取次数',
    `taken_count` BIGINT COMMENT '已领取次数',
    `start_time` STRING COMMENT '可以领取的开始日期',
    `end_time` STRING COMMENT '可以领取的结束日期',
    `operate_time` STRING COMMENT '修改时间',
    `expire_time` STRING COMMENT '过期时间'
) COMMENT '优惠券维度表'
PARTITIONED BY (`dt` STRING)
STORED AS PARQUET
LOCATION '/warehouse/gmall/dim/dim_coupon_info/'
TBLPROPERTIES ("parquet.compression"="lzo");

```

###### dim_activity_rule_info

```
CREATE EXTERNAL TABLE dim_activity_rule_info(
    `activity_rule_id` STRING COMMENT '活动规则ID',
    `activity_id` STRING COMMENT '活动ID',
    `activity_name` STRING  COMMENT '活动名称',
    `activity_type` STRING  COMMENT '活动类型',
    `start_time` STRING  COMMENT '开始时间',
    `end_time` STRING  COMMENT '结束时间',
    `create_time` STRING  COMMENT '创建时间',
    `condition_amount` DECIMAL(16,2) COMMENT '满减金额',
    `condition_num` BIGINT COMMENT '满减件数',
    `benefit_amount` DECIMAL(16,2) COMMENT '优惠金额',
    `benefit_discount` DECIMAL(16,2) COMMENT '优惠折扣',
    `benefit_level` STRING COMMENT '优惠级别'
) COMMENT '活动信息表'
PARTITIONED BY (`dt` STRING)
STORED AS PARQUET
LOCATION '/warehouse/gmall/dim/dim_activity_rule_info/'
TBLPROPERTIES ("parquet.compression"="lzo");

```

######  dim_base_province

```
CREATE EXTERNAL TABLE dim_base_province (
    `id` STRING COMMENT 'id',
    `province_name` STRING COMMENT '省市名称',
    `area_code` STRING COMMENT '地区编码',
    `iso_code` STRING COMMENT 'ISO-3166编码，供可视化使用',
    `iso_3166_2` STRING COMMENT 'IOS-3166-2编码，供可视化使用',
    `region_id` STRING COMMENT '地区id',
    `region_name` STRING COMMENT '地区名称'
) COMMENT '地区维度表'
STORED AS PARQUET
LOCATION '/warehouse/gmall/dim/dim_base_province/'
TBLPROPERTIES ("parquet.compression"="lzo");

```

###### dim_date_info

```
CREATE EXTERNAL TABLE dim_date_info(
    `date_id` STRING COMMENT '日',
    `week_id` STRING COMMENT '周ID',
    `week_day` STRING COMMENT '周几',
    `day` STRING COMMENT '每月的第几天',
    `month` STRING COMMENT '第几月',
    `quarter` STRING COMMENT '第几季度',
    `year` STRING COMMENT '年',
    `is_workday` STRING COMMENT '是否是工作日',
    `holiday_id` STRING COMMENT '节假日'
) COMMENT '时间维度表'
STORED AS PARQUET
LOCATION '/warehouse/gmall/dim/dim_date_info/'
TBLPROPERTIES ("parquet.compression"="lzo");

```



##### DWD

>DIM层DWD层需构建维度模型，一般采用星型模型，呈现的状态一般为星座模型。
>
>DWD层使用维度建模，一般按照以下四个步骤：
>
>**选择业务过程→声明粒度→确认维度→确认事实**

```
业务过程：下单业务，支付业务，退款业务，物流业务，一条业务线对应一张事实表，即一张DWD表。
声明粒度：粒度就是行，决定一行代表什么。一般DWD层都是可用的最小粒度，如一次交易，一个商品，一次浏览等。
确认维度：维度就是列，即关心业务过程的哪些特征维度。如下单业务的时间，下单业务的地区，下单业务的用户等。
确认事实：事实就是度量值（次数、个数、件数、金额，可以进行累加），暂时理解为一种特殊的列，也就是特殊的维度。
```

##### DWS

##### DWT

>DWS层和DWT层统称宽表层，这两层的设计思想大致相同。
>
>一般来说DWT中存储的数据的粒度比DWS大，是DWS的汇总数据，如DWS是一天的订单金额，则DWT是一周、一个月的金额。
>
>设计原则：
>
>1.需要建哪些宽表：以维度为基准。（哪些公用的维度，如都关注地区，但是关注）
>
>2.宽表里面的字段：是站在不同维度的角度去看事实表，重点关注事实表聚合后的度量值。



##### ADS

>对电商系统各大主题指标分别进行分析。
>
>最终需要将统计的数据放入mysql、es等，方便其他分析人员使用。

### hivesql脚本

>sql含义，使用的sql技巧，扩展场景

##### ODS

>将hdfs上的origion_data load进入表中，填充ods。

###### ods_log

>-n 如果字符串长度不为0，为真
>
>-z:长度为0，为真
>
>%F表示格式化为年月日
>
>load data inpath "/origin_data/$APP/log/topic_log/$do_date"  into table   gmall.ods_log  partion(dt='$do_date') 载入hive表
>
>hive -e "$sql"  执行sql
>
>hive  -service hiveserver2
>
>hive -service metastore
>
>

```
#!/bin/bash

# 定义变量方便修改
APP=gmall

# 如果是输入的日期按照取输入日期；如果没输入日期取当前时间的前一天
if [ -n "$1" ] ;then
   do_date=$1
else 
   do_date=`date -d "-1 day" +%F`
fi 

echo ================== 日志日期为 $do_date ==================
sql="
load data inpath '/origin_data/$APP/log/topic_log/$do_date' into table ${APP}.ods_log partition(dt='$do_date');
"

hive -e "$sql"

hadoop jar /opt/module/hadoop-3.1.3/share/hadoop/common/hadoop-lzo-0.4.20.jar com.hadoop.compression.lzo.DistributedLzoIndexer /warehouse/$APP/ods/ods_log/dt=$do_date

#执行
hdfs_to_ods_log.sh 2020-06-14
```

###### ods_db

>ods层都是load data语句，没啥好解释的。然后hive -e执行sql语句就行了。

```
#!/bin/bash

APP=gmall

if [ -n "$2" ] ;then
   do_date=$2
else 
   echo "请传入日期参数"
   exit
fi 

ods_order_info=" 
load data inpath '/origin_data/$APP/db/order_info/$do_date' OVERWRITE into table ${APP}.ods_order_info partition(dt='$do_date');"

ods_order_detail="
load data inpath '/origin_data/$APP/db/order_detail/$do_date' OVERWRITE into table ${APP}.ods_order_detail partition(dt='$do_date');"

ods_sku_info="
load data inpath '/origin_data/$APP/db/sku_info/$do_date' OVERWRITE into table ${APP}.ods_sku_info partition(dt='$do_date');"

ods_user_info="
load data inpath '/origin_data/$APP/db/user_info/$do_date' OVERWRITE into table ${APP}.ods_user_info partition(dt='$do_date');"

ods_payment_info="
load data inpath '/origin_data/$APP/db/payment_info/$do_date' OVERWRITE into table ${APP}.ods_payment_info partition(dt='$do_date');"

ods_base_category1="
load data inpath '/origin_data/$APP/db/base_category1/$do_date' OVERWRITE into table ${APP}.ods_base_category1 partition(dt='$do_date');"

ods_base_category2="
load data inpath '/origin_data/$APP/db/base_category2/$do_date' OVERWRITE into table ${APP}.ods_base_category2 partition(dt='$do_date');"

ods_base_category3="
load data inpath '/origin_data/$APP/db/base_category3/$do_date' OVERWRITE into table ${APP}.ods_base_category3 partition(dt='$do_date'); "

ods_base_trademark="
load data inpath '/origin_data/$APP/db/base_trademark/$do_date' OVERWRITE into table ${APP}.ods_base_trademark partition(dt='$do_date'); "

ods_activity_info="
load data inpath '/origin_data/$APP/db/activity_info/$do_date' OVERWRITE into table ${APP}.ods_activity_info partition(dt='$do_date'); "

ods_cart_info="
load data inpath '/origin_data/$APP/db/cart_info/$do_date' OVERWRITE into table ${APP}.ods_cart_info partition(dt='$do_date'); "

ods_comment_info="
load data inpath '/origin_data/$APP/db/comment_info/$do_date' OVERWRITE into table ${APP}.ods_comment_info partition(dt='$do_date'); "

ods_coupon_info="
load data inpath '/origin_data/$APP/db/coupon_info/$do_date' OVERWRITE into table ${APP}.ods_coupon_info partition(dt='$do_date'); "

ods_coupon_use="
load data inpath '/origin_data/$APP/db/coupon_use/$do_date' OVERWRITE into table ${APP}.ods_coupon_use partition(dt='$do_date'); "

ods_favor_info="
load data inpath '/origin_data/$APP/db/favor_info/$do_date' OVERWRITE into table ${APP}.ods_favor_info partition(dt='$do_date'); "

ods_order_refund_info="
load data inpath '/origin_data/$APP/db/order_refund_info/$do_date' OVERWRITE into table ${APP}.ods_order_refund_info partition(dt='$do_date'); "

ods_order_status_log="
load data inpath '/origin_data/$APP/db/order_status_log/$do_date' OVERWRITE into table ${APP}.ods_order_status_log partition(dt='$do_date'); "

ods_spu_info="
load data inpath '/origin_data/$APP/db/spu_info/$do_date' OVERWRITE into table ${APP}.ods_spu_info partition(dt='$do_date'); "

ods_activity_rule="
load data inpath '/origin_data/$APP/db/activity_rule/$do_date' OVERWRITE into table ${APP}.ods_activity_rule partition(dt='$do_date');" 

ods_base_dic="
load data inpath '/origin_data/$APP/db/base_dic/$do_date' OVERWRITE into table ${APP}.ods_base_dic partition(dt='$do_date'); "

ods_order_detail_activity="
load data inpath '/origin_data/$APP/db/order_detail_activity/$do_date' OVERWRITE into table ${APP}.ods_order_detail_activity partition(dt='$do_date'); "

ods_order_detail_coupon="
load data inpath '/origin_data/$APP/db/order_detail_coupon/$do_date' OVERWRITE into table ${APP}.ods_order_detail_coupon partition(dt='$do_date'); "

ods_refund_payment="
load data inpath '/origin_data/$APP/db/refund_payment/$do_date' OVERWRITE into table ${APP}.ods_refund_payment partition(dt='$do_date'); "

ods_sku_attr_value="
load data inpath '/origin_data/$APP/db/sku_attr_value/$do_date' OVERWRITE into table ${APP}.ods_sku_attr_value partition(dt='$do_date'); "

ods_sku_sale_attr_value="
load data inpath '/origin_data/$APP/db/sku_sale_attr_value/$do_date' OVERWRITE into table ${APP}.ods_sku_sale_attr_value partition(dt='$do_date'); "

ods_base_province=" 
load data inpath '/origin_data/$APP/db/base_province/$do_date' OVERWRITE into table ${APP}.ods_base_province;"

ods_base_region="
load data inpath '/origin_data/$APP/db/base_region/$do_date' OVERWRITE into table ${APP}.ods_base_region;"

case $1 in
    "ods_order_info"){
        hive -e "$ods_order_info"
    };;
    "ods_order_detail"){
        hive -e "$ods_order_detail"
    };;
    "ods_sku_info"){
        hive -e "$ods_sku_info"
    };;
    "ods_user_info"){
        hive -e "$ods_user_info"
    };;
    "ods_payment_info"){
        hive -e "$ods_payment_info"
    };;
    "ods_base_category1"){
        hive -e "$ods_base_category1"
    };;
    "ods_base_category2"){
        hive -e "$ods_base_category2"
    };;
    "ods_base_category3"){
        hive -e "$ods_base_category3"
    };;
    "ods_base_trademark"){
        hive -e "$ods_base_trademark"
    };;
    "ods_activity_info"){
        hive -e "$ods_activity_info"
    };;
    "ods_cart_info"){
        hive -e "$ods_cart_info"
    };;
    "ods_comment_info"){
        hive -e "$ods_comment_info"
    };;
    "ods_coupon_info"){
        hive -e "$ods_coupon_info"
    };;
    "ods_coupon_use"){
        hive -e "$ods_coupon_use"
    };;
    "ods_favor_info"){
        hive -e "$ods_favor_info"
    };;
    "ods_order_refund_info"){
        hive -e "$ods_order_refund_info"
    };;
    "ods_order_status_log"){
        hive -e "$ods_order_status_log"
    };;
    "ods_spu_info"){
        hive -e "$ods_spu_info"
    };;
    "ods_activity_rule"){
        hive -e "$ods_activity_rule"
    };;
    "ods_base_dic"){
        hive -e "$ods_base_dic"
    };;
    "ods_order_detail_activity"){
        hive -e "$ods_order_detail_activity"
    };;
    "ods_order_detail_coupon"){
        hive -e "$ods_order_detail_coupon"
    };;
    "ods_refund_payment"){
        hive -e "$ods_refund_payment"
    };;
    "ods_sku_attr_value"){
        hive -e "$ods_sku_attr_value"
    };;
    "ods_sku_sale_attr_value"){
        hive -e "$ods_sku_sale_attr_value"
    };;
    "ods_base_province"){
        hive -e "$ods_base_province"
    };;
    "ods_base_region"){
        hive -e "$ods_base_region"
    };;
    "all"){
        hive -e "$ods_order_info$ods_order_detail$ods_sku_info$ods_user_info$ods_payment_info$ods_base_category1$ods_base_category2$ods_base_category3$ods_base_trademark$ods_activity_info$ods_cart_info$ods_comment_info$ods_coupon_info$ods_coupon_use$ods_favor_info$ods_order_refund_info$ods_order_status_log$ods_spu_info$ods_activity_rule$ods_base_dic$ods_order_detail_activity$ods_order_detail_coupon$ods_refund_payment$ods_sku_attr_value$ods_sku_sale_attr_value$ods_base_province$ods_base_region"
    };;
esac


#执行shell
hdfs_to_ods_db_init.sh all 2020-06-14
```



##### DIM

>DIM层信息是从ODS清洗过来的，但是一般情况下DIM表是手动导入的，他是设计者规定的一些信息。
>
>编写sql，然后case选择，最后hive -e 执行。
>
>基本的清洗语句为：
>
>insert overwrite gmall.dim_user_info partition(dt='9999-99-99')
>
>select id,name  
>
>from gmall.ods_user_info
>
>where dt="";
>
>这是是常用的select、insert组合语句。两表的列名也一样，故不用指定插入哪些列。
>
>
>
>#使用with as临时表语句和left join
>
>with  spu as
>
>()
>
>insert overwrite table gmall.test
>
>select id,name form
>
>gmall.test1
>
>left join  test2
>
>with as创建的临时表可用于后边的select语句，如语句中的sku临时表。
>
>left join语句以sku为基础连接category1、2、3，spu，连接商标，连接sku属性和属性值
>
>
>
>#join（inner join）和left join使用场景
>
>join保存两个表都存在的行，left join以左表为基础，会保留左表所有的行。

```
#!/bin/bash

APP=gmall

if [ -n "$2" ] ;then
   do_date=$2
else 
   echo "请传入日期参数"
   exit
fi 

dim_user_info="
set hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table ${APP}.dim_user_info partition(dt='9999-99-99')
select
    id,
    login_name,
    nick_name,
    md5(name),
    md5(phone_num),
    md5(email),
    user_level,
    birthday,
    gender,
    create_time,
    operate_time,
    '$do_date',
    '9999-99-99'
from ${APP}.ods_user_info
where dt='$do_date';
"

dim_sku_info="
set hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
with
sku as
(
    select
        id,
        price,
        sku_name,
        sku_desc,
        weight,
        is_sale,
        spu_id,
        category3_id,
        tm_id,
        create_time
    from ${APP}.ods_sku_info
    where dt='$do_date'
),
spu as
(
    select
        id,
        spu_name
    from ${APP}.ods_spu_info
    where dt='$do_date'
),
c3 as
(
    select
        id,
        name,
        category2_id
    from ${APP}.ods_base_category3
    where dt='$do_date'
),
c2 as
(
    select
        id,
        name,
        category1_id
    from ${APP}.ods_base_category2
    where dt='$do_date'
),
c1 as
(
    select
        id,
        name
    from ${APP}.ods_base_category1
    where dt='$do_date'
),
tm as
(
    select
        id,
        tm_name
    from ${APP}.ods_base_trademark
    where dt='$do_date'
),
attr as
(
    select
        sku_id,
        collect_set(named_struct('attr_id',attr_id,'value_id',value_id,'attr_name',attr_name,'value_name',value_name)) attrs
    from ${APP}.ods_sku_attr_value
    where dt='$do_date'
    group by sku_id
),
sale_attr as
(
    select
        sku_id,
        collect_set(named_struct('sale_attr_id',sale_attr_id,'sale_attr_value_id',sale_attr_value_id,'sale_attr_name',sale_attr_name,'sale_attr_value_name',sale_attr_value_name)) sale_attrs
    from ${APP}.ods_sku_sale_attr_value
    where dt='$do_date'
    group by sku_id
)

insert overwrite table ${APP}.dim_sku_info partition(dt='$do_date')
select
    sku.id,
    sku.price,
    sku.sku_name,
    sku.sku_desc,
    sku.weight,
    sku.is_sale,
    sku.spu_id,
    spu.spu_name,
    sku.category3_id,
    c3.name,
    c3.category2_id,
    c2.name,
    c2.category1_id,
    c1.name,
    sku.tm_id,
    tm.tm_name,
    attr.attrs,
    sale_attr.sale_attrs,
    sku.create_time
from sku
left join spu on sku.spu_id=spu.id
left join c3 on sku.category3_id=c3.id
left join c2 on c3.category2_id=c2.id
left join c1 on c2.category1_id=c1.id
left join tm on sku.tm_id=tm.id
left join attr on sku.id=attr.sku_id
left join sale_attr on sku.id=sale_attr.sku_id;
"

dim_base_province="
set hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table ${APP}.dim_base_province
select
    bp.id,
    bp.name,
    bp.area_code,
    bp.iso_code,
    bp.iso_3166_2,
    bp.region_id,
    br.region_name
from ${APP}.ods_base_province bp
join ${APP}.ods_base_region br on bp.region_id = br.id;
"

dim_coupon_info="
set hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table ${APP}.dim_coupon_info partition(dt='$do_date')
select
    id,
    coupon_name,
    coupon_type,
    condition_amount,
    condition_num,
    activity_id,
    benefit_amount,
    benefit_discount,
    create_time,
    range_type,
    limit_num,
    taken_count,
    start_time,
    end_time,
    operate_time,
    expire_time
from ${APP}.ods_coupon_info
where dt='$do_date';
"

dim_activity_rule_info="
set hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
insert overwrite table ${APP}.dim_activity_rule_info partition(dt='$do_date')
select
    ar.id,
    ar.activity_id,
    ai.activity_name,
    ar.activity_type,
    ai.start_time,
    ai.end_time,
    ai.create_time,
    ar.condition_amount,
    ar.condition_num,
    ar.benefit_amount,
    ar.benefit_discount,
    ar.benefit_level
from
(
    select
        id,
        activity_id,
        activity_type,
        condition_amount,
        condition_num,
        benefit_amount,
        benefit_discount,
        benefit_level
    from ${APP}.ods_activity_rule
    where dt='$do_date'
)ar
left join
(
    select
        id,
        activity_name,
        start_time,
        end_time,
        create_time
    from ${APP}.ods_activity_info
    where dt='$do_date'
)ai
on ar.activity_id=ai.id;
"

case $1 in
"dim_user_info"){
    hive -e "$dim_user_info"
};;
"dim_sku_info"){
    hive -e "$dim_sku_info"
};;
"dim_base_province"){
    hive -e "$dim_base_province"
};;
"dim_coupon_info"){
    hive -e "$dim_coupon_info"
};;
"dim_activity_rule_info"){
    hive -e "$dim_activity_rule_info"
};;
"all"){
    hive -e "$dim_user_info$dim_sku_info$dim_coupon_info$dim_activity_rule_info$dim_base_province"
};;
esac
#ods_to_dim_db_init.sh all 2020-06-14
```

###### 拉链表

```
为每一条信息记录开始时间，和生效的结束时间。当该条信息被修改时，该条信息作废，将失效时间设为当前时间。并创建新记录，该记录失效时间设置为9999-99-99。
如何实现：每天根据全量表的创建时间和操作时间的到改天的用户变化表，将该表与之前的拉链表合并得到新的拉链表。
为什么有全量表了，还创建拉链表。全量表不保留历史的信息，比如我修改了自己的昵称，那么旧的昵称就丢失了，只知道在该天修改了昵称。如果再次进行修改昵称的操作，前一次的修改时间都丢失了，只知道最后一次操作的时间。而我们需要知道每一天，每个用户是什么状态，又不可能为每天都维护一个表，所以使用拉链表，将用户信息按照修改的不同阶段进行信息保留。
如果要池逊某一天的用户状态，使用此where语句where dt>=2022-0311 and dt <=2022-03-11。
```

###### 拉链表首日装载

```
insert overwrite table dim_user_info partition(dt='9999-99-99')
select
    id,
    login_name,
    nick_name,
    md5(name),
    md5(phone_num),
    md5(email),
    user_level,
    birthday,
    gender,
    create_time,
    operate_time,
    '2020-06-14',
    '9999-99-99'
from ods_user_info
where dt='2020-06-14';
```

###### 挑选出每天的用户变化表并与原来的拉链表拼接

>nvl函数：当参数1为null时，返回参数2.否则直接返回参数1.

```
#挑选出每天的用户变化表
#与原来的拉链表拼接
#修改过期信息的失效日期
with
tmp as
(
    select
        old.id old_id,
        old.login_name old_login_name,
        old.nick_name old_nick_name,
        old.name old_name,
        old.phone_num old_phone_num,
        old.email old_email,
        old.user_level old_user_level,
        old.birthday old_birthday,
        old.gender old_gender,
        old.create_time old_create_time,
        old.operate_time old_operate_time,
        old.start_date old_start_date,
        old.end_date old_end_date,
        new.id new_id,
        new.login_name new_login_name,
        new.nick_name new_nick_name,
        new.name new_name,
        new.phone_num new_phone_num,
        new.email new_email,
        new.user_level new_user_level,
        new.birthday new_birthday,
        new.gender new_gender,
        new.create_time new_create_time,
        new.operate_time new_operate_time,
        new.start_date new_start_date,
        new.end_date new_end_date
    from
    (
        select
            id,
            login_name,
            nick_name,
            name,
            phone_num,
            email,
            user_level,
            birthday,
            gender,
            create_time,
            operate_time,
            start_date,
            end_date
        from dim_user_info
        where dt='9999-99-99'
    )old
    full outer join
    (
        select
            id,
            login_name,
            nick_name,
            md5(name) name,
            md5(phone_num) phone_num,
            md5(email) email,
            user_level,
            birthday,
            gender,
            create_time,
            operate_time,
            '2020-06-15' start_date,
            '9999-99-99' end_date
        from ods_user_info
        where dt='2020-06-15'
    )new
    on old.id=new.id
)
insert overwrite table dim_user_info partition(dt)
select
    nvl(new_id,old_id),
    nvl(new_login_name,old_login_name),
    nvl(new_nick_name,old_nick_name),
    nvl(new_name,old_name),
    nvl(new_phone_num,old_phone_num),
    nvl(new_email,old_email),
    nvl(new_user_level,old_user_level),
    nvl(new_birthday,old_birthday),
    nvl(new_gender,old_gender),
    nvl(new_create_time,old_create_time),
    nvl(new_operate_time,old_operate_time),
    nvl(new_start_date,old_start_date),
    nvl(new_end_date,old_end_date),
    nvl(new_end_date,old_end_date) dt
from tmp
union all
select
    old_id,
    old_login_name,
    old_nick_name,
    old_name,
    old_phone_num,
    old_email,
    old_user_level,
    old_birthday,
    old_gender,
    old_create_time,
    old_operate_time,
    old_start_date,
    cast(date_add('2020-06-15',-1) as string),
    cast(date_add('2020-06-15',-1) as string) dt
from tmp
where new_id is not null and old_id is not null;

```



##### DWD

>日志信息json的结构：公共字段（地区，手机品牌，渠道，是否新增，手机型号，设备id，会员id，app版本），动作数组（目标类型，时间），曝光数组（对象，类型，顺序，位置），页面信息（持续时间，目标id），错误信息
>
>即基本信息（公共字段），进入了什么页面（页面信息），执行了什么操作（操作目标），曝光数组（页面有哪些东西，曝光了什么）
>
>DWD主要分为：启动日志，页面日志，动作日志，曝光日志，错误日志
>
>学习自定义UDF函数如何编写

###### get_josn_object

```
获取json并解析为对象，不可解析复杂json（带数组的）
```

###### DWD sql例子

>get_josn_object解析后过滤有start字段的，并查询出对应日期的所需字段写入DWD即可

```
insert overwrite table dwd_start_log partition(dt='2020-06-14')
select
    get_json_object(line,'$.common.ar'),
    get_json_object(line,'$.common.ba'),
    get_json_object(line,'$.common.ch'),
    get_json_object(line,'$.common.is_new'),
    get_json_object(line,'$.common.md'),
    get_json_object(line,'$.common.mid'),
    get_json_object(line,'$.common.os'),
    get_json_object(line,'$.common.uid'),
    get_json_object(line,'$.common.vc'),
    get_json_object(line,'$.start.entry'),
    get_json_object(line,'$.start.loading_time'),
    get_json_object(line,'$.start.open_ad_id'),
    get_json_object(line,'$.start.open_ad_ms'),
    get_json_object(line,'$.start.open_ad_skip_ms'),
    get_json_object(line,'$.ts')
from ods_log
where dt='2020-06-14'
and get_json_object(line,'$.start') is not null;

```

###### lateral view 

>默认的explode函数是处理map结构的。
>
>lateral view首先为原始表的每行调用UDTF，UTDF会把一行拆分成一或者多行，lateral view再把结果组合，产生一个支持别名表的虚拟表。
>
>如下语句表示将displays拆成多行，并组成一个名为tmp的表，列名为display。

```
lateral view explode_json_array(get_json_object(line,'$.displays')) tmp as display
```

###### dwd_order_detail

>需要连接多个表，如ods_order_detail、ods_order_info、ods_order_detail_activity、ods_order_detail_coupon

```
insert overwrite table dwd_order_detail partition(dt)
select
    od.id,
    od.order_id,
    oi.user_id,
    od.sku_id,
    oi.province_id,
    oda.activity_id,
    oda.activity_rule_id,
    odc.coupon_id,
    od.create_time,
    od.source_type,
    od.source_id,
    od.sku_num,
    od.order_price*od.sku_num,
    od.split_activity_amount,
    od.split_coupon_amount,
    od.split_final_amount,
    date_format(create_time,'yyyy-MM-dd')
from
(
    select
        *
    from ods_order_detail
    where dt='2020-06-14'
)od
left join
(
    select
        id,
        user_id,
        province_id
    from ods_order_info
    where dt='2020-06-14'
)oi
on od.order_id=oi.id
left join
(
    select
        order_detail_id,
        activity_id,
        activity_rule_id
    from ods_order_detail_activity
    where dt='2020-06-14'
)oda
on od.id=oda.order_detail_id
left join
(
    select
        order_detail_id,
        coupon_id
    from ods_order_detail_coupon
    where dt='2020-06-14'
)odc
on od.id=odc.order_detail_id;

```



##### DWS

###### date_format

```
date_format('2020-06-14','yyyy-MM')
```

###### date_add

```
date_add('2020-06-14',-1)
```

###### next_day

```
#取下一个周一
next_day('2020-06-14','MO')
#取当前周的周一
date_add(next_day('2020-06-14','MO'),-7)
```

###### last_day

```
#当月最后一天
select last_day('2020-06-14')
```

###### 复杂数据类型

```
#map结构数据定义
map<string,string>
#array结构数据定义
array<string>
#struct结构数据定义
struct<id:int,name:string,age:int>
#struct和array嵌套定义
array<struct<id:int,name:string,age:int>>
```



##### DWT

###### dwt_visitor_topic

>

```
DROP TABLE IF EXISTS dwt_visitor_topic;
CREATE EXTERNAL TABLE dwt_visitor_topic
(
    `mid_id` STRING COMMENT '设备id',
    `brand` STRING COMMENT '手机品牌',
    `model` STRING COMMENT '手机型号',
    `channel` ARRAY<STRING> COMMENT '渠道',
    `os` ARRAY<STRING> COMMENT '操作系统',
    `area_code` ARRAY<STRING> COMMENT '地区ID',
    `version_code` ARRAY<STRING> COMMENT '应用版本',
    `visit_date_first` STRING  COMMENT '首次访问时间',
    `visit_date_last` STRING  COMMENT '末次访问时间',
    `visit_last_1d_count` BIGINT COMMENT '最近1日访问次数',
    `visit_last_1d_day_count` BIGINT COMMENT '最近1日访问天数',
    `visit_last_7d_count` BIGINT COMMENT '最近7日访问次数',
    `visit_last_7d_day_count` BIGINT COMMENT '最近7日访问天数',
    `visit_last_30d_count` BIGINT COMMENT '最近30日访问次数',
    `visit_last_30d_day_count` BIGINT COMMENT '最近30日访问天数',
    `visit_count` BIGINT COMMENT '累积访问次数',
    `visit_day_count` BIGINT COMMENT '累积访问天数'
) COMMENT '设备主题宽表'
PARTITIONED BY (`dt` STRING)
STORED AS PARQUET
LOCATION '/warehouse/gmall/dwt/dwt_visitor_topic'
TBLPROPERTIES ("parquet.compression"="lzo");

```



##### ADS

>一个topic，一个topic的看吧，设计的时候也是这样，想要什么topic，设计对应的ods，dwd，dws，dwt。
>
>用户，活动，优惠券，订单，商品

###### ads_user_total数据流向



##### azkaban

## ketttle

>基本操作，操作技巧

##### DIM

## superset图表

>活动的分析信息、优惠券的分析信息、订单在省份维度上的分析、订单在spu（商品）上的分析、订单的总体分析、用户的点击路径分析、商品的回购力度分析、用户行为在1、7、30天的分析（）、用户变动信息（回归、离开）、用户停留时间（不同创建日期）、用户1、7、30天总信息分析（下单、上限、）、用户浏览商品信息
>
>superset+presto：https://www.cnblogs.com/luweiseu/p/9493134.html

>superset可以对mysql中的数据进行可视化展示。

### 订单

#### ads_order_total

>订单统计，订单数、订单金额、下单人数

>画图：1.订单数量、订单金额、下单人数 2.1、7、30间隔    3.不同日期

```
SELECT dt AS dt,
       sum(order_count) AS `SUM(order_count)`,
       SUM(order_user_count) AS `SUM(order_user_count)`,
       SUM(order_amount/10000) AS `SUM(order_amount/10000)`
FROM gmall_report.ads_order_total
WHERE dt >= STR_TO_DATE('2020-06-15', '%Y-%m-%d')
  AND dt < STR_TO_DATE('2020-06-19', '%Y-%m-%d')
  AND recent_days = 1
GROUP BY dt
ORDER BY `SUM(order_count)` DESC
LIMIT 50000;
```

#### ads_order_by_province

>统计订单在省份上的分布，包括地区编码、身份名称、国际标准地区编码、订单数、订单金额

```
SELECT iso_code AS iso_code,
       sum(order_count) AS `SUM(order_count)`
FROM ads_order_by_province
WHERE dt >= STR_TO_DATE('2020-06-14', '%Y-%m-%d')
  AND dt < STR_TO_DATE('2020-06-19', '%Y-%m-%d')
GROUP BY iso_code
ORDER BY `SUM(order_count)` DESC
LIMIT 50000;
```

### 用户

#### ads_visit_stats

>访客统计

```
SELECT DATE(dt) AS __timestamp,
       sum(uv_count) AS `SUM(uv_count)`,
       sum(page_count) AS `SUM(page_count)`,
       sum(duration_sec) AS `SUM(duration_sec)`
FROM ads_visit_stats
WHERE dt >= STR_TO_DATE('2020-06-14', '%Y-%m-%d')
  AND dt < STR_TO_DATE('2020-06-19', '%Y-%m-%d')
GROUP BY DATE(dt)
ORDER BY `SUM(uv_count)` DESC
LIMIT 10000;
```

#### ads_user_total

>用户统计，包括新增用户、新增下单用户数、下单用户数、下单金额、活跃用户未下单用户数

```
SELECT DATE(dt) AS __timestamp,
       sum(new_user_count) AS `SUM(new_user_count)`,
       sum(new_order_user_count) AS `SUM(new_order_user_count)`
FROM ads_user_total
WHERE dt >= STR_TO_DATE('2020-06-14', '%Y-%m-%d')
  AND dt < STR_TO_DATE('2022-04-15', '%Y-%m-%d')
  AND recent_days = 1
GROUP BY DATE(dt)
ORDER BY `SUM(new_user_count)` DESC
LIMIT 50000;
```

#### ads_user_change

>用户变动统计，包括流失用户数、回流用户数

```
SELECT DATE(dt) AS __timestamp,
       sum(user_back_count) AS `SUM(user_back_count)`,
       sum(user_churn_count) AS `SUM(user_churn_count)`
FROM ads_user_change
WHERE dt >= STR_TO_DATE('2020-06-13', '%Y-%m-%d')
  AND dt < STR_TO_DATE('2020-06-18', '%Y-%m-%d')
GROUP BY DATE(dt)
ORDER BY `SUM(user_back_count)` DESC
LIMIT 25;
```

#### ads_user_action

>用户行为漏斗分析，浏览首页人数、浏览商品详情页人数、加入购物车人数、下单人数、支付人数

```
SELECT dt AS dt,
       sum(home_count) AS `SUM(home_count)`,
       sum(good_detail_count) AS `SUM(good_detail_count)`,
       sum(cart_count) AS `SUM(cart_count)`,
       sum(order_count) AS `SUM(order_count)`,
       sum(payment_count) AS `SUM(payment_count)`
FROM ads_user_action
WHERE dt >= STR_TO_DATE('2020-06-15', '%Y-%m-%d')
  AND dt < STR_TO_DATE('2020-06-18', '%Y-%m-%d')
GROUP BY dt
ORDER BY `SUM(home_count)` DESC
LIMIT 10000;
```

#### ads_user_retention

>用户留存率分析，包括留存用户数量、新增用户数量、留存率

```
SELECT create_date AS create_date,
       sum(retention_rate) AS `SUM(retention_rate)`,
       sum(retention_day) AS `SUM(retention_day)`
FROM ads_user_retention
GROUP BY create_date
ORDER BY `SUM(retention_rate)` DESC
LIMIT 10000;
```

### 商品

#### ads_repeat_purchase

>品牌复购率，品牌ID、品牌名称、复购率

```
SELECT tm_name AS tm_name,
       sum(order_repeat_rate) AS `SUM(order_repeat_rate)`
FROM ads_repeat_purchase
WHERE dt >= STR_TO_DATE('2020-06-14', '%Y-%m-%d')
  AND dt < STR_TO_DATE('2020-06-15', '%Y-%m-%d')
  AND recent_days = 30
GROUP BY tm_name
ORDER BY `SUM(order_repeat_rate)` DESC
LIMIT 10000;
```

#### ads_order_spu_stats

>商品统计，包括商品ID、商品名称、品牌ID、品牌名称、三级品类ID、三级品类名称、二级品类ID、二级品类名称、一级品类ID、一级品类名称、订单数、订单金额。

```
SELECT tm_name AS tm_name,
       sum(order_count) AS `SUM(order_count)`
FROM ads_order_spu_stats
WHERE dt >= STR_TO_DATE('2020-06-14', '%Y-%m-%d')
  AND dt < STR_TO_DATE('2020-06-15', '%Y-%m-%d')
GROUP BY tm_name
ORDER BY `SUM(order_count)` DESC
LIMIT 10000;
```

### 页面

#### ads_page_path

>页面路径分析

```
#没有合适的路径图
```

### 优惠

#### ads_coupon_stats

>优惠券统计，包括优惠券ID、优惠券名称、开始日期、优惠规则、领用次数、使用次数、使用优惠券订单原始金额、优惠金额、补贴率

```
SELECT coupon_name AS coupon_name,
       sum(order_original_amount) AS `SUM(order_original_amount)`
FROM ads_coupon_stats
GROUP BY coupon_name
ORDER BY `SUM(order_original_amount)` DESC
LIMIT 25;
```

### 活动

#### ads_activity_stats

>活动统计，包括活动ID、活动名称、参与活动订单数、参与活动订单金额、参与活动订单最终金额、补贴率。

```
SELECT activity_name AS activity_name,
       sum(order_original_amount) AS `SUM(order_original_amount)`
FROM ads_activity_stats
GROUP BY activity_name
ORDER BY `SUM(order_original_amount)` DESC
LIMIT 10000;
```

## 采集迁移脚本

##### sqoop

>

##### flume

>

# summary

```
zookeeper,kafka,flume,sqoop,superset
支持组件(zookeeper，kafka)，数据采集迁移框架(flume，logtash，sqoop)，数仓(hadoop,hive)，数据分析转换引擎（etl）(kettle,hivesql，spark,kylin,presto)，可视化框架(superset)、调度组件(azkaban)、sql封装工具（phoenix）
```

# 服务器配置

```
三台机器，磁盘为500G。Intel Xeon 16核，共48个slot。内存为3*16GB。
```

# 服务分布

| 机器       | 服务                                                         | 教程机器 |
| ---------- | ------------------------------------------------------------ | -------- |
| 187,hbase  | hive,mysql_hive,flume_file_kafka，azkaban,kylin,presto,superset | 102      |
| 186,hbase1 | flume_file_kafka                                             | 103      |
| 185,hbase2 | mysql_platform,mysql_lisa,flume_kafka_hdfs                   | 104      |

# 软件版本

![软件版本](../resources/images/image-20220103163534955.png)

##### 

# 数仓4.0总结

>bili资源：数仓4.0
>
>具体统计了哪些主题，哪些指标，又说不出来。
>
>碰到了哪些问题，如何解决的。
>
>为什么这样设计每一层的数仓。
>
>挑选一个sql写出来。

## 整体架构

>采集数据,离线仓库，迁移数据，可视化。
>
>采集数据时使用sqoop从mysq清洗数据到hdfs，使用flume采集用户行为日志到hdfs。
>
>离线数仓使用hive搭建，包括ods、dwd、dim、dws、dwt、ads层。编写清洗脚本，逐层清洗数据。
>
>迁移数据数据使用sqoop重新存储到mysql数据库，用于可视化和业务使用。
>
>可视化使用superset，借助图表展现数仓统计结果。

## 采集

### 日志结构设计

```
登录日志
登录失败日志

#页面数据，事件数据，启动数据和错误数据
```

#### 页面

>页面数据主要是记录一个页面的用户访问情况，包括访问时间，停留时间，页面路径等。
>
>page_id:属于哪一种页面，如首页、商品详情、下单结算等。
>
>sourceType:页面来源类型，商品推广，查询结果，促销活动等
>
>during_time:停留时间
>
>ts:跳入时间
>
>
>
>last_page_id:上页
>
>page_item_type:页面对象类型，如活动、购物券等
>
>page_item:页面对象id

#### 事件

>事件数据主要记录应用内一个具体操作行为，包括操作类型，操作对象，操作对象描述
>
>action_id:动作id，表示具体动作，如添加收藏、取消收藏、添加购物车、删除购物车、领取优惠券
>
>item_type:动作目标类型，sku_id商品，coupon_id购物券。
>
>item:动作目标id
>
>ts：动作时间

#### 曝光

>曝光主要记录页面所曝光的内容，包括曝光对象，曝光类型等信息。
>
>displayType：曝光类型，商品推广、促销活动、查询结果商品
>
>item_type:曝光对象，sku_id商品id，activity_id活动id
>
>item:曝光对象id
>
>order：曝光顺序

#### 启动

>启动信息
>
>entry:启动入口，图标、通知、安装后启动
>
>loading_time:启动加载时间
>
>open_ad_id:开屏广告id
>
>open_ad_ms:广告时间
>
>open_ad_skip_ms:跳过时间
>
>ts:时间

#### 错误

>error_code:错误码
>
>msg:错误信息

#### 数据埋点

>主流埋点方式：代码埋点（前段）、可视化埋点、全埋点三种
>
>当离开页面时，上传所有日志（页面、事件、曝光、错误）

#### 普通日志

>common，action，page，display，error

```
{
  "common": {                  -- 公共信息
    "ar": "230000",              -- 地区编码
    "ba": "iPhone",              -- 手机品牌
    "ch": "Appstore",            -- 渠道
    "is_new": "1",--是否首日使用，首次使用的当日，该字段值为1，过了24:00，该字段置为0。
	"md": "iPhone 8",            -- 手机型号
    "mid": "YXfhjAYH6As2z9Iq", -- 设备id
    "os": "iOS 13.2.9",          -- 操作系统
    "uid": "485",                 -- 会员id
    "vc": "v2.1.134"             -- app版本号
  },
"actions": [                     --动作(事件)  
    {
      "action_id": "favor_add",   --动作id
      "item": "3",                   --目标id
      "item_type": "sku_id",       --目标类型
      "ts": 1585744376605           --动作时间戳
    }
  ],
  "displays": [
    {
      "displayType": "query",        -- 曝光类型
      "item": "3",                     -- 曝光对象id
      "item_type": "sku_id",         -- 曝光对象类型
      "order": 1,                      --出现顺序
      "pos_id": 2                      --曝光位置
    },
    {
      "displayType": "promotion",
      "item": "6",
      "item_type": "sku_id",
      "order": 2, 
      "pos_id": 1
    },
    {
      "displayType": "promotion",
      "item": "9",
      "item_type": "sku_id",
      "order": 3, 
      "pos_id": 3
    },
    {
      "displayType": "recommend",
      "item": "6",
      "item_type": "sku_id",
      "order": 4, 
      "pos_id": 2
    },
    {
      "displayType": "query ",
      "item": "6",
      "item_type": "sku_id",
      "order": 5, 
      "pos_id": 1
    }
  ],
  "page": {                       --页面信息
    "during_time": 7648,        -- 持续时间毫秒
    "item": "3",                  -- 目标id
    "item_type": "sku_id",      -- 目标类型
    "last_page_id": "login",    -- 上页类型
    "page_id": "good_detail",   -- 页面ID
    "sourceType": "promotion"   -- 来源类型
  },
"err":{                     --错误
"error_code": "1234",      --错误码
    "msg": "***********"       --错误信息
},
  "ts": 1585744374423  --跳入时间戳
}

```

#### 启动日志

>common,error,start

```
{
  "common": {
    "ar": "370000",
    "ba": "Honor",
    "ch": "wandoujia",
    "is_new": "1",
    "md": "Honor 20s",
    "mid": "eQF5boERMJFOujcp",
    "os": "Android 11.0",
    "uid": "76",
    "vc": "v2.1.134"
  },
  "start": {   
    "entry": "icon",         --icon手机图标  notice 通知   install 安装后启动
    "loading_time": 18803,  --启动加载时间
    "open_ad_id": 7,        --广告页ID
    "open_ad_ms": 3449,    -- 广告总共播放时间
    "open_ad_skip_ms": 1989   --  用户跳过广告时点
  },
"err":{                     --错误
"error_code": "1234",      --错误码
    "msg": "***********"       --错误信息
},
  "ts": 1585744304000
}

```



### flume用法

##### flume自定义拦截器

>继承Interceptor，重写两个interceptor方法，一个是处理单个Event的，另一个是处理Event列表的，在第二个种调用第一个即可。

##### 碰到的问题

###### hive无法使用load导入hdfs采集的数据

>flume的hdfs sink有三种类型：SequenceFile,Datastream,CompressedStream，对于hdfs sink数据，如果要导入TEXTFILE格式的hive表，flume sink时必须使用Datastream。

###### vim 本质是创建新文件

>使用flume采集时，如果用vim修改文件进行追加，flume会认为其是新文件，将文件的全部内容进行了发送，所以应该使用echo ”hello“ >>2.txt进行测试，模拟追加文件的功能。

##### flume配置文件

```
## 组件
a1.sources=r1
a1.channels=c1
a1.sinks=k1

## source1
a1.sources.r1.type = org.apache.flume.source.kafka.KafkaSource
a1.sources.r1.batchSize = 5000
a1.sources.r1.batchDurationMillis = 2000
a1.sources.r1.kafka.bootstrap.servers = hadoop102:9092,hadoop103:9092,hadoop104:9092
a1.sources.r1.kafka.topics=topic_log
a1.sources.r1.interceptors = i1
a1.sources.r1.interceptors.i1.type = com.atguigu.flume.interceptor.TimeStampInterceptor$Builder

## channel1
a1.channels.c1.type = file
a1.channels.c1.checkpointDir = /opt/module/flume/checkpoint/behavior1
a1.channels.c1.dataDirs = /opt/module/flume/data/behavior1/


## sink1
a1.sinks.k1.type = hdfs
a1.sinks.k1.hdfs.path = /origin_data/gmall/log/topic_log/%Y-%m-%d
a1.sinks.k1.hdfs.filePrefix = log-
a1.sinks.k1.hdfs.round = false

#控制生成的小文件
a1.sinks.k1.hdfs.rollInterval = 10
a1.sinks.k1.hdfs.rollSize = 134217728
a1.sinks.k1.hdfs.rollCount = 0

## 控制输出文件是原生文件。
a1.sinks.k1.hdfs.fileType = CompressedStream
a1.sinks.k1.hdfs.codeC = lzop

## 拼装
a1.sources.r1.channels = c1
a1.sinks.k1.channel= c1
```

### sqoop用法

```
#导入数据到hdfs
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

## hive仓库

>每一层有哪些表
>
>每一个主题有那些表
>
>每个表的内容是什么，粒度是什么

### ODS

>使用sqoop和flume采集到hdfs目录，在导入到ods表中，ods表的设计不用维度建模，参考采集的数据即可。
>
>ods是原生的数据，没有所谓的维度。可以通过维度建模的维度来考虑和设计这些表，并进行一定扩展。如维度包括优惠券，订单，商品，可扩展为优惠券信息表、优惠券领用表，订单表、订单明细优惠券关联表，商品表、商品销售属性表（如白色，4英寸等属性）。
>
>时间是常用的列，如创建，使用，取消，支付等

#### 用户行为日志

##### ods_log

>当日志服务器接收到日志后，将日志输出到本地文件系统的log文件中，然后再使用flume采集到hdfs上。
>
>用于存储行为日志信息，将hdfs上的日志导入该表。
>
>如果需要实时处理，日志服务器还需要将信息发送到kafka队列，可以通过日志服务编程直接发送，或者通过flume采集到kafka，方便后续的实时处理程序读取。

```
#需要指定存储位置、分区、存储格式
#该表存储每日的原始日志数据，以date为分区。
drop table if exists ods_log;
CREATE EXTERNAL TABLE ods_log (`line` string)
PARTITIONED BY (`dt` string) -- 按照时间创建分区
STORED AS -- 指定存储方式，读数据采用LzoTextInputFormat；
  INPUTFORMAT 'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
  OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/warehouse/gmall/ods/ods_log'  -- 指定数据在hdfs上的存储位置
;
```

#### 业务数据

>按照大块划分，主要有购物车、收藏、下单、支付、退单、退款、活动、商品、评论表、优惠券、用户、省份、地区，都是从业务数据库中迁移过来的。
>
>按照天进行分区，每次从业务数据库拉取一天的完整数据。
>
>使用load data in path将hdfs上的数据加载入表中。

##### 同步策略

>适用于什么场景（数据量大小、数据有哪些操作，如增加、旧数据删除，业务过程是否有多个时间节点和过程。），每日处理怎么处理数据

###### 增量同步策略

>每日增量，每天存储一份增量数据，作为一个分区
>
>适用于表的数据量大，而且数据只会有插入的场景，不能进行数据的删除、修改等，对应于事务型事实表。
>
>例如：退单表、订单状态表、订单与活动关联表、商品评论表、支付流水表、订单详情表

###### 全量同步策略

>每日全量，及时每天存储一份完成数据的快照，作为一个分区。
>
>适用于表数据量不大，且每天会有新数据插入，也会有旧数据修改的场景。
>
>例如：加入购物车表、收藏表、商品三级分类、商品二级分类、商品一级分类、优惠券表、活动表、spu表、sku表

###### 新增及变化策略

>每日新增及变化，存储操作时间都是今天的数据
>
>适用于表的数据量大，既有新增，又会有变化。例如用户表、订单表、优惠券领用表

###### 特殊策略

>一些特殊表，不用遵循。例如某些不会变化的表，如地区表、省份表、名族表可以只存一分固定值。

>小型公司，为了方便，一般使用全量数据导入。
>
>中大型公司，由于数量比较大，还是严格按照同步策略导入数据。                 

##### 活动信息表

>记录活动的基本属性，类型、开始结束时间等

```
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
```

##### 活动规则表

>活动规则，记录活动规则的基本信息，如即满减的金额、满减件数、优惠金额、优惠折扣

```
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
```

##### 一级品类表

>记录一级品类的id、name

```
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
```

##### 二级品类

>二级品类id、name、对应的一级品类id

```
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

```

##### 三级品类

>三级品类id、name、二级品类id

```
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
```

##### 编码字典表

>某些变量在多个地方使用，但是其值比较固定，但是随着系统升级和后期变化，可能需要改变。为了方便维护，将这些变量抽离出来作为编码字典。

```
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
```

##### 省份表

>省份基本信息，如编号、省份名称、地区ID、地区编码、ISO-3166编码、ISO-3166-2编码

```
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
```

##### 地区表

>地区名称、编号

```
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
```

##### 品牌表

>编号、品牌名称

```
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
```

##### 购物车表

>编号、用户id、skuid、创建时间、下单时间、修改时间

```
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
```

##### 评论表

>用户id、商品sku、商品spu、订单id、评价id、评价

```
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

```

##### 优惠券信息表

>编号、名称、优惠券类型、满减件数、满减金额、范围类型、范围、优惠金额、优惠折扣、开始结束时间、过期时间、最多领用次数、当前领用次数。

```
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

```

##### 优惠券领用表

>编号、优惠券ID、skuid、spuid、领取时间、使用时间下单、使用时间支付、过期时间、优惠券状态

```
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

```

##### 收藏表

>用户id、skuid、spuid、收藏时间、取消收藏时间

```
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

```

##### 订单表

```
#订单信息，包括订单号、用户id、支付方式、支付流水号、订单原价金额、运费、运费减免、活动减免、优惠券减免
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

```

##### 订单明细表

```
#订单号、商品id、商品名称、商品价格、商品数量、分摊金额、分摊活动优惠、分摊优惠券优惠。
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

```

##### 订单明细活动关联表

```
#订单编号、活动id、订单明细id、活动规则id
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
```

##### 订单明细优惠券关联表

```
#订单id、订单明细id、优惠券id、优惠券领用id、商品id
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
```

##### 退单表

```
#用户id、订单id、退单原因、商品id
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
```

##### 订单状态日志表

```
#订单状态、订单id、修改时间
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

```

##### 支付表

```
#订单id、用户id、支付金额、创建时间、回调时间
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
```

##### 退款表

```
#订单编号、交易编号、支付金额、创建时间、回调时间
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
```

##### 商品平台属性表

```
#sku的平台属性，属性id、属性值id、平台属性名称、平台属性值名称、商品id
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

```

##### 商品表sku

```
#商品名称、skuid、spuid、品牌id、品类id
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

```

##### 用户表

```
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

##### 商品表spu

```
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

```



### DIM

>描述几个topic的维度表：用户，商品，活动，优惠，时间，地区。和ads基本一致

### DWD

>DIM层DWD层需构建维度模型，一般采用星型模型，呈现的状态一般为星座模型。
>
>DWD层使用维度建模，一般按照以下四个步骤：
>
>**选择业务过程→声明粒度→确认维度→确认事实**

```
业务过程：下单业务，支付业务，退款业务，物流业务，一条业务线对应一张事实表，即一张DWD表。
声明粒度：粒度就是行，决定一行代表什么。一般DWD层都是可用的最小粒度，如一次交易，一个商品，一次浏览等。
确认维度：维度就是列，即关心业务过程的哪些特征维度。如下单业务的时间，下单业务的地区，下单业务的用户等。
确认事实：事实就是度量值（次数、个数、件数、金额，可以进行累加），暂时理解为一种特殊的列，也就是特殊的维度。
```

#### DWD层（用户行为日志）

>主要是从ods_log表读出json，然后使用get_json_object获取json字段填入对应表。

##### 启动日志

##### 动作日志

##### 页面日志

##### 曝光日志

##### 错误日志

#### DWD层（业务数据）

>ods层有很多表，这里把ods的多个表关联起来，形成业务过程的宽表，并将基础粒度作为行粒度。

##### 收藏事实表（周期型快照事实表）

##### 加购事实表（周期型快照事实表）

>购物车是一个主题，加购是一个业务过程。
>
>周期型快照事实表，每日快照。以日期为分区，日期分区存储了当日的全量购物车数据。顾名思义，就是每隔一天就把全量的数据进行一次快照。
>
>而事务型事实表每个分区内是该日的增量数据，如该日的订单、该日的评价。而事务型事实表是将完整的数据分散在多个分区内，不会有冗余数据，类似于常用的业务数据库。
>
>累积性快照，就是每隔一段时间将全量的数据进行快照，但是它会累计在
>
>为什么要使用周期型快照：当我们只关心一个周期结束时的数据状态，那么我们可以使用周期型快照。比如，我们只关心一天结束时购物车里有多少商品，方便后续统计。或者，当我们会对已有的数据进行修改时，如从购物车删除商品，那么也是需要使用周期型快照，因为它会获得完整的全量数据，反映出数据的变化。而事务型快照中，如订单、评价，一般是不需要关注修改和删除的，因为订单一旦发生，就需要统计其金额等信息，删除也只是让用户无法查看，统计时仍然需要计算。
>
>为什么使用累积性快照：方便跟踪业务事实的变化，周期型快照只能关注到数据的增删，而累积性可以关注到数据的修改、字段的变更。

##### 订单明细事实表（事务型事实表）

>将ods层的订单明细活动、订单明细优惠券、订单明细表合并为此表

##### 订单事实表（累积型快照事实表）

>之所以叫累积型，是因为该业务过程有多个步骤和时间节点。
>
>比如订单有创建时间、支付时间、完成时间，分别对应着订单的创建、支付、支付完成
>
>当该日产生一个订单时，如果其完成了整个过程，那么就放入该日的对应分区，否则让如9999-99-99分区，表示当前还未完成整个步骤的记录。
>
>在业务数据库中，其具有多个时间字段，一旦其中一个字段满足等于当前日期，表示其今日进行了修改，将数据取出来。如果其满足完成条件，则放入对应分区，否则放入9999-99-99分区表示没有完成。
>
>他的区别就是，有多个时间字段和阶段，没有完成的会被累计在9999分区中，只有完成的才能增量添加到对应分区中。

##### 支付事实表（累积型快照事实表）

##### 退款事实表（累积型快照事实表）

##### 优惠券领用表（累积型快照事实表）

##### 退单事实表（事务型事实表）

##### 评价事实表（事务型事实表）

### DWS

>dws、dwt以主题为基础建表，和ads最终目的一致。
>
>dws是以主题为基础，聚集该主题的所有属性到一个表，如用户的所有行为、商品的所有行为、活动的所有行为。也就是将每个主题的所有相关业务过程聚集到一个表，而dwd是将业务过程拆开的多个表，
>
>如用户主题的行为，下了什么单、支付了什么、退款了什么、评价了什么。

#### 访客主题

>每日设备行为表，将dwd层的页面日志处理后，得到此表

#### 用户主题

>每日用户行为表，将dwd层的订单、支付、退款、退单、评价等表合并到此表。

#### 商品主题

>每日商品行为，将dwd层关于商品的行为，如下单、支付、退款、退单、收藏、评价等属性统计到此表

#### 地区主题

>订单省份分布

#### 优惠券主题

>优惠金额，原始金额，补贴率

#### 活动主题

>优惠金额，原始金额，补贴率

### DWT

>DWS层和DWT层统称宽表层，这两层的设计思想大致相同。
>
>一般来说DWT中存储的数据的粒度比DWS大，是DWS的汇总数据，如DWS是一天的订单金额，则DWT是一周、一个月的金额。
>
>设计原则：
>
>1.需要建哪些宽表：以维度为基准。（哪些公用的维度，如都关注地区，但是关注）
>
>2.宽表里面的字段：是站在不同维度的角度去看事实表，重点关注事实表聚合后的度量值。

>dws、dwt以主题为基础建表，和ads最终目的一致。

### ADS

>对电商系统各大主题指标分别进行分析。
>
>最终需要将统计的数据放入mysql、es等，方便其他分析人员使用。

#### 访客主题

>ods:
>
>dwd:
>
>dws:
>
>dwt:
>
>ads:

#### 用户主题

>用户新增
>
>用户流失
>
>用户回流

#### 商品主题

>品牌复购率

#### 订单主题

>订单省份分布

#### 优惠券主题

>优惠金额，原始金额，补贴率

#### 活动主题

>优惠金额，原始金额，补贴率

## 迁移数据

##### sqoop用法

```
#导出数据到mysql
/bin/sqoop export \
--connect "jdbc:mysql://hadoop102:3306/${mysql_db_name}?useUnicode=true&characterEncoding=utf-8"  \
--username root \
--password 000000 \
--table $1 \
--num-mappers 1 \
--export-dir /warehouse/$hive_db_name/ads/$1 \
--input-fields-terminated-by "\t" \
--update-mode allowinsert \
--update-key $2 \
--input-null-string '\\N'    \
--input-null-non-string '\\N'
}
```

## 可视化

>活动的分析信息、优惠券的分析信息、订单在省份维度上的分析、订单在spu（商品）上的分析、订单的总体分析、用户的点击路径分析、商品的回购力度分析、用户行为在1、7、30天的分析（）、用户变动信息（回归、离开）、用户停留时间（不同创建日期）、用户1、7、30天总信息分析（下单、上限、）、用户浏览商品信息

## 脚本任务调度

>使用azkaban管理清洗数据的脚本，azkaban可以有效管理任务之间的依赖，查看任务执行进度，检查错误等。通过编写basic.flow文件，我们可以管理自己的调度任务。

```
#一个示例
#创建azkaban.project，写入如下内容
azkaban-flow-version: 2.0
#创建basic.flow文件，写入如下内容
nodes:
  - name: jobA
    type: command
    config:
      command: echo "jobA"
  - name: jobB
    type: command
    dependsOn:
     - jobA
    config:
     command: echo "jobB"
将两者打包到同一个zip下，在azkaban创建project时upload即可。
```

## 数据质量管理

>数据质量管理，是指从对数据从计划、获取、存储、共享、维护、应用、消亡生命周期的各个阶段可能引发的各类数据质量问题，进行识别、度量、监控、预警等一系列管理活动，并通过改善和提高组织的管理水平是的数据质量获得进一步提高。
>
>数据质量管理是循环管理过程，其最终目标是通过可靠的数据提升数据在使用中的价值。

### 数据质量评价指标

>数据质量的目标是改善，如何评估改善的结果呢，通常包括以下内容。

| 评价标准 | 描述                                         | 监控项                                            |
| -------- | -------------------------------------------- | ------------------------------------------------- |
| 唯一性   | 指主键保持唯一                               | 字段唯一性检查                                    |
| 完整性   | 主要包括记录缺失和字段值缺失等方面           | 字段枚举值检查     字段记录数检查                 |
| 精确性   | 数据生成的正确性，数据在整个链路流转的正确性 | 波动阈值检查                                      |
| 合法性   | 主要包括格式、类型、阈值的合法性             | 字段日期格式检查      字段长度检查   字段值域检查 |
| 时效性   | 主要包括数据处理的时效性                     | 批处理是否按时完成                                |

### 数据质量管理实操

#### 需求分析

>数仓4.0主要监控以下指标：
>ODS层数据量，每日环比和每周同比变化不能超过一定范围
>
>DIM层不能出现id控制，重复值。
>
>DWD层不能出现id空值，重复值。
>
>id列：重复值、空值，其他相关列：值域检查，行：数据总量同比、环比增长。

| 表                | **检查项目**   | **依据**     | **异常值下限** | **异常值上限** |      |
| ----------------- | -------------- | ------------ | -------------- | -------------- | ---- |
| ods_order_info    | **同比增长**   | 数据总量     | -10%           | 10%            |      |
| ods_order_info    | **环比增长**   | 数据总量     | -10%           | 50%            |      |
| ods_order_info    | **值域检查**   | final_amount | 0              | 100            |      |
| dwd_order_info    | **空值检查**   | id           | 0              | 10             |      |
| dwd_order_info    | **重复值检查** | id           | 0              | 5              |      |
| dwd_order_info    | **空值检查**   | id           | 0              | 10             |      |
| **dim_user_info** | **重复值检查** | id           | 0              | 5              |      |

#### 功能分块

>数据统计模块：使用shell脚本，统计数据仓库中的指标，然后插入mysql数据库
>
>可视化模块：使用superset读取myslq数据库，可视化告警结果。
>
>告警模块：使用python读取mysql的数据，判断是否需要告警，并发送邮件。
>
>调度模块：使用azkaban监控数仓清洗任务状态，如果有新的清洗任务，就执行 数据统计模块  和 告警模块，检测数据质量。

#### 数据统计模块

##### 空id检查脚本

```sh
#!/usr/bin/env bash
# -*- coding: utf-8 -*-
# 检查id空值
# 解析参数
while getopts "t:d:c:s:x:l:" arg; do
  case $arg in
  # 要处理的表名
  t)
    TABLE=$OPTARG
    ;;
  # 日期
  d)
    DT=$OPTARG
    ;;
  # 要计算空值的列名
  c)
    COL=$OPTARG
    ;;
  # 空值指标下限
  s)
    MIN=$OPTARG
    ;;
  # 空值指标上限
  x)
    MAX=$OPTARG
    ;;
  # 告警级别
  l)
    LEVEL=$OPTARG
    ;;
  ?)
    echo "unkonw argument"
    exit 1
    ;;
  esac
done

#如果dt和level没有设置，那么默认值dt是昨天 告警级别是0
[ "$DT" ] || DT=$(date -d '-1 day' +%F)
[ "$LEVEL" ] || LEVEL=0

# 数仓DB名称
HIVE_DB=gmall

# 查询引擎
HIVE_ENGINE=hive

# MySQL相关配置
mysql_user="root"
mysql_passwd="000000"
mysql_host="hadoop102"
mysql_DB="data_supervisor"
mysql_tbl="null_id"

# 认证为hive用户，如在非安全(Hadoop未启用Kerberos认证)环境中，则无需认证
kinit -kt /etc/security/keytab/hive.keytab hive

# 空值个数
RESULT=$($HIVE_ENGINE -e "set hive.cli.print.header=false;select count(1) from $HIVE_DB.$TABLE where dt='$DT' and $COL is null;")

#结果插入MySQL
mysql -h"$mysql_host" -u"$mysql_user" -p"$mysql_passwd" \
  -e"INSERT INTO $mysql_DB.$mysql_tbl VALUES('$DT', '$TABLE', '$COL', $RESULT, $MIN, $MAX, $LEVEL)
ON DUPLICATE KEY UPDATE \`value\`=$RESULT, value_min=$MIN, value_max=$MAX, notification_level=$LEVEL;"

```

##### 重复id检查脚本

```
RESULT=$($HIVE_ENGINE -e "set hive.cli.print.header=false;select count(1) from (select $COL from $HIVE_DB.$TABLE where dt='$DT' group by $COL having count($COL)>1) t1;")

# 将结果插入MySQL
mysql -h"$mysql_host" -u"$mysql_user" -p"$mysql_passwd" \
  -e"INSERT INTO $mysql_DB.$mysql_tbl VALUES('$DT', '$TABLE', '$COL', $RESULT, $MIN, $MAX, $LEVEL)
ON DUPLICATE KEY UPDATE \`value\`=$RESULT, value_min=$MIN, value_max=$MAX, notification_level=$LEVEL;"
```

##### 值域检查脚本

```
# 查询不在规定值域的值的个数
RESULT=$($HIVE_ENGINE -e "set hive.cli.print.header=false;select count(1) from $HIVE_DB.$TABLE where dt='$DT' and $COL not between $RANGE_MIN and $RANGE_MAX;")

# 将结果写入MySQL
mysql -h"$mysql_host" -u"$mysql_user" -p"$mysql_passwd" \
  -e"INSERT INTO $mysql_DB.$mysql_tbl VALUES('$DT', '$TABLE', '$COL', $RESULT, $RANGE_MIN, $RANGE_MAX, $MIN, $MAX, $LEVEL)
ON DUPLICATE KEY UPDATE \`value\`=$RESULT, range_min=$RANGE_MIN, range_max=$RANGE_MAX, value_min=$MIN, value_max=$MAX, notification_level=$LEVEL;"

```

##### 数据量环比检查脚本

```
# 昨日数据量
YESTERDAY=$($HIVE_ENGINE -e "set hive.cli.print.header=false; select count(1) from $HIVE_DB.$TABLE where dt=date_add('$DT',-1);")

# 今日数据量
TODAY=$($HIVE_ENGINE -e "set hive.cli.print.header=false;select count(1) from $HIVE_DB.$TABLE where dt='$DT';")

# 计算环比增长值
if [ "$YESTERDAY" -ne 0 ]; then
  RESULT=$(awk "BEGIN{print ($TODAY-$YESTERDAY)/$YESTERDAY*100}")
else
  RESULT=10000
fi

# 将结果写入MySQL表格
mysql -h"$mysql_host" -u"$mysql_user" -p"$mysql_passwd" \
  -e"INSERT INTO $mysql_DB.$mysql_tbl VALUES('$DT', '$TABLE', $RESULT, $MIN, $MAX, $LEVEL)
ON DUPLICATE KEY UPDATE \`value\`=$RESULT, value_min=$MIN, value_max=$MAX, notification_level=$LEVEL;"

```

##### 数据量同比检查脚本

```
# 上周数据量
LASTWEEK=$($HIVE_ENGINE -e "set hive.cli.print.header=false;select count(1) from $HIVE_DB.$TABLE where dt=date_add('$DT',-7);")

# 本周数据量
THISWEEK=$($HIVE_ENGINE -e "set hive.cli.print.header=false;select count(1) from $HIVE_DB.$TABLE where dt='$DT';")

# 计算增长
if [ $LASTWEEK -ne 0 ]; then
  RESULT=$(awk "BEGIN{print ($THISWEEK-$LASTWEEK)/$LASTWEEK*100}")
else
  RESULT=10000
fi

# 将结果写入MySQL
mysql -h"$mysql_host" -u"$mysql_user" -p"$mysql_passwd" \
  -e"INSERT INTO $mysql_DB.$mysql_tbl VALUES('$DT', '$TABLE', $RESULT, $MIN, $MAX, $LEVEL)
ON DUPLICATE KEY UPDATE \`value\`=$RESULT, value_min=$MIN, value_max=$MAX, notification_level=$LEVEL;"

```

##### ODS层

```
#!/usr/bin/env bash
DT=$1
[ "$DT" ] || DT=$(date -d '-1 day' +%F)

#检查表 ods_order_info 数据量日环比增长
#参数： -t 表名
#      -d 日期
#      -s 环比增长下限
#      -x 环比增长上限
#      -l 告警级别
bash day_on_day.sh -t ods_order_info -d "$DT" -s -10 -x 10 -l 1

#检查表 ods_order_info 数据量周同比增长
#参数： -t 表名
#      -d 日期
#      -s 同比增长下限
#      -x 同比增长上限
#      -l 告警级别
bash week_on_week.sh -t ods_order_info -d "$DT" -s -10 -x 50 -l 1

#检查表 ods_order_info 订单异常值
#参数： -t 表名
#      -d 日期
#      -s 指标下限
#      -x 指标上限
#      -l 告警级别
#      -a 值域下限
#      -b 值域上限
bash range.sh -t ods_order_info -d "$DT" -c final_amount -a 0 -b 100000 -s 0 -x 100 -l 1 

```

##### DWD层

```
#!/usr/bin/env bash
DT=$1
[ "$DT" ] || DT=$(date -d '-1 day' +%F)

# 检查表 dwd_order_info 重复ID
#参数： -t 表名
#      -d 日期
#      -c 检查重复值的列
#      -s 异常指标下限
#      -x 异常指标上限
#      -l 告警级别
bash duplicate.sh -t dwd_order_info -d "$DT" -c id -s 0 -x 5 -l 0

#检查表 dwd_order_info 的空ID
#参数： -t 表名
#      -d 日期
#      -c 检查空值的列
#      -s 异常指标下限
#      -x 异常指标上限
#      -l 告警级别
bash null_id.sh -t dwd_order_info -d "$DT" -c id -s 0 -x 10 -l 0

```

##### DIM层

```
#!/usr/bin/env bash
DT=$1
[ "$DT" ] || DT=$(date -d '-1 day' +%F)

#检查表 dim_user_info 的重复ID
#参数： -t 表名
#      -d 日期
#      -c 检查重复值的列
#      -s 异常指标下限
#      -x 异常指标上限
#      -l 告警级别
bash duplicate.sh -t dim_user_info -d "$DT" -c id -s 0 -x 5 -l 0

#检查表 dim_user_info 的空ID
#参数： -t 表名
#      -d 日期
#      -c 检查空值的列
#      -s 异常指标下限
#      -x 异常指标上限
#      -l 告警级别
bash null_id.sh -t dim_user_info -d "$DT" -c id -s 0 -x 10 -l 0

```

##### 告警模块

>使用python读取数据，判断是否需要告警，发送邮件

# 尚硅谷spark-streaming实时数仓

>尚硅谷SparkStreaming实时数仓。使用canal、maxwell采集mysql数据库的数据到kafka，使用日志服务器后端将采集到的数据落盘并发送到kafka，使用sparkStreaming编写程序进行处理。统计以下几个指标，当日日活数量、当日首单用户数量。

## 日志采集后端

>为了模拟生产中的场景，开启了一个sprinboot后端服务作为日志服务器，负责接收来自前段的日志，并存储到logfile，并发送到对应的kafka的topic，topic包括日志topic，事件topic。

### 日志模拟生成

>这里前端用一个java程序模拟代替，其源源不断的向后端日志服务器发送模拟的数据。发送的速度为每秒100条，因为设置的每条日志发送的延迟为10ms。

```
#使用okhttp将数据发送到日志服务器的后端接口。
#日志格式如下，第一层的key有common和start，start下是启动的信息，common是其他信息，如浏览的品牌等。
#启动信息：loading_time,open_ad_id,open_ad_ms,open_ad_skip_ms
{"common":{"ar":"110000","ba":"Redmi","ch":"oppo","md":"Redmi k30","mid":"mid_168","os":"Android 11.0","uid":"337","vc":"v2.1.132"},"start":{"entry":"notice","loading_time":17874,"open_ad_id":2,"open_ad_ms":2467,"open_ad_skip_ms":0},"ts":1594652955000}
```

### 日志结构设计

```
登录日志
登录失败日志

#页面数据，事件数据，启动数据和错误数据
```

#### 页面

>页面数据主要是记录一个页面的用户访问情况，包括访问时间，停留时间，页面路径等。
>
>page_id:属于哪一种页面，如首页、商品详情、下单结算等。
>
>sourceType:页面来源类型，商品推广，查询结果，促销活动等
>
>during_time:停留时间
>
>ts:跳入时间
>
>
>
>last_page_id:上页
>
>page_item_type:页面对象类型，如活动、购物券等
>
>page_item:页面对象id
>

#### 事件

>事件数据主要记录应用内一个具体操作行为，包括操作类型，操作对象，操作对象描述
>
>action_id:动作id，表示具体动作，如添加收藏、取消收藏、添加购物车、删除购物车、领取优惠券
>
>item_type:动作目标类型，sku_id商品，coupon_id购物券。
>
>item:动作目标id
>
>ts：动作时间

#### 曝光

>曝光主要记录页面所曝光的内容，包括曝光对象，曝光类型等信息。
>
>displayType：曝光类型，商品推广、促销活动、查询结果商品
>
>item_type:曝光对象，sku_id商品id，activity_id活动id
>
>item:曝光对象id
>
>order：曝光顺序

#### 启动

>启动信息
>
>entry:启动入口，图标、通知、安装后启动
>
>loading_time:启动加载时间
>
>open_ad_id:开屏广告id
>
>open_ad_ms:广告时间
>
>open_ad_skip_ms:跳过时间
>
>ts:时间

#### 错误

>error_code:错误码
>
>msg:错误信息

#### 数据埋点

>主流埋点方式：代码埋点（前段）、可视化埋点、全埋点三种
>
>当离开页面时，上传所有日志（页面、事件、曝光、错误）

#### 普通日志

>common，action，page，display，error

```
{
  "common": {                  -- 公共信息
    "ar": "230000",              -- 地区编码
    "ba": "iPhone",              -- 手机品牌
    "ch": "Appstore",            -- 渠道
    "is_new": "1",--是否首日使用，首次使用的当日，该字段值为1，过了24:00，该字段置为0。
	"md": "iPhone 8",            -- 手机型号
    "mid": "YXfhjAYH6As2z9Iq", -- 设备id
    "os": "iOS 13.2.9",          -- 操作系统
    "uid": "485",                 -- 会员id
    "vc": "v2.1.134"             -- app版本号
  },
"actions": [                     --动作(事件)  
    {
      "action_id": "favor_add",   --动作id
      "item": "3",                   --目标id
      "item_type": "sku_id",       --目标类型
      "ts": 1585744376605           --动作时间戳
    }
  ],
  "displays": [
    {
      "displayType": "query",        -- 曝光类型
      "item": "3",                     -- 曝光对象id
      "item_type": "sku_id",         -- 曝光对象类型
      "order": 1,                      --出现顺序
      "pos_id": 2                      --曝光位置
    },
    {
      "displayType": "promotion",
      "item": "6",
      "item_type": "sku_id",
      "order": 2, 
      "pos_id": 1
    },
    {
      "displayType": "promotion",
      "item": "9",
      "item_type": "sku_id",
      "order": 3, 
      "pos_id": 3
    },
    {
      "displayType": "recommend",
      "item": "6",
      "item_type": "sku_id",
      "order": 4, 
      "pos_id": 2
    },
    {
      "displayType": "query ",
      "item": "6",
      "item_type": "sku_id",
      "order": 5, 
      "pos_id": 1
    }
  ],
  "page": {                       --页面信息
    "during_time": 7648,        -- 持续时间毫秒
    "item": "3",                  -- 目标id
    "item_type": "sku_id",      -- 目标类型
    "last_page_id": "login",    -- 上页类型
    "page_id": "good_detail",   -- 页面ID
    "sourceType": "promotion"   -- 来源类型
  },
"err":{                     --错误
"error_code": "1234",      --错误码
    "msg": "***********"       --错误信息
},
  "ts": 1585744374423  --跳入时间戳
}

```

#### 启动日志

>common,error,start

```
{
  "common": {
    "ar": "370000",
    "ba": "Honor",
    "ch": "wandoujia",
    "is_new": "1",
    "md": "Honor 20s",
    "mid": "eQF5boERMJFOujcp",
    "os": "Android 11.0",
    "uid": "76",
    "vc": "v2.1.134"
  },
  "start": {   
    "entry": "icon",         --icon手机图标  notice 通知   install 安装后启动
    "loading_time": 18803,  --启动加载时间
    "open_ad_id": 7,        --广告页ID
    "open_ad_ms": 3449,    -- 广告总共播放时间
    "open_ad_skip_ms": 1989   --  用户跳过广告时点
  },
"err":{                     --错误
"error_code": "1234",      --错误码
    "msg": "***********"       --错误信息
},
  "ts": 1585744304000
}

```



### Logback

>1创建logback.xml，在其中控制日志的输出级别的输出节点root，logger，以及输出源appender。
>
>2为类添加@Slf4j注解

```
<appender name="console" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
    	<pattern>%msg%n</pattern>
    </encoder>
</appender>

<!-- 将某一个包下日志单独打印日志 -->
<logger name="com.atguigu.logger.controller.LoggerController" level="INFO" additivity="false">
    <appender-ref ref="rollingFile" />
    <appender-ref ref="console" />
</logger>

#当没有子节点logger，会调用root
 <root level="error" additivity="false">
 	<appender-ref ref="console" />
 </root>
```

### Kafka发送

>使用springframework提供的spring-kafka链接kafka。
>
>1首先配置好springframework-kafka的pom依赖，然后编写application.properties填写bootstrap-server,key-serializer,value-serializer。
>
>2然后在代码中创建KafkaTemplate对象，使用Autowired注入对象。
>
>3使用：kafkaTemplate.send(topic,jsonObject);

## 业务数据采集

>主要有以下几个表，使用maxwell、canal采集到kafka对应topic，每个表对应一个topic。
>
>这些表按照业务大致划分为：事实表：活动、优惠券、收藏、加入购物车、下单、支付、退款 、  维度表：商品信息、地区信息、品类信息、地区信息、日期信息
>
>安装好canal、maxwell后，配置好mysql地址，并为canal、maxwell创建好mysql账户，给予权限，为两者配置kafka地址，及发送的topic。

### binlog格式

>binlog支持以下格式：
>
>1row格式，一行为单位保存记录，仅仅保存那条记录被修改，可以不记录sql语句的信息。
>
>2statement，将每一条会修改数据的sql记录在binlog中。不需要记录每一行的变化，减少了日志量。还需要记录每条语句执行时候的一些信息，确保在slave节点上执行时候获得相同的结果。
>
>3Mixed格式
>
>该格式是以上两种level的混合使用

### maxwell的格式

```
#指明数据库，表名，data字段中携带表中的数据，old字段中携带旧的字段值，时间戳，xid（对应的sql的id）。
#maxwell以影响的数据为单位发送日志，每一条数据产生一条日志，如果想知道是否来自同一条sql，可以通过xid判断。
{
"database":"gmall-2020-04",
"table":"z_user_info",
"type":"insert",
"ts":1589385314,
"xid":82982,
"xoffset":0,
"data":{"id":30,
"user_name":"zhang3",
"tel":"1381000101
0"
}
}
```

## nginx反代

>使用nginx反向代理三台日志服务器，因为用户的客户端操作日志数据量很大，使用nginx创建upstream，负载均衡三台机器，这样可以向nginx动态的替换机器，在需要重启服务时，可以逐个重启服务，保证upstream中有一个可用的结点。让用户无法感知服务器已经进行了重启和更新。

## 需求1日活统计

>日活统计要求统计当日登录的人数，通过采集的登录日志进行统计。需要借助redis进行去重，比如某个用户进行了多次登录，只能记作一个活跃用户。然后将去重的日活记录插入到ES数据库中，用于可视化和查询。如果不需要使用kibana可以写入MySQL，给业务使用，也可以用sql统计后提供给其他可视化工具，如DataV、QuickBI。这一部分代码在DauApp类中完成，即day active user。

### 读取kafka的启动日志topic

#### 精确一次消费

>精确一次消费可以通过两种形式达到：
>
>1使用事务，让两步操作必须同时失败或成功。
>
>2手动提交+幂等性，在存储好数据后在提交offset，避免offset成功但数据没存储的情况。然后，确保数据存储是幂等性的操作，也就是说offset提交失败时，虽然会重试数据存储的操作，但是无论重试多少次都是结果一样的，会进行覆盖，例如es相同的id会进行覆盖。

>为了达到精确一次消费，需要进行手动提交及读取offset，避免出现offset提交了，但是数据还没有存储的情况。然后借助es的幂等性达到精确一次消费。

>kafka默认5秒提交一次偏移。
>
>enable.auto.commit 的默认值是 true；就是默认采用自动提交的机制。 
>
>auto.commit.interval.ms 的默认值是 5000，单位是毫秒。

```
#首先,从redis取出kafka偏移量
#获取jedis客户端
val prop = MyPropertiesUtil.load("config.properties")
val host = prop.getProperty("redis.host")
val port = prop.getProperty("redis.port")

val jedisPoolConfig: JedisPoolConfig = new JedisPoolConfig
jedisPoolConfig.setMaxTotal(100)  //最大连接数
jedisPoolConfig.setMaxIdle(20)   //最大空闲
jedisPoolConfig.setMinIdle(20)     //最小空闲
jedisPoolConfig.setBlockWhenExhausted(true)  //忙碌时是否等待
jedisPoolConfig.setMaxWaitMillis(5000)//忙碌时等待时长 毫秒
jedisPoolConfig.setTestOnBorrow(true) //每次获得连接的进行测试

jedisPool=new JedisPool(jedisPoolConfig,host,port.toInt)
jedis=jedisPool.getResource
val offsetMap: util.Map[String, String]=jedis.hgetAll(offsetKey)
```

```
#从kafka创建sparkstreaming输入流
kafkaParam("group.id")=groupId
val dStream = KafkaUtils.createDirectStream[String,String](
      ssc,
      LocationStrategies.PreferConsistent,
      //这句话将整个topic,groupid传递进去,也就是所有partition的offset都传递进去了,这表示,consumer(topic,groupid)接受所有传递进去的分区的消息.
      ConsumerStrategies.Subscribe[String,String](Array(topic),kafkaParam,offsets))
      dStream
```

```
#从kafka读取偏移量。
var offsetRanges: Array[OffsetRange] = Array.empty[OffsetRange]

    val offsetDStream: DStream[ConsumerRecord[String, String]] = recordDStream.transform {
      rdd => {
        //因为recodeDStream底层封装的是KafkaRDD，混入了HasOffsetRanges特质，这个特质中提供了可以获取偏移量范围的方法
        offsetRanges = rdd.asInstanceOf[HasOffsetRanges].offsetRanges
        rdd
      }
    }
```

### 借助redis去重

```
#使用sadd函数，如果存在插入元素，返回0，否则返回1并插入。
val isNew:lang.Long=jedisClient.sadd(dauKey,mid)
去重这一部分使用了mapPartitions函数，对于每个分区创建了一次jedisClient，对于分区内每个记录请求了一次redis服务。
#然后得到首次登录的用户列表。
#将首次登录的用户信息封装好之后，插入ES。
#使用了foreachPartition方法，将整个分区的数据封装好后，借助ES bulk insert批量插入。
```

#### foreachPartition mapPartition

>1foreachPartition是action算子，用于数据处理的结尾，获取处理的结果，并将结果插入es，hbase等数据库中，它不会返回一个新的数据流rdd。mapPartition是转换操作，返回一个可以继续操作的rdd

### 写入结果数据

#### 精确一次消费

## 需求2首单分析

>ods，dwd
>
>交易的每一条记录都在mysql中，使用canal、maxwell采集到kafka，然后使用sparkStreaming对数据分流，放入不同的topic。这一部分分流的代码在BaseDBMaxwellApp。
>
>基础的dim维度信息，由dim层的流处理app处理。
>
>使用hbase过滤掉不是首单的用户，如果是首单还需要修改hbase的标记，最终将首单的信息写入数据库。由于是否首单需要统计历史数据，数据量比较大，如果使用redis内存压力比较大，而且是否首单需要保存很久，不是24小时过期的数据，需要修改hbase的标记，需要是key-value模式的查询，综合上边三点，适合保存在hbase中。这一部分代码在orderInfoApp中。
>
>具体流程：读取ods_order_info，从hbase查出所有ods_order_info中存在的user_id信息，然后将ods_order_info中斗胆用户的标记为1，然后对ods_order_info中同一个用户后续提交的标记为0，最终稿将过滤好的首单结果与省份表、用户表拼接得到宽表，将首单信息写入es，将首单标记写入hbase。

```
#用maxWell采集数据到kafka
#取table字段，根据表名称发送到不同的kafka topic，作为ods层
if(
                ("order_info".equals(tableName)&&"insert".equals(opType))
                  || (tableName.equals("order_detail") && "insert".equals(opType))
                  ||  tableName.equals("base_province")
                  ||  tableName.equals("user_info")
                  ||  tableName.equals("sku_info")
                  ||  tableName.equals("base_trademark")
                  ||  tableName.equals("base_category3")
                  ||  tableName.equals("spu_info")
              ){
                //拼接要发送到的主题
                var sendTopic = "ods_" + tableName
                MyKafkaSink.send(sendTopic,dataJsonObj.toString)
              }
              
#从ods_order_info的kafka主题中创建directStream

#借助phoenix从hbase读取首单信息
Class.forName("org.apache.phoenix.jdbc.PhoenixDriver")
    //建立连接
    val conn: Connection = DriverManager.getConnection("jdbc:phoenix:hbase,hbase1,hbase2:2181")
    //创建数据库操作对象
val ps: PreparedStatement = conn.prepareStatement(sql)
    //执行SQL语句
val rs: ResultSet = ps.executeQuery()

#后去es链接
jestFactory = new JestClientFactory
    jestFactory.setHttpClientConfig(new HttpClientConfig
        .Builder("http://172.18.65.187:9200")
        .multiThreaded(true)
        .maxTotalConnection(20)
        .connTimeout(10000)
        .readTimeout(1000).build())
jestFactory.getObject
```

## 需求3实付分摊金额及交易额统计

>购买商品有时会有优惠，但优惠一般是以订单为单位，现在需要以商品为单位，计算每个商品优惠了多少，进一步计算出每个商品的实付金额。对于除法除不尽的情况，不是最后一件商品的分摊金额使用乘除法比例计算，最后一件商品的分摊金额使用减法计算，用实付金额减去前边计算的其他商品的金额得到。

### 双流join

>由于两个流的数据是独立保存的，独立消费，很有可能同一业务的数据，分布在不同的批次。因为join算子只join同一批次的数据，如果只用简单的join流方式，会丢掉不同批次的数据。
>
>可以通过缓存的形式处理这种情况，将没有匹配join成功的数据放入该流的缓存中，当其需要匹配的数据到来时，再进行join。但是其编写逻辑负责。
>
>也可以通过滑动窗口+数据去重，这种方式处理代码简单，但是会造成数据重复，滑动窗口每次会覆盖一些重复数据，如果使用滚动窗口，每次窗口的数据不重复，无法解决join问题。

### Elasticsearch可视化

>添加indexpattern，添加dashboard

## 需求4ADS聚合及可视化

>将dws的数据进行聚合，插入mysql。编写发布接口，让阿里云DataV使用发布的接口。

## Exactly once

>为了实现精确一次消费，有两种方案。1.使用具有幂等性的操作，如es，redis等，在数据操作完成后，再提交offset。2使用mysql事务，将数据操作和索引操作进行绑定，达到原子性事务。

## kafka分层

>对输入的不同类别的原始日志进行处理

## ods

>将maxwell采集到kafka的数据取出，再根据表名称写入对应的ods消息队列。

## dwd

>读取对应的ods消息队列，将订单、订单明细写入dwd消息队列

## dws

>拼接维度形成宽表

## ads

>将数据从dws取出并处理，结果写入mysql、es等。

## 使用的DStream算子

>按照无状态，有状态，窗口，输出区分。

### 无状态

```
transform
mapPartitions
map
join
```

### 窗口

```
window(windowDuration,slideDuration)
```

### 输出DStream

```
foreachRDD
```

# 拆分可视化服务

>为了方便展示项目效果，将可视化的模块与庞大的数仓集群独立出来，只保留关键的ads数据库、后端、前段可视化展示。包括如下服务：es，mysql，clickhouse，superset，springboot，echarts。为了方便管理这些零散的服务，使用docker-compose将其制作成镜像，方便展示。
>
>可能问题：1集群和项目的服务是在服务器上，只是将清洗结果和可视化服务迁移到本机。
>2实验室没有其他项目嘛，有一些小项目，与当前职位关系很小。

>mysql的数据用navicate导出为sql，再导入新的数据库
>
>elasticsearch借助logstash导入到新的es数据库。

## docker-compose

```
version: "2"
services:
  mysql:
    image: vs-mysql
    build:
      context: ./docker/mysql
      dockerfile: ./Dockerfile
    environment:
      - MYSQL_ROOT_PASSWORD=root
      - MYSQL_USER=vsserver
      - MYSQL_PASSWORD=vsserver
      - MYSQL_DATABASE=gmall_report
    volumes:
      - "./data/mysqldb:/var/lib/mysql"
    ports:
      - 3307:3306
    networks:
      vs_server_net:
        ipv4_address: 172.42.1.10
  superset:
    image: vs-superset
    build:
      context: ./docker/superset
      dockerfile: ./Dockerfile
    ports:
      - 8787:8787
    networks:
      vs_server_net:
        ipv4_address: 172.42.1.11
  elasticsearch:
    image: vs-elasticsearch
    build:
      context: ./docker/elasticsearch
      dockerfile: ./Dockerfile
    deploy:
      resources:
        limits:
          memory: 4096M
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - "./data/elasticsearch:/usr/share/elasticsearch/data"
 #     - "./data/kibana:/usr/local/kibana-7.11.1-linux-x86_64/"
    ports:
      - 9201:9200
      - 5602:5601
    networks:
      vs_server_net:
        ipv4_address: 172.42.1.12
networks:
  vs_server_net:
    ipam:
      driver: default
      config:
        - subnet: 172.42.1.0/24
```

## Dockerfile

### mysql

>platform.sql中是建表语句

```
FROM mariadb:latest
USER root
COPY ./platform.sql /docker-entrypoint-initdb.d
RUN chown -R mysql:mysql /docker-entrypoint-initdb.d/
```

### superset

>如下的dockerfile主要是为了安装python的编译依赖，将python和pip编译并安装好。然后使用pip安装superset及其依赖。

```
FROM centos:7
ENV FLASK_APP=superset
RUN useradd -m platform \
&& yum install -y gcc gcc-c++ libffi-devel \
MySQL-python mysql-devel python-devel \
python-devel python-pip python-wheel python-setuptools\
 openssl-devel cyrus-sasl-devel openldap-devel bzip2-devel expat-devel\
gdbm-devel readline-devel sqlite-devel automake  autoconf libtool make \
libffi-devel wget
RUN cd /usr/local/ \
&& wget https://www.python.org/ftp/python/3.7.0/Python-3.7.0.tgz\
&&tar -zxvf Python-3.7.0.tgz \
&& cd /usr/local/Python-3.7.0/ \
&& ./configure --prefix=/usr/local/python3\
&& make && make install\
&&mv /usr/bin/python /usr/bin/python_bak\
&&ln -s /usr/local/python3/bin/python3 /usr/bin/python\
&& cd /usr/local\
&& wget https://bootstrap.pypa.io/get-pip.py\
&&  python get-pip.py\
&&ln -s /usr/local/python3/bin/pip3 /usr/bin/pip
Run  pip install --upgrade setuptools pip -i https://pypi.douban.com/simple/ 
Run pip install apache-superset==0.36.0  -i https://pypi.douban.com/simple/ \
markupsafe==2.0.1 -i https://pypi.douban.com/simple/ \
WTForms==2.3.3 \
sqlalchemy==1.3.24  \
pymysql
#Run superset db upgrade \
#&&superset fab create-admin \
#&&superset init \
#&&pip install gunicorn -i https://pypi.douban.com/simple/ \
#&&gunicorn --workers 5 --timeout 120 --bind hadoop102:8787  "superset.app:create_app()" --daemon 
#COPY   --chown=platform:platform  ./superset /home/platform/
COPY   --chown=platform:platform  ./entrypoint.sh /home/platform/
#CMD ["python3","-m","http.server","8888"]
USER root
CMD ["sh","/home/platform/entrypoint.sh"]
```

```
#第一次启动还要进行如下操作
#关于为什么可以直接使用superset命令，因为它将site-packages/superset/bin/superset软连接到了/usr/bin或添加了环境变量。有时候不是放在对应安装包的bin目录下，而是在python环境的目录，gunicorn就是这样，如conda/env/my_python/bin/gunicorn
#superset需要创建数据库，指定登陆web界面的用户和密码，及后续的gunicorn这一部分需要如下手动完成。
先使用ln -s将superset、gunicorn的可执行文件链接到/usr/bin
superset db upgrade
export FLASK_APP=superset
superset fab create-admin
superset init
#配置前端页面
pip install gunicorn -i https://pypi.douban.com/simple/
gunicorn --workers 5 --timeout 120 --bind hadoop102:8787  "superset.app:create_app()"
```

### elasticsearch kibana

>将es拷贝进去，修改配置文件，启动
>
>导入数据

```
FROM centos:7

ADD ./jdk-11.0.10_linux-x64_bin.tar.gz /usr/local/java/
ENV JAVA_HOME /usr/local/java/jdk-11.0.10
ENV PATH $JAVA_HOME/bin:$PATH
RUN useradd -m elasticsearch
ADD ./kibana-7.11.1-linux-x86_64.tar.gz  /usr/local
COPY --chown=elasticsearch:elasticsearch ./elasticsearch /usr/share/elasticsearch
#COPY  --chown=elasticsearch:elasticsearch  ./elasticsearch.yml /usr/share/elasticsearch/config/elasticsearch.yml
COPY --chown=elasticsearch:elasticsearch ./kibana.yml /usr/local/kibana-7.11.1-linux-x86_64/config
COPY  --chown=elasticsearch:elasticsearch  ./entrypoint.sh /usr/local
#USER root
#ENTRYPOINT ["/usr/local/entrypoint.sh"]
#CMD ["/usr/local/kibana-7.11.1-linux-x86_64/bin/kibana ","--allow-root"]
USER elasticsearch
ENTRYPOINT  ["/usr/local/entrypoint.sh"]
```

>entrypoint.sh启动脚本

```
#!/bin/bash
/usr/share/elasticsearch/bin/elasticsearch -d
/usr/local/kibana-7.11.1-linux-x86_64/bin/kibana
```

## 迁移数据

>mysql中的数据分为两块，一块是gmall_report，是数据仓库的的统计结果。
>
>一块是data_supervisor,是数据质量控制的统计结果。这两块借助navicate的导出导入功能迁移数据，然后手动填充一些数据，让superset的图形效果好一点。
>
>elasticsearch主要是spakrstreaming的统计结果，使用logstash将数据从集群采集到docker中的elasticsearch中。

## 花生壳映射

```
elasticsearch的映射地址：
1.spark实时数仓可视化，http://49144m9k60.zicp.vip:52494/goto/97163d82d55cb968ef79599d864032c2
2.flink实时推荐： http://49144m9k60.zicp.vip:52494/goto/c25c8c3593c6ed62582aae6ee79da8a0
superset的映射地址：
1.hive数据仓库统计结果，http://49144m9k60.zicp.vip/r/5    
2.数据质量控制统计结果， http://49144m9k60.zicp.vip/r/7  
账号：visitor visitor
```

# 项目困难

## 离线数仓

### 数据倾斜

>1找出倾斜点，单独处理该任务再合并
>
>2试图分散其数据

### 日期间隔连续问题

>借助rank函数获取排名，然后相减获得分组标识。
>
>或者借助lag函数，判断是否是新的一组，达到分组的目的。

## 实时数仓

### exactly-once 精确一次消费

>输入端精确一次消费
>
>处理程序具有恢复状态功能
>
>输出端满足幂等或事务
