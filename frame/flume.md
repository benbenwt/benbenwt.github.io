[TOC]
### 安装

```
#解压缩安装包
#进入conf文件夹
mv flume-env.sh.template flume-env.sh
vi flume-env.sh
export JAVA_HOME=/opt/module/jdk1.8.0_212
```

### 使用

##### 启动flume

```
bin/flume-ng agent --name a1 --conf-file conf/file-flume-kafka.conf  &
nohup /root/module/flume/bin/flume-ng agent --conf-file /root/module/flume/conf/file-flume-kafka.conf --name a1 -Dflume.root.logger=INFO,LOGFILE > /root/module/flume/log1.txt 2>&1 &
```



##### 编写file-kafka配置文件

```
#为各组件命名
a1.sources = r1
a1.channels = c1

#描述source
a1.sources.r1.type = TAILDIR
a1.sources.r1.filegroups = f1
a1.sources.r1.filegroups.f1 = /opt/module/applog/log/app.*
a1.sources.r1.positionFile = /opt/module/flume/taildir_position.json
a1.sources.r1.interceptors =  i1
a1.sources.r1.interceptors.i1.type = com.atguigu.flume.interceptor.ETLInterceptor$Builder

#描述channel
a1.channels.c1.type = org.apache.flume.channel.kafka.KafkaChannel
a1.channels.c1.kafka.bootstrap.servers = hadoop102:9092,hadoop103:9092
a1.channels.c1.kafka.topic = topic_log
a1.channels.c1.parseAsFlumeEvent = false

#绑定source和channel以及sink和channel的关系
a1.sources.r1.channels = c1

```

```
#练习
a2.sources=t2
a2.channels=k2

a2.sources.t2.type= TAILDIR
a2.sources.t2.filegroups= f1 f2
a2.sources.t2.filegroups.f1= /var/log/*.log
a2.sources.t2.filegroups.f2= /var/log2/*test.log

a2.channels.k2.type= org.apache.flume.channel.kafka.KafkaChannel
a2.channels.k2.kafka.bootstrap.servers= host1:9092,host2:9092,host3:9092
a2.channels.k2.topic= test

a2.sources.t2.channels= k2
```

##### 编写kafka-hdfs配置文件

>hdfs.roolInterval=3600,flume在hdfs创建的文件时间超过3600时，使用新的文件装后续数据。
>
>hdfs.roolSize=134217728，文件达到128M时滚动生成新文件
>
>hdfs.rollCount =0

```
## 组件
a1.sources=r1
a1.channels=c1
a1.sinks=k1

## source1
a1.sources.r1.type = org.apache.flume.source.kafka.KafkaSource
a1.sources.r1.batchSize = 5000
a1.sources.r1.batchDurationMillis = 2000
a1.sources.r1.kafka.bootstrap.servers = hadoop102:9092,hadoop103:9092,hadoop104:9092
a1.sources.r1.kafka.topics=topic_log
a1.sources.r1.interceptors = i1
a1.sources.r1.interceptors.i1.type = com.atguigu.flume.interceptor.TimeStampInterceptor$Builder

## channel1
a1.channels.c1.type = file
a1.channels.c1.checkpointDir = /opt/module/flume/checkpoint/behavior1
a1.channels.c1.dataDirs = /opt/module/flume/data/behavior1/


## sink1
a1.sinks.k1.type = hdfs
a1.sinks.k1.hdfs.path = /origin_data/gmall/log/topic_log/%Y-%m-%d
a1.sinks.k1.hdfs.filePrefix = log-
a1.sinks.k1.hdfs.round = false

#控制生成的小文件
a1.sinks.k1.hdfs.rollInterval = 10
a1.sinks.k1.hdfs.rollSize = 134217728
a1.sinks.k1.hdfs.rollCount = 0

## 控制输出文件是原生文件。
a1.sinks.k1.hdfs.fileType = CompressedStream
a1.sinks.k1.hdfs.codeC = lzop

## 拼装
a1.sources.r1.channels = c1
a1.sinks.k1.channel= c1

```

##### 编写flume过滤拦截器

>需要将打包好的拦截器放到flume/lib文件夹下生效

#pom文件

```
<dependencies>
    <dependency>
        <groupId>org.apache.flume</groupId>
        <artifactId>flume-ng-core</artifactId>
        <version>1.9.0</version>
        <scope>provided</scope>
    </dependency>

    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>fastjson</artifactId>
        <version>1.2.62</version>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>2.3.2</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
            </configuration>
        </plugin>
        <plugin>
            <artifactId>maven-assembly-plugin</artifactId>
            <configuration>
                <descriptorRefs>
                    <descriptorRef>jar-with-dependencies</descriptorRef>
                </descriptorRefs>
            </configuration>
            <executions>
                <execution>
                    <id>make-assembly</id>
                    <phase>package</phase>
                    <goals>
                        <goal>single</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>

```

