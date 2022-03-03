### 相关组件版本

```
hadoop3.1.1
hbase-2.3.3
hive-3.1.2
zookeeper3.5.9
mysql-5.7.28
kafka_2.12-2.1.1
flume 1.9.0
```



### 端口

```
namenode rpc  8020
webui 9870
```

# CONFIG

```
mapred.job.maps：一个job可以分配到map数量。这里的map数量和InputSplit的map数量不同，InputSplit控制map任务的数量，控制输入数据的读取来控制map任务数量。而此处的maps是指有多少个核心可用于map任务槽，用来同时执行map任务。
```

# hadoop服务器搭建

>软件及系统版本：hadoop3.1.4,centos7.2，idea2019.3.3，maven为idea的bundle版本。
>
>参考官方文档3.1.4
>
>参考的视频https://www.bilibili.com/video/BV1cW411r7c5
>
>过程中碰到的问题总结在最后

### 初步安装

1进入http://hadoop.apache.org/下载hadoop3.1.4.tar.gz并解压缩。

2服务器终端输入vim  /etc/ profile 。在profile中添加如下内容,添加java和hadoop的环境变量。注意替换自己的JAVA_HOME和HADOOP_HOME目录：

```
#JAVA_HOME
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.181-7.b13.el7.x86_64
export PATH=$PATH:$JAVA_HOME/bin

#HADOOP_HOME
export HADOOP_HOME=/root/module/hadoop-3.1.4
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin

```

终端运行source /etc/profile，载入修改的配置。

3给hadoop配置java目录

```
vim  etc/hadoop/hadoop-env.sh
添加如下：
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.181-7.b13.el7.x86_64
```

source   etc/hadoop/hadoop-env.sh

4终端运行hadoop，显示为帮助信息，即配置成功。

帮助信息以如下文字结尾：

```
SUBCOMMAND may print help when invoked w/o parameters or with -h.
```

### 单机模式运行

官方文档：https://hadoop.apache.org/docs/r3.1.4/hadoop-project-dist/hadoop-common/SingleCluster.html

进入hadoop安装的根目录，执行如下命令。在output目录下生成结果文件，即为成功。注意output由hadoop自动创建，不要手动。

```
mkdir input
cp etc/hadoop/*.xml input
bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.4.jar grep input output 'dfs[a-z.]+'
cat output/*
```

如上命令的功能：将etc/hadoop下的以xml为后缀的文件，复制到input文件夹下面。

然后用hadoop-mapreduce-examples-3.1.4.jar中的java程序统计input下的所有文件，再对结果进行'dfs[a-z.]+'正则匹配，将结果输出到output文件夹。

### 伪分布式

1vim    etc/hadoop/core-site.xml，修改为如下内容，指定默认文件系统路径。:

```
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
```

2vim etc/hadoop/hdfs-site.xml，修改为如下内容:

```
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```

3执行如下命令，使得ssh localhost无需密码。

```
cd 用户根目录
cd .ssh
ssh-keygen -t rsa
ssh-copy-id localhost
输入密码即可
```

执行ssh localhost无需密码，即为成功。

4进入hadoop根目录，执行如下，格式化namenode，启动hdfs：

```
 $ bin/hdfs namenode -format
 $ sbin/start-dfs.sh
```

服务器本机访问 `http://localhost:9870/`即可查看hdfs系统

关闭防火墙后，外部也可访问。

5执行mapreduce作业

创建hdfs文件目录：

```
bin/hdfs dfs -mkdir /user
bin/hdfs dfs -mkdir /user/<username>
```

创建input文件夹，复制一些文件到input中

```
 bin/hdfs dfs -mkdir input
 bin/hdfs dfs -put etc/hadoop/*.xml input
```

执行mapreduce

```
bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.4.jar grep input output 'dfs[a-z.]+'
```

获取hdfs中结果文件存储到本地

```
bin/hdfs dfs -get output output
cat output/*
```

注意，必须使用hdfs命令创建，删除，修改文件目录。hdfs的目录和本地是相互独立的。

6关闭hdfs

```
 sbin/stop-dfs.sh
```

### 完全分布式搭建

>编写好scp分发脚本,快速同步etc配置文件到其他集群机器.
>
>编写好删除日志，data，tmp文件的脚本。当需要初始化时需要删除这些文件，然后格式化namenode再启动
>
>编写脚本参见bili视频
>
>以三台机器为例，同bili视频

