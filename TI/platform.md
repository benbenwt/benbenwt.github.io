```
平台备份时间：20211122
```

```
剔除冗余数据
版本管理，但是有很多块。
正常的：
web的，挖掘的，数据中间工具。
java的，python的

单机的：
docker的
idea很多功能都没使用过-工具性质。
```

### ssh端口

```
52542
```



### 20211124软件园迁移到两台新机器

```
需要存储的部分：
1需要添加的代码：
   1es_search整个image，其中包括(es_search和fourNumberAndImport)
   2前端替换,nginx.conf修改
   3estomysql（漏掉了一个类型的英文，没有转换为中文，好像是keyboard）,platformstatistic （有一个无意义字段也统计了），lisaprovider（修改了什么？）三个后端修改。cti_server修改（pcap备份），pcap修改（计算哈希id，用rb读而不是r）。
2需要导入的数据
3docker，docker-compose文件,docker安装命令
3整个平台完整的docker文件

需要迁移的部分：
安装docker以及docker-compose软件
旧机器上的整个docker文件，包括data挂载目录的数据都要迁移。

需要处理的部分：
上文的需要存储的部分的1与2要完成

基本流程：将旧机器的文件复制到新机器，在新机器安装docker以及docker-compose。构建对应的docker image。成功后添加需要处理的部分，使用构建的方法，不进入内部修改。

docker 20.10.7
docker-compose版本  docker-compose version 1.28.5
两者的安装方式:
docker:使用拷贝的文件，命令如下：sudo dpkg -i /path/to/package.deb。教程https://docs.docker.com/engine/install/ubuntu/
docker-compose使用拷贝的文件即可，教程https://github.com/docker/compose
```

```
docker-compose 版本修改为指定
```

```
光盘放置的位置：
1需要添加的代码,需要导入的数据,docker，docker-compose文件,docker安装命令。放置在外置光驱盒子中光盘。
2整个平台完整的docker文件。放置在袋子中光盘。
```



```
将原来机器的程序迁移到另一台机器上。
更新程序部分，（前端，后端）。
更新数据部分。
```

### 存储数据库json hive

>apache drill
>
>apache impala 计算引擎
>
>apache kylin  存储数据库
>
>kudu 存储数据库
>
> hive get_json_object

|               | 国内活跃度 | 支持json |
| ------------- | ---------- | -------- |
| apache drill  | 少         | 是       |
| apache impala | 正常       | 是       |
| apache kylin  | 正常       | ？否     |



```
https://blog.csdn.net/danpu0978/article/details/107275443
impala:https://blog.csdn.net/vkingnew/article/details/109792607
https://blog.csdn.net/weixin_39911113/article/details/78805272
hive+hbase:https://www.zhihu.com/question/21677041?sort=created
```

```

-----------------es_search：
    将es_search文件复制到目录，添加该模块docker-compose，修改docker-compose将cves.db挂载到/usr/local/cvedb,docker-compose up启动服务。
    
-------------------前端：
启动服务，docker cp将web文件复制到容器里边的新文件夹，修改nginx配置文件。将lisa的api代理过来，将download代理过来，将新的web作为root。

pcap后端的读取r,rb

-------------------estomysql,platformstatistic ，lisaprovider：
将原来的备份，然后测试。最后修改docker文件。

cti_server 后端

##前端twocategory使用为other和vulnerability，后端提供三个值。


```

```
malware组件不解析size
名称都写hash
```

```
需要用网络
docker网络ip+
```

```
配置服务失败的restart
```



```
malware早于other
虚拟机 后端写的other（更新过）   前端写的malware
idea  后端写的malware   前端写的malware
之前就是写的malware所以要更新
```



```
pcap插入了三次？？
java运行时指定参数或外部配置文件
程序的日志管理
```



