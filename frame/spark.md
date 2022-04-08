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

### 变量广播

>哪些变量需要手动广播？哪些会自动传递过去，在任意节点都可以使用？
>
>自定义函数，变量，数据

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
1下载scala包或在idea的project structure中指定自动下载，
2在setting的plugins中下载scala插件。
3创建maven项目，在project structure中添加scala。
4pom中引入spark3.0.0,scala-2.12及scala打包插件
 <dependencies>
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
```

# 理论知识

## spark运行架构

>spark框架采用标准的master-slave的结构，Driver即master，Executor即slave。
>
>其核心组件包括Driver、Executor

### Driver

```
1将用户程序转化为job
2在Executor之间调度任务
3跟踪Executor的执行情况
4通过UI展示查询运行情况
```



### Executor

```
是集群工作节点中的一个JVM进程，负责在Spark作业中运行具体任务，任务彼此之间相互独立。
Executor负责执行任务并返回给驱动进程，通过自身的Block Manager为RDD提供内存式存储，RDD是直接缓存在Executor进程内的。
```

### Master&Worker

>spark自带的资源管理器，类似于rm和nm。

### spark通用运行流程

>1任务提交后，启动driver
>
>2随后driver向集群管理器注册应用程序
>
>3之后集群管理器根据此任务的配置文件分配Executor并启动
>
>4driver开始执行main函数，spark查询为懒执行，当执行到Action算子时开始反向推算，根据宽依赖进行Stage的划分，随后一个Stage对应一个Taskset，Taskset中有很多Task，查找可用的Executor进行调度。
>
>5根据本地化原则，task发往指定的Executor执行。

### spark部署模式

>1Standlone:spark原生的集群管理工具
>
>2hadoop yarn，统一的资源管理工具，在上面可以运行多套计算框架，如MR，Storm等。根据driver在集群中的位置不同，分为yarn client和yarn cluster。

#### yarn cluster模式

>注意几个关键组件：YARN的AM,NM,RM,yarn服务器，yarn客户端        spark：sparksubmit进程，Executor，Driver
>
>sparksubmit->yarn->spark

>1执行脚本提交任务，实际就是启动一个SparkSubmit的JVM进程
>
>2SparkSubmit类中main方法反射调用YarnClusterApplication的main方法
>
>3YarnClusterApplication创建Yarn客户端，然后向Yarn服务端发送指令：bin/java  Applicationmaster;
>
>4Yarn框架收到指令后会在指定的NM中启动ApplicationMaster
>
>5**ApplicationMaster启动Driver线程，执行用户的作业**
>
>6AM 向RM注册，申请资源
>
>7获取资源后AM向NM发送指令，bin/java YarnCoarseGrainedExecutorBackend
>
>8CoarseGrainedExecutorBackend进程会接收消息，跟Driver通信，注册已经启动点Executor，然后启动Executor等待接收任务。
>
>9Driver继续执行作业调度
>
>10dirver分配任务并监控任务的执行。

#### yarn client模式

>AM  ->Driver->Executer

>1提交脚本提价任务，实际是启动一个SparkSubmit的JVM进程
>
>2**SparkSubmit类中的main方法反射调用用户代码的main方法**
>
>3**启动Driver进程**，执行用户的作业，并创建ScheduleBackend
>
>4YarnClientSchedulerBackend向RM发送指令：bin/java ExecutorLauncher
>
>5Yarn框架收到指令后会在指定的NM中启动ExecutorLauncher（实际上还是调用ApplicationMaster的main方法）
>
>6AM向RM注册，申请资源
>
>7获取资源后AM向NM发送指令：bin/java CoarseGrainedExecutorBackend
>
>8.CoarseGrainedExecutorBackend进程会接收消息，
>
>9.Dirver分配任务并监控任务的执行

#### yarn cluster和yarn client的区别

>AM  ->Driver->Executer这个过程
>
>yarnclient模式中，dirver运行在提交的client机器上，client不能关闭，类似于在windows下的idea编写程序提交到远程集群；
>
>而cluster模式driver运行在集群中，client在提交后可以关闭，类似于使用spark-submit的cluster模式提交到远程集群。
>
>yarn cluster适合生产环境 ，由于driver在提交端，所以executor会和driver端交流信息， yarn client适合交互和调试。

## Spark通讯架构

>每个节点有一个RpcEndpoint，Spark针对每个节点都称之为一个RPC终端，且都实现RpcEndpoint接口。每个节点都有inbox用于接收所有来自其他RpcEndpoint的消息，而对于每一个需要发送消息的其他RpcEndpoint，该节点为其创建一个outbox，每个outbox对应一个RpcEndpoint，将需要发送的消息放入outbox。

## Spark任务调度概述

>当Driver起来后，Driver则会根据用户程序逻辑准备任务，并根据Executor资源情况逐步分发任务。spark应用中的四个概念：
>
>1.application，对应于一个spark-submit 
>
>2.job：以action为分界，遇到一个action算子就触发一个job
>
>3.stage是job的子集，以RDD宽依赖为界限，遇到shuffle做一次划分
>
>4.task是stage的子集，以并行度衡量，分区数多少就多少个task

>spark任务调度有两路，一路是stage级别的并行，一路是task级别的调度。实际上一个stage包含多个RDD，一个RDD有包含多个分区，这样连接起来。

#### DAG

>Spark RDD通过其Transactions操作，形成了RDD血缘关系图，即DAG，最后通过Action的调用，触发Job并调度执行，执行过程中会创建两个调度器：DAGScheduler，TaskScheduler。
>
>DAGScheduler负责Stage级的调度，主要是将job切分成若干个Stages，并将每个Stage打包成TaskSset交给TaskScheduler。
>
>TaskScheduler负责Task级别的调度，将DAGScheduler处理好的TaskSet按照指定的调度策略分发到Executor上执行，调度过程中SchedulerBackend负责提供可用资源，其中SchedulerBackend有多种实现，分别对接不同的资源管理系统。

>Driver初始化SparkContext过程中，会分别初始化DAGScheduler，TaskScheduler，SchedulerBackend以及HeartbeatReceiver，并启动SchedulerBackend以及HeartbeatReceiver。SchedulerBackend通过AM申请资源，并从TaskScheduler拿到task分发到executor执行。HeartbeatReceiver负责接收executor的心跳信息，监控Executor的存活状态，并通知TaskScheduler。

#### spark stage级调度

>spark的任务调度是从DAG切割开始的，主要是由DAGScheduler来完成。当遇到一个Action后触发job，并交DAGScheduler来提交，下图是job提交的相关方法。
>
>sparkcontext将job交给DAGScheduler提交，他会根据RDD的血缘关系构成DAG进行切分，将一个job划分为若干个Stages，具体策略是，由最终的RDD不断通过依赖回溯判断是不是宽依赖，就是一个由后向前的深度优先搜索算法。即以shuffle为分界，划分stage，窄依赖的RDD之间被划分到同一个Stage中，可以进行pipline计算。stage分为两类，一类是resultstage，就是action算子所处的stage。另一类是shuffleampstage，为下游stage做准备。

>一个stage是否被提交，取决于父stage是否执行完毕。如果没有父stage就从该stage开始提交。stage提交时会将task等信息序列化打包为taskset交给taskscheduler，一个partition对应一个task，另一方面taskscheduler会监控stage的运行状态，只有executor丢失或task由于fetcj任务失败才需要重新提交stage。
>
>相对来说，DAGScheduler做的事情相对简单，仅仅在stage层面划分DAG，提交stage并监控相关状态信息。TaskScheduler则相对较为复杂。

#### Spark Task 级调度

>spark task的调度是由taskScheduler来完成的，由前文可知，DAGScheduler将Stage打包交给TaskScheTaskSetduler，TaskScheduler会将Taskset封装到TaskSetManager加入到队列。TaskSetManager负责管控同一个stage的tasks，TaskScheduler就是以TaskSetmanager为单元来调度任务。
>
>前边提到，taskScheduler初始化后会启动schedulerBackend，他负责跟外界打交道，接收Executor的注册信息，并维护Executor的状态，所以说schedulerBackend是管理外界沟通的，负责询问taskscheduler是否有task要处理，并分发task到对应资源。

#### 调度策略

>​	TaskScheduler支持两种调度策略，一种是FIFO，也就是默认的调度策略，另一种是FAIR。TaskScheduler初始化会实例化rootPool，表示树的根节点，是Pool类型。
>
>1FIFO调度策略
>
>2FAIR调度策略：FAIR模式有一个rootPool和多个子Pool，各个子Pool中存储着所有待分配的TaskSetManager。在FAIR模式中，需要先对Pool进行排序，再对子Pool里的TaskSetMagager进行排序。

##### FAIR调度策略

>通过三个关键参数确定如何调度，runningTasks，minShare，weight。minshare、weight是在fairscheduler.xml中指定，调度池会读取此文件。
>
>1比较runningTasks和minshare，如果小，则小的比大的先执行。
>
>2如果runningtask都比minshare小，那么比较runningtaks和minshare的比值，谁小排前边。
>
>3比较runningtasks和weight的比值，小的排前边。

#### 本地化调度

>本地化调度的目的是，让数据更少的在网络上发送，占用太多时间。spark调度尽量让task处于最高级别，如果最高级别长时间不可用，会尝试降低级别，迁就一下。
>
>共有五个优先级别：
>
>1PROCESS_LOCAL,进程本地化，task和数据在同一个Executor中，性能最好
>
>2NODE_LOCAL,节点本地化，task和数据在同一个节点中，但是不在同一个executor
>
>3RACK_LOCAL，机架本地化，task和数据在同一个机架的两个节点上，数据需要通过网络在节点上进行传输
>
>4NO_PREF，对于task来说，从哪里都可以。
>
>5ANY,任何地方，甚至跨网络，性能最差。

## Spark Shuffle解析

>这几个的主要区别，是否保存了很多最终文件，是否进行排序操作。

### HashShuffle解析

##### 未优化的HashShuffle解析

>每个task都分成分区个数的文件，那么一共生成了task_num*partition_num的数据。reduce会来拉取子集分区的文件。

##### 优化的HashShuffle

>优化后的启用合并机制，就是复用buffer，spark.shuffle.consolidateFiles。该参数默认值为false，将其设置为true即可开启。
>
>开启后，所有task相同的key都输出到一个buffer。然后把buffer写入以core数量为单位的本地文件。也就是core_nums*partition_num

### SortShuffle

#### 普通SortShuffle

>数据先写入一个数据结构，一边聚合一边写入内存，到达阈值写入磁盘文件。类似mapreduce的shuffle。

#### bypass SortShuffle

>1当shuffle reduce task数量小于等于spark.shuffle.sort.bypassMergeThreshold参数的值，默认200
>
>2不是聚合类的shuffle算子，如reducebykey，然后为每个task创建磁盘文件，通过溢写放入对应的磁盘文件，最后merge多个磁盘文件。但是，该步骤与sortShuffle以及mapreduce不同，它不进行排序。

## spark内存管理

### 堆内和堆外内存规划

>作为一个JVM进程，Executor的内存管理建立在JVM的内存管理之上，Spark对JVM的堆内空间进行了更详细的分配，以充分利用内存。同事，spark引入了堆外内存，使之可以在工作节点的系统内存中开辟空间，进一步优化了内存的使用。堆内内存受到了JVM统一管理，堆外内存直接向操作系统进行内存的申请和释放。

#### 堆内内存

>由executor-memory参数或spark.executor.memory参数配置。executor内运行的并发任务共享JVM堆内存，这些任务在缓存RDD数据或广播数据时占用的内存被规划为存储内存，而这些任务在执行shuffle时使用的内存被规划为执行内存，剩余的部分不做特殊规划，那些spark内部的对象实例，用于定义的实例，使用剩余的空间。
>
>spark只是记录，实际的对象实例的内存申请和释放由JVM完成。由于spark无法准确地估计需要的内存大小，而且其标记为释放的对象，可能还没有被gc处理，这可能导致实际内存与spark预估的不同，造成oom。

#### 堆外内存

>引入堆外内存是为了可以在工作节点的系统空间中开辟空间，存储经过序列化的二进制数据。
>
>堆外内存可以精准的申请和释放，而且序列化对象的内存你可以精准计算。默认情况不启用，通过spark.memory.offHeap.enabled启用，通过spark.memory.offHeap.size设置大小。

### 内存空间分配

#### 静态内存管理

>存储内存、执行内存、和其他内存都都是固定的

#### 统一内存管理

>执行内存与存储内存共享同一块空间，两者动态占用对方的空闲区域。

### 存储内存管理

>存储内存管理RDD，RDD是只读的集合。task启动之初，需要读取分区，判断分区是否已经在内存中或磁盘中，若没有需要检查checkpoint或按照血统重新计算。对于多次使用的RDD，可以常见persist或cache，提升后续访问性能。
>
>RDD持久化由spark的storage模块负责，实现了RDD与物理存储的解耦合。Storage模块负责管理spark在计算过程中产生的数据，将那些在内存或磁盘存取数据的功能封装好。实现时，dirver的blockmanager为master，executor的blockmanager为slave，storage以block为单位存储，RDD的每个分区经过处理后对应一个Block。Driver端的Master负责整个spark应用程序的block的元数据信息的管理和维护。 

>存储级别是以下几个属性的组合，1存储位置，2是否序列化3是否副本

#### RDD缓存过程

>RDD在缓存到存储内存以前，Partition中的数据一般以迭代器的数据结构来访问。通过iterator逐个访问记录，这些记录逻辑上占用了堆内的other空间。通过unroll将partition由不连续的转换为连续的内存。

#### 淘汰与落盘

>由于同一个Executor的所有计算任务共享有限的存储空间，当有新的Block需要存储，而LinkedHashMap中的旧Block进行淘汰，而被淘汰的block有存储到磁盘要求时，要进行落盘，否则直接删除。
>
>淘汰规则：
>
>1被淘汰的与新的memory模式相同，即同为堆外或堆内
>
>2新旧Block不能属于同一个RDD，避免循环淘汰。
>
>3旧block不能处于被读状态
>
>4遍历LinkedHashMap中的block，采用LRU算法。LRU是linkedhashmap自带的属性。

### 执行内存管理

#### shuffle write

>暂时不背

#### shuffle read

>暂时不背

## RDD

### RDD序列化

#### 闭包检查

>从计算的角度，算子以外的代码都在Driver端执行，算子里边的代码都在Executor端执行。那么在scala的函数式编程中，就会导致算子内经常会用到算子外的数据，这样就形成了闭包的效果，如果使用的算子外的数据无法序列化，就意味着无法传值给Executor端执行，就会发生错误，所以需要在执行任务计算以前，检测闭包对象是否可以进行序列化操作，这个操作称为闭包检测。

#### 序列化方法和属性

>从计算的角度，算子以外的代码都是在Driver端执，算子里边的代码都是在Executor端执行。

#### Kryo序列化框架

>Java 的序列化能够序列化任何的类。但是比较重（字节多），序列化后，对象的提交也 比较大。Spark 出于性能的考虑，Spark2.0 开始支持另外一种 Kryo 序列化机制。Kryo 速度 是 Serializable 的 10 倍。
>
>注意：即使使用 Kryo 序列化，也要继承 Serializable 接口。

### RDD依赖关系

### RDD血缘关系

>RDD只支持粗粒度转换，即在大量记录上执行的单个操作。将创建RDD的一系列Lineage（血统）记录下来，以便恢复丢失的分区。RDD的Lineage会记录RDD的元数据信息和转换行为。当该RDD的部分分区数据丢失时，它可以根据这些信息来重新运算和恢复丢失的数据分区。

### RDD依赖关系

>这里所谓的依赖关系就是两个RDD之间的关系

#### RDD窄依赖

>窄依赖表示每一个父（上游）RDD的partition最多被子RDD的一个Partition使用，窄依赖我们形象的比喻为独生子女。

```
class OneToOneDependency[T](rdd: RDD[T]) extends NarrowDependency[T](rdd) 
```



#### RDD宽依赖

>宽依赖表示同一个父上游RDD的Partition被多个子RDD的Partition依赖，会引起Shuffle，总结：宽依赖我们形象的比喻为多生。

```
class ShuffleDependency[K: ClassTag, V: ClassTag, C: ClassTag](
 @transient private val _rdd: RDD[_ <: Product2[K, V]],
 val partitioner: Partitioner,
 val serializer: Serializer = SparkEnv.get.serializer,
 val keyOrdering: Option[Ordering[K]] = None,
 val aggregator: Option[Aggregator[K, V, C]] = None,
 val mapSideCombine: Boolean = false)
 extends Dependency[Product2[K, V]] 
