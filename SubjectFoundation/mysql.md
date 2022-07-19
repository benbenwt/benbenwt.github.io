```
mysql有快照嘛
```



### DDL
constraint用于防止非法信息进入
#### 索引
#### 表结构

##### 指定外键

```
CREATE TABLE Orders
(
Id_O int NOT NULL PRIMARY KEY,
OrderNo int NOT NULL,
Id_P int FOREIGN KEY REFERENCES Persons(Id_P)
)
```



### DML
插入
#### 常用函数
##### 日期函数
```sql
date_format(date_var,"%Y-%m-%d %H:%m:%s")
str_to_date("2022-02-02","%Y-%m-%d %H:%m:%s")
```
##### 数学运算
##### 逻辑运算

```
isnull
```
### DCL


### platform相关sql

##### 时区问题

>utc世界协调时间，比北京时间
>
>CST的含义比较多，有很多歧义，可代表中国标准时间，美国时间等。

```
show variables like "%time_zone"
set global time_zone = '+8:00';
#永久修改
vim /etc/my.conf
```

```
#查看数据库和系统时间
select now();
select sysdate();
```





##### 从lisadb删除id_重复的

>更改SUBSTRING_INDEX的第三个参数(1,2,3,4,5,6,7)，迭代完成任务
>
>

```
delete from celery_taskmeta where id in  
(select  SUBSTRING_INDEX(SUBSTRING_INDEX(ids,',',1),',',-1) AS id from
(select SUBSTRING_INDEX(id,',',-(num-1)) AS ids,num-1 as newnum from 
 (select id,num from
	(select GROUP_CONCAT(id) as id,id_ as id_,count(id_) as num from celery_taskmeta   where status='SUCCESS' group by id_   order by count(id_) desc) t1 where num >1) t2) t3 )
```

##### id_重复的需要删除的有多少条

```
(select  SUBSTRING_INDEX(SUBSTRING_INDEX(ids,',',1),',',-1) AS id,sum(newnum) from
(select SUBSTRING_INDEX(id,',',-(num-1)) AS ids,num-1 as newnum from 
 (select id,num from
	(select GROUP_CONCAT(id) as id,id_ as id_,count(id_) as num from celery_taskmeta   where status='SUCCESS' group by id_   order by count(id_) desc) t1 where num >1) t2) t3 )
```





### shell命令

```
#连接mysql,-p与密码之间不要加空格，加空格后无法识别正确的密码。
mysql -h localhost -u root -p'root'
#查看用户和允许的连接host
use mysql
select user, host from user;
update user set host="%" where user="root";
 flush privileges;
#执行mysql文件
source create_table.sql
```

### 批量PrepareStatement插入

>时间花费=网络传输时间（传输的数据量，建立的连接次数）+数据库向应时间（索引结构、存储原理）

```
#批量insert
for (int k = 1; k <= 10000; k++) {
					pst.setLong(1, k * i);
					pst.setLong(2, k * i);
					pst.addBatch();
				}
pst.executeBatch();
#prepareStatement使用batch插入15000条要200s。添加后两个参数，速度直接到达一秒。
conn = DriverManager.getConnection("jdbc:mysql://hbase2:3306/platform?useUnicode=true&characterEncoding=utf-8&useSSL=true&serverTimezone=UTC&rewriteBatchedStatements=true&useServerPrepStmts=true","root","root");
#使用这两个参数后，只传递一条预编译语句，一次请求将数据放在一起传递过去。减少了传递的预编译语句数量，以及创建连接的次数。
```

### 重设自增id

```
清空自增id
alter table 表名 auto_increment=数字
```



```
#pcap表
CREATE TABLE pcap(
id int(11) NOT NULL AUTO_INCREMENT,
filename VARCHAR(500) NOT NULL,
task_id VARCHAR(500) NOT NULL,
date_done  datetime,
md5 VARCHAR(500) NOT NULL,
PRIMARY KEY(id)
)ENGINE=InnoDB DEFAULT CHARSET=utf8;
```



