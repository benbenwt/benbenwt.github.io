### 版本

java8,hadoop2.10.x,3.1.1+,3.2.x,hbase2.3.x，hive

java8,hadoop3.1.4,hive3.1.2,hbase2.3.3,mysql5.7.28,mysql-connector-5.1.37



```
#import org.apache.commons.io.FileUtils;常用io包
String json_str = FileUtils.readFileToString(file, "UTF-8");
```

```
#import org.apache.hadoop.io.IOUtils; hadoop提供的io工具包
 Configuration configuration = new Configuration();
        configuration.set("fs.hdfs.impl", org.apache.hadoop.hdfs.DistributedFileSystem.class.getName());
        configuration.set("fs.file.impl", org.apache.hadoop.fs.LocalFileSystem.class.getName());
        FileSystem fs = FileSystem.get(URI.create(hdfsPath), configuration);
        FSDataOutputStream out = fs.create(new Path(hdfsPath));//创建一个输出流
        InputStream in = new FileInputStream(new File(localPath));//从本地读取文件
        IOUtils.copyBytes(in, out, 100, true);
        System.out.println("上传完毕");
```



```
ntpdate  pool.ntp.org
date -R 
执行时间
17   1,4   6 
```

```
date
timedatectl  list-timezones  
date  -R
timedatectl set-timezone Asia/Shanghai
ntpdate  pool.ntp.org
```

```text
sudo yum -y install ntp
```



dump_hive

```
#!/bin/sh
PATH=/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/bin:/sbin
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib
hadoop jar /root/software/mapreduce_start-1.0-SNAPSHOT.jar  "hdfs://hbase:9000/user/root/stix/$(date -d last-day +%Y-%m-%d)/" "hdfs://hbase:9000/user/root/dump_hive_result/$(date -d last-day +%Y-%m-%d)/"

hadoop jar /root/software/mapreduce_start-1.0-SNAPSHOT.jar  "hdfs://hbase:9000/user/root/stix/$(date -d last-day +%Y-%m-%d)/" "hdfs://hbase:9000/user/root/dump_hive_result/$(date -d last-day +%Y-%m-%d)/"&&hdfs df
s -mv "hdfs://hbase:9000/user/root/dump_hive_result/$(date -d last-day +%Y-%m-%d)/"  "/user/hive/warehouse/platform.db/sample"
```



```
#centos crontab
vim /etc/crontab
crontab -e
serivce crond restart

```

crontab

```
#crontab失效，将命令卸载sh内，添加环境变量。且不要再crontab中指定执行用户即可。
/usr/bin/command.sh:
#!/bin/sh
PATH=/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/bin:/sbin
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib
java -jar /root/module/dump_hdfs-1.0-SNAPSHOT.jar  "/home/node/platform_data/stix/$(date -d last-day +%Y-%m-%d)" "hdfs://hbase:9000/user/root/stix/$(date -d last-day +%Y-%m-%d)/">/var/log/dump_hdfs.log 2>&1

crontab -e
提交stix
1 0  * * *   /root/module/dump_hdfs.sh
1 0  * * *   /root/module/dump_es.sh
提交到lisa，生成stix
1 0 * * *   /root/module/clock_py.sh
1 0 * * *  /root/module/clock_py1.py


```



```
crontab  -e

```



```
55 17 * * *  root python3.7 /root/module/lisa/clock_py.py

0  4  * * *  root java -jar /root/module/dump_hdfs-1.0-SNAPSHOT.jar  "/home/node/platform_data/stix/$(date -d last-day +%Y-%m-%d)" 
"hdfs://hbase:9000/user/root/stix/$(date -d last-day +%Y-%m-%d)"

0  4  * * *  root java -jar /root/module/dump_hdfs-1.0-SNAPSHOT.jar  "/home/node/platform_data/stix1/$(date -d last-day +%Y-%m-%d)" "hdfs://hbase:9000/user/root/stix/$(date -d last-day +%Y-%m-%d)"

```



小作业模式：mapreduce.job.ubertask.enable

​    -D mapreduce.job.ubertask.enable=true 

