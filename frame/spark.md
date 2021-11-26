```
异步数据并行每步迭代需要先从参数服务器获取参数，计算完
梯度后还要将梯度值传回参数服务器。相比单机训练，异步数据并行需要额外的通信开
销，因而无法达到线性加速比。假设工作者每步迭代平均耗时为𝑇�
�
，平均计算开销为𝑇�

平均通信耗时为𝑇

，计算开销占比定义为： 

通信开销占比定义为： 

```

### RDD

```
rdd的缺点是，无法控制分布的细节。如，无法指定特定机器获取特定的数据，这些都有spark进行了高度封装，没有暴露给用户，用户只能像编写单机程序一样编写程序。
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
#本地编译器调试运行，以setHadoophome形式控制整个逻辑。
spark和hadoop都以hdfs//或spark//接口形式接受请求和控制作业。
这种情况hadoop或spark提供了执行环境，结果写出到本地。
python负责编写逻辑，提交到服务端spark后，spark进行
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

