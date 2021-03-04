服务器密码:240711.wt

hive和hbase表：

单个信息，用来检索，统计；md5,SHA256,sha1,size,architecture,language,endianess,type,              		sampletime,ip,url,cveid,location,identity,      hdfs				

其他:进程，字符，注册表，filepath

,identity,,

hbase

```
 #malware
 list_namespace
 list_namespace_tables 'platform'
 create_namespace 'platform'
create 'platform:sample','baseinfo','moreinfo','store'
```

```
#cve    id,

```



hive

```
create table sample(md5 string,SHA256 string,sha1 string,size string,architecture string,endianess string,type string,sampletime string,ip string,url string,cveid string,location string,identity string,hdfs string);
```



### 数据

lisa生成,各种格式ti。

stix原始数据

	待统计数据,单个样本

##### 				          nginx前端个体样本数据及原始数据

##### 		  搜索页面

				mysql统计数据

##### 								                         spring统计模块

								nginx前端统计页面

到网站或自己的库查询ti

## 基本

Springcloud搭建

前端页面搭在nginx上，使用echart。Bootstrap。

请求consumer，consumer会依据eureka名称请求对应的客户端。

数据库用mongodb，对于·统计过慢，用redis缓存。

 

Docker，使用dockerfile搭建服务环境环境，挂载目录上传html和jar包。

### 数据库

Springcloud搭建

前端页面搭在nginx上，使用echart。Bootstrap。

请求consumer，consumer会依据eureka名称请求对应的客户端。

数据库用mysql，统计后的关系型数据放到mysql。 

Docker，使用dockerfile搭建服务环境环境，挂载目录上传html和jar包。

 

计算统计结果，展示统计信息后台，crudstix文件，nginx前端，数据库。

category value  percent time 

family value

 

CREATE TABLE `category_tbl`

(

    `category_id` INT UNSIGNED AUTO_INCREMENT,
    
    `category` VARCHAR(20) NOT NULL,
    
    `value`  INT NOT NULL,
    
    `percent` INT UNSIGNED NOT NULL,
    
    `time` DATE，//截至time日累积的数目。可得月周。
    
    PRIMARY KEY(`category_id`)

)ENGINE=InnoDB DEFAULT CHARSET=utf8;

柱状图当前统计=源数据截至当前日每个类数量，样本时间<当前时间。

折线：源数据截至某日累积样本数，样本时间<n日,n=1……。每周和每月数据取该段时间最大日期即可。某日月周没有，即取前一周月日。新增即是相减。

单个样本：该样本所处时间点。源数据，按某策略更新mysql统计数据即可。前端要展示，不存mysql里。

CREATE TABLE `family_tbl`

(

    `family_id` INT UNSIGNED AUTO_INCREMENT,
    
    `family` VARCHAR(20) NOT NULL,
    
    `value` INT NOT NULL,
    
    PRIMARY KEY(`family_id`)

)ENGINE=InnoDB DEFAULT CHARSET=utf8;

 

mysql事件删除过期数据。日统计信息。后端要提供单个样本具体信息，

 

最新日期，每类一个：

SELECT category_id categoryId,category,value,percent,time FROM `category_tbl` WHERE category_id IN(SELECT     SUBSTRING_INDEX(GROUP_CONCAT(category_id ORDER BY `time` DESC),',',1) FROM `category_tbl` GROUP BY category )

由于此数据保证每天都存储，即每类的最新数据就是MAX(time)，所以不用concat。

SELECT category_id categoryId,category,value,percent,time FROM `category_tbl` WHERE time=(SE

最新的前10天，占比最高的前两类。

最新的前10天，占比最高的前两类。特殊情况，每天都记录了。

SELECT category_id categoryId,category,value,percent,time FROM category_tbl a WHERE a.time IN 

(SELECT time FROM(SELECT DISTINCT b.time FROM `category_tbl` b ORDER BY b.time DESC LIMIT 10)AS times) AND a.category IN

(SELECT * FROM (SELECT category FROM category_tbl GROUP BY category ORDER BY count(category) DESC LIMIT 2)AS category_list) ORDER BY category,time

最新的前10周

最新的前10周，特殊情况，每天都记录了。

 

特例前10个月。

特例前10个月，特殊情况，每天都记录了。

源数据，stix2文件。待统计数据：单个结构化的样本，包含发生时间等属性。统计数据：截至某日，共发生各种类多少次等。



##### 版本

java8,hadoop2.10.x,3.1.1+,3.2.x,hbase2.3.x，hive

java8,hadoop3.1.4,hive3.1.2,hbase2.3.3,mysql5.7.28,mysql-connector-5.1.37