```

/user/root/stix/$(date -d last-day +%Y-%m-%d)
```



```
java -jar D:\DevInstall\IdeaProjects\dump_hdfs\target\dump_hdfs-1.0-SNAPSHOT.jar "C:\\Users\\guo\\Desktop\\dump_hdfs" "hdfs://hbase:9000/"
java -jar /root/module/dump_hdfs-1.0-SNAPSHOT.jar  "/home/node/platform_data/stix/$(date -d last-day +%Y-%m-%d)" "hdfs://hbase:9000/"
java -jar /root/module/dump_hdfs-1.0-SNAPSHOT.jar  "/home/node/platform_data/stix/$(date +%Y-%m-%d)" "hdfs://hbase:9000/"
前一天:date -d last-day +%Y-%m-%d
java -jar dump_es-1.0-SNAPSHOT.jar  "/home/node/platform_data/stix/$(date -d last-day +%Y-%m-%d)"  "hbase2"
```



| hostname | ip   | 服务                                                   |
| -------- | ---- | ------------------------------------------------------ |
| hbase    | 187  | hdfs-mater,hive,dump_hive_mysql,kafka                  |
| hbase1   | 186  | yarn-master,nginx                                      |
| hbase2   | 185  | es,mysql,lisa2,grakn                                   |
| lisa     | 184  | lisa1,java,lisa_submit1,lisa_submit2,dump_hdfs,dump_es |

1   /home/node/paltform_data/sample1

2  /home/node/paltform_data/sample

### lisa

>重新build遇到的主要问题：
>
>1radare无法下载，到radareorg/radare2下载后COPY的镜像内。linux_images无法下载,到网站下载好COPY进去。对于无法COPY的images，在.dockerignore中注释掉该行。
>
>2无法安装r2pipe，将requirements.txt中版本改为1.5.3，进行build。

```
#lisa-worker的内容,再将requirements.txt中ripe2改为1.5.3版本,将.dockerignore中的images注释掉。
FROM python:3.6-slim

ARG maxmind_key

RUN apt-get update && apt-get install -y --no-install-recommends \
    curl\
    gcc \
    g++ \
    libpcap-dev \
    make \
    patch \
    git \
    qemu \
    qemu-system \
    openvpn \
    binutils \
    iprange \
    wget \
    tar \
    e2tools 

COPY  ./new  ./radare2

RUN 	./radare2/sys/install.sh \
    && useradd -m lisa \
    && echo "Copying LiSa Linux images ..." 

COPY --chown=lisa:lisa ./images /home/lisa/images

COPY --chown=lisa:lisa ./data /home/lisa/data
COPY --chown=lisa:lisa ./docker /home/lisa/docker
COPY --chown=lisa:lisa ./lisa /home/lisa/lisa
COPY --chown=lisa:lisa ./requirements.txt /home/lisa/requirements.txt

ENV PYTHONPATH /home/lisa

WORKDIR /home/lisa

RUN /usr/local/bin/python -m pip install --upgrade pip

RUN pip install -r requirements.txt --extra-index-url  http://pypi.douban.com/simple/ --trusted-host pypi.douban.com
RUN iprange -j data/blacklists/* > data/ipblacklist \
    && ./docker/worker/maxmind.sh $maxmind_key \
    && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false \
    git \
    gcc \
    g++ \
    make \
    patch \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf /radare2/.git

CMD ["./docker/worker/init.sh"]
```

##### 清除历史数据

```
docker-compose up 
docker exec  -it container_id /bin/bash
mysql -ulisa -plisa
use lisadb
delete  from cel...
#清除./data/storate中文件即可。
```

##### 更改监听ip和端口

```
修改docker-compose.yml中的nginx服务的的args->webhost,让后重新docker-compose build
```

sample提交-lisa-lisa-stix2-hdfs,es

按日期，t+1天再处理t天的数据。

type多个分多条，location多个分多条

自定义分组group by规则

