```
可进行数据的划分，不可以控制分布到哪台机器。
不可进行任务的划分，结点只能施加相同的操作。对于前后依赖的操作，必须等前一个完成，这和（同时施加不相同的操作）不同。yarn能否控制任务分配的机器，是可以的，比如你的某台机器坏了，他会分配给其他机器。但是，你只能向集群这个整体提交作业，无法让yarn只给某个机器单独分配任务。一个map任务到底分为多少个结点任务，是由rdd分区数目决定的。那可以不使用rdd进行map嘛？不行，本来就是用来处理数据的，你没有rdd处理什么数据。这么说，我提交一个任务，分区为1，然后yarn分配是不是就完成了单个机器的分配。那如果跟在它后边再提交一个不依赖于它的任务，是不是就实现了同时执行不相同的任务呢。这样做消耗了哪些资源，时间如何？需要yarn的调度，节点的分发，manager管理等。时间主要花费在爬取，用map爬取会一直维持一个map任务。
```

# pysaprk

```
sc = SparkSession.builder.master("local[1]").appName("myApp").config("spark.executor.memory", "5g").config(
        "spark.driver.memory", "5g").config("spark.driver.maxResultSize", "0").getOrCreate().sparkContext
sc.setLogLevel("INFO")
```



```
1本地编译器调试运行，以setHadoophome形式控制整个逻辑。
2spark和hadoop都以hdfs//或spark//接口形式接受请求和控制作业。这种情况hadoop或spark提供了执行环境，结果写出到本地。python负责编写逻辑，提交到服务端spark后，spark调用python的解释器运行对应的python程序。
```

### 配置使用环境

```
下载spark压缩包解压到本地，配置spark的环境变量。
pip安装pyspark的依赖，测试是否安装成功。
```

### 提交

>关于提交参数:
>
>https://books.japila.pl/apache-spark-internals/tools/spark-submit/#command-line-options_1
>
>https://blog.csdn.net/weixin_42649077/article/details/84976960

```
spark-submit
--master  spark://hbase:7077
--py-files /home/uther/uther/uther.zip
--conf "spark.pyspark.driver.python=""
--conf  "spark.pyspark.python=""
--files 1.zip
--py-files  2.py,3.py
main.py

注意打包时进入要打包的文件夹内，然后，zip -r test.zip ./*。再外部打包会多一层文件夹，导致无法访问。
访问路径
--py-files用于提交多个单一的py文件
--files用于提交打包的zip
访问路径类似如下：
对于zip包内的test文件夹下的1.json文件,在python程序中这样访问
open('test/1.json','r') as f

对于提交的zip中的py文件，要使用如下方式访问
对于test.zip根下的util.py：
from util  import start
```

```
#提交jar包
pyspark --packages com.johnsnowlabs.nlp:spark-nlp_2.11:2.5.5
此操作会去repo1.maven.org下载对应jar包和依赖。
```

### spark-submit .sh

```
spark-submit 
--master spark://172.18.65.187:7077
--py-files /root/software/spark-elephas/spark_elephas.zip 
--conf "spark.pyhspark.driver.python=/root/miniconda3/envs/elephas1/bin/python" 
--conf "spark.pyspark.python=/root/miniconda3/envs/elephas1/bin/python"  
/root/software/spark-elephas/myModel.py
```

### 创建sparkcontext

```
SparkSession.builder.getOrCreate().sparkContext
```

### 从list创建rdd

```
pairs = [(x, y) for x, y in zip(features, labels)]
sc.parallelize(pairs)
```

### DataFrame

```
创建规则化行列，之后才可以执行sql语句。
```

##### 创建dataframe的方法

```
https://blog.csdn.net/weixin_39198406/article/details/104916715
dataframe的表现力有限，只适合部分的逻辑。对某些操作不方便，如非结构化数据，或有其他复杂的计算。它必须基于表进行操作，所以表达复杂的逻辑智能借助创建中间表来表示。
```