我的三台机器对应如下

| ip主机号         | 190               | 189                         | 188                        |
| ---------------- | ----------------- | --------------------------- | -------------------------- |
| 我的hostname     | hbase             | hbase1                      | hbase2                     |
| bili的hostname : | hadoop102         | hadoop103                   | hadoop104                  |
| hdfs分工：       | namenode,datanode | datanode                    | secondarynamenode,datanode |
| hadoop分工:      | nodemanager       | resourcemanager,nodemanager | nodemanager                |

##### 配置环境

>主要配置这些firewall,hosts,hostname,ssh.
>
>java_home,hadooop_home,hadoop-env.sh

进入190主机，即hbase。

用单机模式的方法配置好，java环境变量，hadoop环境变量，hadoop的java_home。

关闭firewall:

```
systemctl stop firewalld.service
```

手动分配服务器ip,可省略。

hosts修改ip和域名映射:

```
vim /etc/hosts
追加如下内容：
0.0.0.0 hbase  #此处是个坑，不要配置为localhost，按此配置。
111.111.111.189 hbase1
111.111.111.188 hbase2

```

hostname改为对应域名:

```
vim /etc/sysconfig/network
修改为ip对应的hostname，我的190对应为
hbase
#也可使用此命令
hostname -b hbase2
```

ssh配置使用公私钥不用密码：

```
cd 用户根目录
cd .ssh
ssh-keygen -t rsa
ssh-copy-id hbase
输入密码即可
ssh-copy-id hbase1
输入密码即可
ssh-copy-id hbase2
输入密码即可
```

此时hbase访问hbase1和hbase2都不需要密码。这样，之后才能控制hbase1和hbase2，启动运行在他们之上的hdfs结点。

配置ssh公钥的例子：

希望机器a可以无密码访问机器b。进行如下操作，进入a机器用户根目录下的.ssh文件夹，执行ssh-keygen -t  rsa。随后执行ssh-copy-id   b的ip，然后输入账号密码即可。可以进入b机器的.ssh的authorized_hosts查看，发现已经添加成功。希望a访问某台机器，使用同上方法拷贝到对应机器即可。

##### 配置etc下的文件

>配置env和xml，workers等。

1hadoop-env.sh引入java_home:

```
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.181-7.b13.el7.x86_64
```

2core-site.xml:

```
<configuration>
        <property>
                <name>fs.defaultFS</name>
                <value>hdfs://hbase:9000</value>
        </property>
        <property>
                <name>hadoop.tmp.dir</name>
                <value>/root/hadoop/dfs/tmp</value>
        </property>
</configuration>
```

为了在网页界面可以操作文件夹，需要配置代理用户及staticuser。完整的配置文件如下。

```
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
	<!-- 指定NameNode的地址 -->
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://hadoop102:8020</value>
</property>
<!-- 指定hadoop数据的存储目录 -->
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/opt/module/hadoop-3.1.3/data</value>
</property>

<!-- 配置HDFS网页登录使用的静态用户为atguigu -->
    <property>
        <name>hadoop.http.staticuser.user</name>
        <value>atguigu</value>
</property>

<!-- 配置该atguigu(superUser)允许通过代理访问的主机节点 -->
    <property>
        <name>hadoop.proxyuser.atguigu.hosts</name>
        <value>*</value>
</property>
<!-- 配置该atguigu(superUser)允许通过代理用户所属组 -->
    <property>
        <name>hadoop.proxyuser.atguigu.groups</name>
        <value>*</value>
</property>
<!-- 配置该atguigu(superUser)允许通过代理的用户-->
    <property>
        <name>hadoop.proxyuser.atguigu.users</name>
        <value>*</value>
</property>
</configuration>

```



3hdfs-site.xml:

```
<configuration>
        <property>
                <name>dfs.replicatin</name>
                <value>3</value>
        </property>
        <property>
                <name>dfs.namenode.secondary.http-address</name>
                <value>hbase2:50090</value>
        </property>
        <property>
                <name>dfs.namenode.name.dir</name>
                <value>/root/hadoop/dfs/name</value>
        </property>
        <property>
                <name>dfs.datanode.data.dir</name>
                <value>/root/hadoop/dfs/data</value>
        </property>
</configuration>

```

4etc/workers,3.x由slaves更名为workers。

```
vim workers
改为：
hbase
hbase1
hbase2
```

##### 转发hadoop到其他机器

