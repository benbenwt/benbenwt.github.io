```
location
前端点击切换的
```

写一个pageHelper类

发现应使用如下方法创建输入流，而不是用path.tostring().通过查看TextInputFormat的源码，发现它使用此方法创建输入流。hdfs使用的FSDataInputStream是一个继承自java.io.InputFormat的类，但不是直接继承的，中间还有几个父类。

```
Path path=split.getPath();
            FileSystem fileSystem=path.getFileSystem(configuration);
            InputStream inputStream=fileSystem.open(path);
            BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(inputStream));
```



```
hadoop jar mapreduce_start-1.0-SNAPSHOT.jar com.ti.mr.getSingleInfo.wordcount.WordCountDriver input output
```

新的stix加入，添加。

```
<!--        <dependency>-->
<!--            <groupId>org.apache.hadoop</groupId>-->
<!--            <artifactId>hadoop-hdfs</artifactId>-->
<!--            <version>3.1.4</version>-->
<!--        </dependency>-->
<!--        <dependency>-->
<!--            <groupId>org.apache.hadoop</groupId>-->
<!--            <artifactId>hadoop-mapreduce-client-core</artifactId>-->
<!--            <version>3.1.4</version>-->
<!--        </dependency>-->
<!--        <dependency>-->
<!--            <groupId>org.apache.hadoop</groupId>-->
<!--            <artifactId>hadoop-mapreduce-client-jobclient</artifactId>-->
<!--            <version>3.1.4</version>-->
<!--            <scope>provided</scope>-->
<!--        </dependency>-->
<!--        <dependency>-->
<!--            <groupId>org.apache.hadoop</groupId>-->
<!--            <artifactId>hadoop-mapreduce-client-common</artifactId>-->
<!--            <version>3.1.4</version>-->
<!--        </dependency>-->
```

github不上传某些文件



hivesql

```
#java api不要写分号结尾
 select type,sampletime, count(*) nums   from sample where type!='NULL' and sampletime!='NULL' group by type,sampletime;
 select architecture,count(1) nums from sample group by architecture
 !connect jdbc:hive2://hbase:10000/platform root root
 hive存储裁掉time
```

```
补全没有的日期，计算截至的累计数量。
求每一天的category百分比
```



多个sql不可以公用statement

type添加

connector,三元组，jar主方法

服务器密码:240711.wt

hive和hbase表：

单个信息，用来检索，统计；md5,SHA256,sha1,size,architecture,language,endianess,type,              		sampletime,ip,url,cveid,location,identity,      hdfs				

其他:进程，字符，注册表，filepath

identity

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
disable 'tablename'
drop 'tablename'
```



hive

```
create table sample(md5 string,SHA256 string,sha1 string,size string,architecture string,languages string,endianess string,type string,sampletime string,ip string,url string,cveid string,location string,identity string,hdfs string);
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

category

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.ti.platform_provider_statistic8001.mapper.CategoryMapper">

    <select id="getCategoryInfoNow" resultType="Category">
        SELECT category_id categoryId,category,value,percent,time FROM `category_tbl` WHERE  time=(SELECT MAX(time) FROM `category_tbl`)
    </select>

    <select id="getCategoryInfoByDay" resultType="Category">
SELECT category_id categoryId,category,value,percent,time FROM category_tbl a WHERE a.time IN
(SELECT time FROM(SELECT DISTINCT b.time FROM  `category_tbl` b ORDER BY b.time DESC LIMIT 10)AS times) AND a.category IN
(SELECT * FROM (SELECT category FROM category_tbl  GROUP BY category  ORDER BY  count(category) DESC LIMIT 2)AS category_list) ORDER BY category,time
    </select>

</mapper>
```



Springcloud搭建

前端页面搭在nginx上，使用echart。Bootstrap。

请求consumer，consumer会依据eureka名称请求对应的客户端。

数据库用mysql，统计后的关系型数据放到mysql。 

Docker，使用dockerfile搭建服务环境环境，挂载目录上传html和jar包。

 

计算统计结果，展示统计信息后台，crudstix文件，nginx前端，数据库。

category value  percent time 

family value

 ```
