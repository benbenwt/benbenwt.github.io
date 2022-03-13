### atlas

>atlas支持元数据分类、元数据检索、元数据血缘依赖

### 安装

>atlas需要hbase、solr的支持，建议使用外置的hbase、solr。

```
#解压
tar -zxvf apache-atlas-2.1.0-server.tar.gz -C /opt/module/
mv /opt/module/apache-atlas-2.1.0 /opt/module/atlas
#集成hbase
vim /opt/module/atlas/conf/atlas-application.properties
atlas.graph.storage.hostname=hadoop102:2181,hadoop103:2181,hadoop104:2181
vim /opt/module/atlas/conf/atlas-env.sh
export HBASE_CONF_DIR=/opt/module/hbase/conf

#集成solr
vim /opt/module/atlas/conf/atlas-application.properties
atlas.graph.index.search.backend=solr
atlas.graph.index.search.solr.mode=cloud
atlas.graph.index.search.solr.zookeeper-url=hadoop102:2181,hadoop103:2181,hadoop104:2181
#创建solr collection
sudo -i -u solr /opt/module/solr/bin/solr create  -c vertex_index -d /opt/module/atlas/conf/solr -shards 3 -replicationFactor 2

#集成kafka
vim /opt/module/atlas/conf/atlas-application.properties
atlas.notification.embedded=false
atlas.kafka.data=/opt/module/kafka/data
atlas.kafka.zookeeper.connect= hadoop102:2181,hadoop103:2181,hadoop104:2181/kafka
atlas.kafka.bootstrap.servers=hadoop102:9092,hadoop103:9092,hadoop104:9092

#atlas server配置
#########  Server Properties  #########
atlas.rest.address=http://hadoop102:21000
# If enabled and set to true, this will run setup steps when the server starts
atlas.server.run.setup.on.start=false

#########  Entity Audit Configs  #########
atlas.audit.hbase.zookeeper.quorum=hadoop102:2181,hadoop103:2181,hadoop104:2181

#记录性能指标
vim atlas-log4j.xml
#去掉如下代码的注释
<appender name="perf_appender" class="org.apache.log4j.DailyRollingFileAppender">
    <param name="file" value="${atlas.log.dir}/atlas_perf.log" />
    <param name="datePattern" value="'.'yyyy-MM-dd" />
    <param name="append" value="true" />
    <layout class="org.apache.log4j.PatternLayout">
        <param name="ConversionPattern" value="%d|%t|%m%n" />
    </layout>
</appender>

<logger name="org.apache.atlas.perf" additivity="false">
    <level value="debug" />
    <appender-ref ref="perf_appender" />
</logger>

```

###### 添加hive hook

```
#添加hive-hook插件
tar -zxvf apache-atlas-2.1.0-hive-hook.tar.gz
cp -r apache-atlas-hive-hook-2.1.0/* /opt/module/atlas/

#修改hive配置
vim hive-env.sh
export HIVE_AUX_JARS_PATH=/opt/module/atlas/hook/hive
vim /hive/conf/hive-site.xml
<property>
      <name>hive.exec.post.hooks</name>
      <value>org.apache.atlas.hive.hook.HiveHook</value>
</property>

#修改atlas配置
vim conf/atlas-application.properties
######### Hive Hook Configs #######

atlas.hook.hive.synchronous=false
atlas.hook.hive.numRetries=3
atlas.hook.hive.queueSize=10000
atlas.cluster.name=primary

#拷贝atlas配置到hive
cp /opt/module/atlas/conf/atlas-application.properties  /opt/module/hive/conf/
```

### 启动

```
python2 bin/atlas_start.py
```

###### 导入hive表元数据

```
hook-bin/imort_hive.sh
```



#### problem

##### javasocketreadtimeout

>导入几个表之后，后边的就无法导入成功了，显示status404。再重启，再次导入，又能成功几个。

```
写一个脚本循环启动服务、尝试导入hive表。
```