1转发hadoop-3.1.4到hbase1，hbase2（也可以直接克隆整个虚拟机，但注意要修改ip，hostname，hosts文件）

```
scp hadoop-3.1.4  root@hbase1:`pwd`
scp hadoop-3.1.4  root@hbase2:`pwd`
```

如果修改配置文件，需要再次同步：

```
cd    hadoop-3.1.4/etc/hadoop
scp core-site.xml root@hbase1:`pwd`
scp core-site.xml root@hbase2:`pwd`
```

2配置hbase1，hbase2

对于hbase1和hbase2我们只需修改/etc/hosts,/etc/sysconfg/network,其他部分保持和hbase一致。

以hbase1为例子，主要改一下hostname和ip与域名对应关系。

hbase1的hosts修改ip和域名映射:

```
vim /etc/hosts
追加如下内容：
111.111.111.190  hbase  #此 处是个坑，不要配置为localhost，按此配置。
0.0.0.0        hbase1
111.111.111.188  hbase2

```

hbase1的hostname改为对应域名:

```
vim /etc/sysconfig/network
修改为ip对应的hostname，我的189对应为
hbase1
```

hbase2原理同上

##### 格式化namenode并启动

注意：若之前启动过hdfs，格式化时必须停止所有模块，并删除data，logs，tmp文件夹。

进入namenode所在的服务器，即hbase，执行如下命令。

```
hdfs namenode -format
sbin/start-dfs.sh
```

会看到他按照配置文件，在三台不同机器启动了不同node。

访问hbase的9870端口，可查看hdfs存储情况。

##### 启动yarn

进入resourcemanager所在的服务器，执行sbin/start-yarn.sh

访问resourcemanager所在机器的ip的8088端口,查看mapreduce执行情况。

##### 测试

```
上传文件，可在9870查看
hdfs dfs -put filename
执行mr，可在8088查看
bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.4.jar grep input output 'dfs[a-z.]+'
```

##### 开启historyjob server

yarn-site.xml：

```
<!--配置历史服务器-->
<property>  
<name>mapreduce.jobhistory.address</name>  
 <value>hbase:10020</value>  
</property>  
<property>  
<name>mapreduce.jobhistory.webapp.address</name>  
<value>hbase:19888</value>  
</property>
```

```
cd  $HADOOP_HOME/sbin
mr-jobhistory-daemon.sh start historyserver
```



### hdfs

```
Configuration configuration = new Configuration();
configuration.set("fs.hdfs.impl", org.apache.hadoop.hdfs.DistributedFileSystem.class.getName());
configuration.set("fs.file.impl", org.apache.hadoop.fs.LocalFileSystem.class.getName());
FileSystem fs = FileSystem.get(URI.create(hdfsPath), configuration);
FSDataOutputStream out = fs.create(new Path(hdfsPath));//创建一个输出流
InputStream in = new FileInputStream(new File(localPath));//从本地读取文件
IOUtils.copyBytes(in, out, 100, true);
System.out.println("上传完毕");
```



# MapReduce开发环境搭建

1官网下载hadoop3.1.4.tar.gz并解压缩

下载hadoop.dll及winexe，下载链接：

https://github.com/ordinaryload/Hadoop-tools

将hadoop.dll,winexe复制到hadoop-3.1.4/bin

2编写程序，按照bili视频编写map,reduce,driver类。

也可按照此博客，使用源码示例，博客地址：

https://www.cnblogs.com/xingluo/p/9512961.html

3添加NativeIO，由于win10摒弃此函数，需自己创建。

参考此博客处理：https://blog.csdn.net/weixin_42229056/article/details/82686172

4直接运行driver的main函数，直接将输入输出目录写死在程序中，不使用控制台输入参数。

##### 源码

driver:

```
package com.weitao.mr.wordcount;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.IOException;

public class WordCountDriver {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        System.setProperty("hadoop.home.dir", "C:\\Users\\asus\\Desktop\\hadoop-3.1.4\\hadoop-3.1.4");
        //获取job对象
        Configuration conf=new Configuration();
        Job job=Job.getInstance(conf);
        //设置jar位置
        job.setJarByClass(WordCountDriver.class);
        //关联map和reduce
        job.setMapperClass(WordCountMapper.class);
        job.setReducerClass(WordCountReducer.class);

        //设置mapper阶段输出数据key和value类型
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(IntWritable.class);
        //设置最终数据输出的key和value类型
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        //设置输入路径和输出路径
        FileInputFormat.setInputPaths(job, new Path("C:\\Users\\asus\\Desktop\\words.txt"));
        FileOutputFormat.setOutputPath(job, new Path("C:\\Users\\asus\\Desktop\\output"));
        //提交job
        boolean result=job.waitForCompletion(true);
        System.exit(result?0:1);
    }
}

```