# flume理论

### source

>以行为单位检测指定的所有文件，每有一行增加，将改行的内容作为body进行封装Event，并发送到channel。

### channel

### sink

# flume用法
### windows 命令
>>flume-ng.cmd agent --conf ..\conf -conf-file ..\conf\flume_file_kafka.conf --name agent1 -property flume.root.logger=INFO,console

### ExecSource

### Sink

##### hdfs Sink

>hdfs sink有三种类型：SequenceFile,Datastream,CompressedStream
>
>https://blog.csdn.net/helloxiaozhe/article/details/88417091.
>
>对于要导入TEXTFILE的hive表的数据，flume sink时必须使用Datastream。

# Flume架构和用法示例

### Flume架构

>Flume是一个分布式的日志数据收集、聚合系统，它可以收集很多不同来源的数据到数据中心存储。
>
>Flume Event是一个数据单元，body中包含一定字节的数据，headers包含一些属性，如时间戳。
>
>Flume Agent是一个JVM进程，它运行程序组件，以便支持将数据从一个数据源清洗到目的源进行存储。
>
>Flume Source消费从外部源传递给它的Event，外部源的发送Event格式必须与Source一致，以便Source能识别。当Source接收到Event，它将Event存储到一个或多个channels中。
>
>Flume Channel是一个被动的存储，它会保存数据直到Flulme Sink主动消费其存储的数据。例如File Channel，它存储数据到本地filesystem。
>
>Flume Sink从Channel中移除Event，并将Event的数据存储到目的源，例如HDFS。Sink也可以将数据发送到下一个Agent的source。
>
>Flume Source和Flume Sink两者是异步的，各自分别与channel中的Event进行交流。

### Source

>Source用于从指定接受Event来自那种类型，如kafka、thrift、tialdir等。



##### Exec Source

>Exec Source在启动时执行给定的命令，并期望该指令进程持续产生数据输出到standard out。默认情况下stderr关闭，可通过logStdErr=true开启。如果进程因为任何原因退出了，Flume Source也会退出并不会产生数据了。例如，tail -F [file]将持续产生期望的结果，而date命令只产生一个Event并退出。
>
>下表的粗体部分为必须属性，其他属性是可选属性。

| **Property Name** | **Default** | **Description**                                         |
| ----------------- | ----------- | ------------------------------------------------------- |
| **channels**      | -           |                                                         |
| **type**          | -           | exec                                                    |
| **command**       | -           | 需要执行的命令，如 tail -F  /log                        |
| shell             | -           | 指定运行shell，如/bin/sh -c 指定使用sh执行command的内容 |
| logStdERR         | false       | 是否输出标准错误的日志                                  |
| batchSize         | 20          | 一次发往channel的最大批次大小                           |
| batchTimeout      | 3000        | 如果没达到buffer size，隔多久强行发送一次               |
| restartThrottle   | 10000       | 重试等待时间                                            |
| restart           | false       | command失败死亡，是否重试                               |
| interceptors      | -           | 拦截器                                                  |

>关于ExecSource的注意要点，ExecSource无法提供任何保证确保数据不丢失，如果souce尝试将数据放入channel时失败，那么此条数据可能丢失。比如channel已经装满时，source还在发送，但是channel拒绝了接受。总之，ExecSource不够可信，对数据完整性要求较高时，可使用Spooling Directory Source，Taildir Source 。

```
#只包含必填属性的示例
a1.sources = r1
a1.channels = c1
a1.sources.r1.type = exec
a1.sources.r1.command = tail -F /var/log/secure
a1.sources.r1.channels = c1
```

##### Spooling Directory Source

>Spooling Directory Source使用将数据放入spooling directory的形式引入数据，Spooling Directory Source会监控指定文件夹，并将新增的文件按照解析逻辑封装为Event，解析逻辑是可插拔的。当一个文件完全写入channel后，它会将文件后缀添加Completed，或者使用追踪文件记录，或者删除完成的文件。Spooling Directory Source是完全可信赖的，他存储了发送的信息，当重启和kill时不会丢失数据。
>
>Spooling Directory Source使用要确保两点，1文件夹下的文件必须一成不变，一旦生成不会变动内容。2文件夹下文件不可重名。当然，其并非百分百可靠，这取决于下游任务是否发生一定的错误。

| Property Name  | Default     | Description                         |
| -------------- | ----------- | ----------------------------------- |
| **channels**   | -           |                                     |
| **type**       | -           | spooldir                            |
| **spoolDir**   | -           | 监视的文件夹                        |
| fileSuffix     | .COMPLETED  | 完成后添加的后缀                    |
| deletePolicy   | never       | 是否删除完成的文件，never,immediate |
| fileHeader     | false       | 在Event头部添加文件路径             |
| fileHeaderKey  | file        | 添加的key                           |
| batchSize      | -           | 同上文                              |
| trackingPolicy | -           | 追踪策略，rename，tacker_dir        |
| trackerDir     | .flumespool | 追踪文件存储地址                    |