```



### RDD阶段划分

>DAG有向无环图是由点和线组成的拓扑图形，该图形具有方向，不会闭环。例如，DAG记录了RDD的转换过程和任务的阶段。
>
>当使用map，mappartition时，不会发生shuffle，只会有窄依赖，不需要多个分区互相等待。但是，使用groupby、reducebyKey时，分区需要重新组合，形成了宽依赖，导致多个分区需要互相等待，然后进入下一个阶段，生成新的RDD。
>
>每当有一个Shuffle依赖时，就增加一个阶段，也就是发生的宽依赖个数加1。

### RDD任务划分

>RDD任务切分中间分为：Application，Job，Stage和Task，这次个每一个由前往后都是1对n的关系。
>
>Application：初始化一个SparkContext即生成一个Application
>
>Job：一个Action算子就会生成一个Job
>
>Stage：Stage等于宽依赖的个数加1
>
>Task:一个Stage阶段中，最后一个RDD的分区个数就是Task的个数。

### RDD持久化

#### RDD Cache缓存

>RDD通过Cache和Persist方法将前边的计算结果缓存，默认情况下会把·数据以缓存在JVM的内存中。但是并不是两个方法调用时立即缓存，而是触发后面的action算子时，该RDD将会被缓存在计算节点的内存中，并供后面重用。

```
#cache操作会增加血缘关系，不改变原有的血缘关系
println(wordToOneRdd.toDebygString)
#数据缓存
wordToOneRdd.cache()
#可以更改存储级别
mapRdd.persist(StorageLevel.MEMORY_AND_DISK_2)
```

>缓存有可能丢失，或者存储于内存的数据优于内存不足而被删除，RDD的缓存容错机制保证了及时缓存丢失也能执行。基于RDD的一系列转换，丢失的数据会被重新计算，由于RDD的各个Partition是相对独立的，因此只需要计算丢失的部分即可。

#### RDD checkpoint检查点

>将RDD中间结果写入磁盘
>
>由于血缘依赖过长会造成容错成本过高，这样就不如在中间阶段做检查点容错，如果检查点最后有节点出现问题，可以从检查点开始重做血缘，减少开销。设置的检查点需要action算子触发。

```
sc.setCheckpointDir("./checkpoint1")
```

#### 缓存和检查点区别

>1缓存只会将数据存储，不切断血缘关系，后边的rdd仍然能找到自己的血缘，checkpoint切断血缘
>
>2cache数据存储在磁盘、内存。检查点一般存储在hdfs等文件系统。
>
>3建议对checkpoint的rdd使用Cache缓存，这样checkpoint的job只需要从Cache缓存中读取数据即可，否则需要从头计算。

### RDD分区器

>hash分区，range分区，自定义分区



## Spark核心编程

##### 开发依赖pom

>scala的编译依赖，spark_core对应的scala版本。

```java
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
#在idea的project structure的Global Libraries中导入scala-sdk-2.12.11，并在setting->plugins中搜索scala，按装此插件
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



