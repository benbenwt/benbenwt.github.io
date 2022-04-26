# 安装

## 集群搭建

>解压安装包
>
>配置jobmanager
>
>workers
>
>bin/start-cluster.sh

## 开发环境

```
    <properties>
        <flink.version>1.13.0</flink.version>
        <java.version>1.8</java.version>
        <scala.binary.version>2.12</scala.binary.version>
        <slf4j.version>1.7.30</slf4j.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-java</artifactId>
            <version>${flink.version}</version>
        </dependency>

        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-streaming-java_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>

        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-clients_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
```

# Flink 理论

>其在实时处理上有着独特的优势，当单个任务抵达时，立刻就执行该任务。相比于spakr streaming的秒级延迟，flink可以达到毫秒级别，提供了exact-once语义

## lambda架构

>批处理通过积攒一批数据，然后一起执行，保证数据的完整性，提高海量数据的处理能力，但是其完全的牺牲了实时性。
>
>第一代流处理框架，如storm则是为了实时性完全牺牲了高吞吐和精确消费。
>
>很自然的一个想法是，将两者结合，既能达到实时性，又能处理大量历史数据。这就是lambda架构，比如使用离线数仓统计以天为粒度的指标，而用实时数仓统计当日的实时数据，基于这两个统计结果，我们可以计算获得当日的实时数据，批处理层也会定时处理离线数仓中的数据，提升数据实时性。但是这样提高了工作量，相当于用离线数仓和实时数仓实现了两套完全相同语义的系统。

## flink运行时架构

### 系统架构

>两大组件：作业管理器和任务管理器
>
>对于提交的任务jm是管理者master，而tm是工作者worker，slave。
>
>客户端在负责将代码转换为数据流图，并生成jobgraph，发送给jobmanager。提交之后，任务就和客户端无关了，客户端可以选择断开，也可以继续保持链接。-d参数用于detached mode，即断开链接。
>
>taskmanager启动后，jobmanager会与它建立链接，并将作业图转换为可执行的执行图分发给taskmanager，然后就由taskmanager具体执行任务。

#### JobManager

>每个应用应该被唯一的jobmanager控制。
>
>jobmanager包含三个组件：jobmaster，resourceManager，Dispatcher

##### jobmaster

>负责处理单独得到job，所以jobmaster和具体的job是一一对应的，如一个应用中有多个job，那么每个job都有一个jobmaster。应用一般指jar包、数据流图、作业图等。
>
>jobmaster会把jobgraph转换为一个物理层面的数据流图，这个图叫做执行图，它包含了所有可以并发执行的任务。jobMaster会向资源管理器发出请求，申请任务必要的资源。获取到资源后，将执行图发送到具体的taskmanager

##### resourceManager

>负责资源调配，资源主要指taskmanager的任务槽。任务槽是flink集群中资源调配单位，包含机器用来执行的一组cpu和内存资源。每个人任务都需要分配到一个slot上执行。

##### Dispatcher

>提供REST接口，用来提交应用，并为每个作业启动一个新的jobmaster。Dispatcher不是必须组件

#### TaskManager

>taskManager是flink的工作进程，数据流的具体计算就是它来做的，被称为worker。flink集群中至少有一个taskmanager，每个tm包含一定数量的任务槽。slot是任务资源调度的最小单位，slot限制了tm的并行处理的任务数量。
>
>启动之后，taskmanager会向资源管理器注册它的slots；收到资源管理器的指令后，taskmanger就会将一个或多个槽位提供给jobmaster调用。

### 作业提交流程

#### 高级抽象视角

>在宏观角度，提交的大体流程如下：
>
>1客户端将作业图通过dispathcer的rest接口，发送到jobmanager
>
>2分发器启动jobmaster，并提交作业图给jobmaster
>
>3jobmaster将作业图接卸我i执行图，得到所需的资源数量，然后请求slots
>
>4资源管理器判断是否有足够资源，没有，就启动新的taskmanager
>
>5taskmanager启动后，向resourcemanager注册自己的可用任务槽
>
>6资源管理器通知taskmanager为新的作业提供slots
>
>7taskmanager连接到对应的jobmaster，提供slot
>
>8jm发送任务给tm
>
>9tm执行，tm之间可以交换数据

#### standalone模式

>standalone模式只有会话模式和应用模式，会话模式jobmaster是预先启动，应用模式jobmaster则是在作业提交时启动。
>
>两种模式的taskmanager数量都是固定的，不会去创建新的taskmanager。

#### yarn模式

##### 会话模式

>在会话模式下，我们需要启动一个yarn session，这个会话创建一个flink集群。
>
>通过yarn session会启动jobmanager，而taskmanager需要根据资源动态的启动，在jobmanager内部，由于还没有提交作业，所以只有resourceManager和dispatcher在运行。

##### 单作业模式

>单作业模式，flink集群不会预先启动，而是在提交作业时，才启动新的jobmanager。
>
>通过yarn管理器创建一个jobmanager，jobmanager中的jobmaster会与资源管理器、yarn资源管理器进行交流，让其创建taskmanager，并反向注册到资源管理器，然后提供给jobmaster使用slot插槽。

### 关键概念

#### 数据流图

>它的程序结构就是定义了一连串的操作，每一个数据输入后都调用每一步计算。
>
>这与mapreduce有些不同，mr是对于输入的整的元素，进行编程，而flink处理的对象是一个元素。
>
>flink中每一个操作叫做算子operator，所以程序就是算子构成的管道，数据就像是水流有序流过。所有的flink程序由三部分构成：source，transformation和sink。
>
>source表示源算子，负责读取数据源
>
>transformation表示转换算子，利用各种算子进行处理加工
>
>Sink表示下沉算子，负责数据的输出。
>
>提交之后在flink网页可以看到数据流图

#### 并行度

>spark通过划分stage、task进行分配任务，并将task进行并行执行。
>
>而对于flink这样的流式引擎没有必要划分stage，因为数据是源源不断到来的，一个操作完成就发往下一个操作结点，不会出现stage中的shuffle阶段，宽依赖的分区需要等待其他数据。spark基于mapreduce的思想是数据不动代码动，那么flink就是代码不动数据流动，这是因为流式数据是连续到来的，不会像批处理一样传输所有数据。

##### 并行子任务

>为了提高数据并行的吞吐量，将一个算子任务拆分成多个并行的子任务subtasks，并发往不同的结点。
>
>在flink执行过程中，每个operator算子可以包含一个或多个子任务operator subtask，这些子任务在不同的线程、不同的物理机或不同的容器中完全独立的执行。

>一个特定算子的子任务个数被称为并行度，这样包含了多个子任务的算子就是并行数据流，它需要多个分区。一个流的并行度，可以认为是其所有算子中最大的并行度，不同算子可以有不同的并行度。

##### 并行度的设置

```
#可以根据setParallelism方法设置并行度
stream.map(x->x).setParallelism(2)
env.setParallelism(2)
#提交应用时设置
bin/flink run –p 2 –c com.atguigu.wc.StreamWordCount 
./FlinkTutorial-1.0-SNAPSHOT.jar
#配置文件
parallelism.default: 2
```

#### 算子链

>web ui界面给出的数据流图会把多个任务连接在一起，形成一个大的任务。
>
>这实际上涉及到一对一、充分区的问题，类似shuffle

##### 一对一

>one-to-one,
>
>map，filter，flatmap算子都是one-to-one的

##### 重分区

>数据流分区变化，Redistributing
>
>如keyBy/window算子，会根据策略不同，架构数据发送到不同的下游目标任务，例如不同算子的并行度发生改变。

##### 合并算子链

>对于一对一的算子操作，可以连接起来形成一个大的任务，每个task被一个线程执行。这样可以减少线程之间的切换和数据交换，可以减少时延提升吞吐量。

#### 作业图与执行图

>图的生成顺序：逻辑流图->作业图->执行图->物理图
>
>逻辑流图和作业图不涉及子任务并行度的划分，比较粗略。
>
>作业图相比逻辑流图会进行合并算子链，如keyed aggregation和输出算子sink。
>
>执行图会画出详细的数据在子任务间如何流动。
>
>物理图相比执行图，进一步确定数据存放的位置和收发方式

#### 任务和任务槽

>任务槽表示taskmanager拥有计算资源的一个固定大小的子集，这些资源用来独立执行一个子任务。
>
>同一个taskmanage的多个slot运行在同一个jvm中，共享数据集和数据结构，降低了隔离的级别，减少了任务开销
>
>对于不同算子的子任务，它们之间可以共享slot，可以将一连串的程序处理的任务都放在一个slot中，叫做运行管道。这样允许将资源密集型任务和非密集型任务同时运行在一个slot中，让计算任务平均分配。并且，一个slot中包含了所有的一连串操作，那么即使一个tm宕机，其他也能正常工作。由于同一个任务的子任务不可共享slot，那么很明显执行作业所需的slot数量正好就是作业中所有算子的并行度最大值。

```
taskmanager.numberOfTaskSlots: 8
#希望某个算子独立
.map(word->word).slotSharingGroup("1")
```

#### 任务槽和 并行度

>parallelism.default指定程序运行时的并行度，但是其不一定能成功，如果slot比并行度小，则无法分配资源。
>
>slot是程序的最大可用并行度，是静态的。

## 部署模式

>flink提供了会话模式，session mode
>
>单作业模式，per-job mode
>
>应用模式，application mode

### 会话模式

>先启动一个集群然后通过客户端提交作业到集群，提交的作业竞争集群中的资源。这样的好处是，集群的生命周期是超越于作业之上的，作业结束就释放资源，集群依然运转。
>
>缺点：资源是共享的，所以资源不足时，作业就会失败。同时，taskManager可能运行了很多作业，如果其中一个发生故障导致TaskManager宕机，那么所有作业都会收到影响。
>
>会话模式比较适合于单个规模小、执行时间短的大量作业

### 单作业模式

>单作业模式可以更好的隔离资源，我们可以考虑为每一个提交的作业启动一个集群，也就是所谓的单作业模式。作业完成集群就关闭，释放资源。即便它的taskmanager发生故障，也不会影响其他作业。这些特性使得单作业模式在生产环境更加稳定，是实际应用的首选模式。flink本身无法直接这样运行，所以单作业需要借助一些工具，如yarn，kubernetes。

### 应用模式

>此模式不适用客户端，而是为每个应用启动一个jobmanager，也就是创建一个集群。执行结束时jobmanager关闭。单作业模式通过客户端提交，客户端解析出每一个作业对应一个集群。而应用模式下，直接由jobmanager执行，即使一个应用包含了多个job，也只创建一个集群。

## 独立模式 Standalone

### 会话模式

```
bin/start-cluster.sh
bin/flink run  -m hbase:8081  -c com.wt.wc.WordCount  ./Flink-wordcount.jar
```

### 单作业模式

>flink本身无法直接以单作业方式启动，一般需要借助一些资源管理平台。所以flink的独立集群不支持单作业模式部署。

### 应用模式

```
cp ./flink-wordcount.jar ./lib
./bin/standalone-job.sh start  --job-classname com.atguigu.wc.StreamWordCount
bin/taskmanager.sh start
```

### 高可用

```
vim flink-conf.yaml
high-availability: zookeeper
high-availability.storageDir: hdfs://hadoop102:9820/flink/standalone/ha
high-availability.zookeeper.quorum: 
hadoop102:2181,hadoop103:2181,hadoop104:2181
high-availability.zookeeper.path.root: /flink-standalone
high-availability.cluster-id: /cluster_atguigu

vim masters
hbase1:8081
lisa:8081

#需要在每台机器配置hadoop classpath
#需要启动hdfs集群和zookeeper集群
#分别访问两个master的页面
#查看谁是leader
get   /flink-standalone/cluster_atguigu/leader/rest_server_lock
```

## YARN模式

>session
>
>per-job
>
>application

```
vim flink-conf.yaml
jobmanager.memory.process.size: 1600m
taskmanager.memory.process.size: 1728m
taskmanager.numberOfTaskSlots: 8
parallelism.default: 1
```

### 会话模式

>会话模式首先申请一个YARN会话来启动flink集群。

#### 1启动集群

>启动hadoop集群，并执行命令向yarn申请资源，开启yarn会话，启动flink集群
>
>bin/yarn-session.sh  -nm  test
>
>参数解析：
>
>-d：分离模式，不想让flink yarn运行在前台，可以使用这个参数
>
>-jm(--jobManagerMemory):配置jobmanager所需内存，默认单位为MB
>
>-nm(--name):配置yarn ui界面上显示的任务名
>
>-qu(--queue):指定YARN队列名
>
>-tm(--taskManager)：配置每个TaskManager所使用内存。

#### 2提交作业

>1通过webui提交作业
>
>2通过命令行提交作业
>
>将打包好的jar包上传到集群，执行以下命令：
>
>bin/flink run -c  com.atguigu.wc.StreamWordCount FlinkTutorial-1.0-SNAPSHOT.jar
>
>3任务提交后可以在webui查看