```
# which mysqld
/usr/local/mysql/bin/mysqld
#显示从何处加载my.cnf文件
/usr/local/mysql/bin/mysqld --verbose --help |grep -A 1 'Default options'
#在[mysqld]后加上skip-grant-tables，若没有[mysqld],自己添加一个。
#充值密码
set password for 'root'@'localhost'=password('123456');
```



```
mysql -uroot -proot
#指定端口按照此顺序指定参数。
mysql -P3307 -uemove -h180.89.32.56 -p
xs.glgoo.net
```

```
SELECT NOW();
SELECT CURTIME();
SHOW VARIABLES LIKE "%time_zone%";
SET GLOBAL time_zone = 'SYSTEM';
SET time_zone = 'SYSTEM';
FLUSH PRIVILEGES;
#根据存入数据，设置正确的时区。若显示时需要转换，可以通过jackson转换时区，jackson也可以控制映射的日期格式。
```



### problem

多个sql不可以公用statement

### 数据表

##### 创建数据库

CREATE DATABASE `cloud_db_one` CHARACTER SET utf8 COLLATE utf8_bin;

##### 创建数据表

CREATE TABLE `dept`(
`dept_no` int(11) NOT NULL AUTO_INCREMENT,
`dept_name` varchar(500) DEFAULT NULL,
`db_source` varchar(500) DEFAULT NULL,
PRIMARY KEY(`dept_no`)
)ENGINE=InnoDB DEFAULT CHARSET=utf8;

# 理论部分

## 存储引擎

### InnoDB

>InnoDB支持B+树索引、全文索引、哈希索引
>
>哈希索引是自适应的，不能人为干预，InnoDB会根据这张表的使用情况自动生成。每张表只能有一个全文检索的索引。B+树索引是传统意义上的索引，它不能根据键值找到具体的行数据，而是找到行数据所在的页，然后把页读取到内存查找行数据。

#### B+树

>B+树是一种平衡查找树
>
>优势：
>
>1单一节点存储多个元素，减少IO
>
>2所有查询都要找到叶子结点，查询效率稳定。
>
>3所有叶子结点形成有序链表，方便范围查询。
>
>一般b+树的层数维持在2-4层。

## mysql事务和锁

>

## mysq数据安全管理

>数据库是企业的核心资产，如果发生意外的损坏，如删除、更新会导致很大的灾难，可通过以下方式减小损失和发生的可能性。
>
>1在资源允许的情况下，定期备份重要数据库的信息。
>
>2开启sql_safe_update模式，此模式可避免忘记写where的update和delete操作。
>
>3数据库用户权限管理，只给需要查询的用户对应权限账户，update和delete权限要慎重分配，这样也可以防止部分攻击行为。

## Java MySQL数据类型对照