## mr的shuffle和spark的shuffle有什么区别稍微讲一下

## 执行流程

>https://blog.csdn.net/u010745505/article/details/81261019

```
注意：action操作才会触发sparkContext的runjob方法。然后进行stage和task的划，进行资源分配和任务调度。这个rdd上的过程是不可控的...，而mapreduce是可以通过mapreduce进行控制的，但mapreduce局限了表达方式。就算用rdd，rdd也只是用来传递数据和通信吧。实际上mapreduce和rdd控制的是
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

### spark scala api

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

>RDD转换算子数量很多，根据执行操作不同，分为map类，reduce类，分区类、聚合函数类、集合操作类。

>RDD根据数据处理方式不同，分为Value类型、双Value类型和Key-Value类型。

###### Value类型

>value类型包括map、mappartion

###### map

>对每一条rdd元素调用函数

###### mapPartion

>以分区上的所有数据为单位进行处理，传入传出参数都是一个迭代器。

###### mapPartionWithIndex

>与mapPartion相比，可以多传入一个参数index，可以在函数内获取分区编号。

>mapPartitionsWithIndex详解https://blog.csdn.net/yilulvxing/article/details/98968750
>
>mapPartitionsWithIndex通过额外传递一个int类型的index给mappartion函数，是的在mapPartion内部可以打印分区id。
>
>也可以通过如下函数在mapPartion调用函数内部打印partionId。
>
>```
>TaskContext.getPartitionId()
>```



###### flatMap

>将列表扁平化后，再进行处理。

###### groupBy

>分组，一个组的数据一定在同一个分区

###### filter

>数据过滤，此操作不改变分区，但是可能造成数据倾斜。

###### sample

>根据规则从数据集中抽取数据，第一个参数抽取是否放回、第二个用于比较的数值，小于此参数则要，否则不要。第三个参数，随机数种子。

###### distinct

>去重

###### coalesce

>缩减分区

###### repartion

>重新分区

###### sortby

>排序，需要shuffle。按照f函数处理，然后按照处理结果排序。

```
val dataRDD = sparkContext.makeRDD(List(
 1,2,3,4,1,2
),2)
val dataRDD1 = dataRDD.sortBy(num=>num, false, 4)
```

###### glom

>将同一个分区的数据转换为内存数组

###### 双value类型

>输入未两个数值

###### intersection

>求交集

###### union

>两个RDD并集

###### substract

>集合的差集，去除重复元素，保留其他元素

###### zip

>取第一个RDD的key，第二个RDD的key对应的value值，组成新的key、value

###### key-value类型

>多个输入，reduce、聚合等

###### reduceByKey

>将数据按照key相同的对value进行聚合

###### groupByKey

>将数据按照key对value进行分组，相比于reduceByKey，groupByKey只是对数据进行分组，而没有进行聚合。reduceByKey可以借助combine在shuffle之前进行预聚合，所以聚合性能更好。

###### aggregateByKey

>将数据根据不同的规则进行分区内计算和分区间计算
>
>它具有两个参数列表，第一个参数列表表示分区内的计算规则，第二个参数列表表示分区间的计算规则。

```
val resultRDD=rdd.aggregateByKey(10)(
	(x,y)=> math.max(x,y)
	(x,y)=> x+y
)
```

###### foldByKey

>当分区内和分区间计算规则相同时，使用此函数

###### combineByKey

>相当于aggregateByKey的区内部分

###### sortByKey

>在一个<K,V>的RDD上调用，K必须实现Ordered接口，返回一个按照key排序的RDD

###### join

>使用相同key连接两个RDD，将(K,V)(K,W)组成(K,(V,W))

###### leftOuterJoin

>左外连接，即以左表为基表，保留左表的空值行。

###### cogroup

>将(K,V)(K,W)组成(K,(Iterable<V>,Iterable<W>))

###### treeReduce算子

>源码详解：https://blog.csdn.net/jiang_jinyue/article/details/59109837

###### treeAggregate算子

>

##### RDD Action算子

>有很多action算子用于从rdd中取出数据，不触发额外的计算，如take，first，takeOrdered，collect
>
>还有部分算子触发操作，常见的操作有reduce函数，聚合函数，如count，reduce，aggregate，fold，countByKey

###### reduce

>聚合RDD中的所有元素

###### collect

>以数据形式返回所有元素

###### count

>返回RDD中元素个数

###### first

>返回RDD中的第一个元素

###### take

>返回前n个元素

###### takeOrdered

>排序后返回前n个

###### aggregate

>分区的数据通过初始值和分区内的数据进行聚合，然后再和初始值进行分区间的数据聚合

###### fold

>折叠操作，aggregate 的简化版操作

###### countByKey

>统计每种 key 的个数

###### save

```scala
def saveAsTextFile(path: String): Unit
def saveAsObjectFile(path: String): Unit
def saveAsSequenceFile(
 path: String,
 codec: Option[Class[_ <: CompressionCodec]] = None): Unit