### 单作业模式

>在yarn环境中，我们可以直接向yarn提交一个作业，从而启动flink集群
>
>1提交作业
>
>bin/flink  run -d -t yarn-per-job -c  com.atguigu.wc.StreamWordCount  FlinkTutorial-1.0-SNAPSHOT.jar
>
>2在yarn页面查看applicatinmaster，flink web ui也可以查看
>
>3可以取消作业：
>
>bin/flink list -t yarn-per-job -Dyarn.application.id=application_XXXX_YY
>
>bin/flink cancel -t yarn-per-job -Dyarn.application.id=application_XXXX_YY  jobId

### 应用模式

>1提交作业
>
>bin/flink run-application -t yarn-application -c com.atguigu.wc.StreamWordCount  FlinkTutorial-1.0-SNAPSHOT.jar
>
>2在命令行查看和取消作业
>
>./bin/flink list -t yarn-application -Dyarn.application.id=application_XXXX_YY $ 
>
>./bin/flink cancel -t yarn-application  -Dyarn.application.id=application_XXXX_YY 
>
>3yarn.provided.lib.dirs配置选项指定位置，将jar上传到远程。
>
>./bin/flink run-application -t yarn-application -Dyarn.provided.lib.dirs="hdfs://myhdfs/my-remote-flink-dist-dir" hdfs://myhdfs/jars/my-application.jar

## 容错机制

### JobManager

### TaskManager

## 其他八股文

### flink怎么实现exactly once（几乎是flink必问问题）

### **flink和spark streaming的区别**

### **详细说一下flink checkpointing吧，最好底层一些**

# Flink 用法

## Flink Data Stream API 

>基本流程
>
>1获取执行环境
>
>2读取数据源
>
>3定义基于数据的转换操作
>
>4定义计算结果的输出位置
>
>5触发程序执行

### 执行环境

#### 1getExecutionEnvironment

>此种方式，根据运行方式返回正确的运行环境
>
>比如，如果你是在本地运行，那么它就返回本地执行环境。如果是提交到集群那么它就返回集群的执行环境

```
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
```

#### 2createLocalEnvironment

```
#创建本地执行环境
StreamExecutionEnvironment localEnv = StreamExecutionEnvironment.createLocalEnvironment();
```

#### 3createRemoteEnvironment

```
StreamExecutionEnvironment remoteEnv = StreamExecutionEnvironment
 .createRemoteEnvironment(
 "host", // JobManager 主机名
 1234, // JobManager 进程端口号
 "path/to/jarFile.jar" // 提交给 JobManager 的 JAR 包
);
```

#### 执行模式

>分为三种：批执行模式、自动模式、流执行模式
>
>虽然流处理能处理所有的批处理任务，但是有时，批处理比流处理更高效，如wordcount中，流处理会对每条输入进行计算并输出，但是批处理作为整体进行处理，最终再输出，我们并不关心中间的结果。

```
#batch，命令行配置
bin/flink run -Dexecution.runtime-mode=BATCH ...
StreamExecutionEnvironment env = 
StreamExecutionEnvironment.getExecutionEnvironment();
env.setRuntimeMode(RuntimeExecutionMode.BATCH);
```

#### 触发程序执行

>env.execute();

### 源算子 Source

```
#从集合中读取数据
env.fromCollection(clicks)
env.fromElements(new Person("bob",12),new Person("alice",11))
#从文件读取数据
DataStream<String> stream=env.readTextFile("clicks.csv")
相对路径使用系统属性user.dir获取路径；idea下是project的根目录，standlone模式下是集群节点根目录。
也可以使用hdfs：//，但是需要引入依赖
<dependency>
 <groupId>org.apache.hadoop</groupId>
 <artifactId>hadoop-client</artifactId>
 <version>2.7.5</version>
 <scope>provided</scope>
</dependency>

#从socket读取数据
env.socketTextStream("localhost",7777);

#从kafka读取数据,需要引入依赖，然后调用addSource传入对象实例
<dependency>
 <groupId>org.apache.flink</groupId>
 <artifactId>flink-connector-kafka_${scala.binary.version}</artifactId>
 <version>${flink.version}</version>
</dependency>
env.addSource(new 
FlinkKafkaConsumer<String>(
 "clicks",
 new SimpleStringSchema(),
 properties
 ));

```

#### 自定义source

>自定义source需要实现sourceFunction接口。主要重写两个关键方法：run（）和cancel（）
>
>run（）方法：使用运行时上下文对象sourceContext向下游发送数据
>
>cancel（）方法：通过标示位控制退出循环，达到中断数据流的效果。

#### Flink支持的类型系统

>flink的数据类型使用TypeInformation统一描述所有类型，是类型描述符的基类。

#### Flink支持的数据类型

>对于常见的java或scala类型，flink都是支持的
>
>flink对不同类型进行了划分，在Types类中可以找到

##### 基本类型

>Void，String，Date，BigDecimal，BigInteger

##### 数组类型

>基本数据类型和对象数据

##### 复合数据类型

>java元组类型：这是flink内置的元组类型，是java api的一部分。最多25个字段。
>
>scala样例类及元组：不支持空字段
>
>行类型：可以认为是具有任意个字段的元组，且支持空字段
>
>POJO：Flink自定义的类似于java bean
>
>POJO有一些要求：1类是public的独立的，即没有非静态的内部类2有公共的无参构造方法3类中所有字段都是public且非final，或者有公共的getter和setter，这些方法要符合java bean命名规范。

>辅助类型：option，either，list，map
>
>泛型类型：如果没有按照pojo定义的类，被当做泛型处理。

#### 类型提示

>returns用于进行类型提示，因为对于map里传入的lambda表达式，系统只能推断出是tuple2类型，无法得到tuple2<String，Long>。只有显示的告诉系统返回类型，才能正确解析数据。



### 转换操算子Transformation

#### 基本转换算子

##### map

>用于将数据流中的数据进行转换，形成新的数据流。

```
#可以通过匿名类、lambda、直接创建一个类等方法表示mapfunction，map是一个可以调用用户自定义操作的算子
// 传入匿名类，实现 MapFunction
 stream.map(new MapFunction<Event, String>() {
 @Override
 public String map(Event e) throws Exception {
 return e.user;
 }
 });
```

##### filter

>filter传入的参数需要实现filterFunction接口

```
stream.filter(new FilterFunction<Event>() {
 @Override
 public boolean filter(Event e) throws Exception {
 return e.user.equals("Mary");
 }
 });
```

##### flatMap

>先map，然后将复合元素拆分成单个，最终返回一个拆分后的列表。

```
stream.flatMap(new MyFlatMap()).print();
public static class MyFlatMap implements FlatMapFunction<Event, String> {
 @Override
 public void flatMap(Event value, Collector<String> out) throws Exception 
{
 if (value.user.equals("Mary")) {
 out.collect(value.user);
 } else if (value.user.equals("Bob")) {
 out.collect(value.user);
 out.collect(value.url);
 }
 }
 }

```

#### 聚合算子

##### keyBy

```
#flink没有直接聚合的api，因为对数据进行聚合时需要进行分区处理，这样才能提高效率。所以在flink中，要做聚合需要自己调用keyBy进行分区。keyBy用于从逻辑上划分不同分区，这里的分区是指并行处理的子任务，也就是任务槽。
#可以指定tuple的多个位置的子弹组合作为分区，对于pojo类型可以指定属性（String），另外可以传入lambda表达式实现一个键选择器，用于从数据中提取key的逻辑。
KeyedStream<Event, String> keyedStream1 = stream.keyBy(new
KeySelector<Event, String>() {
 @Override
 public String getKey(Event e) throws Exception {
 return e.user;
 }
 });
 env.execute();
 }
}
```

##### 简单聚合

>sum
>
>min
>
>max
>
>minBy
>
>maxBy
>
>通过指定位置或名称进行聚合，元组通过f0、f1.... ..进行指定位置。
>
>一个聚合算子会为每一个key保存一个聚合的值，在flink中叫做状态，所以当有一个新的数据输入，算子就会更新保存的聚合结果，并发送一个带有更新后聚合值的事件到下游算子。对于无界流来说，这个状态是永远不会清除的。所以聚合算子的key应该是有限个的。

##### 规约聚合

>**简单聚合是对一些特定统计需求的实现，那么reduce算子就是一个一般化的聚合统计操作**
>
>reduce操作会将keyedstream转换为datastream，调用reduce传入一个参数，此类需要实现reduceFunction接口。定义的方法有两个参数，是输入事件，将输入事件合并可以得到输出事件。
>
>在流处理中，将两个输入事件合并后的状态进行保存，当到来一个新事件时，对其进行计算并更新状态。

>所有聚合的操作保存在flink的状态内存中，因为他需要跨越多条记录，需要根据key保存状态。数据流入的过程，就是不断计算并更新flink中保存的状态的过程。

##### aggregate

>aggregate需要指定一个aggregatefunction函数，可以看做reduce函数的通用版本，这里有三种类型IN,ACC,OUT,分别代表输入类型，累加器类型，输出类型。
>
>接口中有四个方法：
>
>createAccumulator（）：创建一个累加器，这就是为聚合创建一个初始状态，每个聚合任务只会调用一次
>
>add（）：将输入的元素添加到累加器中，这就是聚合状态，对于新来的数据进行进一步聚合的过程。传入两个参数，当前新到来的数据value，和当前的累加器accumulator；返回一个新的累加器值，对聚合状态进行更新。
>
>getResult（）：从累加器提取聚合的输出结果。也就是说我们可以定义多个状态，然后基于这些聚合的状态计算出一个结果进行输出。比如计算平均，我们可以设置sum和count两个状态，最终调用这个方法时相除得到最终的结果。这个方法只在窗口要输出结果时调用。
>
>merge（）：合并两个累加器，并将合并后的状态作为一个累加器返回。这个方法只在需要合并窗口的场景下才会被调用；最常见的合并窗口的场景就是会话窗口。
>
>与reduce相比，aggregate的输入格式与输出格式可以不同。

##### 用户自定义函数UDF

>我们可以通过匿名函数、函数类、lambda表达式来表示一个function。

###### 富函数类

>富函数也是DataStream API提供的一个函数类的接口，所有的Flink函数类都有其Rich版本。富函数一般以抽象类的形式出现，例如RichMapFunction，RichFilterFunction、RichReduceFunction等。与常规函数类的不同主要自在于，富函数类可以获取运行环境的上下文，并拥有一些生命周期写法，所以可以实现更复杂的功能。
>
>这样自定义的函数，会改变map等函数的功能，如在其中加入对状态的操作和访问，它们还能称为无状态函数吗，显然不能，我们可以改写

```
#典型的生命周期方法有：
open（）方法，是RichFunciton的初始化方法，也就是开启一个算子的生命周期。当一个算子的实际工作方法例如map（）或filter（）方法被调用前，open（）方法首先被调用。所以像文件io创建，数据库链接，配置文件读取都可以在open方法中完成
close（）方法，是生命周期中的最后一个调用的方法，类似于解构方法。一般用来做一些清理工作。
#这里的生命周期方法对于一个并行子任务来说只会调用一次，而对应的实际工作方法，例如RichMapFunction中的map（），在每条数据到来后都会触发一次调用
#getRuntimeContext（）方法，可以获取到运行时上下文的信息，如并行度、任务名称、甚至状态（state）。这对应了后续的状态管理和状态编程。
```



##### 物理分区

>与keyBy区别，这是直接控制物理上的分布

### 输出算子 Sink

>addSink可以创建新的sink，添加的类需要实现一SinkFunction接口

#### 输出到文件

```
StreamingFileSink<String> fileSink = StreamingFileSink
 .<String>forRowFormat(new Path("./output"),
 new SimpleStringEncoder<>("UTF-8"))
 .withRollingPolicy(
 DefaultRollingPolicy.builder()
 .withRolloverInterval(TimeUnit.MINUTES.toMillis(15)
)
 .withInactivityInterval(TimeUnit.MINUTES.toMillis(5
))
 .withMaxPartSize(1024 * 1024 * 1024)
.build())
 .build();
 // 将 Event 转换成 String 写入文件
 stream.map(Event::toString).addSink(fileSink);

```



#### 输出到kafka

```
stream
 .addSink(new FlinkKafkaProducer<String>(
 "clicks",
 new SimpleStringSchema(),
 properties
 ));

```



#### 输出到redis

```
<dependency>
 <groupId>org.apache.bahir</groupId>
 <artifactId>flink-connector-redis_2.11</artifactId>
 <version>1.0</version>
</dependency>

public static class MyRedisMapper implements RedisMapper<Event> {
 @Override
 public String getKeyFromData(Event e) {
 return e.user;
 }
 @Override
 public String getValueFromData(Event e) {
 return e.url;
 }
 @Override
 public RedisCommandDescription getCommandDescription() {
 return new RedisCommandDescription(RedisCommand.HSET, "clicks");
 }
}

FlinkJedisPoolConfig conf = new 
FlinkJedisPoolConfig.Builder().setHost("hadoop102").build();
 env.addSource(new ClickSource())
 .addSink(new RedisSink<Event>(conf, new MyRedisMapper()));

```