reducer:

```
package com.weitao.mr.wordcount;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

public class WordCountReducer extends Reducer<Text, IntWritable,Text,IntWritable> {
    IntWritable v=new IntWritable();
    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
        //合并相同key
        int sum=0;
        for(IntWritable value:values)
        {
            sum+=value.get();
        }

        v.set(sum);
        context.write(key,v);
    }
}
```

mapper

```
package com.weitao.mr.wordcount;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

public class WordCountMapper extends Mapper<LongWritable, Text,Text, IntWritable> {
    Text k=new Text();
    IntWritable v=new IntWritable(1);
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        //1转为string并切分
        String line=value.toString();
        String[] words=line.split(" ");
        //循环写出
        for(String word:words)
        {
            k.set(word);
            context.write(k,v);
        }
    }
}
```

pom.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.weitao.mr</groupId>
    <artifactId>mapreduce_start</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
            <version>2.8.2</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-common</artifactId>
            <version>3.1.4</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-client</artifactId>
            <version>3.1.4</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.apache.hadoop/hadoop-hdfs -->
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-hdfs</artifactId>
            <version>3.1.4</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-mapreduce-client-core</artifactId>
            <version>3.1.4</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-mapreduce-client-jobclient</artifactId>
            <version>3.1.4</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-mapreduce-client-common</artifactId>
            <version>3.1.4</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hive</groupId>
            <artifactId>hive-jdbc</artifactId>
            <version>3.1.2</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hive</groupId>
            <artifactId>hive-exec</artifactId>
            <version>3.1.2</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hive</groupId>
            <artifactId>hive-metastore</artifactId>
            <version>3.1.2</version>
        </dependency>
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-clients</artifactId>
            <version>2.4.1</version>
        </dependency>
    </dependencies>

    <repositories>
        <repository>
            <id>aliyun</id> <!-- id可以随便取，只要不重名即可 -->
            <url>https://mvnrepository.com/</url>
        </repository>
    </repositories>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>8</source>
                    <target>8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>3.2.0</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <transformers>
                                <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <mainClass>com.ti.mr.getSingleInfo.utils.ConsumeHdfs</mainClass>
                                </transformer>
                            </transformers>
                            <filters>
                                <filter>
                                    <artifact>*:*</artifact>
                                    <excludes>
                                        <exclude>META-INF/*.SF</exclude>
                                        <exclude>META-INF/*.DSA</exclude>
                                        <exclude>META-INF/*.RSA</exclude>
                                    </excludes>
                                </filter>
                            </filters>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

</project>
```



# MapReduce更多功能

### 命令

```
yarn application -list
for i in  `yarn application  -list | grep -w  ACCEPTED | awk '{print $1}' | grep application_`; do yarn  application -kill $i; done
```



### NLineInputFormat

driver:

```
#5行为一个分片，共有  总行数/5个split分片。
NLineInputFormat.setNumLinesPerSplit(job,5);
job.setInputFormatClass(NLineInputFormat.class);
```

### 自定义InputFormat

>一次读取一个文件，一个文件作为一个分片
>
>控制map，让map一次读取整个文件，即多行。
>
>NullWritable用于填充空白，如不想要key，key就写NullWritable类型。对于部分无需reduce的程序，也可以不写reducer。但仍要设置OutputKeyClass,OutputValueClass，其值和MapOutputKeyClass,MapOutputValueClass一致。
>
>要想控制单次map的输入就要重写NextKeyValue方法，修改Context中的key和value，这样在调用Context.NextKeyValue()时，才能控制是否执行map，并控制map读取的key和value。getCurrentKey()和getCurrentValue()返回，在重写的NextKeyValue中修改后的值。
>
>```
>public void run(Mapper<KEYIN, VALUEIN, KEYOUT, VALUEOUT>.Context context) throws IOException, InterruptedException {
>    this.setup(context);
>
>    try {
>        while(context.nextKeyValue()) {
>            this.map(context.getCurrentKey(), context.getCurrentValue(), context);
>        }
>    } finally {
>        this.cleanup(context);
>    }
>
>}
>```

WholeFileInputFormat

```jAVA
package com.weitao.mr.wordcount;

import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.BytesWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.InputSplit;
import org.apache.hadoop.mapreduce.JobContext;
import org.apache.hadoop.mapreduce.RecordReader;
import org.apache.hadoop.mapreduce.TaskAttemptContext;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;

import java.io.IOException;

public class WholeFileInputFormat extends FileInputFormat<Text, Text> {
    @Override
    protected boolean isSplitable(JobContext context, Path filename) {
        return false;
    }

    @Override
    public RecordReader<Text,Text> createRecordReader(InputSplit inputSplit, TaskAttemptContext taskAttemptContext) throws IOException, InterruptedException {
        WholeRecordReader recordReader=new WholeRecordReader();
        recordReader.initialize(inputSplit,taskAttemptContext);
        return recordReader;
    }
}

```

WholeRecordReader

```java
package com.weitao.mr.wordcount;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FSDataInputStream;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.BytesWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.InputSplit;
import org.apache.hadoop.mapreduce.RecordReader;
import org.apache.hadoop.mapreduce.TaskAttemptContext;
import org.apache.hadoop.mapreduce.lib.input.FileSplit;
import sun.misc.IOUtils;

import java.io.BufferedReader;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStreamReader;

public class WholeRecordReader extends RecordReader<Text, Text> {
    FileSplit split;
    Configuration configuration;
    Text k;
    Text v;
    boolean isProgress;
    @Override
    public void initialize(InputSplit inputSplit, TaskAttemptContext taskAttemptContext) throws IOException, InterruptedException {
        this.split=(FileSplit)inputSplit;
        System.out.println("name:"+this.split.getPath().getName());
        this.configuration=taskAttemptContext.getConfiguration();
        k=new Text();
        v=new Text();
        isProgress=true;
    }

    @Override
    public boolean nextKeyValue() throws IOException, InterruptedException {
        if(isProgress)
        {
        	/*根据path读取文件流，一次性将所有内容读出。并设置到context的K和v中。控制返回值，使得一个文件只读取一次*/
            Path path=split.getPath();
            String path_str=path.toString();
            path_str = path_str.substring(6);
            BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(new FileInputStream(path_str)));
            StringBuffer json=new StringBuffer();
            String tem="";
            while(true){
                tem = bufferedReader.readLine();
                if (tem != null) {
                    json.append(tem);
                } else break;
            }
            v.set(json.toString());
            k.set(path.toString());
            isProgress=false;
            return true;
        }
        return false;
    }

    @Override
    public Text getCurrentKey() throws IOException, InterruptedException {

        return k;
    }

    @Override
    public Text getCurrentValue() throws IOException, InterruptedException {
        return v;
    }

    @Override
    public float getProgress() throws IOException, InterruptedException {
        return 0;
    }

    @Override
    public void close() throws IOException {

    }
}
```

driver

```
job.setInputFormatClass(WholeFileInputFormat.class);
```

### 自定义分区

>控制结果按照条件输出到不同文件

driver

```
#reduce的个数和partitioner的逻辑对应。
job.setPartionerClass(SelfPartitioner);
job.setNumReduceTask(2);
```

PartionClass

```
package com.weitao.mr.wordcount;