### RDD

>RDD中操作分为Transformation,Action。对于Transformation是累积的，当有Action执行时才会执行累积的Transformation。Action常见的有collect，map，reduce，groupby等。
>map是对一个数组的单个对象重复调用map函数内容。
>reduce是将多个对象递归调用，最终归约为一个值。

```
rdd的缺点是，无法控制分布的细节。如，无法指定特定机器获取特定的数据，这些都有spark进行了高度封装，没有暴露给用户，用户只能像编写单机程序一样编写程序。
```

##### mappartion和foreach等

```
mappartion从单个节点上的所有数据角度编写函数，每个分区调用一次。foreach从rdd的单个元素角度编写处理函数，每个元素调用一次。使用dataframe的select从整个rdd角度编写程序，整个rdd调用一次，select如何在节点间迁移数据，是shuffle嘛。
```



### 常用函数

```
函数清单:https://blog.csdn.net/qq_32595075/article/details/79918644
```

##### 层级

```
mappartion从单个节点上的所有数据角度编写函数，每个分区调用一次。
foreach从rdd的单个元素角度编写处理函数，每个元素调用一次。
使用dataframe的select从整个rdd角度编写程序，整个rdd调用一次。
```

```
broadcast 介绍:https://blog.csdn.net/weixin_42155006/article/details/118517464
发送一个变量到每个节点， parameters = rdd.context.broadcast(parameters)，适用于变量大的情况。如果不使用,直接使用对应变量，对于同一个节点也不能复用。
```

##### map

```
在mapreduce中map对于每个Filesplit执行一次,Filesplit默认为一个key值执行一次。
```

##### flatmap

```
相比map，flatmap会将结果中所有数组展开，flat为一个统一的数组。
```







##### 创建

```
rddMine=sc.parallelize(list，3)
list是列表，可以存储任意类别的多个元素。
3是分区数，list的数据被分散在3个分区上。list中共用len(list)个元素，单个元素是划分到分区的最小的单位。
rddMine.mapPartion(train).collect
mapPartion为每个partion上的数据分别调用train函数，train函数定义时只能有一个参数，即rdd。
```

### problem

```
Exception: Python in worker has different version 3.9 than that in driver 3.7, PySpark cannot run with different minor versions. Please check environment variables PYSPARK_PYTHON and PYSPARK_DRIVER_PYTHON are correctly set.
worker,driver的python版本不匹配，但是两者不都是本机嘛。
在环境变量中指定PYSPARK_PYTHON，PYSPARK_DRIVER_PYTHON，值设定唯python的位置.或在代码中使用config("PYSPARK_PYTHON","/anaconda/env/env_name/python")
```



# 安装

>spark-3.1.2-bin-hadoop3.2,默认we端口8080,绑定失败会用8081端口。
>
>从官网下载spark的压缩包，其不区分平台无需编译，说明是由java语言和scala编写的源码或字节码或jar包，然后由java虚拟机进行跨平台编译执行。
>进入bin目录，pyspark即可启动交互命令行和webui，其是由java编写的程序，带有web程序后端和html。

````
安装教程:http://spark.apache.org/docs/latest/
````

### 本地模式

```
./bin/run-example SparkPi 10
./bin/spark-shell --master local[2]
```

### standalone模式

```
mv slaves.template slaves
vim slaves
#修改从属节点
linux1
linux2
linux3
mv spark-env.sh.template spark-env.sh
#配置java路径和master,此处的7077相当于hadoop3中内部通信的8020，要确保hadoop配置。
export JAVA_HOME=/java
SPARK_MASTER_HOST=linux1
SPARK_MASTER_PORT=7077
xsync spark-standlone
sbin/start-all.sh
进入http://linux1:8080观看
```

##### 配置历史服务器

