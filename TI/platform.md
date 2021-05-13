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



### lisa

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
ps -ef |grep lisa.web_api.tasks|grep -v "grep"|awk '{print $2}'|xargs kill -9
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