| **类型名称**  | **显示长度** | **数据库类型**            | **JAVA类型**             | **JDBC类型索引(int)** | **描述** |
| ------------- | ------------ | ------------------------- | ------------------------ | --------------------- | -------- |
|               |              |                           |                          |                       |          |
| **VARCHAR**   | **L+N**      | **VARCHAR**               | **java.lang.String**     | **12**                |          |
| **CHAR**      | **N**        | **CHAR**                  | **java.lang.String**     | **1**                 |          |
| **BLOB**      | **L+N**      | **BLOB**                  | **java.lang.byte[]**     | **-4**                |          |
| **TEXT**      | **65535**    | **VARCHAR**               | **java.lang.String**     | **-1**                |          |
|               |              |                           |                          |                       |          |
| **INTEGER**   | **4**        | **INTEGER UNSIGNED**      | **java.lang.Long**       | **4**                 |          |
| **TINYINT**   | **3**        | **TINYINT UNSIGNED**      | **java.lang.Integer**    | **-6**                |          |
| **SMALLINT**  | **5**        | **SMALLINT UNSIGNED**     | **java.lang.Integer**    | **5**                 |          |
| **MEDIUMINT** | **8**        | **MEDIUMINT UNSIGNED**    | **java.lang.Integer**    | **4**                 |          |
| **BIT**       | **1**        | **BIT**                   | **java.lang.Boolean**    | **-7**                |          |
| **BIGINT**    | **20**       | **BIGINT UNSIGNED**       | **java.math.BigInteger** | **-5**                |          |
| **FLOAT**     | **4+8**      | **FLOAT**                 | **java.lang.Float**      | **7**                 |          |
| **DOUBLE**    | **22**       | **DOUBLE**                | **java.lang.Double**     | **8**                 |          |
| **DECIMAL**   | **11**       | **DECIMAL**               | **java.math.BigDecimal** | **3**                 |          |
| **BOOLEAN**   | **1**        | **同TINYINT**             |                          |                       |          |
|               |              |                           |                          |                       |          |
| **ID**        | **11**       | **PK (INTEGER UNSIGNED)** | **java.lang.Long**       | **4**                 |          |
|               |              |                           |                          |                       |          |
| **DATE**      | **10**       | **DATE**                  | **java.sql.Date**        | **91**                |          |
| **TIME**      | **8**        | **TIME**                  | **java.sql.Time**        | **92**                |          |
| **DATETIME**  | **19**       | **DATETIME**              | **java.sql.Timestamp**   | **93**                |          |
| **TIMESTAMP** | **19**       | **TIMESTAMP**             | **java.sql.Timestamp**   | **93**                |          |
| **YEAR**      | **4**        | **YEAR**                  | **java.sql.Date**        | **91**                |          |

分页查询

- mysql实现
  select * from table limit begin,size; begin为数据的开始下标，size为数据个数。select count(*) from table; 求出totalPage; 创建PageInfo封装分页信息。pageinfo如下public class PageInfo<T>{ int pageNum; 当前页码 int pageSize; 尺寸 List<T> list; 查询的数据 int pages; 总页码 int total; 总记录数 int navigatePageNums[]; 导航条页号 int navigatePages; 导航条页码数​}导航条计算方法：​若总页码小于导航条页码数，全部存入数组。 若startNum<1,startNum=1,从startNum开始存navigatePages个页号。//（只会有一段越界，因为pages>navigatePages） 若startNum>pages，从后向前存。 若两端都合法，从startNum开始即可。​
- pageHelper实现
  参考https://github.com/pagehelper/Mybatis-PageHelper/blob/master/wikis/zh/HowToUse.md1，引入依赖，在mybatis或spring中配置拦截器，二选一即可。2PageHelper.startPage(pageNumber,5); 之后的第一个查询会分页List<User> userList=userService.findAll();PageInfo pageInfo=new PageInfo(userList,5); 封装return Msg.success().add("pageInfo",pageInfo); ajax返回前端

## 数据库优化

### 索引

>有什么类型的索引（hash索引、b树、b+树、min_max、bitmap、bloom、稀疏索引）

索引是一个排序和快速查找的数据结构。若在某一列上有索引，可通过列值快速查找到所需数据。 