```
mv spark-defaults.conf.template spark-defaults.conf
vim spark-default.conf
spark.eventLog.enabled    true
spark.eventLog.dir        hdfs://linux1:8020/directory

sbin/start-dfs.sh
hadoop fs -mkdir /directory
#修改spark-env.sh
export SPARK_HISTORY_OPTS="
-Dspark.history.ui.port=18080
-Dspark.history.fs.logDirectory=hdfs://linux1:8020/directory
-Dspark.history.retainedApplications=30
"

xsync conf
sbin/start-all.sh
sbin/start-history-server.sh
```

##### 配置高可用HA

>提供高可用，借助zookeeper，启用备用Master，解决单点故障问题。

```
vim spark-env.sh  #注释掉端口，master是由zookeeper挑选的，故而不配置了。
#SPARK_MASTER_HOST=linux1
#SPARK_MASTER_PORT=7077

#因为8080与zookeeper冲突
SPARK_MASTER_WEBUI_PORT=7077

export SPARK_DAEMON_JAVA_OPTS="
-Dspark.deploy.recoveryMode=ZOOKEEPER
-Dspark.deploy.zookeeper.url=linux1,linux2,linux3
-Dspark.deploy.zookeeper.dir=/spark/
"
xsync conf/

sbin/start-all/sh
```

### yarn模式集群

>官网:http://spark.apache.org/docs/latest/running-on-yarn.html
>3.1.2_version:https://spark.apache.org/docs/3.1.2/spark-standalone.html
>
>下载spark-3.0.0-bin-hadoop3.2.tgz并解压，注意要对应hadoop和spark版本。

```
#修改yarn-site.xml
<property>
 <name>yarn.nodemanager.pmem-check-enabled</name>
 <value>false</value>
</property>
<!--是否启动一个线程检查每个任务正使用的虚拟内存量，如果任务超出分配值，则直接将其杀掉，默认
是 true -->
<property>
 <name>yarn.nodemanager.vmem-check-enabled</name>
 <value>false</value>
</property>
```



```
修改spark-env.sh
vim spark-en.sh
export JAVA_HOME=/opt/java
export YARN_CONF_DIR=..../
./bin/spark-submit --class org.apache.spark.examples.SparkPi \
    --master yarn \
    --deploy-mode cluster \
    --driver-memory 4g \
    --executor-memory 2g \
    --executor-cores 1 \
    examples/jars/spark-examples*.jar \
    10
```

##### 配置历史服务

```
#在standlone模式的历史服务器配置基础上，添加如下
vim spark-defaults.conf
spark.yarn.historyServer.address=linux1:18080
spark.history.ui.port=18080

sbin/start-history-server.sh
```

### hive配置spark引擎

```
#修改hive-site.xml并重启
<property>
<name>execute.engine</name>
</property>
```



### scala安装

```
下载scala包或在idea的project structure中指定自动下载，在setting的plugins中下载scala插件。
创建maven项目，在project structure中添加scala。
sprk3.0.0,scala-2.12
```

# 理论知识

### spark运行架构

>spark框架采用标准的master-slave的结构，Driver即master，Executor即slave。
>
>其核心组件包括Driver、Executor

##### Driver

```
1将用户程序转化为job
2在Executor之间调度任务
3跟踪Executor的执行情况
4通过UI展示查询运行情况
```



##### Executor

```
是集群工作节点中的一个JVM进程，负责在Spark作业中运行具体任务，任务彼此之间相互独立。Executor负责执行任务并返回给驱动进程，通过自身的Block Manager为RDD提供内存式存储，RDD是直接缓存在Executor进程内的。
```

##### Master&Worker

>spark自带的资源管理器，类似于rm和nm。

### Spark核心编程

##### 开发依赖pom

>scala的编译依赖，spark_core对应的scala版本。