|               | 主键检索             | 非主键检索（全文检索） | 聚合统计                                 |
| ------------- | -------------------- | ---------------------- | ---------------------------------------- |
| mysql         | 能                   | 能，但默认情况性能低   | 能                                       |
| hbase         | 能，能处理的数据量大 | 能，但默认情况性能低   | 能，能处理的数据量大                     |
| elasticsearch | 能                   | 能，很适合             | 能，编写复杂，不适合大量数据、表的此场景 |
| hive          | 不适合               | 不适合                 | 适合                                     |

```
二级索引：https://www.jianshu.com/p/d20769abfa0e
```



```
从package.json依赖中删除了
//    "vue": "^2.5.2",
//    "vue-router": "^3.0.1"
```



```
Apriori算法
```



```
对应的后端，前端，数据库都存在才行。
差异：数据库不同，分块导入没有，founumber没有。elastictomysql，platform statistics更新。pcap模块没有特征。
对应的后端，前端，数据库都存在才行。确保差异的东西，在这边测试通过，在那边也能工作，用docker测试吧。
差异：数据库不同，分块导入没有，founumber没有。elastictomysql，platform statistics更新。
后端：加sqlite版本的fournumber，sqlite版本的分模块导入。elastictomysql，platform statistics更新。
前端：接口不要写错，不要写多了如（pcap特征）
前后端衔接：nginx配置

pcap多次插入

```



```
report.vue
search
searchResult.vue
```

```
es数据库，mysql数据库，jdk，python解释器，nginx服务器。一些系统依赖，如qbittorrent，disspcap,libpcap
爬取，沙箱，web
```

框架

```
数据库的功能：存储，查找，聚合统计，全文检索
数据湖实际上只实现存储和查找功能。
parquet 数据库：列式存储，当然，也是处理关系型数据的。
apache Ozone 对象存储
s3 存储：可存储任何数据，最大可达5TB，可用于网盘存储。
COS 对象存储：可用于存储file文件，但是只提供存储和查找，无统计和检索功能。
iceberg 表引擎
flink 流处理
presto 处理
kylin 计算
sqoop 数据转换工具

如何实现的查找呢，三种：
https://zhuanlan.zhihu.com/p/111822325
1目录按规则命名
2过滤列名
3
数据湖里存一下json，然后再处理，不用直接放到hdfs上，自己维护查找的索引。

文件数据，非关系型（json数据），关系型数据，
```

cve

```
National Vulnerability Database (NVD)
MITRE CVE LIST
Vulnerability Database (VULDB)
WhiteSource Vulnerability Database
```





##### stix2要用python

```
malware-stix2
pcap-stix2
cve多种格式-stix2
```



### hadoop

```
hive，hbase只可处理结构化数据，和存储文件等，无法处理json，xml，html等。es处理可处理json。
pcapFeature都是结构化的，图片，文本，malware等无需处理的只需存储的信息，是不是小文件不能呀。
hadoop和web应用关系，hadoop是存储。

```

##### hbase+elasticsearch

```
hbase存储真实数据
es存储hbase的rowkey，rowkey为自己需要实现检索的字段。

存储功能
搜索功能本质在干什么？搜索就是根据某个词，找到所有包含这个词的文档，也就是倒排索引。（目前需要hash，各种name，各种type，扩展不方便）
聚合统计功能是针对结构化数据或json等按照一定规则统计。（目前需要恶意样本类型、时间，样本架构，时间地点，扩展比较方便）
方案1，hbase只实现存储，存储完整json。es实现倒排索引，并实现聚合。倒排索引和聚合都只覆盖部分需要的字段，自己定义。
```

##### hbase,mysql,es

```
hbase只支持rowkey查询，无法像mysql或es那样进行字段筛选并搜索。支持按rowkey搜索，按单列统计。
mysql可进行字段筛选，但是对字段内部或部分进行检索会使用like模糊查询，性能低下。都支持，但搜索性能低。
es适合分词和全文检索，当然其效果取决于分词效果。都支持，但查询存储没有hbase
```

##### elasticsearch 2.x官方文档

```
es的大部分场景是：“一个常见的设置是使用其它数据库作为主要的数据存储，使用 Elasticsearch 做数据检索”（2.X官方文档里说的）。和RDBMS是辅助关系。
```



