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

>Source用于从指定

### Channel

### Sink

