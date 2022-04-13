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
 
<artifactId>flink-connector-elasticsearch7_${scala.binary.version}</artifactI
d>
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



## 窗口函数

## 多流操作

## 状态编程

### 指定key的几种方法

>https://www.jianshu.com/p/faaa059453fb
>
>1.使用tuples指定key
>
>keyBy(0)
>
>keyBy("f0.f1")
>
>2.Field Expression
>
>使用xx.f0表示tuple的第一个元素
>
>"xx.0"
>
>3使用KeySelector
>
>返回想指定为key值的数据即可。

### Flink CDC

### Fink SQL

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

## 模块说明

>在flink-2hbase中，主要分为6个flink任务

### 用户-产品浏览历史->实现基于协同过滤的推荐逻辑

>通过flink记录用户浏览过这个类目下的哪些产品，为后面的基于Item的协同过滤做准备，实时的记录用户的评分到Hbase中，为后续离线处理做准备。

>从kafka的con topic读取数据，继承mapFunction编写map函数，将用户id、产品id分别存入u_history表和p_history表。增加计数。

### 用户-兴趣->实现基于上下文的推荐逻辑

>根据用户对同一个产品的操作计算兴趣度，计算规则通过操作事件间隔（如购物-浏览 <100s）则判定为一次兴趣事件，通过flink的valueState实现如果用户的操作Aciton=3（购物），则清除这个产品的state，如果超过100s没有出现Action=3的事件，也清除这个state。

>从kafka的con topic读取数据，获取到日志后封装为Log对象，然后按照userid分组。最后调用map函数，继承RichMapFunction编写内容，实现记录用户兴趣。实际上就是在100s内进行了连续向后的操作，就记为一次兴趣事件，每触发一个兴趣事件，将userid为rowkey的记录的，列族中列名为productid的数值加一。增加计数。

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
        Action actionLastTime = state.value();
        Action actionThisTime = new Action(logEntity.getAction(), logEntity.getTime().toString());
        int times = 1;
        // 如果用户没有操作 则为state创建值
        if (actionLastTime == null) {
            actionLastTime = actionThisTime;
            saveToHBase(logEntity, 1);
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

>从kafka的con topic读取数据，查询对应的产品信息（country、color、style），在以userId为rowkey的记录中找到以（country、color、style）为列的cell，增加计数。

### 产品画像记录->实现基于标签的推荐逻辑

>用两个维度记录产品画像，一个是喜爱该产品的年龄段，另一个是性别，数据存储在Hbase的prod表。

>从kafka的con topic中读取数据，继承mapFunction编写map函数，从mysql的user表查询出该用户的信息，数据存入hbase的prod表。增加计数。

### 实时热度榜->实现基于热度的推荐逻辑

>通过flink时间窗口机制，统计当前时间的实时热度，并将数据缓存在Redis中，通过Flink的窗口机制计算实时热度，使用ListState保存一次热度榜。数据存储在redis中，按照时间戳存储list。增加计数。

```
#将数据统计后写入redis，用于实时热度榜使用
首先按照productId分组，对于每个productId内部使用滑动窗口，对于窗口内的进行aggregate操作。
aggregate目的是，每有一条数据累加一次。
```



### 日志导入

>从kafka接受的数据直接写入Hbase事实表，保存完整的日志log，日志中包含了用户id，用户操作的产品id，操作时间，行为（如购买，点击，推荐等）
>
>数据按时间窗口统计数据大屏需要的数据，返回前段显示
>
>数据存储在Hbase的con表

>从kafka的logs topic读取数据，继承mapFunction编写map函数，将日志解析为LogEntity(userid,produceid,time,action)，然后根据用户id、产品id、时间戳拼接hbase的rowkey，最终将每一条记录插入hbase的con表中。

## web模块

### 前台用户

>该页面返回给用户推荐的产品list

### 后台监控

>该页面返回给管理员指标监控

## 推荐引擎说明

## 前台推荐页面

>分为三列，分别是热度榜推荐，协同过滤推荐和产品画像推荐

## 后台数据大屏

>包含热度榜和1小时日志接入量两个指标，其真实数据位置在resource/database.sql