#### 输出到Elasticsearch

```
<dependency>
 <groupId>org.apache.flink</groupId>
<artifactId>flink-connector-elasticsearch7_${scala.binary.version}</artifactId>
 <version>${flink.version}</version>
</dependency>

ElasticsearchSinkFunction<Event> elasticsearchSinkFunction = new 
ElasticsearchSinkFunction<Event>() {
 @Override
 public void process(Event element, RuntimeContext ctx, RequestIndexer 
indexer) {
 HashMap<String, String> data = new HashMap<>();
 data.put(element.user, element.url);
 IndexRequest request = Requests.indexRequest()
 .index("clicks")
 .type("type") // Es 6 必须定义 type
 .source(data);
 indexer.add(request);
 }
 };
 stream.addSink(new ElasticsearchSink.Builder<Event>(httpHosts, 
elasticsearchSinkFunction).build());
```

```
#builder添加setBulkFlushMaxActions参数后，可以控制多久写入es数据库一次。
       ElasticsearchSinkFunction<TopProductEntity> elasticsearchSinkFunction=new ElasticsearchSinkFunction<TopProductEntity>() {
            @Override
            public void process(TopProductEntity topProductEntity, RuntimeContext runtimeContext, RequestIndexer requestIndexer) {
                HashMap<String,String> data=new HashMap<>();
                data.put("productid",Integer.valueOf(topProductEntity.getProductId()).toString());
                data.put("times",Integer.valueOf(topProductEntity.getActionTimes()).toString());
                data.put("windowEnd",Long.valueOf(topProductEntity.getWindowEnd()).toString());
                IndexRequest request= Requests.indexRequest().index("topproduct").source(data);
                System.out.println(data);
                requestIndexer.add(request);
            }
        };
        ArrayList<HttpHost> httpHosts=new ArrayList<>();
        httpHosts.add(new HttpHost("192.168.244.128",9201,"http"));
        ElasticsearchSink.Builder<TopProductEntity> builder = new ElasticsearchSink.Builder<TopProductEntity>(httpHosts,elasticsearchSinkFunction );
        builder.setBulkFlushMaxActions(1);
        topProduct.addSink(builder.build());
```

#### 输出到MySQL （JDBC）

```
<dependency>
 <groupId>org.apache.flink</groupId>
 <artifactId>flink-connector-jdbc_${scala.binary.version}</artifactId>
 <version>${flink.version}</version>
</dependency>
<dependency>
 <groupId>mysql</groupId>
 <artifactId>mysql-connector-java</artifactId>
 <version>5.1.47</version>
</dependency>

stream.addSink(
 JdbcSink.sink(
 "INSERT INTO clicks (user, url) VALUES (?, ?)",
(statement, r) -> {
 statement.setString(1, r.user);
statement.setString(2, r.url);
 },
JdbcExecutionOptions.builder()
 .withBatchSize(1000)
.withBatchIntervalMs(200)
 .withMaxRetries(5)
 .build(),
 new 
JdbcConnectionOptions.JdbcConnectionOptionsBuilder()
 .withUrl("jdbc:mysql://localhost:3306/userbe
havior")
 // 对于 MySQL 5.7，用"com.mysql.jdbc.Driver"
 .withDriverName("com.mysql.cj.jdbc.Driver")
.withUsername("username")
 .withPassword("password")
 .build()
 )
 );

```

#### 自定义sink输出

```
#实现RichSinkFunction，重写如下方法
open() 在其中建立链接
invoke（）在其中执行sink的操作
close（）在其中关闭链接
```

## 窗口函数

### 时间语义

>事件像水流一样到来，经过pipline进行处理，为了划定窗口进行计算，需要以时间作为标准，也就是流中元素事件的先后以及间隔描述。
>
>flink是一个分布式系统，如何让所有机器保证时间的完全同步呢。比如上游任务8点59分59秒发送了消息，到达下游时是9点零1秒，那么应该放到哪个窗口内计算呢。所以，我们需要决定到底以哪个时间为标准。

#### 处理时间

>processing time,指执行处理操作的机器的系统时间，就是说什么时候到达处理机器，就将其划分到哪个滑动窗口。这是最直接的时间语义，各结点按照自己的系统时钟划分窗口。

>处理时间由于数据一旦到来就处理，所以效率很高，延迟很低。一般用在对准确性要求不太高的场景。

#### 事件时间

>事件时间，是指每个事件在对应的设备上发生的时间，也就是数据生成的时间。数据一旦产生，这个时间自然就确定了，所以它可以作为一个属性嵌入到数据中。这就是时间戳。这种情况，我们无法看到系统时间，假设只以到来时间的时间戳为基准，控制滑动窗口，那么到来数据必须是有序的，时间戳也是不断增长的。这实际上不可能，所以我们需要借助另外的标志，水位线wateramrks。

>事件语义允许设置水位线，并可以接收乱序的数据，可以进行一定的延迟等待，让窗口内的所有数据都到齐，再进行计算。

#### 摄入时间

>它指数据进入flink数据流的时间，也就是source算子读入数据的时间。他是事件时间和处理时间的中和，不会引入太大的延迟，他的具体行为跟事件时间非常像，可以当做特殊的事件时间处理。

### 水位线

>关于sparkstreaming和flink水位线的思考对比：
>
>https://www.sohu.com/a/270444235_494938
>
>流计算中我们需要保存状态，但是Dstream是无状态的，那么其count算子是如何工作的呢，答案是将前一个时间步的RDD作为当前时间步的前继结点，就能达到状态更替的效果。

>watermark用来度量事件时间，watermark是为了服务事件时间产生的。
>
>在处理时间语义中，每个事件以到达处理机器的时间作为时间戳，当机器时间到达9点（窗口尾部），那么触发计算8点到9点这个窗口的数据，左闭右开。
>
>在事件时间语义中，每个事件以事件产生的时间为时间戳，以事件时间作为窗口的起始和截止，何时触发操作呢？每当一个事件到来，我们读取它的时间戳，作为当前的水位线时钟，当9点的事件到来时，我们认为数据到齐了，开始触发计算。当然，我们可以通过调节触发时机，来调整数据延迟的容忍度及处理效率。

>通过在数据流中插入一条记录，这条记录包含从数据流事件中读取的时间戳，称为水位线。通过将水位线广播到下游所有子任务，可以更新下游子任务的时钟

>水位线的特性：
>
>1必须递增
>
>2可以设置延迟，保证处理完整的乱序流
>
>3一个水位线t表示t之前的数据都到齐了，之后流中不会再出现小于t的事件了。

#### 有序流中的水位线

>有序流中可以保证水位线有序增长，在实际应用中，我们需要水位线的插入周期，当大量相同时间戳的数据到来时，不要频繁插入值相同的水位线。以系统时间为基准，每隔一段时间，插入水位线。

#### 乱序流中的水位线

>乱序流中有很多迟到数据，我们需要容忍这些数据全部到期，所以我们可以为水位线添加延迟，当读取时间戳时，减去2s作为时间戳，再当做水位线插入数据流。

#### 生成水位线

##### 生成水位线原则

>我们知道，完美的水位线是绝对正确的，一旦水位线t出现表示t之前的数据都到齐了，之后流中不会再出现小于t的事件了。但是，这在实际中很难达成，我们需要考虑效率，及一些意外延迟。如何确保大部分数据不迟到，设置合理的水位线呢。另一种做法是，创建一个单独的flink作业监控事件流，统计事件概率，学习事件的迟到规律。然后，选择置信区间确定延迟，作为水位线的生成策略。

##### 水位线生成策略

>flink的datastream api中，使用assignTimestampsAndWatermarks（）为流中的数据分配时间戳，并生成水位线指示时间。

```
assignTimestampsAndWatermarks方法需要传入一个WatermarkStrategy作为参数，这就是所谓的“水位线生成策略”。WatermarkStrategy中包含一个时间戳分配器TimestampAssigner和一个“水位线生成器”WatermarkGenerator。
TimestampAssigner：主要负责从流中数据元素的某个字段中提取时间戳，并分配给元素。时间戳的分配是生成水位线的基础
WatermarkGenerator主要负责按照既定的方式，基于时间戳生成水位线。在WatermarkGenerator接口中，主要又有两个方法：onEvent（）和onPeriodicEmit（）
onEvent：每个事件到来都会调用的方法，它的参数由当前时间、时间戳，以及允许发出水位线的一个WatermarkOutput，可以基于事件做各种操作。
onPeriodicEmit：周期性调用的方法，可以由WatermarkOutput发出水位线。周期时间为处理时间，可以调用环境配置的.setAutoWatermarkInternal()方法来设置，默认为200ms。
env.getConfig().setAutoWatermarkInternal(60*1000L)
```

##### flink内置水位线生成器

>flink提供了内置的生成器，可以使用WatermarkStrategy的静态辅助方法来创建，它们都是周期性生成水位线的，分别对应着处理有序流和乱序流的场景

###### 有序流

>对于有序流，主要特点就是时间戳单调递增，所以永远不会出现迟到数据的问题。直接调用WatermarkStrategy.forMonotonousTimestamps()就可以实现，就是拿当前最大的时间戳作为水位线。
>
>我们使用withTimestampAssigner方法将数据中的timestamp字段取出来，作为时间戳分配给数据元素；然后用内置的有序流水线生成器构造策略。

```
stream.assignTimestampsAndWatermarks(
 WatermarkStrategy.<Event>forMonotonousTimestamps()
 .withTimestampAssigner(new SerializableTimestampAssigner<Event>() 
{
 @Override
 public long extractTimestamp(Event element, long recordTimestamp) 
{
 return element.timestamp;
 }
 })
);
```

###### 乱序流

>乱序流需要设置延迟时间。调用WatermarkStrategy.  forBoundedOutOfOrderness()可以实现，这个方法需要传入一个maxOutOfOrderness参数，表示最大乱序程度。

```
env
 .addSource(new ClickSource())
 // 插入水位线的逻辑
 .assignTimestampsAndWatermarks(
 // 针对乱序流插入水位线，延迟时间设置为 5s
 
WatermarkStrategy.<Event>forBoundedOutOfOrderness(Duration.ofSeconds(5))
 .withTimestampAssigner(new 
SerializableTimestampAssigner<Event>() {
 // 抽取时间戳的逻辑
@Override
public long extractTimestamp(Event element, long 
recordTimestamp) {
 return element.timestamp;
 }
 })
 )
 .print();

```

##### 自定义水位线

>水位线分为周期性的，断点式的，分别在onPeriodicEmit和onEvent编写程序。

###### 周期性水位线生成器

>周期性水位线一般通过onEvent观察输入事件，而在onPerioddicEmit发出水位线。

```
public static class CustomPeriodicGenerator implements 
WatermarkGenerator<Event> {
 private Long delayTime = 5000L; // 延迟时间
 private Long maxTs = Long.MIN_VALUE + delayTime + 1L; // 观察到的最大时间戳
 @Override
 public void onEvent(Event event, long eventTimestamp, WatermarkOutput 
output) {
 // 每来一条数据就调用一次
 maxTs = Math.max(event.timestamp, maxTs); // 更新最大时间戳
 }
 @Override
 public void onPeriodicEmit(WatermarkOutput output) {
 // 发射水位线，默认 200ms 调用一次
 output.emitWatermark(new Watermark(maxTs - delayTime - 1L));
 }
 }
}
```

###### 断点式水位线生成器

>断点式生成器会不停地检测 onEvent()中的事件，当发现带有水位线信息的特殊事件时， 就立即发出水位线。一般来说，断点式生成器不会通过 onPeriodicEmit()发出水位线。

```
@Override
 public void onEvent(Event r, long eventTimestamp, WatermarkOutput output) {
// 只有在遇到特定的 itemId 时，才发出水位线
 if (r.user.equals("Mary")) {
 output.emitWatermark(new Watermark(r.timestamp - 1));
 }
 }
 @Override
 public void onPeriodicEmit(WatermarkOutput output) {
 // 不需要做任何事情，因为我们在 onEvent 方法中发射了水位线
 }
```

#### 水位线的传递

>当有多个来自上游的水位线时，取值最小的水位线，因为要确保水位线以前的消息都到齐了。

### 窗口

>窗口将无界数据流切分为多个数据块
>
>当使用处理时间时，窗口的概念很直观。
>
>当使用事件时间时，窗口的触发时机发生了变化。我们把窗口理解为一个桶，讲不同的数据收集到正确的窗口桶中。
>
>主要关注几个属性：事件到来，窗口范围及存储数据，水位线。
>
>注意触发计算和窗口关闭是可以分开的，只是一般情况无需这么复杂。

