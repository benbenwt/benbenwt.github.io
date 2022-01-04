```
可进行数据的划分，不可以控制分布到哪台机器。
不可进行任务的划分，结点只能施加相同的操作。对于前后依赖的操作，必须等前一个完成，这和（同时施加不相同的操作）不同。yarn能否控制任务分配的机器，是可以的，比如你的某台机器坏了，他会分配给其他机器。但是，你只能向集群这个整体提交作业，无法让yarn只给某个机器单独分配任务。一个map任务到底分为多少个结点任务，是由rdd分区数目决定的。那可以不使用rdd进行map嘛？不行，本来就是用来处理数据的，你没有rdd处理什么数据。这么说，我提交一个任务，分区为1，然后yarn分配是不是就完成了单个机器的分配。那如果跟在它后边再提交一个不依赖于它的任务，是不是就实现了同时执行不相同的任务呢。这样做消耗了哪些资源，时间如何？需要yarn的调度，节点的分发，manager管理等。时间主要花费在爬取，用map爬取会一直维持一个map任务。
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

```
rdd的缺点是，无法控制分布的细节。如，无法指定特定机器获取特定的数据，这些都有spark进行了高度封装，没有暴露给用户，用户只能像编写单机程序一样编写程序。
```

##### mappartion和foreach等

```
mappartion从单个节点上的所有数据角度编写函数，每个分区调用一次。foreach从rdd的单个元素角度编写处理函数，每个元素调用一次。使用dataframe的select从整个rdd角度编写程序，整个rdd调用一次，select如何在节点间迁移数据，是shuffle嘛。
```



##### 创建

```
rddMine=sc.parallelize(list，3)
list是列表，可以存储任意类别的多个元素。
3是分区数，list的数据被分散在3个分区上。list中共用len(list)个元素，单个元素是划分到分区的最小的单位。
rddMine.mapPartion(train).collect
mapPartion为每个partion上的数据分别调用train函数，train函数定义时只能有一个参数，即rdd。
```

### 创建sparkcontext

```
SparkSession.builder.getOrCreate().sparkContext
```

从list创建rdd

```
pairs = [(x, y) for x, y in zip(features, labels)]
sc.parallelize(pairs)
```

### problem

```
Exception: Python in worker has different version 3.9 than that in driver 3.7, PySpark cannot run with different minor versions. Please check environment variables PYSPARK_PYTHON and PYSPARK_DRIVER_PYTHON are correctly set.
worker,driver的python版本不匹配，但是两者不都是本机嘛。
在环境变量中指定PYSPARK_PYTHON，PYSPARK_DRIVER_PYTHON，值设定唯python的位置.或在代码中使用config("PYSPARK_PYTHON","/anaconda/env/env_name/python")
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



### 关于安装

```
从官网下载spark的压缩包，其不区分平台无需编译，说明是由java语言和scala编写的源码或字节码或jar包，然后由java虚拟机进行跨平台编译执行。
进入bin目录，pyspark即可启动交互命令行和webui，其是由java编写的程序，带有web程序后端和html。
```

### spark提交远程jar包

```
pyspark --packages com.johnsnowlabs.nlp:spark-nlp_2.11:2.5.5
此操作会去repo1.maven.org下载对应jar包和依赖。
```

### pysaprk

```
sc = SparkSession.builder.master("local[1]").appName("myApp").config("spark.executor.memory", "5g").config(
        "spark.driver.memory", "5g").config("spark.driver.maxResultSize", "0").getOrCreate().sparkContext
sc.setLogLevel("INFO")
```



```
1本地编译器调试运行，以setHadoophome形式控制整个逻辑。
2spark和hadoop都以hdfs//或spark//接口形式接受请求和控制作业。这种情况hadoop或spark提供了执行环境，结果写出到本地。python负责编写逻辑，提交到服务端spark后，spark调用python的解释器运行对应的python程序。
```

##### 提交

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

##### spark-submit .sh

```
spark-submit 
--master spark://172.18.65.187:7077
--py-files /root/software/spark-elephas/spark_elephas.zip 
--conf "spark.pyhspark.driver.python=/root/miniconda3/envs/elephas1/bin/python" 
--conf "spark.pyspark.python=/root/miniconda3/envs/elephas1/bin/python"  
/root/software/spark-elephas/myModel.py
```



### spark安装

>spark-3.1.2-bin-hadoop3.2,默认we端口8080,绑定失败会用8081端口。

````
安装教程:http://spark.apache.org/docs/latest/
````

##### 本地模式

```
./bin/run-example SparkPi 10
./bin/spark-shell --master local[2]
```

##### standalone模式

```
复制压缩包解压即可
sbin/start-master.sh
netstat -lntp
curl localhost:8081
```

##### 不使用yarn等集群，开启集群

```
sbin/start-master.sh   
curl  http://hbase:8081
sbin/start-worker.sh  spark://hbase:7077
```



##### yarn模式集群

```
官网:http://spark.apache.org/docs/latest/running-on-yarn.html
3.1.2_version:https://spark.apache.org/docs/3.1.2/spark-standalone.html
```

```
vim spark-en.sh
export HADOOP_CONF_DIR=..../
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



### scala安装

```
下载scala包或在idea的project structure中指定自动下载，在setting的plugins中下载scala插件。
创建maven项目，在project structure中添加scala。
sprk3.0.0,scala-2.12
```



##### RDD

```
RDD中操作分为Transformation,Action。对于Transformation是累积的，当有Action执行时才会执行累积的Transformation。Action常见的有collect，map，reduce，groupby等。
map是对一个数组的单个对象重复调用map函数内容。
reduce是将多个对象递归调用，最终归约为一个值。
```