```

###### foreach

>分布式遍历 RDD 中的每一个元素，调用指定函数

### spark sql

>三者都是分布式弹性数据集**Resilient** 
>
>RDD相比DataFrame不支持sql操作，一般与mlib一起使用。DataFrame是指定了列名的，可以通过列名访问。
>
>DataFrame是Dataset的一个特例，其类型为Dataset[Row]。两者都支持sql操作，比如select，groupby。

#### DataFrame

##### 创建DataFrame

```scala
#查看支持的数据源格式
spark.read
val df=spark.read.json("data/user.json")
```

##### SQL语法

```
#对DataFrame创建一个临时表，这样可以使用sql进行操作
df.createOrReplaceTempView('user')
val sqlDF=spark.sql("SELECT * FROM user")
sqlDF.show()
#创建全局表
df.createGlobalTempView("user")
spark.sql("SELECT * FROM global_temp.user").show()
```

##### DSL语法

>domain-specific language，DSL语法用于管理结构化数据，可以使用scala、java、python等编写DSL语法语句，无需创建临时视图使用sql了。

```
#等同于sql的select语句
df.select($"username",$"age" + 1).show
df.select('username, 'age + 1).show()
#
df.filter($"age">30).show
df.groupBy("age").count.show
```

##### RDD转换为DataFrame

>在IDEA开发程序时，如果需要将RDD于DF和DS之间互相操作，需要import spark.implicits._

```
val idRDD=sc.textFile("id.txt")
idRDD.toDF("id").show
#DataFrame转RDD
df.rdd
```

#### Dataset

>Dataset是具有强类型的数据集合，需要提供对应的类型信息。

##### 创建Dataset

```
#使用样例类序列创建DataSet
case class Person(name:String,age:Long)
val caseClassDS=Seq(Person("zhangsan",2)).toDS()
caseClassDS.show
#使用基本类型的序列创建DataSet
val ds=Seq(1,2,3,4,5).toDS
ds.show
#RDD转DataSet
ds1=sc.makeRDD(List(("zhangsna",30),("lisa",60))).map(t=>User(t._1,t._2)).toDS
#Dataset转RDD
ds1.rdd
#DataFrame和DataSet转换
val df=sc.makeRDD(List(("zhangsan"，30)))
val ds=df.as[User]
VAL df=ds.toDF
```

#### 用户自定义函数

>可以使用spark.udf添加自定义函数

```
spark.udf.register("addName",(x:String)=>"Name:"+x)
df.createOrReplaceTempView("people")
spark.sql("Select addName(name),age from people").show()
```

#### 数据保存和加载

```
#数据加载通用方式，format设置类型：csv，jdbc，json，parquet
load：在csv，jdbc，json，parquet格式下需要传入数据的路径
option：在jdbc格式下，需要传入jdbc相应参数，url、user、password和dbtable
spark.read.format("…")[.option("…")].load("…")

