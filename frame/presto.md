### 理论

>presto是分布式sql查询引擎，支持多数据源，包括hive、redis、mysql等。
>
>如果向快速查询hive仓库，或者和其他源合并查询，可以使用presto。

### 安装

##### presto server安装

```
tar -zxvf presto-server-0.196.tar.gz -C /opt/module/
mv presto-server-0.196/ presto
mkdir data
mkdir etc
#添加配置文件jvm.config
cd etc 
vim jvm.config：
-server
-Xmx16G
-XX:+UseG1GC
-XX:G1HeapRegionSize=32M
-XX:+UseGCOverheadLimit
-XX:+ExplicitGCInvokesConcurrent
-XX:+HeapDumpOnOutOfMemoryError
-XX:+ExitOnOutOfMemoryError
#添加配置文件hive.properties ,此处的hive-hadoop2不是主机host，是指的使用hive服务，对应plugin下的文件夹，文件夹内是jar包依赖。
mkdir etc/catalog
vim hive.properties 
connector.name=hive-hadoop2
hive.metastore.uri=thrift://hbase:9083
#分发
xsync presto
#添加node.properties，注意每个节点的ndoe.id不同
vim node.properties
node.environment=production
node.id=1
node.data-dir=/opt/module/presto/data
#添加config.properties,注意coordinator一项只有master设置为true
vim config.properties
coordinator=true
node-scheduler.include-coordinator=false
http-server.http.port=8881
query.max-memory=50GB
discovery-server.enabled=true
discovery.uri=http://hbase:8881
#启动hive的metastore
nohup bin/hive --service metastore >/dev/null 2>&1 &
#分别在三台机器上启动Presto Server
bin/launcher run
```

##### presto cli安装

```
mv presto-cli-0.196-executable.jar  prestocli
chmod +x prestocli
#使用prestocli连接server服务，然后在shell中测试是否功能正常。
./prestocli --server hbase:8881 --catalog hive --schema default
show tables;
use default;
select * from test_table;
```

##### 可视化web服务安装

```
#修改conf/yanagishima.properties，修改web端口号和hive连接url，用户，密码等信息。
#启动web服务
nohup bin/yanagishima-start.sh >y.log 2>&1 &
```

###### yanagishima.properties内容如下

>修改presto-server服务url，端口：presto.coordinator.server.atguigu-presto、presto.redirect.server.your-presto
>
>修改web端口号：jetty.port
>
>hive连接url，用户，密码：presto.coordinator.server.atguigu-presto、resource.manager.url.your-hive、hive.jdbc.user.your-hive、hive.jdbc.password.your-hive

```
# yanagishima web port
jetty.port=7080
# 30 minutes. If presto query exceeds this time, yanagishima cancel the query.
presto.query.max-run-time-seconds=1800
# 1GB. If presto query result file size exceeds this value, yanagishima cancel the query.
presto.max-result-file-byte-size=1073741824
# you can specify freely. But you need to specify same name to presto.coordinator.server.[...] and presto.redirect.server.[...] and catalog.[...] and schema.[...]
presto.datasources=atguigu-presto
auth.your-presto=false
# presto coordinator url
presto.coordinator.server.atguigu-presto=http://hbase:8881
# almost same as presto coordinator url. If you use reverse proxy, specify it
presto.redirect.server.your-presto=http://hbase:8881
# presto catalog name
catalog.atguigu-presto=hive
# presto schema name
schema.atguigu-presto=default
# if query result exceeds this limit, to show rest of result is skipped
select.limit=500
# http header name for audit log
audit.http.header.name=some.auth.header
use.audit.http.header.name=false
# limit to convert from tsv to values query
to.values.query.limit=500
# authorization feature
check.datasource=false
hive.jdbc.url.your-hive=jdbc:hive2://hbase:10000/default;auth=noSasl
hive.jdbc.user.your-hive=root
hive.jdbc.password.your-hive=root
hive.query.max-run-time-seconds=3600
hive.query.max-run-time-seconds.your-hive=3600
resource.manager.url.your-hive=http://hbase:8088
sql.query.engines=presto
hive.datasources=your-hive
hive.disallowed.keywords.your-hive=insert,drop
# 1GB. If hive query result file size exceeds this value, yanagishima cancel the query.
hive.max-result-file-byte-size=1073741824
hive.setup.query.path.your-hive=/usr/local/yanagishima/conf/hive_setup_query_your-hive
cors.enabled=false
```



### 启动

```
#启动后端服务
python2  /opt/module/presto/bin/launcher.py start
python2  /opt/module/presto/bin/launcher.py stop
#进入cli控制台
./prestocli --server hadoop102:8881 --catalog hive --schema default
#修改conf/yanagishima.properties，修改web端口号和hive连接url，用户，密码等信息。
#启动web服务
nohup bin/yanagishima-start.sh >y.log 2>&1 &
```

### 使用效果

##### 对比hive查询

```
#执行此语句SELECT count(*) FROM hive.gmall."ads_user_change"  LIMIT 100，presto需要0.54s，而hive需要22s。
```

![presto](..\resources\images\presto.png)

### problem

##### 查表无法访问lzo文件

```
将hadoop-lzo.jar移到presto/plugin/hive-hadoop2下即可，这样能查询ads层，其他层依然不可以。
```