##### 平台可展现部分

```
平台可展现的及有实际研究意义的。
平台可展现的：前端框架和界面,代码整合 
有实际研究意义的：2步骤，3步骤。pcap流式大量，stix2文件在hadoop存储方式
实际的研究意义：如两个例子，方法和有意义的研究。

m
爬虫，分析，数据统计、聚合，web模块。
```

```
挖掘部分，统计数据部分，web平台部分
java代码整理:web前后端，数据（esToMysql），其他模块（malwareMine爬虫）
py代码整理：web后端，脚本（lisa相关脚本），其他模块（threat-broadcast爬虫）

文件结构按照功能、开发测试部署方便划分，块少一点，块之间耦合高，调试方便，服务划分。
```



```
多态的作用，你需要三个子类进行一些类似的动作，但是动作又有各自的实现。可以通过将父类赋值为不同子类来实现。
```



```
批量插入es时，可能由于数据量过大、文件过大、其他原因导致插入中断，es会直接报异常不会返回错误的id，若后续操作依赖于es插入结果决策，则没办法。只能解决文件过大和数据量过大的问题，限制大小或修改配置允许大小，控制发送的数据量来避免错误。
```



```
删除lisadb中重复id_，保留一个。
删除es中无索引数据。
分块插入并检测是否已经存在。
```



### 数量关系

```
四个模块的数量：malware模块,cve模块,pcap模块,apt模块 ,模块必须和list对应
ok --->   malware模块=<malware 数量(malware组件数量（即动态分析，其他类别）+apt模块的malware（木马+....+蠕虫）) 
ok --->   cve模块=<cve数量(cve模块+apt模块)

爬取重复的是不进行提交析的，也不增加数量。若提交一个重复的，则默认不进行提交，但增加数量。避免malware模块超出malware 数量太多。


其他恶意软件为未识别的，不就是malware模块数量?

location
category(malware,cve)
统计malware组件无malware_types个数存入category(elasticsearch)，分类统计时（platform-statistic），别统计其他类别。
architecture
```



```
将dockerfile的每个版本写固定
```



```
平台就仿照https://www.opencti.io/en/吧
pcap用github上star的哪些项目，改一下。cve看opencti是否有
```



```
漏洞统计变少了，是由于新的json覆盖了MD5相同的旧的json，但是新的json格式又不正确，无法得到统计结果，故少了。
json
```



```
pcap,stix2,malware批量
pcapanalyze有错误，uploadstix2有错误
```



```
es可以用hadoop作为存储，spark可以不做统计，而是作为其他模块。看看opencti怎么做的。
```



```
若使用hadoop，则抽取的单个样本信息应存入hbase，因为hbase是数据库，而hive是数据仓库我们需要进行实时聚合统计的查询，hive不适合这种场景。那么此处使用hbase和mysql有什么区别呢。
```



```
部署时需要修改的代码和配置：
清空data,只保留cti_server,threat-broadcast-master对应结构。

1前端的down_loadapi需要替换为宿主机ip，前端宿主机服务器ip也要更为宿主机ip。
2修改ulimit  1048570并重启sysctl_max_heap.修改vm.max_map_count并重启
2修改es的max_result_windows
docker 20.10.7  docker-compose 1.28.5
安装:https://docs.docker.com/engine/install/ubuntu/。
```

### elasticsearch 安装的相关操作

##### ulimit

```
too many open files
blog：https://blog.csdn.net/qq_18298439/article/details/83896777
调节宿主机的文件和进程限制
lsof -p 进程id 查看进程使用的文件
ulimit -a
ulimit -n 4096


vim /etc/security/limits.conf  
* soft nofile 4096  
* hard nofile 4096  

#ulimit  -a，信息解释
core file size          (blocks, -c) 0  #
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 31574
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 10240  #打开文件数量，包括socker链接等。
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 31574
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```

##### vm.max_map_count

```
blog:https://blog.csdn.net/xcc_2269861428/article/details/100186654
报错:max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
vim /etc/sysctl.conf
vm.max_map_count=262144
/sbin/sysctl -p 
```