#保存数据
#查看
df.write.
df.write.format("")[.option("...")].save("...")

```

>core-site.xml   hdfs-site.xml  mapred-site.xml  yarn-site.xml
>
>hadoop-env.sh,yarn-env.sh,mapred-env.sh
>
>workers

### spark streaming

#### wordcount例子

```
package com.wt.learn

import org.apache.spark.streaming.{Seconds, StreamingContext}
import org.apache.spark.{SparkConf, SparkContext}

object StreamWordCount {
  def main(args: Array[String]):Unit={
    val sparkConf=new SparkConf().setMaster("local[*]").setAppName("streaming")
    val ssc=new StreamingContext(sparkConf,Seconds(3))
    val lineStreams=ssc.socketTextStream("66.66.66.66",9999)

    val wordStreams=lineStreams.flatMap(_.split(" "))
    val wordAndOneStreams=wordStreams.map((_,1))
    val wordAndCountStreams=wordAndOneStreams.reduceByKey(_+_)

    wordAndCountStreams.print()

    ssc.start()
    ssc.awaitTermination()
  }
}
nc  -lk 9999
hello world
```

#### DStream创建

##### RDD队列

>测试过程中，可以通过ssc.queueStream(queueOfRDDs)创建DStream，每一个推送到这个队列的RDD，都会作为一个DStream处理。

```
package com.wt.learn