b树和b+树![img](https://api2.mubu.com/v3/document_image/3b048d78-9fe6-4498-8531-9de3bc23be78-6012434.jpg)
树：由边和结点构成的无回路的数据结构。b树的结点构成，由k个关键字和k+1个指向子节点的指针交替构成，关键字递增。叶子结点不存储关键字，用来存储具体信息。b树在内部结点的关键字不存储具体信息，所有信息都在叶子节点，叶子结点连接成链表。相比之下有，都在叶子，所以性能稳定 ，内部无卫星数据指针，范围查询块，因为有叶子链表。

#### 聚集索引，非聚集索引

定义1：数据库表的物理顺序与聚集索引的顺序一样，一个表只有一个聚集索引。若有PK,PK就是聚集索引。聚集索引的索引树叶子结点直接存储行记录。而非聚集索引叶子节点存储主键值即聚集索引。

#### 联合索引，单个索引

联合索引就是建立在多列上的索引

#### 唯一索引

对该列每个索引键值，只有一条记录匹配。

#### 索引覆盖

原理：select所需数据从索引树就可取得，一次查询即可，不必再通过id回表来查询数据表。方法：所建立的非聚集联合索引包含覆盖了where和查询的列字段，所以索引树上就有所需要的全部数据（叶子上存储了主键id）。

#### 索引失效

索引失效情况：

#### 索引的创建删除

CREATE INDEX name ON table(col1,col2....)DROP INDEX name;

优化

#### 索引覆盖优化

全表count查询优化：添加索引列查询回表索引：覆盖即可分页查询避免回表：覆盖即可​

#### 索引失效优化

like以%开头or左右均为索引才生效varchar未加单引号is null ,is not nullnot ，<>,!= 可以优化为range字段上使用函数全表更快​就不使用索引​

explain（8+7）

select_type
simplepirmarysubquery​unionderived​

id
id若相同，由上向下顺序。若不同，大的先执行。

table
表名称，临时表derived后数字为id

type
表的查询类型ALL:全表扫描index:全索引扫描range:给定范围检索行ref:多表连接使用非唯一索引eq_ref:多表连接使用PK，unique key.const:通过PK直接查询，只读一次。system:表只有一列

possible_keys
可能使用的索引

key
实际使用的索引

key_len
索引长度

ref
使用的列字段和常量

rows
扫描出的行数

extra
using filesort :索引排序不可用using temporary:临时表using join buffer:多表连接未使用索引using where：使用了where过滤查找结果using index:索引覆盖了

#### hash索引的缺点

>只需存储对应的hash值，索引结构紧凑，查找速度快。
>
>缺点：
>
>1无法实现排序，因为hash不是按照索引值顺序存储的。
>
>2只支持等值查询，不支持大于小于。
>
>3hash冲突，不同的索引列值却有相同的hash。

#### b树和b+树的区别

>https://www.jianshu.com/p/ace3cd6526c4
>
>B+树是B-树的变体，也是一种多路搜索树, 它与 B- 树的不同之处在于:
>
>1. 所有关键字存储在叶子节点出现,内部节点(非叶子节点并不存储真正的 data)
>2. 为所有叶子结点增加了一个链指针

>**页是计算机管理存储器的逻辑块，硬件及操作系统往往将主存和磁盘存储区分割为连续的大小相等的块，每个存储块称为一页（在许多操作系统中，页得大小通常为4k）**，主存和磁盘以页为单位交换数据。当程序要读取的数据不在主存中时，会触发一个缺页异常，此时系统会向磁盘发出读盘信号，磁盘会找到数据的起始位置并向后连续读取一页或几页载入内存中，然后异常返回，程序继续运行。
>
>**所以IO一次就是读一页的大小**
>
>b+树一次io能读取出更多的节点信息，因为其data存储在叶子节点中。
>
>一般来说，索引本身也很大，不可能全部存储在内存中，因此索引往往以索引文件的形式存储的磁盘上。这样的话，索引查找过程中就要产生磁盘I/O消耗，相对于内存存取，I/O存取的消耗要高几个数量级，所以评价一个数据结构作为索引的优劣最重要的指标就是在查找过程中磁盘I/O操作次数的渐进复杂度。换句话说，索引的结构组织要尽量减少查找过程中磁盘I/O的存取次数。下面先介绍内存和磁盘存取原理，然后再结合这些原理分析B-/+Tree作为索引的效率。
>
>1.**B+树内节点不存储数据，所有 data 存储在叶节点导致查询时间复杂度固定为 log n。而B-树查询时间复杂度不固定，与 key 在树中的位置有关，最好为O(1)。**
>
>**2.B+树叶节点两两相连可大大增加区间访问性，可使用在范围查询等，而B-树每个节点 key 和 data 在一起，则无法区间查找。**
>
>**3.B+树更适合外部存储。由于内节点无 data 域，每个节点能索引的范围更大更精确**

### 引擎

>了解数据库的什么引擎（myisam、innodb、kylin、doris）
