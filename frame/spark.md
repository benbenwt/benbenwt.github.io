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



### spark安装

>spark-3.1.2-bin-hadoop3.2,默认we端口8080,绑定失败会用8081端口。

````
安装教程:http://spark.apache.org/docs/latest/
````

##### 本地模式

```
./bin/run-example SparkPi 10
./bin/sprk-shell --master local[2]
```

##### standalone模式

```
复制压缩包解压即可
sbin/start-master.sh
netstat -lntp
curl localhost:8081
```

##### yarn模式集群

```
官网:http://spark.apache.org/docs/latest/running-on-yarn.html
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