import org.apache.spark.SparkConf
import org.apache.spark.rdd.RDD
import org.apache.spark.streaming.{Seconds, StreamingContext}

import scala.collection.mutable

object RDDStream {
  def main(args:Array[String]):Unit={
    val conf=new SparkConf().setMaster("local[*]").setAppName("RDDStream")
    val ssc=new StreamingContext(conf,Seconds(4))

    val rddQueue=new mutable.Queue[RDD[Int]]()

    val inputStream=ssc.queueStream(rddQueue,oneAtATime = false)
    val mappedStream=inputStream.map((_,1))
    val reducedStream=mappedStream.reduceByKey(_+_)

    reducedStream.print()

    ssc.start()

    for(i<-1 to 5)
      {
        println(i)
        rddQueue+=ssc.sparkContext.makeRDD(1 to 300, 10)
        Thread.sleep(2000)
      }
    ssc.awaitTermination()
  }
}

```

#### 自定义数据源

>需要继承Receiver类，并实现onStrat、onStop方法来自定义数据源采集。

#### Kafka数据源

>通过sparkStreaming连接上kafka特定的topic，并将数据读取出来进行计算。

```
package com.wt.learn

import org.apache.kafka.clients.consumer.{ConsumerConfig, ConsumerRecord}
import org.apache.spark.SparkConf
import org.apache.spark.streaming.dstream.{DStream, InputDStream}
import org.apache.spark.streaming.kafka010.{ConsumerStrategies, KafkaUtils, LocationStrategies}
import org.apache.spark.streaming.{Seconds, StreamingContext}