CREATE TABLE `category_tbl`
(
    `category_id` INT UNSIGNED AUTO_INCREMENT,
    `category` VARCHAR(20) NOT NULL,
    `value`  INT NOT NULL,
    `time`  DATE,
    PRIMARY KEY(`category_id`)
)ENGINE=InnoDB DEFAULT CHARSET=utf8;
 ```

柱状图当前统计=源数据截至当前日每个类数量，样本时间<当前时间。

折线：源数据截至某日累积样本数，样本时间<n日,n=1……。每周和每月数据取该段时间最大日期即可。某日月周没有，即取前一周月日。新增即是相减。

单个样本：该样本所处时间点。源数据，按某策略更新mysql统计数据即可。前端要展示，不存mysql里。

```
CREATE TABLE `family_tbl`
(
    `family_id` INT UNSIGNED AUTO_INCREMENT,
    `family` VARCHAR(20) NOT NULL,
    `value` INT NOT NULL,
    PRIMARY KEY(`family_id`)
)ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

 ```
CREATE TABLE `architecture`
(
    `arch_id` INT UNSIGNED AUTO_INCREMENT,
    `architecture` VARCHAR(20) NOT NULL,
    `value` INT NOT NULL,
    PRIMARY KEY(`arch_id`)
)ENGINE=InnoDB DEFAULT CHARSET=utf8;
 ```



mysql事件删除过期数据。日统计信息。后端要提供单个样本具体信息，

1

```
#查询最新的一天，每个category的累积数量。
SELECT category_id categoryId,category,value,time FROM `category_tbl` WHERE category_id IN(SELECT     SUBSTRING_INDEX(GROUP_CONCAT(category_id ORDER BY `time` DESC),',',1) FROM `category_tbl` GROUP BY category )
```

```
#查询最新的一天，每个category的累积数量。如果此数据保证每天都存储，即每类的最新数据就是MAX(time)，所以不用concat。
SELECT category_id categoryId,category,value,time FROM `category_tbl` WHERE  time=(SELECT MAX(time) FROM `category_tbl`) LIMIT 4
```

2

```
最新的前10天，占比最高的前两类。
```

```
#最新的前10天，占比最高的前两类。如果每一天的累积数量都有一条记录，使用如下。
SELECT category_id categoryId,category,value,time FROM category_tbl a WHERE a.time IN 
(SELECT time FROM(SELECT DISTINCT b.time FROM `category_tbl` b ORDER BY b.time DESC LIMIT 10)AS times) AND a.category IN
(SELECT * FROM (SELECT category FROM category_tbl GROUP BY category ORDER BY count(category) DESC LIMIT 2)AS category_list) ORDER BY category,time
```

3

```
最近的10周
```

```
最近的10周，特殊情况，每天都记录了。
```

4

 ```
最近的10个月。
 ```

```
最近的10个月，特殊情况，每天都记录了。
```

源数据，stix2文件。待统计数据：单个结构化的样本，包含发生时间等属性。统计数据：截至某日，共发生各种类多少次等。



##### 版本

java8,hadoop2.10.x,3.1.1+,3.2.x,hbase2.3.x，hive

java8,hadoop3.1.4,hive3.1.2,hbase2.3.3,mysql5.7.28,mysql-connector-5.1.37

dump_mysql