import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Partitioner;

public class SelfPartitioner extends Partitioner<Text, Text> {

    @Override
    public int getPartition(Text text, Text text2, int i) {
        int partition;
        if(true){
            partition=0;
        }else
        {
            partition=1;
        }
        return partition;
    }
}
```



# 基本操作

### hdfs

```

hdfs dfs -put filename  hdsf_path
hdfs dfs -mkdir directoryname
hdfs dfs -rm -r directoryname
hdfs dfs -ls 
hadoop fs -ls
```



### mapreduce

合并多个文件

```
CombineTextInputFormat.setMaxInputSplitSize(job,20971520);
job.setInputFormatClass(CombineTextInputFormat.class);
```

读取多行，使用自定义或NLine，不能两全。

希望多个文件公用mapper，一个文件一次map取完（NLine可能？）。

key为从文件头处的偏移量

文件数目，文件读写数目，mapper数目，分片尺寸，map数目

默认格式：文件数=文件读写数目=分片数目=mapper数目  map数目=行数

WhomeFileInputFormat：文件数=文件读写数目=分片数目=mapper数目   map数目等于=控制行数

setMaxInputSplitSize:文件数=读写数  mapper数目= 分片数=根据分片尺寸，按照规则所得    map数目=行数

设置最大分片尺寸，控制mapper数量。

使用默认inputformat,mapper执行多少次，map执行次数等于行数

setup，cleanup每个mapper只执行一次。

一个maptask对应一个分片。通过LineInputFormat，NLineInputFormat控制分片数量。

通过重写InputFormat类，实现自己的类。重写RecordReader中的NextKeyValue控制map一次读取的行数。



# 其他

### 大数据之源

Google的GFS、MapReduce、BigTable

### 常用配置文件

##### 文件夹

etc/hadoop:环境配置sh文件，各组件配置xml文件。

sbin：启动，停止各组件控制sh文件。

bin：执行mapreduce任务的sh文件。

share：基础配置模板等。

##### core-site.xml

配置hdfs的ip和端口

hdfs的存储文件夹

##### hdfs-site.xml

配置hdfs的详细信息，例如hdfs的replication，namenode位置和datanode的位置。

##### hadoop-env.sh,mapred-env.sh,yarn-env.sh

环境配置，至少配置JAVA_HOME绝对路径。

##### mapred-site.xml

指定管理框架，如yarn

##### yarn-site.xml

配置resourceManagers

### 常用端口

9870  用户查看hdfs节点信息，hdfs的http端口

  8088  用户查看YARN的http端口，显示任务执行信息，查找错误等。

| 端口  | 用途                                                         |
| ----- | ------------------------------------------------------------ |
| 9000  | fs.defaultFS，如：hdfs://172.25.40.171:9000                  |
| 9001  | dfs.namenode.rpc-address，DataNode会连接这个端口             |
| 50070 | dfs.namenode.http-address                                    |
| 50470 | dfs.namenode.https-address                                   |
| 50100 | dfs.namenode.backup.address                                  |
| 50105 | dfs.namenode.backup.http-address                             |
| 50090 | dfs.namenode.secondary.http-address，如：172.25.39.166:50090 |
| 50091 | dfs.namenode.secondary.https-address，如：172.25.39.166:50091 |
| 50020 | dfs.datanode.ipc.address                                     |
| 50075 | dfs.datanode.http.address                                    |
| 50475 | dfs.datanode.https.address                                   |
| 50010 | dfs.datanode.address，DataNode的数据传输端口                 |
| 8480  | dfs.journalnode.rpc-address                                  |
| 8481  | dfs.journalnode.https-address                                |
| 8032  | yarn.resourcemanager.address                                 |
| 8090  | yarn.resourcemanager.webapp.https.address                    |
| 8030  | yarn.resourcemanager.scheduler.address                       |
| 8031  | yarn.resourcemanager.resource-tracker.address                |
| 8033  | yarn.resourcemanager.admin.address                           |
| 8042  | yarn.nodemanager.webapp.address                              |
| 8040  | yarn.nodemanager.localizer.address                           |
| 8188  | yarn.timeline-service.webapp.address                         |
| 10020 | mapreduce.jobhistory.address                                 |
| 19888 | mapreduce.jobhistory.webapp.address                          |
| 2888  | ZooKeeper，如果是Leader，用来监听Follower的连接              |
| 3888  | ZooKeeper，用于Leader选举                                    |
| 2181  | ZooKeeper，用来监听客户端的连接                              |
| 60010 | hbase.master.info.port，HMaster的http端口                    |
| 60000 | hbase.master.port，HMaster的RPC端口                          |
| 60030 | hbase.regionserver.info.port，HRegionServer的http端口        |
| 60020 | hbase.regionserver.port，HRegionServer的RPC端口              |
| 8080  | hbase.rest.port，HBase REST server的端口                     |
| 10000 | hive.server2.thrift.port                                     |
| 9083  | hive.metastore.uris                                          |

### 基本分工

1，平台。搭建平台，集群调优

2，数据仓库。sqlboy

3，数据挖掘。算法

4，开发。java开发



# problem

##### source core-site.xml第一行未预期的错误,newline。

不必理会

##### sbin/strat-dfs.sh报错

解决：对于start-dfs.sh和stop-dfs.sh文件，添加下列参数：

```
#!/usr/bin/env bash
HDFS_DATANODE_USER=root
HADOOP_SECURE_DN_USER=hdfs
HDFS_NAMENODE_USER=root
HDFS_SECONDARYNAMENODE_USER=root
```

对于start-yarn.sh和stop-yarn.sh文件，添加下列参数：

```
#!/usr/bin/env bash
YARN_RESOURCEMANAGER_USER=root
HADOOP_SECURE_DN_USER=yarn
YARN_NODEMANAGER_USER=root
```

##### 9000端口未启动

端口也没有被占用就是没有启动服务。

解决:在hosts中添加如下设置:0.0.0.0 hbase

相应的core-site.xml应设置如下:fs.defaultFS的value为hdfs://hbase:9000。随后，使用sbin/stop-all.sh停止服务，使用sbin/start-all.sh开启即可。

相关，查看hadoop官方configuration，其默认的设置ip的确为0。但使用其他教程修改了

##### outputfile already exists

在本地用rm -rf这种方式删除output文件夹是错误的，在hdfs系统中仍能查看到output文件存在。应使用hadoop指令删除相关文件。如下 bin/hdfs -rm -r -f  /user/root/output.

##### 重启dfs

停止进程，删除dfs的data。删除hadoop的logs和tmp。再重启

##### 无法访问9870

1关闭selinux和防火墙

2使用http访问，不支持https

##### no such file

 hadoop fs -mkdir -p /user/root/

##### Exception in thread "main"java.lang.UnsatisfiedLinkError:org.apache.hadoop.io.nativeio.NativeIO$Windows.access0(Ljava/lang/String;I)Z

解决：参考此博客处理：https://blog.csdn.net/weixin_42229056/article/details/82686172

也可能：缺少hadoop.dll和winexe组件，下载winexe和hadoop.dll放到hadoop3.1.4的bin目录下。下载链接：https://github.com/ordinaryload/Hadoop-tools

##### Retrying connect to server: 0.0.0.0/0.0.0.0:8032. Already tried 0 time(s); retry policy is...

在yarn-master执行mr无错误，其他机器有错误

修改yanr-site.xml:

```
<property>
    <name>yarn.resourcemanager.address</name>
    <value>master:8032</value>
  </property>
  <property>
    <name>yarn.resourcemanager.scheduler.address</name>
    <value>master:8030</value>
  </property>
  <property>
    <name>yarn.resourcemanager.resource-tracker.address</name>
    <value>master:8031</value>
  </property>