```
a1.channels = ch-1
a1.sources = src-1

a1.sources.src-1.type = spooldir
a1.sources.src-1.channels = ch-1
a1.sources.src-1.spoolDir = /var/log/apache/flumeSpool

a1.sources.src-1.fileHeader = true
```

##### Taildir Source

>note:此功能无法在windwos工作
>
>监控特定的文本文件，并不断读取追加的内容，以行为单位封装为Event，发送到channels。
>
>Taildir Source是可信的，当flume重启，kill时不会丢失数据。

| Property Name                 | Default | Description                                                |
| :---------------------------- | :------ | :--------------------------------------------------------- |
| **channels**                  | –       |                                                            |
| **type**                      | –       | TAILDIR                                                    |
| **filegroups**                | –       | 文件组，表示一系列需要tail的文件。                         |
| **filegroups.filegroupsName** | –       | 使用绝对路径表示需要监控的文件组，一般使用正则表示文件组。 |

```
#必须属性示例
a1.sources = r1
a1.channels = c1
a1.sources.r1.type = TAILDIR
a1.sources.r1.channels = c1
a1.sources.r1.filegroups = f1 f2
a1.sources.r1.filegroups.f1 = /var/log/test1/example.log
a1.sources.r1.headers.f1.headerKey1 = value1
a1.sources.r1.filegroups.f2 = /var/log/test2/.*log.*
```

##### Kafka Source

>Kafka Source是一个kafka consumer从对应topic读取消息。你可以配置多个kafka source，通过设置相同的groupid让他们读取不同的topic分区。

| Property Name               | Default | Description                                                  |
| :-------------------------- | :------ | :----------------------------------------------------------- |
| **channels**                | –       |                                                              |
| **type**                    | –       | org.apache.flume.source.kafka.KafkaSource                    |
| **kafka.bootstrap.servers** | –       | broker的服务地址                                             |
| kafka.consumer.group.id     | flume   | 为多个source设置相同的groupid，让他们作为同一个消费者组进行消费。 |
| **kafka.topics**            | –       | 消费的topics                                                 |
| **kafka.topics.regex**      | –       | 使用正则表示消费的topic，如果此参数存在会覆盖topics          |

```
tier1.sources.source1.type = org.apache.flume.source.kafka.KafkaSource
tier1.sources.source1.channels = channel1
tier1.sources.source1.kafka.bootstrap.servers = localhost:9092
tier1.sources.source1.kafka.topics = test1, test2
tier1.sources.source1.kafka.consumer.group.id = custom.g.id
```

```
tier1.sources.source1.type = org.apache.flume.source.kafka.KafkaSource
tier1.sources.source1.channels = channel1
tier1.sources.source1.kafka.bootstrap.servers = localhost:9092
tier1.sources.source1.kafka.topics.regex = ^topic[0-9]$
```

##### Event Deserializers

| Property Name              | Default | Description                       |
| :------------------------- | :------ | :-------------------------------- |
| deserializer.maxLineLength | 2048    | 单行最大字符数，超过的会被截断    |
| deserializer.outputCharset | UTF-8   | 发送到channel的数据采用的字符编码 |

##### NetCat TCP Source

>监听给定的端口，获取每一行的数据并发送

| Property Name | Default | Description                                   |
| :------------ | :------ | :-------------------------------------------- |
| **channels**  | –       |                                               |
| **type**      | –       | The component type name, needs to be `netcat` |
| **bind**      | –       | Host name or IP address to bind to            |
| **port**      | –       | Port # to bind to                             |

```
a1.sources = r1
a1.channels = c1
a1.sources.r1.type = netcat
a1.sources.r1.bind = 0.0.0.0
a1.sources.r1.port = 6666
a1.sources.r1.channels = c1
```

### Channel

>channels是为agent提供存储库用于存储Event，source向其添加，sink消费其中的Event。

##### Memory Channel

>使用内存队列存储接受的Event，1用于处理高吞吐量、高速度的数据流2必须能接受event失败导致的数据丢失结果。

| Property Name       | Default       | Description                                               |
| :------------------ | :------------ | :-------------------------------------------------------- |
| **type**            | –             | memory                                                    |
| capacity            | 100           | 最大event数目                                             |
| transactionCapacity | 100           | source每个事务添加的最大event数目，和给予sink的最大数目。 |
| keep-alive          | 3             | source和sink交互的超时时间                                |
| byteCapacity        | jvm -Xmx的80% | 最大字节数，默认为jvm -Xmx的80%。                         |