```
 <dependencies>
        <!-- https://mvnrepository.com/artifact/org.apache.spark/spark-core -->
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-core_2.12</artifactId>
            <version>3.1.2</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/net.alchim31.maven/scala-maven-plugin -->
        <dependency>
            <groupId>net.alchim31.maven</groupId>
            <artifactId>scala-maven-plugin</artifactId>
            <version>3.2.2</version>
        </dependency>
</dependencies>
#在idea的project structure的Global Libraries中导入scala-sdk-2.12.11
```

##### RDD

>Resilient distributed Dataset,弹性分布式数据集，他代表一个弹性的、不可变的、可分区的、里边的元素可并行计算的集合。 他是计算逻辑的封装，当需要新的操作时，只能再创建新的RDD，不可修改。

###### getPartion函数

```
分区列表，用于执行任务是并行计算
```



###### compute函数

```
使用分区函数对每一个分区进行计算
```

###### getDependencies

```
当需要将多个计算模型组合时，就需要将多个RDD建立依赖关系
```

###### partioner

```
为KV类型数据时，通过设定分区器自定义数据的分区
```





### 执行流程

>https://blog.csdn.net/u010745505/article/details/81261019

```
注意：action操作才会触发sparkContext的runjob方法。然后进行stage和task的划，zau进行资源分配和任务调度。这个rdd上的过程是不可控的...，而mapreduce是可以通过mapreduce进行控制的，但mapreduce局限了表达方式。就算用rdd，rdd也只是用来传递数据和通信吧。实际上mapreduce和rdd控制的是
1、Driver程序的代码运行到action操作，触发了SparkContext的runJob方法。 
2、SparkContext调用DAGScheduler的runJob函数。 
3、DAGScheduler把Job划分stage，然后把stage转化为相应的Tasks，把Tasks交给TaskScheduler。 
4、通过TaskScheduler把Tasks添加到任务队列当中，交给SchedulerBackend进行资源分配和任务调度。 
5、调度器给Task分配执行Executor，ExecutorBackend负责执行Task。
```

### spark运行原理，从提交一个jar到最后返回结果，整个过程

```
Driver项rm申请资源，rm分配资源后在合适的机器上启动AM，AM再申请资源启动Executor，Excutor注册到Driver，Driver开始执行，划分任务并分发。
```

# spark用法

### spark java api

##### RDD创建

>makeRdd底层调用了parallize

```
#从集合创建
val sparkConf=new SparkConf().setMaster("local[4]").setAppName("RDDLearn")
val sparkContext=new SparkContext(sparkConf)
val rdd1=sparkContext.parallelize(List(1,2,3,4))
val rdd2=sparkContext.makeRDD(List(1,2,3,4))
rdd1.collect().foreach(println)
rdd2.collect().foreach(println)
sparkContext.stop()
```

```
#从文件创建
val rdd:RDD[String]=sparkContext.textFile("input")
```

##### RDD并行度与分区

```
#设定partion数量为4
spark.text.makeRDD(List(1,2,3,4),4)
```

```
#分区源码
def positions(length: Long, numSlices: Int): Iterator[(Int, Int)] = {
 (0 until numSlices).iterator.map { i =>
 val start = ((i * length) / numSlices).toInt
 val end = (((i + 1) * length) / numSlices).toInt
 (start, end)
 }
```

##### RDD转换算子

### spark sql

### spark streaming

### spark shell

##### 交互shell

```
#启动
bin/spark-shell
sc.textFile("data/word.txt").flatMap(_.split("")).map((_,1)).recudeByKey(_+_).collect
#退出
:quit
```

##### 提交应用

```
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master local[2] \
./example/jars/spark-examples_2.12-3.0.0.jar \
10
#常用参数
--executor-memory 1G ，每个executor使用的内存
--total-executor-cores 2 ，所有executor使用的cpu核数
--executors , 每个executor使用的cpu核数
application-jar ，打包好的jar，此url全局可见，比如hdfs。
application-arguments，jar包该类main方法所需要的参数
```