#### 窗口的分类

##### 按照驱动类型分类

>窗口可以按照驱动类型分为时间窗口、计数窗口。
>
>计数窗口按照某个固定的个数，来截取一段数据集，这种窗口叫做计数窗口。

###### 时间窗口

>flink中有一个时间窗口的类叫做TimeWindow，这个类有两个私有属性：start和end，表示窗口的开始和结束的时间戳，单位为毫秒。

```
private final long start;
private final long end;
```

>我们可以调用公有的getStrat()和getEnd()方法直接获取这两个时间戳。另外，TimeWindow还提供了一个maxTimestamp()方法，用来获取窗口中能够包含的数据的最大时间戳。

```
public long maxTimestamp(){
   return end-1;
}
#很明显，窗口中的数据，最大的允许的时间戳是end-1，也就代表了窗口时间范围是左闭右开的。
```

###### 计数窗口

>计数窗口基于元素的个数来截取数据，到达固定个数时就触发计算并关闭窗口。这相当于座位有限，人满就发车。每个窗口截取的个数，就是窗口的大小。flink通过全局窗口global window来实现计数窗口。

###### 全局窗口

##### 按照窗口分配数据的规则分配

>时间窗口和计数窗口，只是对窗口的一个大致划分；在具体应用时，还需要定义更加精细的规则，来控制数据应该划分到哪个窗口中去。不同的分配数据的方式，就可以有不同的功能应用。
>
>根据分配的规则，窗口的具体实现可以分为4类：滚动窗口（Tumbling Window）、滑动窗口（Sliding Window）、会话窗口（Session Window），以及全局窗口（Global  Window）。

###### 滚动窗口Tumbling Windows

>滚动窗口对数据进行均匀分片。窗口之间没有重叠，也不会有间隔，是首尾相接的状态。如果把多个窗口的创建看作一个窗口的移动，那么他就像在滚动一样。

###### 滑动窗口Sliding Windows

>由窗口大小和滑动距离确定，每个窗口之间有一定重叠部分。滑动窗口是滚动窗口的一种广义方式，当滑动步长等于滑动窗口大小时，就是滚动窗口。

###### 会话窗口Session Windows

>简单来说，就是数据来了之后开启一个会话窗口，如果接下来还有数据陆续到来，那么一直保持会话，如果一段时间没有接收到数据，那就认为会话超时失效，窗口自动关闭。
>
>会话窗口只能基于时间来定义，而没有会话计数窗口的概念，因为会话的意思就是“隔一段时间没有数据来”。会话窗口的关键参数是窗口大小，如果两个数据到来的间隔小于指定的大小，那么说明还在保持会话。
>
>在乱序流中，如果gap间隔超过size就关闭，可能导致迟到的消息丢失。为了处理这种情况，每当来一个新的数据，都会创建一个新的会话窗口；然后判断已有窗口之间的距离，如果小于给定的size，就对它进行合并操作。相当于会话窗口永远不关闭，一直在维护，无论迟到的数据何时到来，总能合并到正确的会话中。注意会话窗口的一些特性：
>
>1不同分区是不相关的
>
>2会话窗口的长度不固定
>
>3起始和结束时间不确定

###### 全局窗口Global Windows

>还有一类比较通用的窗口，就是全局窗口。这种窗口全局有效，会把相同key的所有数据分配到同一个窗口中。说直白点，就是没分窗口一样，这种窗口没有结束的时候，默认是不会做触发计算的，必须编写触发器。

#### 窗口API

```
#按键分区Keyed Windows
stream.keyBy().window()
#非按键分区Non-Keyed Windows
stream.windowAll()
#窗口api使用
steram.keyBy(<key selector>)
       .window(<window assigner>)
       .aggregate(<window function>)
```

##### 窗口分配器

###### 时间窗口

>分为滚动、滑动和会话三种

```
#滚动处理时间窗口
stream.keyBy(...)
.window(TumblingProcessingTimeWindows.of(Time.seconds(5)))
.aggregate(...)
#滑动处理时间窗口
stream.keyBy(...)
.window(SlidingProcessingTimeWindows.of(Time.seconds(10), Time.seconds(5)))
.aggregate(...)
#处理时间会话窗口
stream.keyBy(...)
    .window(ProcessingTimeSessionWindows.withGap(Time.seconds(10)))
	.aggregate(...)
#动态定义超时时间
.window(ProcessingTimeSessionWindows.withDynamicGap(new 
SessionWindowTimeGapExtractor<Tuple2<String, Long>>() {
 @Override
 public long extract(Tuple2<String, Long> element) { 
// 提取 session gap 值返回, 单位毫秒
 return element.f0.length() * 1000;
 }
}))
#滚动事件时间窗口
stream.keyBy(...)
.window(TumblingEventTimeWindows.of(Time.seconds(5)))
.aggregate(...)
#滑动事件时间窗口
stream.keyBy(...)
.window(SlidingEventTimeWindows.of(Time.seconds(10), Time.seconds(5)))
.aggregate(...)
#事件时间会话窗口
stream.keyBy(...)
.window(EventTimeSessionWindows.withGap(Time.seconds(10)))
.aggregate(...)
```

###### 计数窗口

```
#滚动计数窗口
stream.keyBey()
   .countWindow(10)
#滑动计数窗口
stream.keyBy()
       .countWindow(10,3)
```

###### 全局窗口

```
stream.keyBy()
      .window(GlobalWindows.create())
```

##### 窗口函数

###### 增量聚合函数

>**简单聚合是对一些特定统计需求的实现，那么reduce算子就是一个一般化的聚合统计操作**
>
>**1.归约函数**
>
>reduce操作会将keyedstream转换为datastream，调用reduce传入一个参数，此类需要实现reduceFunction接口。定义的方法有两个参数，是输入事件，将输入事件合并可以得到输出事件。
>
>在流处理中，将两个输入事件合并后的状态进行保存，当到来一个新事件时，对其进行计算并更新状态。

>所有聚合的操作保存在flink的状态内存中，因为他需要跨越多条记录，需要根据key保存状态。数据流入的过程，就是不断计算并更新flink中保存的状态的过程。

>**2.聚合函数**
>
>aggregate需要指定一个aggregatefunction函数，可以看做reduce函数的通用版本，这里有三种类型IN,ACC,OUT,分别代表输入类型，累加器类型，输出类型。
>
>接口中有四个方法：
>
>createAccumulator（）：创建一个累加器，这就是为聚合创建一个初始状态，每个聚合任务只会调用一次
>
>add（）：将输入的元素添加到累加器中，这就是聚合状态，对于新来的数据进行进一步聚合的过程。传入两个参数，当前新到来的数据value，和当前的累加器accumulator；返回一个新的累加器值，对聚合状态进行更新。
>
>getResult（）：从累加器提取聚合的输出结果。也就是说我们可以定义多个状态，然后基于这些聚合的状态计算出一个结果进行输出。比如计算平均，我们可以设置sum和count两个状态，最终调用这个方法时相除得到最终的结果。这个方法只在窗口要输出结果时调用。
>
>merge（）：合并两个累加器，并将合并后的状态作为一个累加器返回。这个方法只在需要合并窗口的场景下才会被调用；最常见的合并窗口的场景就是会话窗口。
>
>与reduce相比，aggregate的输入格式与输出格式可以不同。

###### 全窗口函数

>全窗口函数与增量聚合函数不同，全窗口函数需要先收集窗口中的所有数据，然后在计算全部数据。
>
>这种计算方式相比于流处理是低效的，但是有的时候必须获取到所有数据才能计算，或者需要获取窗口的起始时间等，那么就必须使用全窗口函数。这是典型的批处理思想。

>1.窗口函数（WindowFunction）
>
>WindowFunction字面上就是“窗口函数”，他其实就是老版本的通用窗口函数接口。我们可以基于WindowedStream调用apply方法，传入一个WindowFunction的实现类。
>
>WindowFunction可用的功能较少，一般使用ProcessWindowFunction

```
stream
      .keyBy(<key selector>)
      .window(<window assigner>)
      .apply(new MyWindowFunction());
```

>2.处理窗口函数（ProcessWindowFunction）
>
>ProcessWindowFunction是windowAPI中最底层的通用窗口函数接口。相比于WindowFunction，他可以拿到Context上下文对象，不仅能获得窗口信息，还能访问当前时间和内存状态。这里的时间包括当前时间和时间水位线。

```
stream
      .keyBy(<key selector>)
      .window(<window assigner>)
      .process(UvCountByWindow())
public static class UvCountByWindow extends ProcessWindowFunction<Event, 
    String, Boolean, TimeWindow>{
         @Override
         public void process(Boolean aBoolean, Context context, Iterable<Event> 
                elements, Collector<String> out) throws Exception {
                 HashSet<String> userSet = new HashSet<>();
                 // 遍历所有数据，放到 Set 里去重
                 for (Event event: elements){
                 userSet.add(event.user);
                 }
                 // 结合窗口信息，包装输出内容
                 Long start = context.window().getStart();
                 Long end = context.window().getEnd();
                 out.collect(" 窗 口 : " + new Timestamp(start) + " ~ " + new 
                Timestamp(end)
                 + " 的独立访客数量是：" + userSet.size());
    }
}
}

```

###### 增量窗口和全窗口函数的结合使用

>对于reduce和aggregate函数，我们除了可以传入一个ReduceFunction 或 AggregateFunction 进行增量聚合，还可以传入WindowFunction获取更多丰富的信息，传入的可以是WindowFunction或ProcessWindowFunction。
>
>示例中apply的输入就是ReduceFunction 或 AggregateFunction 处理完的结果，当窗口到达结尾触发计算才会调用第二个窗口函数的apply方法，将其传入windowFunction处理后，作为aggregate函数的处理结果。

```
#可以借助aggregate函数调用窗口函数
stream
      .keyBy(<key selector>)
      .window(<window assigner>)
      .aggregate(new AggreagteFunction(),new WindowResultFunction())
     
public class WindowResultFunction
        implements WindowFunction<Long, TopProductEntity, Tuple, TimeWindow> {
    @Override
    public void apply(Tuple key, TimeWindow window, Iterable<Long> aggregateResult, Collector<TopProductEntity> collector) throws Exception {
		int itemId = key.getField(0);
		Long count = aggregateResult.iterator().next();
        collector.collect(TopProductEntity.of(itemId,window.getEnd(),count));
    }
}
```

##### 其他API

>对于一个窗口算子，其由窗口分配器+窗口函数构成。flink还提供了其他可选的api，让我们灵活的控制窗口行为。

###### 触发器

>触发器主要用来控制窗口什么时候触发计算。所谓的触发计算，本质上就是执行窗口函数，所以可以认为是计算得到的结果。
>
>基于WindowedStream调用trigger
>
>Trigger 是窗口算子的内部属性，每个窗口分配器（WindowAssigner）都会对应一个默认 的触发器；对于 Flink 内置的窗口类型，它们的触发器都已经做了实现。例如，所有事件时间 窗口，默认的触发器都是 EventTimeTrigger；类似还有 ProcessingTimeTrigger 和 CountTrigger。 所以一般情况下是不需要自定义触发器的，不过我们依然有必要了解它的原理。

```
stream.keyBy(...)
     .window(...)
     .trigger(new MyTrigger())
```

```
#Trigger是一个抽象类，自定义时需要实现如下四个抽象方法
onElement（）：对元素的响应
onEventTime（）：对事件时间的响应
onProcessingTime（）：对处理时间的响应
clear（）：窗口销毁时调用此方法
前三个方法通过放回enum值，控制窗口行为。
枚举值为：CONTINUE,FIRE,PURGE,FIRE_AND_PURGE
如上的枚举值说明触发计算和窗口销毁是可以分开的，并不一定一起触发。

```

###### 移除器

>移除器主要用来定义移除某些数据的逻辑。基于WindowedStream调用.evictor()方法，就可以传入一个自定义的移除器。evictor是一个接口，不同的窗口类型都有各自实现的移除器。

```
stream.keyBy()
      .window()
      .evictor(new MyEvictor())
evictor定义了两个方法
evictBefore（）：定义执行窗口函数之前的移除数据操作
evictAfter（）：定义执行窗口函数之后的移除数据操作
```

###### 允许延迟

>为了让迟到的数据也参与计算，修正结果的误差，使用允许延迟控制窗口的销毁时间。
>
>这里可以看出，窗口的触发计算和清除操作确实可以分开。不过在默认情况下，允许的延迟是0，这样一旦水位线到达了窗口结束时间就会触发计算并清除窗口。

```
stream.keyBy()
      .window(TumblingEventTimeWindows.of(Time.hours(1)))
      .allowedLateness(Time.minutes(1))      
```

###### 将迟到的数据放入侧输出流