```
a1.channels = c1
a1.channels.c1.type = memory
a1.channels.c1.capacity = 10000
a1.channels.c1.transactionCapacity = 10000
a1.channels.c1.byteCapacity = 800000
```

##### Kafka Channel

>使用kafka集群作为存储Event的介质，必须是分布式kafka才能作为channel。

| Property Name               | Default       | Description                                   |
| :-------------------------- | :------------ | :-------------------------------------------- |
| **type**                    | –             | `org.apache.flume.channel.kafka.KafkaChannel` |
| **kafka.bootstrap.servers** | –             | kafka集群的broker                             |
| kafka.topic                 | flume-channel | 存储在哪个topic                               |

```
a1.channels.channel1.type = org.apache.flume.channel.kafka.KafkaChannel
a1.channels.channel1.kafka.bootstrap.servers = kafka-1:9092,kafka-2:9092,kafka-3:9092
a1.channels.channel1.kafka.topic = channel1
a1.channels.channel1.kafka.consumer.group.id = flume-consumer
```

##### File Channel

| Property Name Default | Description                      |            |
| :-------------------- | :------------------------------- | :--------- |
| **type**              | –                                | file       |
| checkpointDir         | ~/.flume/file-channel/checkpoint | 检查点目录 |

```
a1.channels = c1
a1.channels.c1.type = file
a1.channels.c1.checkpointDir = /mnt/flume/checkpoint
a1.channels.c1.dataDirs = /mnt/flume/data
```

### Sink

##### HDFS Sink

>将event数据写入HDFS，目前支持创建sequence files和text，支持压缩。支持对hdfs存储目录进行分区，可以如下表格的转义字符生成分区目录

| Alias        | Description                               |
| :----------- | :---------------------------------------- |
| %{host}      | headers中的key为host的value               |
| %t           | 毫秒时间milliseconds                      |
| %a           | 周几缩写 (Mon, Tue, ...)                  |
| %A           | 周几完整名称 (Monday, Tuesday, ...)       |
| %b           | 月份缩写 (Jan, Feb, ...)                  |
| %B           | 月份完整 (January, February, ...)         |
| %c           | datetime (Thu Mar 3 23:05:25 2005)        |
| %d           | 该月几号(01)                              |
| %e           | 月份，无填充 (1)                          |
| %D           | 日期：%m/%d/%y                            |
| %H           | 小时(00..23)                              |
| %I           | 小时(01..12)                              |
| %j           | 一年的第几天 (001..366)                   |
| %k           | 小时，无填充( 0..23)                      |
| %m           | 月份(01..12)                              |
| %n           | 月份 (1..12)                              |
| %M           | 分钟(00..59)                              |
| %p           | 上午下午 am or pm                         |
| %s           | 自1970-01-01 00:00:00 UTC以来秒数         |
| %S           | 秒，(00..59)                              |
| %y           | 年的后两位(00..99)                        |
| %Y           | 年(2010)                                  |
| %z           | 时区 (for example, -0400)                 |
| %[localhost] | hostname of the host 主机名               |
| %[IP]        | IP address of the host 地址IP             |
| %[FQDN]      | canonical hostname of the host 规范主机名 |

>必须属性

| Name              | Default | Description                              |
| :---------------- | :------ | :--------------------------------------- |
| **channel**       | –       |                                          |
| **type**          | –       | `hdfs`                                   |
| **hdfs.path**     | –       | HDFS路径，可以使用前文的转义字符拼接路径 |
| hdfs.rollInterval | 30      | 触发写入时间                             |
| hdfs.rollSize     | 1024    | 触发写入大小，字节为单位                 |
| hdfs.rollCount    | 10      | 触发写入event数目                        |

```
a1.channels = c1
a1.sinks = k1
a1.sinks.k1.type = hdfs
a1.sinks.k1.channel = c1
a1.sinks.k1.hdfs.path = /flume/events/%y-%m-%d/%H%M/%S
a1.sinks.k1.hdfs.filePrefix = events-
a1.sinks.k1.hdfs.round = true
a1.sinks.k1.hdfs.roundValue = 10
a1.sinks.k1.hdfs.roundUnit = minute
```

##### File Roll Sink

| Property Name      | Default | Description  |
| :----------------- | :------ | :----------- |
| **channel**        | –       |              |
| **type**           | –       | `file_roll`. |
| **sink.directory** | –       | sink目录     |

```
a1.channels = c1
a1.sinks = k1
a1.sinks.k1.type = file_roll
a1.sinks.k1.channel = c1
a1.sinks.k1.sink.directory = /var/log/flume
```

### 相关链接

>https://flume.apache.org/releases/content/1.9.0/FlumeUserGuide.html#flume-sources