##### 修改max_result_size

```
PUT policy_document/_settings
{
  "index":{
    "max_result_window":1000000
  }
}
```

### navicate破解版

navicate破解版:http://fankey.blog365.cn/database/129.html

```
threat-broad
nonetype object is not subscriptable
```



```
关于hadoop和elasticsearch的使用场景
es着力于搜索，它在聚合统计上支持很差，性能也很差。一般通过使用painless来完成聚合统计工作，因为它不支持join和中间结果存储、后续分析等。只能使用拼接key值的方法统计聚合字段。例如7.26号的新增人数有多少。
```



```
hadoop慢的原因，在hive仓库中，数据分散在多个小文件中，每个几kb或更大，由于过大的全局统计，进行多次小文件IO，使得开销变大。
```



192.168.31.177

```
Apache Maven 3.6.1
```

python:pcapAnalyze,batchwork,,fournumber,threat-broadcast,

java:esto_mysql,es_provider,lisa_provider,statistic_provider

### 服务在机器上的分布

##### 分布

```
一个前端，两种后端，三个数据库，2爬虫，1沙箱。
hbase1: ngignx前端，pcap的python后端
hbase2:cve爬虫，malware爬虫，malware沙箱，mysql数据库。
lisa：其他java的后端
```



```
一个前端，两种后端，三个数据库，爬虫。
```



| hostname | ip   | 服务                                                         |
| -------- | ---- | ------------------------------------------------------------ |
| hbase    | 187  | esto_mysql                                                   |
| hbase1   | 186  | nginx,pcapAnalyze,batchwork，python后端及nginx前端           |
| hbase2   | 185  | elasticsearch,mysql,lisa,threat-broadcast,fournumber,数据库，沙箱，爬虫 |
| lisa     | 184  | es_provider,lisa_provider,statistic_provider，java后端       |

```
部署时需要修改的参数
1前端代码中的跳转目标，修改为部署的宿主机目标。
2nginx配置文件中的跳转目标，修改为正确目标
3备份的超链接ip要修改
```



```
部署环境有问题时，需要修改代码，在开发环境打包再上传，再运行，麻烦。
1个jar提供一个功能，无法灵活的调试，.........jar包阻碍调试和修改，
想要什么，让打包，上传，运行的过程透明，不手动。
```



```
lisa建表语句，flask插入md5，前端的下载
```



```
统计
```

```
#opencti架构
```



```
1统计
2字段搜索
3下载，多个文件压缩包
4分页,临时解决如下：完全解决使用scroll和scroll-scan
PUT policy_document/_settings
{
  "index":{
    "max_result_window":1000000
  }
}
```



```
Deluge2.03
libTorrent0.13.6
qbittorrent
```



```
CST因代表4个不同的地方，故时区有4个，分别是UT-6:00、UT+9:30、UT+8:00和 UT-4:00。

GMT表示UTC和UT1。
```



```
xs.glgoo.net
```



```
curl -X GET localhost:9200/_cat/indices?v
```

```
#select by month，7 month
SELECT category_id categoryId,category,value,time FROM category_tbl a WHERE a.time IN (SELECT lastday from(SELECT   MAX(time) AS lastday,DATE_FORMAT(time,'%Y-%m') AS subtime FROM category_tbl  GROUP BY subtime ORDER BY lastday DESC LIMIT 7) as lastdaylist)AND a.category IN
(SELECT * FROM (SELECT category FROM category_tbl  GROUP BY category  ORDER BY  count(category) DESC LIMIT 2)AS category_list) ORDER BY category,timexxxxxxxxxx SELECT category_id categoryId,category,value,time FROM category_tbl a WHERE a.time IN (SELECT lastday from(SELECT   MAX(time) AS lastday,DATE_FORMAT(time,'%Y-%m') AS subtime FROM category_tbl  GROUP BY subtime ORDER BY lastday DESC LIMIT 7) as lastdaylist)AND a.category IN(SELECT * FROM (SELECT category FROM category_tbl  GROUP BY category  ORDER BY  count(category) DESC LIMIT 2)AS category_list) ORDER BY category,timeSELECT   MAX(time) AS lastday,DATE_FORMAT(time,'%Y-%m') AS subtime FROM category_tbl  GROUP BY subtime
```