>即使可以设置延迟，仍然会有数据 超过延迟。如果不想丢弃任何数据，可以借助side output进行另外的处理。所谓的侧输出流，相当于数据流的一个分支，这个流中单独放置那些错过了该上的车、本该被丢弃的数据。
>
>基于WindowedStream调用.sideOutputLateData()方法，就可以实现这个功能。方法需要传入一个输出标签，用来标记分支的迟到数据流。因为保存的就是流中的原始数据，所以OutputTag的类型和流中数据类型相同。

```
DataStream<Event> stream = env.addSource(...);
OutputTag<Event> outputTag = new OutputTag<Event>("late") {};
SingleOutputStreamOperator<AggResult> winAggStream = stream.keyBy(...)
     .window(TumblingEventTimeWindows.of(Time.hours(1)))
    .sideOutputLateData(outputTag)
    .aggregate(new MyAggregateFunction())
DataStream<Event> lateStream = winAggStream.getSideOutput(outputTag);
```



##### 窗口的生命周期

###### 窗口的创建

>窗口的类型和基本信息由窗口分配器指定，但是窗口不会预先创建好，而是由数据驱动创建。当第一个应该属于这个窗口的数据元素到达时，就会创建窗口。

###### 窗口计算的触发

>除了窗口分配器，还有自己的窗口函数的触发器。窗口函数可以分为增量聚合函数和全窗口函数，主要定义了计算的逻辑。而触发器就是窗口函数触发计算的条件。

###### 窗口的销毁

>flink只对时间窗口进行销毁，计数窗口由于是基于全局窗口实现的，而全局窗口不会清除状态，所以就不会销毁。

###### 窗口api总结

```
stream.
     . keyBy()
     .window()
     [.trigger()]
     [.evictor()]
     [.allowedLateness()]
     [.sideOutputLateData]
     .redluce/aggregate/fold/apply()
     [.getSideOutput()]

stream
     .windowAll()
     [.trigger()]
     [.evictor()]
     [.allowedLateness()]
     [.sideOutputLateData()]
     .reduce/aggregate/fold/apply()
     [.getSideOutput]

DataStream,KeyStream,WindowedStream,singleOutputStream
```

### 迟到数据的处理

>1水位线延迟
>
>2窗口延迟销毁
>
>3侧输出流

## 处理函数

>处理函数中，我们直接面对的是数据流中的最基本元素：数据事件event,状态state，时间time。处理函数比较抽强，是底层的api，理论上可以完成任何事情。

### 基本处理函数

#### 功能和使用

>基本的转换算子，都是针对某种具体操作来定义的，能够拿到的信息比较有限。比如map算子，我们实现的MapFunction中，只能获取到当前的数据，定义它转换之后的形式。而像AggregateFunction，还可以获取到当前的状态Accumulator。RichMapFunction可以拿到getRuntimeContext，得到并行度，任务名称等。
>
>
>
>但是，这些算子都无法访问事件时间戳，水位线信息。处理函数提供这些功能，包括定时服务，流中的事件，时间戳，水位线，甚至可以注册定时事件，并且具有富函数类的所有特性，同样可以访问state和其他运行时信息。

```
stream.process(new MyProcessFunction())
```

>这里ProcessFunction是一个抽象类，继承了AbstractRichFunction；

#### ProcessFunction解析

>ProcessFunction继承了AbstractRichFunction，有两个泛型类型参数I表示Input，也就是输入的数据类型；O表示Output，也就是处理完成之后输出的数据类型。
>
>其内部定义了两个方法：一个是必须要实现的抽象方法processElement（）；另一个是非抽象方法onTimer（）

##### processElement

>这个方法对于流中的每个元素都会调用一次， 参数包括三个：输入数据值 value，上下文 ctx，以及“收集器”（Collector）out。

##### 非抽象方法onTimer

>通过TimerService来注册定时器，它是一个基于时间的回调方法。他有三个参数：时间戳，上下文ctx，收集器out。这里的timestamp是指设定好的触发时间，事件时间语义下当然就是水位线了。
>
>只有keyedStream才能设定定时器

#### 处理函数的分类

>DataStream在调用一些方法后，有可能生成新的流类型；例如调用keyBy之后得到的KeyedStream，进一步调用window得到WindowedStream。对于不同类型的流都可以调用process方法。
>
>flink提供了8个不同的处理函数

##### ProcessFunction

>基本处理函数

##### KeyedProcessFunction

>对流按键分区后的处理函数，基于KeyedStream调用process时作为参数从传入。要想使用定时器，比如基于KeyedStream。

##### ProcessWindowFunction

>开窗之后的处理函数，也就是全窗口函数的代表，基于WindowedStream调用process。

##### ProcessAllWindowFunction

>同样是开窗之后的处理函数，基于 AllWindowedStream 调用.process()时作为参数传入。

##### CoProcessFunction

>合并（connect）两条流之后的处理函数，基于 ConnectedStreams 调用.process()时作为参 数传入。

##### ProcessJoinFunction

>间隔连接两条流之后的处理函数，基于IntervalJoined调用

##### BroadcastProcessFunction

>广播连接流处理函数，基于BroadcastConnectedStream调用process时作为参数传入，这里的广播连接流，是一个未keyBy的普通DataStream与一个广播流连接之后的产物

##### KeyedBroadcastProcessFunction

>是一个keyedStream与一个广播流连接之后的产物

### 按键分区处理函数

### 窗口处理函数

### 侧输出流

>processFunction还有一个特定功能，就是将自定义的数据放入侧输出流输出。绝大多数转换算子都是单一流，借助侧输出流我们可以从主流分叉出支流，实现分流操作。
>
>具体使用时，在processElement或onTimer中，调用context的output方法即可。

```
DataStream<Integer> stream = env.addSource(...);
SingleOutputStreamOperator<Long> longStream = stream.process(new 
ProcessFunction<Integer, Long>() {
 @Override
 public void processElement( Integer value, Context ctx, Collector<Integer> 
out) throws Exception {
 // 转换成 Long，输出到主流中
 out.collect(Long.valueOf(value));
 // 转换成 String，输出到侧输出流中
 ctx.output(outputTag, "side-output: " + String.valueOf(value));
 }
});

OutputTag<String> outputTag = new OutputTag<String>("side-output") {};
DataStream<String> stringStream = longStream.getSideOutput(outputTag);
```

## 多流转换

>在处理数据时，有时我们需要对不同数据源的数据进行连接合并，也可能需要拆分数据流。即分流和合流操作

### 分流

>分流将一条数据流拆分成完全独立的两条、甚至多条流。也就是基于一个DataStream，得到多个完全平等的子DataStream。通常通过定义筛选条件，控制分到哪个子流中。

#### filter

>实现FilterFunction我们就可以实现分流操作，但是其不仅代码重复，而且效率低。通过将原始DataStream复制三次，执行三次筛选才能得到子流

#### 侧输出流

>侧输出流可以输出与主流不同类型的DataStream，极为方便。

```
public class SplitStreamByOutputTag {
     // 定义输出标签，侧输出流的数据类型为三元组(user, url, timestamp)
     private static OutputTag<Tuple3<String, String, Long>> MaryTag = new 
    OutputTag<Tuple3<String, String, Long>>("Mary-pv"){};
     private static OutputTag<Tuple3<String, String, Long>> BobTag = new 
    OutputTag<Tuple3<String, String, Long>>("Bob-pv"){};
     public static void main(String[] args) throws Exception {
         StreamExecutionEnvironment env = 
        StreamExecutionEnvironment.getExecutionEnvironment();
         env.setParallelism(1);
         SingleOutputStreamOperator<Event> stream = env
         .addSource(new ClickSource());
         SingleOutputStreamOperator<Event> processedStream = stream.process(new 
        ProcessFunction<Event, Event>() {
         @Override
         public void processElement(Event value, Context ctx, Collector<Event> 
        out) throws Exception {
         if (value.user.equals("Mary")){
         ctx.output(MaryTag, new Tuple3<>(value.user, value.url, 
        value.timestamp));
         } else if (value.user.equals("Bob")){
        213
         ctx.output(BobTag, new Tuple3<>(value.user, value.url, 
        value.timestamp));
         } else {
         out.collect(value);
         }
         }
         });
         processedStream.getSideOutput(MaryTag).print("Mary pv");
         processedStream.getSideOutput(BobTag).print("Bob pv");
         processedStream.print("else");
         env.execute();
     }
}

```

### 基本合流操作

>合流操作比较普遍，api也较多

#### 联合Union

>将多条相同类型的流合并为一条流，简单粗暴。
>
>合并后水位线如何定义，当然以最慢的定义，因为有一个流没有到达该时刻，就表示需要维持这个窗口，不能进行计算，只有当所有流都越过了endtime，才可以触发计算，所以是以最慢的流为基准。

```
stream1.union(stream2,stream3,......)
```

#### 连接Connect

>流的连接可以处理不同数据类型的两个流。

##### 连接流 ConnectedStreams

>由于连接的流数据类型可以不同，所以连接后不是DataStream，而是ConnectedStreams。要想得到新的DataStream，需要进一步定义同处理co-process转换操作，用来说明如何转换不同类型的数据。
>
>分别用map1和map2表示对两个流的处理

```
ConnectedStreams<Integer, Long> connectedStreams = stream1.connect(stream2);
 SingleOutputStreamOperator<String> result = connectedStreams.map(new 
CoMapFunction<Integer, Long, String>() {
 @Override
 public String map1(Integer value) {
 return "Integer: " + value;
 }

 @Override
 public String map2(Long value) {
 return "Long: " + value;
 }
 });

```

##### CoProcessFunction

>用于处理Co的process函数，也是处理函数的一员

```
appStream.connect(thirdpartStream)
 .keyBy(data -> data.f0, data -> data.f0)
 .process(new OrderMatchResult())
 .print();

public void processElement1(Tuple3<String, String, Long> value, Context ctx, 
    Collector<String> out) throws Exception {
     // 看另一条流中事件是否来过
     if (thirdPartyEventState.value() != null){
     out.collect(" 对 账 成 功 ： " + value + " " + 
    thirdPartyEventState.value());
     // 清空状态
     thirdPartyEventState.clear();
     } else {
     // 更新状态
     appEventState.update(value);
     // 注册一个 5 秒后的定时器，开始等待另一条流的事件
     ctx.timerService().registerEventTimeTimer(value.f2 + 5000L);
     }
 }
 
 public void processElement2(Tuple3<String, String, Long> value, Context ctx, 
    Collector<String> out) throws Exception { ......}
```

##### 广播连接流BroadcastConnectedStream

>当连接的流中一个为BroadcastStream时，合并的流变成了广播连接流。
>
>这种流主要用于动态定义某些规则或配置的场景，因为规则是实时变动的，所以可以使用一个单独的流获取规则数据；而这些规则是全局有效的，所以必须使用广播传递给所有子任务，这就是广播状态。
>
>广播状态底层是使用一个映射map结构来爆粗你的，在代码上，可以直接调用DataStream的broadcast方法，传入一个MapStateDescriptor说明状态的名称和类型，就可以达到规则数据的广播流。

### 基于时间的合流-双流联结Join

#### 窗口联结Window Join

>将两者通过where，equalTo连接起来的元素，通过window进行划分，然后执行apply的JoinFunction方法。

```
stream1.join(stream2)
 .where(<KeySelector>)
 .equalTo(<KeySelector>)
 .window(<WindowAssigner>)
 .apply(<JoinFunction>)
```

##### 窗口联结的处理流程

>两条流的数据到来时，按照key分组、窗口时间，进入对应的地方存储。当到达endTime时，挑选出key相互匹配的数据对，然后交给join函数处理。

```
stream1
     .join(stream2)
     .where(r -> r.f0)
     .equalTo(r -> r.f0)
     .window(TumblingEventTimeWindows.of(Time.seconds(5)))
     .apply(new JoinFunction<Tuple2<String, Long>, Tuple2<String, Long>, 
    String>() {
     @Override
     public String join(Tuple2<String, Long> left, Tuple2<String, 
            Long> right) throws Exception {
             return left + "=>" + right;
     }
     })
```

#### 间隔联结Interval Join

>允许流中的元素，与另一个流中的元素进行匹配，匹配的事件时间范围为一个自定义区间，一般是元素事件时间的附近区间。

```
stream1
 .keyBy(<KeySelector>)
 .intervalJoin(stream2.keyBy(<KeySelector>))
 .between(Time.milliseconds(-2), Time.milliseconds(1))
 .process (new ProcessJoinFunction<Integer, Integer, String(){
 @Override
 public void processElement(Integer left, Integer right, Context ctx, 
Collector<String> out) {
 out.collect(left + "," + right);
 }
 });

```

#### 窗口同组联结Window CoGroup

>与window join类似，只不过在apply方法传入一个coGroupFunction自定义类。
>
>对于一整个窗口数据，它只调用一次，而不是对于每个元素调用一次。他可以实现更加灵活的其他join，如inner join，left outer join，right outer join，full outer join。事实上窗口join的底层，也是通过coGroup实现。