docker-compose up --scale worker=10

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
hadoop jar /root/software/mapreduce_start-1.0-SNAPSHOT.jar  input output
hadoop jar /root/software/mapreduce_start-1.0-SNAPSHOT.jar  input1 output
hadoop fs -rm -r  ouput
```









### hive和hbase

##### hive

```
#指定字符集
create database amon DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
alter table sample modify column COMMENT varchar(256) character set utf8;
```



连接

```
#java api不要写分号结尾
 select type,sampletime, count(*) nums   from sample where type!='NULL' and sampletime!='NULL' group by type,sampletime;
 
 select architecture,count(1) nums from sample group by architecture
 !connect jdbc:hive2://hbase:10000/platform root root
 hive存储裁掉time
 create external table sample(md5 string,SHA256 string,sha1 string,size string,architecture string,languages string,endianess string,type string,sampletime string,ip string,url string,cveid string,location string,identity string,hdfs string)row format delimited fields terminated by '\t';
load data inpath 'hdfs://hbase:9000/user/root/output/part-r-00000' into table sample; #从hdfs中上传数据
```

```
补全没有的日期，计算截至的累计数量。
求每一天的category百分比
```



单个信息，用来检索，统计；md5,SHA256,sha1,size,architecture,language,endianess,type,              		sampletime,ip,url,cveid,location,identity,      hdfs				

其他:进程，字符，注册表，filepath，identity



创建hbase表

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



创建hive表

```
create table sample(md5 string,SHA256 string,sha1 string,size string,architecture string,languages string,endianess string,type string,sampletime string,ip string,url string,cveid string,location string,identity string,hdfs string);
```





### 数据库

**category**

```
#建表语句
CREATE TABLE `category_tbl`
(
    `category_id` INT UNSIGNED AUTO_INCREMENT,
    `category` VARCHAR(20) NOT NULL,
    `value`  INT NOT NULL,
    `time`  DATE,
    PRIMARY KEY(`category_id`)
)ENGINE=InnoDB DEFAULT CHARSET=utf8
```

```
#统计cve漏洞,恶意软件数量
select category,sum(tmp.value) value from  
(select CASE WHEN `category` IN ('vulnerabilities') THEN 'vulnerabilities'
ELSE "other" END AS `category`,sum(value) value from category_tbl  group by  `category`) tmp group by tmp.category
```



```
#获取最近七天，排名靠前的两类。
SELECT category_id categoryId,category,value,percent,time FROM category_tbl a WHERE a.time IN
(SELECT time FROM(SELECT DISTINCT b.time FROM  `category_tbl` b ORDER BY b.time DESC LIMIT 10)AS times) AND a.category IN
(SELECT * FROM (SELECT category FROM category_tbl  GROUP BY category  ORDER BY  count(category) DESC LIMIT 2)AS category_list) ORDER BY category,time
```

```
#获取最新的一天，其各个类别的累积数量。
SELECT category_id categoryId,category,value,percent,time FROM `category_tbl` WHERE  time=(SELECT MAX(time) FROM `category_tbl`)
```



**family**

```
#建表语句
CREATE TABLE `family_tbl`
(
    `family_id` INT UNSIGNED AUTO_INCREMENT,
    `family` VARCHAR(20) NOT NULL,
    `value` INT NOT NULL,
    PRIMARY KEY(`family_id`)
)ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

**architecture**

 ```
CREATE TABLE `architecture`
(
    `arch_id` INT UNSIGNED AUTO_INCREMENT,
    `architecture` VARCHAR(20) NOT NULL,
    `value` INT NOT NULL,
    PRIMARY KEY(`arch_id`)
)ENGINE=InnoDB DEFAULT CHARSET=utf8;
 ```

**location**

```
CRAEATE TABLE `location`
(
	`location_id` INT UNSIGNED AUTO_INCREMENT,
	 `location` VARCHAR(20) NOT NULL,
	 `value` INT NOT NULL,
	 PRIMARY KEY(`location_id`)
)ENGINE=InnoDB DEFAULT CHARSET=utf8

select location_id locationId,location,value  from location
```



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
最新的近10天，占比最高的前两类。
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





### 备忘

服务器密码:240711.wt

3.25：统一中英文location