```
package com.ti.dump_mysql.utils;



import java.sql.*;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.time.LocalDate;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;


public class MysqlConnect {
    public static final  String driverName="com.mysql.cj.jdbc.Driver";
    Connection conn;
    public void init(String url,String user,String root) throws ClassNotFoundException, SQLException {
        Class.forName(driverName);
        conn= DriverManager.getConnection(url,user,root);
        System.out.println(conn);
        Statement st=conn.createStatement();
        st.execute("delete from category_tbl");
        st.execute("delete from  architecture");
    }
    public static void main(String[] args) throws SQLException, ClassNotFoundException {
        Map<String, Map<String,Integer>> info=new HashMap<>();
        System.out.println(info.get("a"));
//        MysqlConnect mysqlConnect=new MysqlConnect();
//        String url="jdbc:mysql://localhost:3306/platform?useUnicode=true&characterEncoding=utf-8&useSSL=true&serverTimezone=UTC";
//        mysqlConnect.init(url,"root","root");
//        mysqlConnect.insertSample();
    }
    public Map<String, Map<String,Integer>>  caculateAccuNums(ResultSet resultSet) throws SQLException {
        /*三维数组吧，才能随机读取.category,time,value*/
        Map<String, Map<String,Integer>> info=new HashMap<>();
        Map<String,Integer> tmp=new HashMap<>();
        Map<String,Integer> categoryCount=new HashMap<>();
        while(resultSet.next())
        {
            String category=resultSet.getString(1);
            String time=resultSet.getString(2);
            Integer value=resultSet.getInt(3);

            String subTime=time.substring(0,10);

            tmp.clear();
            tmp.put(category,value);

            info.put(subTime,new HashMap<>(tmp));

            categoryCount.put(category,0);
        }
        System.out.println(categoryCount);

        LocalDate localDate=LocalDate.now();
        LocalDate startDate=localDate.minusDays(360);
        LocalDate vardate=startDate;
        LocalDate endBorder=localDate.plusDays(1);

        Map<String, Map<String,Integer>> result=new HashMap<>();
        Map<String, Integer> resultTmp=new HashMap<>();
        Map<String, Integer> categoryValue;
        int newCount;
        //
        while(vardate.isBefore(endBorder))
        {//循环日期
            System.out.println("vardate："+vardate);

            categoryValue= info.get(vardate.toString());
            System.out.println("add ："+categoryValue);

            if(categoryValue!=null)
            {//遍历不为0类别
                //count
                for(Map.Entry<String,Integer> entry:categoryValue.entrySet())
                {
                    newCount=categoryCount.get(entry.getKey())+entry.getValue();
                    categoryCount.put(entry.getKey(),newCount);
                }
            }
            //当前vardate，累计categoryCount数目存储
            for(Map.Entry<String,Integer> entry:categoryCount.entrySet())
            {
                resultTmp.put(entry.getKey(),categoryCount.get(entry.getKey()));
                result.put(vardate.toString(),resultTmp);
            }
            System.out.println("accu :"+categoryCount);
            vardate=vardate.plusDays(1);
        }
        return result;
    }
    public void  insertCategory(ResultSet resultSet) throws SQLException, ParseException {
        Map<String, Map<String,Integer>> result=caculateAccuNums(resultSet);

        PreparedStatement ps=conn.prepareStatement("insert into category_tbl(time,category,`value`) values(?,?,?)");
        /*time category nums*/
        for(Map.Entry<String,Map<String,Integer>> entry:result.entrySet())
        {
            //time
            SimpleDateFormat simpleDateFormat=new SimpleDateFormat("yyyy-MM-dd");
            Date date=simpleDateFormat.parse(entry.getKey());
            java.sql.Date sqlDate=new java.sql.Date(date.getTime());
            ps.setDate(1,sqlDate);

           for(Map.Entry<String,Integer> entry1:entry.getValue().entrySet())
           {
               String categoryStr=entry1.getKey();
               ps.setString(2,categoryStr);

               int nums=entry1.getValue();
               ps.setInt(3,nums);
               ps.execute();
           }
        }
        ps.close();
    }

    public void insertArch(ResultSet arch) throws SQLException {
        Statement st=conn.createStatement();
        while(arch.next())
        {
            String sql="insert into architecture(architecture,`value`) values('"+arch.getString(1)+"',"+arch.getString(2)+");";
            System.out.println(sql);
            st.execute(sql);
        }
        st.close();
    }

}

```