```
stream1
 .coGroup(stream2)
 .where(r -> r.f0)
 .equalTo(r -> r.f0)
 .window(TumblingEventTimeWindows.of(Time.seconds(5)))
 .apply(new CoGroupFunction<Tuple2<String, Long>, Tuple2<String, 
    Long>, String>() {
         @Override
         public void coGroup(Iterable<Tuple2<String, Long>> iter1, 
        Iterable<Tuple2<String, Long>> iter2, Collector<String> collector) throws 
        Exception {
         collector.collect(iter1 + "=>" + iter2);
     }
 })
```

## 状态编程

>flink的理念就是，有状态的流式计算。对于简单聚合，窗口聚合，处理函数，规约都需要状态的使用。状态如同数据库中的数据。

### Flink中的状态

#### 有状态算子

>算子任务可以分为无状态和有状态两种
>
>无状态的算子独立的观察每个事件，根据输入直接得出结果。如map，filter，flatMap等，计算时不依赖其他数据，就属于无状态算子。
>
>有状态的算子任务，则除了当前数据以外，还需要其他的数据来计算结果。这里的其他数据就是state，比如sum。聚合，规约，窗口算子都是有状态算子。
>
>有状态算子基本流程：
>
>1接收上游数据
>
>2获取当前状态
>
>3计算，并更新状态
>
>4得到计算结果，发往下游任务

#### 状态管理

>flink中，一个算子任务有多个并行子任务，分布在slot上，状态就是子任务在内存中维护的本地变量。
>
>但是，为了考虑低延迟、高吞吐，容错性等问题，需要解决如下问题：
>
>1状态的访问权限。一个slot（一个分区）可能包含多个key的数据，他们同时访问和更改本地变量，就会导致错误，这时状态就不是单纯的本地变量。
>
>2容错性，也就是故障后的恢复。
>
>3分布式扩展性，当数据量增大，应该对计算资源扩容，调大并行度。

#### 状态分类

##### 托管状态和原始状态

>托管状态：由flink统一管理，包括容错问题，存储访问，重组等问题
>
>原始状态：flink不进行任何自动管理，只把他当做字节存储。

##### 算子状态和按键分区状态

>算子状态：只对当前并行子任务实例有效
>
>按键分区状态：使用key值来维护和访问状态，只能用于KeyedStream

### 按键分区状态

#### 基本概念和特点

>以key为作用范围进行隔离。
>
>根据到来的数据的key值，去状态中取出key对应的状态。这保证了数据流和状态的分区一致性，不会错误的处理数据流，或取出错误的状态。
>
>当并行度变化，要进行状态重组，状态组成键组key groups，每一组对应一个并行子任务。key state必须基于keyedStream。

#### 支持的结构类型

>支持如List，Map等

##### 值状态

>只保存一个值

```
public interface ValueState<T> extends State {
    T value() throws IOException;
    void update(T value) throws IOException;
}

```

##### 列表状态

>ListState提供了一系列方法和属性，操作状态
>
>Iterable<T> get()
>
>update(List values)
>
>add(T value)
>
>addAll(List values)

##### 映射状态

>UV get(UK key)
>
>put(UK key, UV value)
>
>putAll(Map map)
>
>remove(UK key)
>
>boolean contains(UK key)
>
>Iterable> entries()
>
>Iterable keys()
>
>Iterable values()
>
>boolean isEmpty()

##### 规约状态

```
public ReducingStateDescriptor(
 String name, ReduceFunction<T> reduceFunction, Class<T> typeClass) {...}
这里的描述器有三个参数，其中第二个参数就是定义了归约聚合逻辑的 ReduceFunction，
另外两个参数则是状态的名称和类型。
```

##### 聚合状态

>与归约状态非常类似，聚合状态也是一个值，用来保存添加进来的所有数据的聚合结果。 与 ReducingState 不同的是，它的聚合逻辑是由在描述器中传入一个更加一般化的聚合函数 （AggregateFunction）来定义的；这也就是之前我们讲过的 AggregateFunction，里面通过一个 累加器（Accumulator）来表示状态，所以聚合的状态类型可以跟添加进来的数据类型完全不 同，使用更加灵活。
>
>同样地，AggregatingState 接口调用方法也与 ReducingState 相同，调用.add()方法添加元素 时，会直接使用指定的 AggregateFunction 进行聚合并更新状态。

```
ValueStateDescriptor<Long> descriptor = new ValueStateDescriptor<>(
    "my state", // 状态名称
    Types.LONG // 状态类型
);
 state = getRuntimeContext().getState(descriptor);
```

##### 状态生存时间

```
StateTtlConfig ttlConfig = StateTtlConfig
 .newBuilder(Time.seconds(10))
 .setUpdateType(StateTtlConfig.UpdateType.OnCreateAndWrite)
 .setStateVisibility(StateTtlConfig.StateVisibility.NeverReturnExpired)
 .build();
ValueStateDescriptor<String> stateDescriptor = new ValueStateDescriptor<>("my
state", String.class);
stateDescriptor.enableTimeToLive(ttlConfig);

```



### 算子状态

>算子状态的实际应用场景不如 Keyed State 多，一般用在 Source 或 Sink 等与外部系统连接 的算子上，或者完全没有 key 定义的场景。比如 Flink 的 Kafka 连接器中，就用到了算子状态。
>
>算子状态也支持不同的结构类型，主要有三种：ListState、UnionListState 和 BroadcastState。
>
>在我们给 Source 算子设置并行度后，Kafka 消费者的每一个并行实例，都会为对应的主题 261 （topic）分区维护一个偏移量， 作为算子状态保存起来。这在保证 Flink 应用“精确一次” （exactly-once）状态一致性时非常有用

#### 状态类型

##### 列表状态

>表示一组数据的列表
>
>与keyed state的区别是，不会按照key区分处理状态，一个子任务上只会保留一个列表，也就是并行子任务上所有状态项的集合。列表中的状态项就是可以重新分配的最细粒度，彼此之间互相独立。当进行重组时，将所有状态收集起来，再均匀的分配给所有的任务。

##### 联合列表状态

>也是表示为一个列表，但是与常规列表状态的区别在于，算子并行度缩放时对于状态的分配方式不同。
>
>当重组时，直接广播整个列表，让节点自己挑选状态使用、

##### 广播状态

>当我们需要算子并行子任务都保持同一份全局状态，用来做统一的配置和规则设定。那么所有分区都会访问到同一个状态，状态就像被广播了一样。这种算子状态叫做广播状态。
>
>在底层其是由map结构的键值对保存的，必须基于broadcaststream创建。

### 广播状态

>广播状态的概念很好理解，但是其应用与其他算子不同。

#### 基本用法

```
在代码上，可以直接调用 DataStream 的.broadcast()方法，传入一个“映射状态描述器”
（MapStateDescriptor）说明状态的名称和类型，就可以得到一个“广播流”（BroadcastStream）；
进而将要处理的数据流与这条广播流进行连接（connect），就会得到“广播连接流”
（BroadcastConnectedStream）。注意广播状态只能用在广播连接流中。

MapStateDescriptor<String, Rule> ruleStateDescriptor = new 
MapStateDescriptor<>(...);
BroadcastStream<Rule> ruleBroadcastStream = ruleStream
 .broadcast(ruleStateDescriptor);
DataStream<String> output = stream
 .connect(ruleBroadcastStream)
 .process( new BroadcastProcessFunction<>() {...} );


对 于 广 播 连 接 流 调 用 .process() 方 法 ， 可 以 传 入 “ 广 播 处 理 函 数 ”
KeyedBroadcastProcessFunction 或者 BroadcastProcessFunction 来进行处理计算。广播处理函数
里面有两个方法.processElement()和.processBroadcastElement()
这里的.processElement()方法，处理的是正常数据流，第一个参数 value 就是当前到来的流
数据；而.processBroadcastElement()方法就相当于是用来处理广播流的，它的第一个参数 value
就是广播流中的规则或者配置数据。
```

### 状态持久化和状态后端

>通过将状态持久化建立检查点或保存点到外部系统，一般是分布式文件系统。

#### 检查点

>保存内存状态的一个快照，如果发生故障，读取最近一次的检查点来恢复状态。
>
>如果检查点之后进行了其他数据处理，那么这些状态就会丢失，为了让结果正确，还需要让source算子重新读取这些数据，再次处理一遍。这就需要流有数据重放的功能，kafka通过记录便宜可以实现重放。

```
#开启检查点,传入参数单位为毫秒，是检查点的时间间隔
StreamExecutionEnvironment env = StreamExecutionEnvironment.getEnvironment();
env.enableCheckpointing(1000);
```

>保存点也可以保存状态持久化，但是要用户手动触发，定义镜像的保存，主要用于有计划的停止、重启等。

#### 状态后端

##### 状态后端的分类

>1哈希表状态后端：将状态当做对象，保存在TaskManager的JVM堆上。如触发器等都会以键值对保存在内存中，底层是一个哈希表。
>
>2内嵌RocksDB状态后端
>
>RocksDB是一种内嵌的key-value存储介质，可以把数据持久化到本地硬盘。默认存储在taskmanager的本地数据目录中。
>
>不适合

##### 状态后端的配置

>1默认后端
>
>flink-conf.yaml,使用state.backend来配置hashmap、rocksdb
>
>2为每个作业配置单独配置状态后端

```
state.backend:hashmap
state.checkpoints.dir: hdfs://namenode:40010/flink/checkpoints


#为每个作业配置单独配置状态后端
StreamExecutionEnvironment env = 
StreamExecutionEnvironment.getExecutionEnvironment();
env.setStateBackend(new EmbeddedRocksDBStateBackend());

```

```
#如果在idea中需要使用EmbeddedRocksDBStateBackend，需要引入如下依赖。
<dependency>
 <groupId>org.apache.flink</groupId>
 
<artifactId>flink-statebackend-rocksdb_${scala.binary.version}</artifactId>
 <version>1.13.0</version>
</dependency>
```

## 容错机制

### 检查点

>在有状态的流处理中，我们想恢复计算程序，并不需要之前的数据，只需要内存状态。故障恢复后的处理的结果应该与故障发生前一致。

#### 检查点的保存

>检查点的保存时机

##### 周期性的触发保存

>每隔一定时间间隔，将每个任务当前的状态复制一份，并持久化保存起来。

##### 保存的时间点

>当检查点触发时，任务可能正在处理某个数据，为了能进行快照恢复，我们需要让任务完成处理手头的数据，这样在恢复时，只通过内存状态和当前处理到哪个数据了，就能恢复任务。但是不同的并行子任务可能处理的进度不一样，所以我们必须等待所有任务都完成了该数据的处理。然后在恢复时，只需要根据偏移量重放需要继续处理的数据。
>
>那如果有些操作改变了下游数据库，又被重放了呢。

##### 保存的具体流程

>当需要保存时，等到所有任务将同一个数据处理完毕，然后将任务中的算子状态都保存起来。比如source的偏移量，聚合算子的内存状态。

#### 从检查点恢复状态

>当发生故障时，找到最近一次成功保存的检查点来恢复状态。
>
>1.重启应用，重启后所有状态会清空
>
>2.读取检查点，重置状态。将检查点中的状态填充到对应的状态中，将状态恢复到flink保存检查点的那一刻。
>
>3.重放数据，让source任务向外部数据源重新提交偏移量，这样整个系统就恢复到了保存快照的那一刻。
>
>4.继续处理数据

#### 检查点算法

>如何等待所有任务将同一个数据处理完毕呢，数据在算子间变换、合并，根本无法判断那些数据是同一个。较为简单的想法是，当需要快照时，将source算子暂停，然后等后续任务处理完所有数据，这样所有任务都将同一个数据处理完成了。但是这样效率很低，资源闲置。

##### 检查点分界线 Barrier

>如何让每个任务认出触发检查点保存的那条数据呢，我们可以借鉴流水线的思想，在数据中插入一个特殊的数据结构barrier，当下游算子处理barrier时，将此刻的状态保存进入快照。而后到来的数据不会被保存在这个快照中。barrier将数据流划分成了不同部分，也叫分界线checkpoint barrier。
>
>在 JobManager 中有一个“检查点协调器”（checkpoint coordinator），专门用来协调处理检 查点的相关工作。检查点协调器会定期向 TaskManager 发出指令，要求保存检查点（带着检查 点 ID）；TaskManager 会让所有的 Source 任务把自己的偏移量（算子状态）保存起来，并将带 有检查点 ID 的分界线（barrier）插入到当前的数据流中，然后像正常的数据一样像下游传递； 之后 Source 任务就可以继续读入新的数据了

##### 分布式快照算法