```

##### 启动后无法访问web页面

查看日志发现是由于只修改了一台的core-site.xml，没有同步。

##### 修改hadoop默认ssh端口

hadoop-env.sh

```
export HADOOP_SSH_OPTS="-p 52542"
```

##### problem

     chmod g-w /home/your_user # 或　chmod 0755 /home/your_user
     chmod 700 /home/your_user/.ssh
     
     chmod 600 /home/your_user/.ssh/authorized_keys

 然后重启ssh服务，就可以免密码登陆了。原因在于ssh服务会检查文件权限码，如777这种，会被认为不安全。

##### 克隆后删除旧网卡

/etc/sysconfig/network-scripts/ifcfg-exxx，

ip addr查看网卡

单节点配置文件

#####  is running 626469376B beyond the 'VIRTUAL' memory limit. Current usage: 208.7 MB of 1 GB physical memory used; 2.7 GB of 2.1 GB virtual memory used. Killing container.

mapreduce

```
For our example cluster, we have the minimum RAM for a Container (yarn.scheduler.minimum-allocation-mb) = 2 GB. We’ll thus assign 4 GB for Map task Containers, and 8 GB for Reduce tasks Containers.

In mapred-site.xml:

mapreduce.map.memory.mb: 4096

mapreduce.reduce.memory.mb: 8192

Each Container will run JVMs for the Map and Reduce tasks. The JVM heap size should be set to lower than the Map and Reduce memory defined above, so that they are within the bounds of the Container memory allocated by YARN.

In mapred-site.xml:

mapreduce.map.java.opts: -Xmx3072m

mapreduce.reduce.java.opts: -Xmx6144m

The above settings configure the upper limit of the physical RAM that Map and Reduce tasks will use.

```

##### stop-dfs.sh失效

```
需要通过pid来停止程序，但是pid存储在temp文件夹中，linux定期将其清理了，所以无法正常停止，可修改pid存储位置。修改hadoop-env.sh
export HADOOP_PID_DIR=/usr/local/hadoop/pids/
```

##### safemode

```
bin/hadoop dfsadmin -safemode leave
```



spark解决

```
yarn.nodemanager.resource.memory-mb 
```

