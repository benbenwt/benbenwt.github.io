### 安装

```
```



```
批量insert
for (int k = 1; k <= 10000; k++) {
					pst.setLong(1, k * i);
					pst.setLong(2, k * i);
					pst.addBatch();
				}
pst.executeBatch();
#prepareStatement使用batch插入15000条要200s。添加后两个参数，速度直接到达一秒。
  conn = DriverManager.getConnection("jdbc:mysql://hbase2:3306/platform?useUnicode=true&characterEncoding=utf-8&useSSL=true&serverTimezone=UTC&rewriteBatchedStatements=true&useServerPrepStmts=true","root","root");
```



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

### Java MySQL数据类型对照

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

- 分页查询

  - mysql实现
    select * from table limit begin,size; begin为数据的开始下标，size为数据个数。select count(*) from table; 求出totalPage; 创建PageInfo封装分页信息。pageinfo如下public class PageInfo<T>{ int pageNum; 当前页码 int pageSize; 尺寸 List<T> list; 查询的数据 int pages; 总页码 int total; 总记录数 int navigatePageNums[]; 导航条页号 int navigatePages; 导航条页码数​}导航条计算方法：​若总页码小于导航条页码数，全部存入数组。 若startNum<1,startNum=1,从startNum开始存navigatePages个页号。//（只会有一段越界，因为pages>navigatePages） 若startNum>pages，从后向前存。 若两端都合法，从startNum开始即可。​

  - pageHelper实现
    参考https://github.com/pagehelper/Mybatis-PageHelper/blob/master/wikis/zh/HowToUse.md1，引入依赖，在mybatis或spring中配置拦截器，二选一即可。2PageHelper.startPage(pageNumber,5); 之后的第一个查询会分页List<User> userList=userService.findAll();PageInfo pageInfo=new PageInfo(userList,5); 封装return Msg.success().add("pageInfo",pageInfo); ajax返回前端

- 数据库优化

  - 索引

    索引是一个排序和快速查找的数据结构。若在某一列上有索引，可通过列值快速查找到所需数据。 

    - b树和b+树![img](https://api2.mubu.com/v3/document_image/3b048d78-9fe6-4498-8531-9de3bc23be78-6012434.jpg)
      树：由边和结点构成的无回路的数据结构。b树的结点构成，由k个关键字和k+1个指向子节点的指针交替构成，关键字递增。叶子结点不存储关键字，用来存储具体信息。b树在内部结点的关键字不存储具体信息，所有信息都在叶子节点，叶子结点连接成链表。相比之下有点：1，都在叶子，所以性能稳定 2，内部无卫星数据指针，io块 3，范围查询块，因为有叶子链表。

    - 聚集索引，非聚集索引
      定义1：数据库表的物理顺序与聚集索引的顺序一样，一个表只有一个聚集索引。若有PK,PK就是聚集索引。聚集索引的索引树叶子结点直接存储行记录。而非聚集索引叶子节点存储主键值即聚集索引。

    - 联合索引，单个索引
      联合索引就是建立在多列上的索引

    - 唯一索引
      对该列每个索引键值，只有一条记录匹配。

    - 索引覆盖
      原理：select所需数据从索引树就可取得，一次查询即可，不必再通过id回表来查询数据表。方法：所建立的非聚集联合索引包含覆盖了where和查询的列字段，所以索引树上就有所需要的全部数据（叶子上存储了主键id）。

    - 索引失效
      索引失效情况：

    - 索引的创建删除
      CREATE INDEX name ON table(col1,col2....)DROP INDEX name;

    - 优化

      - 索引覆盖优化
        全表count查询优化：添加索引列查询回表索引：覆盖即可分页查询避免回表：覆盖即可​

      - 索引失效优化
        like以%开头or左右均为索引才生效varchar未加单引号is null ,is not nullnot ，<>,!= 可以优化为range字段上使用函数全表更快​就不使用索引​

  - explain（8+7）

    - select_type
      simplepirmarysubquery​unionderived​

    - id
      id若相同，由上向下顺序。若不同，大的先执行。

    - table
      表名称，临时表derived后数字为id

    - type
      表的查询类型ALL:全表扫描index:全索引扫描range:给定范围检索行ref:多表连接使用非唯一索引eq_ref:多表连接使用PK，unique key.const:通过PK直接查询，只读一次。system:表只有一列

    - possible_keys
      可能使用的索引

    - key
      实际使用的索引

    - key_len
      索引长度

    - ref
      使用的列字段和常量

    - rows
      扫描出的行数

    - extra
      using filesort :索引排序不可用using temporary:临时表using join buffer:多表连接未使用索引using where：使用了where过滤查找结果using index:索引覆盖了