>通过在分界线barrier，我们可以明确地指示触发检查点保存的时间。但是，对于分布式数据流，数据是乱序的，如何保持数据的顺序，让barrier正确保存快照就不容易了。
>
>1jm发送指令，触发检查点保存。source任务保存状态，插入分界线。
>
>jm周期性的向tm发送带有新检查点id的消息，通过这种方式启动检查点。tm收到指令后，插入barrier，并将偏移量保存持久化。
>
>2.状态快照保存完成，分界线传递给下游
>
>状态存入持久化内存后，会返回通知给Source任务，source任务就会向jm确认检查点完成，然后将barrier传递给下游。
>
>3.向下游多个并行子任务广播分界线，执行分界线对齐
>
>4.对齐后，保存状态的持久化存储
>
>5.先处理缓存数据，然后继续正常处理。



#### 检查点配置

```
StreamExecutionEnvironment env = 
StreamExecutionEnvironment.getExecutionEnvironment();
// 每隔 1 秒启动一次检查点保存
env.enableCheckpointing(1000);

// 配置存储检查点到 JobManager 堆内存
env.getCheckpointConfig().setCheckpointStorage(new 
JobManagerCheckpointStorage());
// 配置存储检查点到文件系统
env.getCheckpointConfig().setCheckpointStorage(new 
FileSystemCheckpointStorage("hdfs://namenode:40010/flink/checkpoints"));

```

```
StreamExecutionEnvironment env = 
StreamExecutionEnvironment.getExecutionEnvironment();
// 启用检查点，间隔时间 1 秒
env.enableCheckpointing(1000);
CheckpointConfig checkpointConfig = env.getCheckpointConfig();
// 设置精确一次模式
checkpointConfig.setCheckpointingMode(CheckpointingMode.EXACTLY_ONCE);
// 最小间隔时间 500 毫秒
checkpointConfig.setMinPauseBetweenCheckpoints(500);
// 超时时间 1 分钟
checkpointConfig.setCheckpointTimeout(60000);
// 同时只能有一个检查点
checkpointConfig.setMaxConcurrentCheckpoints(1);
// 开启检查点的外部持久化保存，作业取消后依然保留
checkpointConfig.enableExternalizedCheckpoints(
 ExternalizedCheckpointCleanup.RETAIN_ON_CANCELLATION);
// 启用不对齐的检查点保存方式
checkpointConfig.enableUnalignedCheckpoints();
292
// 设置检查点存储，可以直接传入一个 String，指定文件系统的路径
checkpointConfig.setCheckpointStorage("hdfs://my/checkpoint/dir")
```

#### 保存点

>保存点与检查点最大的区别，就是触发的时机。检查点是由 Flink 自动管理的，定期创建， 发生故障之后自动读取进行恢复，这是一个“自动存盘”的功能；而保存点不会自动创建，必 须由用户明确地手动触发保存操作，所以就是“手动存盘”

```
DataStream<String> stream = env
 .addSource(new StatefulSource())
 .uid("source-id")
 .map(new StatefulMapper())
 .uid("mapper-id")
 .print();

bin/flink savepoint :jobId [:targetDirectory]
#通过flink-conf.yaml设定保存路径
state.savepoints.dir: hdfs:///flink/savepoints
#通过程序设定保存路径
env.setDefaultSavepointDir("hdfs:///flink/savepoints");
#从保存点启动程序
bin/flink run -s :savepointPath [:runArgs]
```

### 状态一致性

>flink中的一致性主要用于故障恢复的描述中
>
>对于分布式系统而言，强调的是不同节点中相同数据的副本应该总是一致的，也就是不同节点读取时总能取得相同的值。
>
>而对于事务而言，是要求提交更新操作后，能够读取到新的数据。
>
>对于flink来说，就是保证不漏掉任何一个数据，也不会重复处理同一个数据。

#### 一致性的概念和级别

>一般来说有三种级别：
>
>1最多一次
>
>2至少一次
>
>3精确一次

#### 端到端的精确一次

>我们可以保证状态的一致性
>
>但是对于输出端和输入端如何保证呢，包括了数据源、流处理器和外部存储系统三个部分。这个完 整应用的一致性，就叫作“端到端（end-to-end）的状态一致性”，它取决于三个组件中最弱的 那一环。
>
>一般来说，能否达到 at-least-once 一致性级别，主要看数据源能够重放数据；而能否 达到 exactly-once 级别，流处理器内部、数据源、外部存储都要有相应的保证机制。

##### 输入端保证

>想要在故障后不丢失数据，就需要重放数据，回到快照拍摄的那一刻的偏移量。达到至少一次的语义要求，也是精确一次的基本要求。

##### 输出端保证

>通过输入端重放已经能达到至少次的要求，但是精确一次仍有问题。
>
>当一个检查点保存后，后续到来的数据也会进行处理，并写入外部系统，如果此时出现故障，那么内存状态还没有改变，但是外部系统已经改变了。所以，当恢复应用时，就会回到之前的检查点，重复处理这些数据，再次计算并写入外部系统。虽然状态回滚了，保持了状态的一致性，但是写入了外部系统两次。
>
>为了实现端到端的一致性，还需要对外部存储系统、以及Sink连接器有额外的要求。有两种处理方式：
>
>1.幂等写入
>
>2.事务写入

###### 幂等写入

>幂等也就是，一个操作可以重复多次，但是只导致一次的结果更改，也就是说后边重复的操作不会对结果起作用。
>
>在数据处理领域就是hashmap，如果是相同的键值对，后边的插入就没有作用了
>
>数据存储中的redis键值存储，或者关系型数据库中满足查询条件的更新操作
>
>对于幂等写入，故障恢复时会出现短暂的不一致，因为保存点之后的数据其实已经写入数据库了，当恢复的任务执行到故障点时，就恢复正常了。

###### 事务写入

>就是当故障发生时，从上一个检查点到当前数据的操作，都不生效。只有当检查点完成提交后，再真正存入数据库中，让快照和数据库的数据保持一致性。
>
>具体来说有两种方式：预写日志WAL和两阶段提交2PC
>
>1.预写日志write-ahead-log，WAL
>
>对于不支持的事务的存储系统，借助WAL实现
>
>具体步骤是（
>
>（1）先把结果数据作为日志保存起来
>
>（2）进行检查点保存时，也会讲这些结果数据一并做持久化存储
>
>  (3)  在收到检查点完成的通知时，将所有结果一次性写入外部系统。
>
>这就相当于检查点完成时做一个批处理，flink中DataStream API提供了一个模板类GenericWriteAheadSink，用来实现事务型的写入方式。
>
>这种方式需要保存数据批量写入成功的信息，在故障恢复检查点时，必须有成功写入的信息对应。
>
>
>
>2.两阶段提交
>
>（1).当一条数据到来时，或者收到检查点的分界线时，sink都会启动一个事务
>
>  (2).接下来收到的所有数据，都可以通过这个事务写入外部系统；这时由于事务没有提交，所以数据尽管写入了外部系统，但是不可用，是“预提交”的状态。
>
>  (3).当sink任务收到jobmanager发来的检查点完成的通知时，正式提交事务，写入结果就可用了。
>
>flink提供了TwoPhaseCommitSinkFunction接口，方便实现两阶段提交的SinkFunction，提供了精确一次处理。
>
>但是其对外部系统有如下要求：
>
>1. 外部系统提供事务支持
>2. 在检查点间隔期间，能开启一个事务并接收数据写入
>3. 收到完成通知以前，都是等待提交状态。
>4. sink任务必须能够在进程失败后恢复事务
>5. 提交事务必须是幂等操作。也就是说，事务的重复提交应该是无效的。

#### Flink和kafka连接时的精确一次消费

##### Flink内部

>借助检查点机制保证状态和处理结果的精确一次

##### 输入端

>可以重置偏移量，当故障发生后，恢复时从source算子状态中读取偏移量，重放数据。

##### 输出端

>flink实现的kafka连接器中，提供了kafka的flinkkafkaproducer，他就实现了twophasecommitsinkfunction接口。

>需要的配置
>
>1.启用检查点
>
>2.在 FlinkKafkaProducer 的构造函数中传入参数 Semantic.EXACTLY_ONCE
>
>3.配置 Kafka 读取数据的消费者的隔离级别
>
>​      这里所说的 Kafka，是写入的外部系统。预提交阶段数据已经写入，只是被标记为“未提 交”（uncommitted），而 Kafka 中默认的隔离级别 isolation.level 是 read_uncommitted，也就是 可以读取未提交的数据。这样一来，外部应用就可以直接消费未提交的数据，对于事务性的保 证就失效了。所以应该将隔离级别配置
>
>为 read_committed，表示消费者遇到未提交的消息时，会停止从分区中消费数据，直到消 息被标记为已提交才会再次恢复消费。当然，这样做的话，外部应用消费数据就会有显著的延 迟。

>Flink 的 Kafka连接器中配置的事务超时时间 transaction.timeout.ms 默认是 1小时，而Kafka 集群配置的事务最大超时时间 transaction.max.timeout.ms 默认是 15 分钟。所以在检查点保存 时间很长时，有可能出现 Kafka 已经认为事务超时了，丢弃了预提交的数据；而 Sink 任务认 为还可以继续等待。如果接下来检查点保存成功，发生故障后回滚到这个检查点的状态，这部 分数据就被真正丢掉了。所以这两个超时时间，前者应该小于等于后者。

## Fink SQL和Table API

>Flink提供的多层级API中，核心是DataStream API，使我们开发的基本途径；底层则是处理函数，可以访问事件时间信息，内存状态，窗口信息，定时器等。
>
>在企业应用中，会有大量类似的处理逻辑，所以一般会将底层API包装成具体的应用级接口，也就是SQL，flink提供了Table API和SQL处理表结构数据。

>spark，mapreduce，flink这些框架的流批处理功能和接口：
>
>mapreduce：基础批处理编程接口
>
>spark：基础批处理编程接口，spark streaming DStream流处理接口，spark sql结构化数据处理接口
>
>flink：DataStream流批一体接口，Table API和SQL 流批一体接口
>
>
>
>离线数仓的分层结构，不可能达到实时的处理速度，实时要求都是1秒甚至毫秒为单位，离线数仓的分层结构导致有大量的落盘操作，而且它是为单次大批量输入设计的，以天等时间单位为粒度。
>
>实时数仓只有处理的数据流，在最后阶段进行数据的sink。中间步骤一般使用kafka进行临时存储，统计的一般是当天的数据，用于一些实时的指标。



## Flink CEP

## problem

### flink elasticsearch sink无法写入

>flink elasticsearch sink无法写入，程序也不报错。
>
>就是sink的触发机制问题啊，没想到，它肯定是做了缓存的，到达某个时间或数据量到达多少，就会触发sink操作。

```
#builder添加setBulkFlushMaxActions参数后，可以控制多久写入es数据库一次。
       ElasticsearchSinkFunction<TopProductEntity> elasticsearchSinkFunction=new ElasticsearchSinkFunction<TopProductEntity>() {
            @Override
            public void process(TopProductEntity topProductEntity, RuntimeContext runtimeContext, RequestIndexer requestIndexer) {
                HashMap<String,String> data=new HashMap<>();
                data.put("productid",Integer.valueOf(topProductEntity.getProductId()).toString());
                data.put("times",Integer.valueOf(topProductEntity.getActionTimes()).toString());
                data.put("windowEnd",Long.valueOf(topProductEntity.getWindowEnd()).toString());
                IndexRequest request= Requests.indexRequest().index("topproduct").source(data);
                System.out.println(data);
                requestIndexer.add(request);
            }
        };
        ArrayList<HttpHost> httpHosts=new ArrayList<>();
        httpHosts.add(new HttpHost("192.168.244.128",9201,"http"));
        ElasticsearchSink.Builder<TopProductEntity> builder = new ElasticsearchSink.Builder<TopProductEntity>(httpHosts,elasticsearchSinkFunction );
        builder.setBulkFlushMaxActions(1);
        topProduct.addSink(builder.build());
```



# Fink zhihu

>知乎是一个问答平台，每个使用者可以提出自己的问题，也可以回答自己擅长的问题。也可以在平台上发布文章，短视频等。用户主要通过推送的方式接收信息，也可以通过搜索检索特定的问题。
>
>主要流程：用户的行为日志（浏览，点击，停留，搜索），用户的数据库信息（收藏，关注，点赞）
>
>采集到flink的pipline，实时统计热门话题（话题浏览量）、问题、答主，基于浏览历史推荐
>
>用户：行为日志，从后端、前段日志收集，用于推荐。新老用户识别统计，流失新增回流
>
>内容：统计各种维度（浏览量，回答数，关注数），从业务数据库、后端日志收集。
>
>活动广告：外链来源，从链接信息收集。

>关注的业务过程，哪些指标。如电商的那么多业务过程，如用户的维度的浏览等行为，内容、商品的pv、uv量、活动的外链来源。

## 内容

### 浏览量

>内容浏览量包括创建以来，以及以天为粒度（离线），和截至目前。
>
>当用户浏览某个回答时，会向后端对应接口请求文章内容，这时后端服务器可以存储日志文件并发送日志到kafka，借助切面编程可以高效实现。借助flink统计好之后可以写入hbase持久化。

#### hbase与redis