**服务分布**

| hostname | ip   | 服务                                                        |
| -------- | ---- | ----------------------------------------------------------- |
| hbase    | 187  | hdfs-mater,hive,dump_hive,dump_mysql,kafka,zookeeper        |
| hbase1   | 186  | yarn-master,nginx,cve后端，pcap后端                         |
| hbase2   | 185  | elastic,mysql,lisa2,grakn,dump_hdfs,dump_es                 |
| lisa     | 184  | lisa1,es_provider,mapreduce,lisa_submit1,,dump_hdfs,dump_es |

```
/usr/bin/q
```

```
scp   root@hbase1:/root/module/webpages
```

```
年
```

![f0b9a578f02dc6bbd3b5fc08dbb09ac3.png](https://img-blog.csdnimg.cn/img_convert/f0b9a578f02dc6bbd3b5fc08dbb09ac3.png)

```
lisa 得stix文件filenotfound ，字符串多了个""
```



```

全部单个提交到hdfs，存储位置？
mapreduce一次多个信息或一次一个？得到很多独立小文件，然后合并？
dumpmysql较随意
```



```

# Configure logging for testing: optionally with log file
log4j.rootLogger=WARN, stdout
# log4j.rootLogger=WARN, stdout, logfile

log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d %p [%c] - %m%n

log4j.appender.logfile=org.apache.log4j.FileAppender
log4j.appender.logfile.File=target/spring.log
log4j.appender.logfile.layout=org.apache.log4j.PatternLayout
log4j.appender.logfile.layout.ConversionPattern=%d %p [%c] - %m%n
```

```
	81eae37d2085933c9c3052bf53eac459,	91a33cf58ed899057876e80b1e083854这个样本有公网ip结果
	a5c26b11-03a7-4c13-a89e-1b4708aefa62
```



```
一个页面，三个分页
```



```
pip安装stix2，使用清华源。pip安装kafka
复制transfer.py,producer.py
更改tasks.py,添加transfer调用和kafka。
```



```
java请求createfile接口，开始分析文件。
查看pending，fail，success。
success一个report，消息通知python进行获取转换（或者直接耦合进lisa，进行调用）
·····················································································
转换完成一个消息通知java进行提交到hdfs和es
```

```
推荐版本Java8+Hadoop2.7+Spark2.4.5
```

```
chmod +x /etc/rc.d/rc.local
vim /etc/rc.d/rc.local
```

```
提交的malware没写kafka
```

### 平台使用的软件版本

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
timedatectl set-timezone Asia/Shanghai
date  -R
#网络同步时间
ntpdate  pool.ntp.org
#修改时区，文件软连接方式.实际上其所有时间都快了8小时，直接用utc算了，刚好少了8小时。
ln -s /usr/share/zoneinfo/Universal /etc/localtime
#将系统时间写入硬件时间
hwclock -w
#硬件时间写入系统时间
hwclock -s
```

```text
sudo yum -y install ntp
```

```
export HADOOP_HOME=/root/module/hadoop-3.1.4
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
PATH=$PATH:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/bin:/sbin
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib
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
crontab -e
service crond restart

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
1 0  * * *   * i/root/module/dump_hdfs.sh
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

hbase失败重启脚本

```
#!/bin/bash
export HADOOP_HOME=/root/module/hadoop-3.1.4
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
PATH=$PATH:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/bin:/sbin
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib

check_one_service(){
  run=$(ps -ef|grep "$1"|grep -v "grep")
  if [ "$run" ];then
  echo "$1  is alive!"
  else
  echo "$1 was shutdown!"
  echo "Starting $1..."
  eval $2
  echo "$1 was started!"  
  fi
  sleep 5
}
while :
do
#check hdfs,hiveserver2
check_one_service "org.apache.hadoop.hdfs.server.namenode.NameNode" "nohup start-dfs.sh &"
check_one_service "org.apache.hive.service.server.HiveServer2"  "nohup hiveserver2 &"
#check zookeeper,kafka
check_one_service "org.apache.zookeeper.server.quorum.QuorumPeerMain" "nohup /root/module/apache-zookeeper-3.5.9-bin/bin/zkServer.sh start"
check_one_service "kafka" "nohup /root/module/kafka_2.12-2.4.1/bin/kafka-server-start.sh  /root/module/kafka_2.12-2.4.1/config/server.properties &"
#check dump_hive ,dump_mysql
check_one_service "mapreduce_start-1.0-SNAPSHOT.jar"    "nohup java11 -jar  /root/software/mapreduce_start-1.0-SNAPSHOT.jar &"
check_one_service "dump_mysql_jar.jar"  "nohup java11 -jar /root/software/dump_mysql_jar.jar &"
done
```

```
#hbase1
#!/bin/bash
export HADOOP_HOME=/root/module/hadoop-3.1.4
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
PATH=$PATH:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/bin:/sbin
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib

check_one_service(){
  run=$(ps -ef|grep "$1"|grep -v "grep")
  if [ "$run" ];then
  echo "$1  is alive!"
  else
  echo "$1 was shutdown!"
  echo "Starting $1..."
  eval $2
  echo "$1 was started!"  
  fi
  sleep 5
}
while :
do
check_one_service "org.apache.hadoop.yarn.server.resourcemanager.ResourceManager"       "nohup start-yarn.sh &"
check_one_service "nginx"  "nginx"
done
```

hbase2

```
#!/bin/bash
export HADOOP_HOME=/root/module/hadoop-3.1.4
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
PATH=$PATH:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/bin:/sbin
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib

check_one_service(){
  run=$(ps -ef|grep "$1"|grep -v "grep")
  if [ "$run" ];then
  echo "$1  is alive!"
  else
  echo "$1 was shutdown!"
  echo "Starting $1..."
  eval $2
  echo "$1 was started!"  
  fi
  sleep 5
}
while :
do
check_one_service "org.elasticsearch.bootstrap.Elasticsearch"  "su - elas -c  \"nohup elasticsearch  &\" "
check_one_service "mysql" "nohup systemctl start mysqld  &"
check_one_service "grakn" "nohup grakn server start &"
check_one_service "lisa.web_api.tasks" "service docker start && cd /root/module/lisa_/lisa/ && nohup docker-compose up --scale worker=5 &"
done
```

```
#lisa
#!/bin/bash
export HADOOP_HOME=/root/module/hadoop-3.1.4
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
PATH=$PATH:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/bin:/sbin
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib

check_one_service(){
  run=$(ps -ef|grep "$1"|grep -v "grep")
  if [ "$run" ];then
  echo "$1  is alive!"
  else
  echo "$1 was shutdown!"
  echo "Starting $1..."
  eval $2
  echo "$1 was started!"  
  fi
  sleep 5
}
while :
do
check_one_service "es_provider_search8011-0.0.1-SNAPSHOT.jar"  "nohup java11 -jar /root/module/es_provider_search8011-0.0.1-SNAPSHOT.jar  &"
check_one_service "platform_provider_statistic8001-0.0.1-SNAPSHOT.jar" "nohup java11 -jar platform_provider_statistic8001-0.0.1-SNAPSHOT.jar &"
check_one_service "lisa.web_api.tasks" "service docker start && cd /root/module/lisa_/lisa/ && nohup docker-compose up --scale worker=5 &"
done
```

| hostname | ip   | 服务                                         |
| -------- | ---- | -------------------------------------------- |
| hbase    | 187  | esto_mysql                                   |
| hbase1   | 186  | nginx,cve                                    |
| hbase2   | 185  | elasticsearch,mysql,lisa2                    |
| lisa     | 184  | es_provider,lisa_provider,statistic_provider |

| hostname | ip   | 服务                                                        |
| -------- | ---- | ----------------------------------------------------------- |
| hbase    | 187  | hdfs-mater,hive,dump_hive,dump_mysql,kafka,zookeeper        |
| hbase1   | 186  | yarn-master,nginx                                           |
| hbase2   | 185  | elastic,mysql,lisa2,grakn,dump_hdfs,dump_es                 |
| lisa     | 184  | lisa1,es_provider,mapreduce,lisa_submit1,,dump_hdfs,dump_es |

```
lisa在/root/module/lisa_/lisa
```



1   /home/node/paltform_data/sample1

2  /home/node/paltform_data/sample



```
PATH=$PATH:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/bin:/sbin
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib
```



### lisa 构建docker碰到的问题

>重新build遇到的主要问题：
>
>1radare无法下载，到radareorg/radare2下载后COPY的镜像内。linux_images无法下载,到网站下载好COPY进去。对于无法COPY的images，在.dockerignore中注释掉该行。
>
>2无法安装r2pipe，将requirements.txt中版本改为1.5.3，进行build。

```
#lisa-worker的内容,再将requirements.txt中r2pipe2改为1.5.3版本,将.dockerignore中的images注释掉。
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

##### rabbitmq不消费

```
closing AMQP connection <0.13372.1> (172.42.0.10:41766 -> 172.42.0.13:5672):
missed heartbeats from client, timeout: 60s
添加定时重启celery的功能，缓解这种情况。由于它不消费没有什么特征，服务也在跑，只能定时了。
在宿主机执行此命令，启动新的consumer。
在宿主机中杀死无用的消费者，发现自动创建新的消费者开始消费消息队列了。而且docker容器不会消失。但不知道何时阻塞，执行kill。设定定时半小时清除一次。
杀死worker
ps -ef |grep lisa.web_api.tasks|grep -v "grep"|awk '{print $2}'|xargs kill -9
创建lisa worker
sudo docker exec -it $DOCKER_ID /bin/bash -c 'cd /home/lisa && ./docker/worker/init.sh'
```



###### 快照

```
删除了lisa的storage，无成功，是由于恶意类型不支持。
对于消息队列阻塞，进入worker重新启动celery即可。
对于想暂停队列，进入./data/storage，删除所有.随后，消费者会快速调用消息，消息队列会自动清空。
```

##### 结构

```
	REACT build之后，将编译好的文件复制到nginx基础镜像中，路径为/usr/share/nginx/html,让nginx代理.代理ip和端口由docker build的webhost参数决定。
	前端用React编写，编译后由nginx代理。后端由flask编写，使用uwsgi进行代理。tasks运行在worker容器中，在worker中运行qemu，radare2等。worker和api为复用调用关系，api通过celery的远程调用模块，异步调用full_analysis或pcap_analysis.当worker数量增加时，也只用继续监听对应的celery队列，实现多个worker异步调用。
	相比于kafka，它省略了编写生产者和消费者的代码，直接用调用的代码形式，实现了消息的添加和读取。他和rpc还是不同的，rpc是复用调用的关系。当然，它也可以非异步方式调用，就成了rpc类似的，但无必要。它使用形参形式，传递所需要传递的消息。
	但是，为了实现直接本地调用函数名的抽象使用，必须在每台机器都有一份完整的生产者和消费者代码。而传统的消息队列是不需要的。
	实际上api模块只用操作消息队列和数据库，以及存储文件。由worker控制读取。

D:\lisa_docker\lisa\lisa\web_api中为后端的是实现代码，python的flask实现。
@app.route('/api/tasks/create/file', methods=['POST']）提交样本的接口.

routes.py为后端接口，在其中远程调用tasks。
tasks.py为lisa的监听脚本，他会调用r2pipe等处理提交的恶意样本。

查看container是否运行:
进入worker： 
apt-get update
apt-get install procps
ps -aux|grep tasks
进入api：
ps -aux|grep routes
```

处理提交的恶意样本,routes:

```
    task_id = uuid()
    //存储恶意样本
	os.mkdir(f'{storage_path}/{task_id}')
    file_path = f'{storage_path}/{task_id}/{file.filename}'
    file.save(file_path)

    # run analysis
    args = (file_path,)
    kwargs = {'pretty': pretty, 'exec_time': exec_time}
    #通过消息队列远程调用worker执行分析任务。
    tasks.full_analysis.apply_async(args, kwargs, task_id=task_id)
	#返回task_id
    res = {
        'task_id': task_id
    }
    return jsonify(res)
```

监听消息队列

```
@celery_app.task(bind=True, base=LiSaBaseTask)
def full_analysis(self, file_path, pretty=False, exec_time=20):
    """Full sandbox analysis task.

    :param file_path: Path to file.
    :param pretty: Output json indentation.
    :param exec_time: Execution time.
    """
    #刷新消息队列状态
    self.update_state(meta={'filename': os.path.basename(file_path)})

    data_dir = f'{storage_path}/{self.request.id}'
	#调用worker
    # run top level and submodules
    master = Master(file_path, data_dir, exec_time)
    master.load_analyzers()
    master.run()

    output_file = f'{data_dir}/report.json'

    save_output(master.output, output_file, pretty)

    return 'binary'
```

创建analyzer

```
def create_analyzer(analyzer_path, file_path):
    """Imports analyzer class and creates analyzer object

    :param analyzer_path: Path to analyzer in format 'module.Class'.
    :returns: Instantiated analyzer object.
    """
    mod_name, class_name = analyzer_path.rsplit('.', 1)

    analyzer_module = import_module(mod_name)
    analyzer_class = getattr(analyzer_module, class_name)

    return analyzer_class(file_path)
通过字符串，动态引入自己编写的module，并获取类属性。
```

启动r2pipe

```
 def run_analysis(self):
        """Main analysis method.

        :returns: Dictionary containing analysis results.
        """
        log.info('Static Analysis started.')

        # start radare2
        self._r2 = r2pipe.open(self._file.path, ['-2'])
        self._r2.cmd('aaa')

        # binary info
        self._r2_info()

        # strings
        self._load_strings()

        self._r2.quit()
        log.info('Static Analysis finished.')

        return self._output
r2pipe说radare所支持的一个python api。
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









### hive和hbase的sql语句

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





### 平台的mysql数据库的建表语句、crud语句

**category**

```
#建表语句
CREATE TABLE `category_tbl`
(
    `category_id` BIGINT AUTO_INCREMENT,
    `category` VARCHAR(20) NOT NULL,
    `value`  BIGINT NOT NULL,
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

**PCAP**

```
CREATE TABLE `pcap`
(
    `id` BIGINT  AUTO_INCREMENT,
    `filename` VARCHAR(20) NOT NULL,
    `task_id` varchar(20) NOT NULL,
    `date_done` datetime NOT NULL,
    `md5` varchar(100) NOT NULL,
     PRIMARY KEY(`id`)
)ENGINE=InnoDB DEFAULT CHARSET=utf8;
```



**family**

```
#建表语句
CREATE TABLE `family_tbl`
(
    `family_id` BIGINT  AUTO_INCREMENT,
    `family` VARCHAR(20) NOT NULL,
    `value` BIGINT NOT NULL,
    PRIMARY KEY(`family_id`)
)ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

**architecture**

 ```
CREATE TABLE `architecture`
(
    `arch_id` BIGINT AUTO_INCREMENT,
    `architecture` VARCHAR(20) NOT NULL,
    `value` BIGINT NOT NULL,
    PRIMARY KEY(`arch_id`)
)ENGINE=InnoDB DEFAULT CHARSET=utf8;
 ```

**location**

```
CREATE TABLE `location`
(
	`location_id` BIGINT  AUTO_INCREMENT,
	 `location` VARCHAR(20) NOT NULL,
	 `value` BIGINT NOT NULL,
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

服务器密码:240711.wt  111111

3.25：统一中英文location

+