object DirectAPI {
  def main(args: Array[String]): Unit = {
    val sparkConf:SparkConf=new SparkConf().setAppName("ReceiverWordCount").setMaster("local[*]")
    val ssc=new StreamingContext(sparkConf,Seconds(3))

    val kafkaPara:Map[String,Object]=Map[String,Object](
      ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG->
      "hbase:9092,hbase1:9092,hbase2:9092",
      ConsumerConfig.GROUP_ID_CONFIG->
      "atguigu",
      "key.deserializer"->
      "org.apache.kafka.common.serialization.StringDeserializer",
      "value.deserializer"->
      "org.apache.kafka.common.serialization.StringDeserializer"
    )
    val kafkaDStream:InputDStream[ConsumerRecord[String,String]]=KafkaUtils.createDirectStream[String,String](ssc,LocationStrategies.PreferConsistent,ConsumerStrategies.Subscribe[String,String](Set("atguigu_topic"),kafkaPara))

    val valueDStream:DStream[String]=kafkaDStream.map(record=>record.value())
    valueDStream.flatMap(_.split(" ")).map((_,1)).reduceByKey(_+_).print()

    ssc.start()
    ssc.awaitTermination()
  }
}

#查看goup id为atguigu的kafka消费情况
bin/kafka-consumer-groups.sh --describe --bootstrap-server hbase:9092 --group  atguigu