>redis是高速缓存，借助内存提供高速读写，时间在几十微秒，内存中可以存储的数据量是有限的。主要考虑的指标有：读写性能，数据的可靠性，存储的数据量
>
>hbase是持久化的key-value存储，可以满足高频append操作，但是读取较慢，时间在几毫秒。主要考虑指标：读写性能，数据的可靠性，存储的数据量

#### lambda与kappa

>https://bbs.huaweicloud.com/blogs/detail/282421
>
>lambda一个业务逻辑，两套编码方案，通过将离线历史数据和实时增量数据合并输出。
>
>缺点：1开发难度大，维护难度大
>
>2批处理的任务需要在夜间处理完所有前一天的数据
>
>3数据仓库的典型设计，会产生大量的中间表，数据急速膨胀。
>
>
>
>kappa用kafka或类似的分布式队列保存数据，需要几天的数据就保存几天。当需要重新计算时，重新创建一个流计算实例，从头开始读取数据进行输出，并保存到一个新的结果存储中。当新的实例计算完后，停止老的计算实例。在kappa架构中，只有在必要时才会对历史数据进行重复计算，并且批处理和流处理是同一份代码。kappa两个流协作输出，queries使用最新一个流的处理结果。
>
>缺点：1消息中间件缓存的数据量和回溯数据有性能瓶颈。通常需要过去的历史数据，如果都存在消息中间件里，无疑有非常大的压力。同时，一次性回溯180天的数据量，资源消耗非常大。
>
>相比之下，lambda批处理能力更强，但是机器开销大。lambda将数据存储在kafka中，离线将原始数据分层分区存储在hdfs或hive表中，可以取任意时间段，不止可以取全部数据。kappa架构必须在数据中维护时间的信息，不然如何实现重新处理，因为编写流处理时，可能只关注当天的的累计数量。那如果我需要每天粒度的统计呢。当业务逻辑更新时通过重新处理历史数据kafka文件达到目的。

| 项目           | Lambda                                 | Kappa                                    |
| -------------- | -------------------------------------- | ---------------------------------------- |
| 数据处理能力   | 可以处理超大规模的历史数据             | 历史数据处理的能力有限                   |
| 机器开销       | 批处理和实时计算需一直运行，机器开销大 | 必要时进行全量计算，机器开销相对较小     |
| 存储开销       | 只需要保存一份查询结果，存储开销较小   | 需要存储新老实例结果，存储开销相对较大   |
| 开发、测试难度 | 实现两套代码，开发、测试难度较大       | 只需面对一个框架，开发、测试难度相对较小 |
| 运维成本       | 维护两套系统，运维成本大               | 只需维护一个框架，运维成本小             |

### 回答数

### 关注数

## 用户

# Flink 实时项目

## 流处理模块说明

>在flink-2hbase中，主要分为6个flink任务
>
>mysql中中主要存储用户信息，商品信息，相当于维表，主要用于拼接获得信息。
>
>hbase用于存储从flink处理完的数据结果。

### 用户-产品浏览历史->实现基于协同过滤的推荐逻辑

>通过flink记录用户浏览过这个类目下的哪些产品，为后面的基于Item的协同过滤做准备，实时的记录用户的评分到Hbase中，为后续离线处理做准备。

>从kafka的con topic读取数据，继承mapFunction编写map函数，将用户id、产品id分别存入u_history表和p_history表的对应行，并增加计数。

### 用户-兴趣->实现基于上下文的推荐逻辑

>根据用户对同一个产品的操作计算兴趣度，计算规则通过操作事件间隔（如购物-浏览 <100s）则判定为一次兴趣事件，通过flink的valueState实现如果用户的操作Aciton=3（购物），则清除这个产品的state，如果超过100s没有出现Action=3的事件，也清除这个state。

>从kafka的con topic读取数据，获取到日志后封装为Log对象，然后按照userid分组。最后调用map函数，继承RichMapFunction编写内容，实现记录用户兴趣。实际上就是在100s内进行了连续向后的操作，就记为一次兴趣事件，每触发一个兴趣事件，将u_interest表中以userid为rowkey的记录的，列族中列名为productid的数值加一。增加计数。

>RichMapFunction的内容如下。可以说对于连续性的操作才会定义为兴趣，如100s内浏览并分享，100s内浏览并购物。

```
#从getRuntimeContext中获取state，放入类的属性，供map函数使用
@Override
    public void open(Configuration parameters) throws Exception {

        // 设置 state 的过期时间为100s
        StateTtlConfig ttlConfig = StateTtlConfig
                .newBuilder(Time.seconds(100L))
                .setUpdateType(StateTtlConfig.UpdateType.OnCreateAndWrite)
                .setStateVisibility(StateTtlConfig.StateVisibility.NeverReturnExpired)
                .build();

        ValueStateDescriptor<Action> desc = new ValueStateDescriptor<>("Action time", Action.class);
        desc.enableTimeToLive(ttlConfig);
        state = getRuntimeContext().getState(desc);
    }
    
#map函数操作state，进行更新和计算
@Override
    public String map(LogEntity logEntity) throws Exception {
    //flink状态中保存的值
        Action actionLastTime = state.value();
      //当前这个数据封装的Action对象（动作类型，动作时间）
        Action actionThisTime = new Action(logEntity.getAction(), logEntity.getTime().toString());
        int times = 1;
        // 如果用户没有操作 则为state创建值
        //如果flink状态中没有state，那么需要为其创建一个新的flink状态，此次状态当作flink的状态存入。同时，该条计数加一保存到hbase。
        if (actionLastTime == null) {
            actionLastTime = actionThisTime;
            saveToHBase(logEntity, 1);
            //另外，如果flink中已经有该state，那么根据规则计算后，也存入hbase计数加对应次数。
        }else{
            times = getTimesByRule(actionLastTime, actionThisTime);
        }
        saveToHBase(logEntity, times);

        // 如果用户的操作为3(购物),则清除这个key的state
        if (actionThisTime.getType().equals("3")){
            state.clear();
        }
        return null;
}
```



### 用户画像计算->实现基于标签的推荐逻辑

>按照三个维度计算用户画像，分别是用户的颜色兴趣，用户的产地兴趣和用户的风格兴趣，根据日志不断的修改用户画像的数据，记录在Hbase中。数据存储在hbase的user表中

>从kafka的con topic读取数据，查询对应的产品信息（country、color、style），在以userId为rowkey的记录中找到以（country、color、style）为列的cell，增加计数。存入的表名为user表。

### 产品画像记录->实现基于标签的推荐逻辑

>用两个维度记录产品画像，一个是喜爱该产品的年龄段，另一个是性别，数据存储在Hbase的prod表。

>从kafka的con topic中读取数据，继承mapFunction编写map函数，从mysql的user表查询出该用户的sex、age信息，数据存入hbase的prod表。增加计数。

### 实时热度榜->实现基于热度的推荐逻辑

>通过flink时间窗口机制，统计当前时间的实时热度，并将数据缓存在Redis中，通过Flink的窗口机制计算实时热度，使用ListState保存一次热度榜。数据存储在redis中，按照时间戳存储list。增加计数。

```
#将数据统计后写入redis，用于实时热度榜使用
首先按照productId分组，对于每个productId内部使用滑动窗口，对于窗口内的进行aggregate操作，统计商品次数封装为topProduct对象。
aggregate目的是，每有一条数据累加一次。然后使用keyBy按照windwoEnd分组，然后对相同windwoEnd内的商品借助keyBy分组到一起，再进行排序，获得商品的排名。最后将结果写入。这里要理解为什么还要按照windwoEnd分组，就要理解窗口函数的输出传递到下游是什么，他是一个由分区号+窗口时间段唯一确定的一条记录。
```

```
#流处理的概念理解
当我们对流施加keyBy操作，本质是创建n个分区，当数据到来时，数据被分发到不同的分区。如果keyBy后续有操作，那么本质就是在多个分区上施加该操作。如果后续是聚合，那么会得到对应数目的聚合结果，聚合结果是针对整个流进行累积的状态呢，还是当前单个流呢？是针对整个流，来了新数据它会尝试更新状态，然后发往下一个任务。如果加了窗口呢？那就是针对一个窗口的状态，一个计算完就结束了，没有累计状态。
这个的问题是，为了统计相同窗口的，不同分区的聚合数据。如果不添加按照窗口分区，那么数据由同一个map处理。就是累积了嘛？额，所有没加窗口的聚合都是累积吧。
理清的关键是确定1.何时触发计算2计算的数据对象是什么3计算的结果是累积状态嘛4计算的结果发往哪里

即使用window后，当水位线到达windwoend时，就触发计算。计算的对象为该widows内数据，如果分区了，还要截取对应分区。计算的结果不是累积state，计算结果数量时分区数*窗口个数，该数目的记录数全部发往下游的任务。下游的任务，如果需要对应窗口的数据，只能借助窗口时间分组，才能获取到对应的数据，自己使用窗口时间分组，就涉及到触发时间的问题，因为分组无法自己触发计算，所以要借助定时器，当到达end时触发计算，统计该时间分组的所有数据。flatmap。
```

```
aggregate：对数据进行统计聚合
process:可以对流施加复杂的操作，包括设定定时器、触发定时器等。除了设定map函数，还可以设定其他如open、onTimer等函数，时较为底层的api，功能丰富。
```



### 日志导入

>从kafka接受的数据直接写入Hbase事实表，保存完整的日志log，日志中包含了用户id，用户操作的产品id，操作时间，行为（如购买，点击，推荐等）
>
>数据按时间窗口统计数据大屏需要的数据，返回前段显示
>
>数据存储在Hbase的con表

>从kafka的con  topic读取数据，继承mapFunction编写map函数，将日志解析为LogEntity(userid,produceid,time,action)，然后根据用户id、产品id、时间戳拼接hbase的rowkey，最终将每一条记录插入hbase的con表中。

### Elasticsearch Sink

```
PUT /topproduct
{
	"settings": { 
 	"number_of_shards": 3
 },
	"mappings": {
		  "properties":{
		     "productid":{
		     	"type":"keyword"
		     },
		     "times":{
		     	"type":"keyword"
		     },
		     "windowEnd":{
		     	"type":"date"
		     }
		  }
		}
	
}
```



## web模块

### 前台用户

>该页面返回给用户推荐的产品list，使用html编写

>前端的推荐模块主要分为三块：1热榜推荐2基于协同过滤3产品画像

#### 热榜

##### 热榜推荐

>从redis查出热榜的所有信息，根据热榜中的产品id查询产品基本信息product表、产品详情表contact，这两个是mysql中的维度表。

##### 热榜+协同过滤

>首先计算px表，对于一个produceid，为它启动一个单独的线程，执行协同过滤的计算，然后将结果写入px表。对于给定的两个productid，为了计算评分，从p_history表根据productid查出所有相关的列。然后比对两个product是否拥有相同的用户，每有一个相同的用户，sum就加一。然后使用sum除以total，total就是（product1有的用户数*product2有的用户数）再开根号。就相当于在计算，在所有用户中，两种产品有多少共同用户。数据写入px表，rowkey为产品id，列名为产品id。

>在正常情况下，是要计算每个商品的被用户打的分数，或者其他度量，如被用户的操作次数等，这比单纯的计数更有意义。

>从toplist中取出热榜商品的pid，然后从px表中取出pid对应的所有行记录，返回一个结果id的集合。然后根据结果id集合查询产品详情表、产品基本信息表。最后返回结果。

##### 热榜+产品标签画像

>首先计算ps表，产品的画像计算产品的相似度。对于给定的两个产品。从prod用户画像表查出数据，统计共有的标签属性有那些，这些标签的总数sum起来，标签有性别、年龄等，然后除以sqrt（product1的属性累计数量*product2的属性累计数量），相当于在计算两个商品共有的标签属性有多少个，在总的属性中占有多少。这里实际上可以加以改进，加入具体标签的数值计算相关性。数据写入ps表，rowkey为产品id，列名为产品id。

>从toplist中取出热榜商品的pid，然后从ps表取出所有对应的id。最后拼接出结果并返回。

#### 用户

##### 用户协同顾虑

>先根据用户历史表u_history计算每个用户的相近的用户，并记录数据到hbase的表。

##### 用户标签画像

>先根据用户画像表user计算每个用户的相近的用户，并记录数据到hbase的表。
>
>主要由商品的国家、风格、颜色组成。

##### 用户兴趣

>先根据用户兴趣表u_interest计算每个用户的相近的用户，并记录数据到hbase的表。

>用户兴趣的定义，

### 后台监控

>使用superset，es kibana查看效果

>该页面返回给管理员指标监控，主要包括热榜产品，日志接入量

## 推荐引擎说明

## 前台推荐页面

>分为三列，分别是热度榜推荐，协同过滤推荐和产品画像推荐

## 后台数据大屏

>包含热度榜和1小时日志接入量两个指标，其真实数据位置在resource/database.sql

