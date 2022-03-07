### 安装

>安装Kylin前需先部署好Hadoop、Hive、Zookeeper、HBase

```
tar -zxvf apache-kylin-3.0.2-bin.tar.gz -C /opt/module/
mv /opt/module/apache-kylin-3.0.2-bin /opt/module/kylin
#解决依赖冲突
vim /opt/module/kylin/bin/find-spark-dependency.sh
在 spark_dependency后追加! -name '*jackson*' ! -name '*metastore*'
```

### 启动

```
bin/kylin.sh start
进入http://hbase:7070/kylin访问
```

### 使用

##### 创建project

##### 创建datasource

##### 创建model

```
```

##### 创建cube

##### 执行cube

### 理论

```
Kylin不能处理Hive表中的复杂数据类型（Array,Map,Struct）,即便复杂类型的字段并未参与到计算之中。故在加载Hive数据源时，不能直接加载带有复杂数据类型字段的表。需要创建table view过滤掉复杂字段。
kylin将创建好的model和cube进行执行，让后将任务定时执行并放入hbase用于查询。它本质上是在做数据的聚合工作，并将结果存入可以快速访问的hbase中。kylin处理好数据后，可用于可视化bi等工具使用。presto主要提供sql查询，没有完整的存储结果方案。
```

![kylin](..\resources\images\kylin.png)

### problem

##### Error: org.apache.hadoop.hbase.util.FSUtils.setStoragePolic

##### failed to find metadata store by url:kylin_metadata@hbase

```
这两个错误都是因为修改了bin/hbase中的CLASSPATH
```

##### 执行到75%的时候，抛出错误了

```
org.apache.kylin.engine.mr.exception.MapReduceException: no counters for job job_1646418870937_0674Job Diagnostics:Task failed task_1646418870937_0674_r_000000
Job failed as tasks failed. failedMaps:0 failedReduces:1 killedMaps:0 killedReduces: 0
Failure task Diagnostics:
Error: org.apache.hadoop.hbase.util.FSUtils.setStoragePolicy(Lorg/apache/hadoop/fs/FileSystem;Lorg/apache/hadoop/fs/Path;Ljava/lang/String;)V

	at org.apache.kylin.engine.mr.common.MapReduceExecutable.doWork(MapReduceExecutable.java:223)
	at org.apache.kylin.job.execution.AbstractExecutable.execute(AbstractExecutable.java:178)
	at org.apache.kylin.job.execution.DefaultChainedExecutable.doWork(DefaultChainedExecutable.java:71)
	at org.apache.kylin.job.execution.AbstractExecutable.execute(AbstractExecutable.java:178)
	at org.apache.kylin.job.impl.threadpool.DefaultScheduler$JobRunner.run(DefaultScheduler.java:114)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
```

```
详细博客：https://blog.csdn.net/qq_43688472/article/details/111501792，通过改写sh脚本实现。
Kylin兼容性问题解决
问题：由于Kylin的安装需要很多依赖，和你架构系统中安装的各种依赖的版本不一致，会导致兼容性问题

1.kylin启动时会从hbase classpath命令的输出中寻找hbase-common-.jar。但是自hbase2.1之后，hbase classpath的输出不在包含hbase-common-.jar，取而代之的是hbase-shaded-client*.jar，故需要做以下修改。

1）修改/opt/module/kylin/bin/find-hbase-dependency.sh
[atguigu@hadoop102 sorfware]$ vim /opt/module/kylin/bin/find-hbase-dependency.sh
1
2
修改内容如下：

35 arr=(`echo $hbase_classpath | cut -d ":" -f 1- | sed 's/:/ /g'`)
 36 hbase_common_path=
 37 for data in ${arr[@]}
 38 do
 39     result=`echo $data | grep -E 'hbase-(common|shaded\-client)[a-z0-9A-Z\.-]*jar' | grep -v tests`
 40     if [ $result ]
 41     then
 42         hbase_common_path=$data
 43     fi
 44 done
1
2
3
4
5
6
7
8
9
10
2.Kylin启动之后的classpath会包含hbase lib目录下的所有jar包，由于之前安装phoenix时，向hbase的lib目录中加入了phoenix的jar包，导致kylin与其发生冲突，故需要做以下修改，将phoenix的jar包排除在kylin的classpath之外。

1）复制/opt/module/hbase/bin/hbase脚本，命名为hbase_kylin
cp /opt/module/hbase/bin/hbase /opt/module/hbase/bin/hbase_kylin
1
2
2）修改/opt/module/hbase/bin/hbase_kylin，内容如下

270 else
271   for f in $HBASE_HOME/lib/*.jar; do
272     result=`echo $f | grep -v phoenix`
273     if [ $result ];then
274       CLASSPATH=${CLASSPATH}:$f;
275     fi
276   done
277   # make it easier to check for shaded/not later on.

3.修改/opt/module/kylin/bin/kylin.sh

119     start_command="hbase_kylin ${KYLIN_EXTRA_START_OPTS} \
```