```

### DStream转换

>DStream与RDD类似，也分为Transformations转换和Output Operations输出两种。

#### 无状态转换

##### transform

>执行任何自定义的函数，本质每一批次调度一次，其实也是对DSream应用转换

##### join

>两个流之间的join需要两个流的批次大小一直，这样才能做到同时触发计算。触发时，两个流中的各自的RDD进行join，与两个RDD的join效果相同。

#### 有状态转换

>指定如何根据之前的状态和来自输入流的新值对状态进行更新。

##### UpdateStateByKey

>它会传入两个参数，第一个参数是当前时间区间的状态，第二个参数是之前的状态

```
#生成一个新的DStream，为每个时间区间对应的（键，状态）对组成的。
#自定义函数，foldLeft的第一个参数是默认初始值值，相当于累加的初始sum。Some表示有类型，不是None，scala中option的子类有some和none。
val updateFunc = (values: Seq[Int], state: Option[Int]) => {
     val currentCount = values.foldLeft(0)(_ + _)
    val previousCount = state.getOrElse(0)
     Some(currentCount + previousCount)
 }
#调用
val stateDstream = pairs.updateStateByKey[Int](updateFunc)
```



#### WindowOperations

>由两个参数决定，窗口时长是计算内容的时间范围，滑动步长是隔多久触发一次计算。

### DStream输出

#### print

>在驱动节点上打印最开始10个元素

#### save

>saveAsTextFiles
>
>saveAsObjectFiles
>
>saveAsHadoopFiles

#### foreachRDD

>对于每个到来的批次

### 案例实操

>广告黑名单：将每天对某个广告点击超过100次的用户统计出来，将其保存到mysql作为黑名单。
>
>广告点击量：统计每天各地区各城市各广告的点击总流量，并将其存入MySQL
>
>最近一小时广告点击量
>
>这两个能称为实时streaming？都以天为单位进行处理了。

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

