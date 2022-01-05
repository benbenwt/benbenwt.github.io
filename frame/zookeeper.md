### 安装

##### 修改myid

```
下载压缩包并解压
mkdir zkData
cd zkData
#保持每个结点不同即可
vim myid
xsync myid
```

##### 修改zoo.cfg

```
cp  zoo-sample.cfg  zoo.cfg
vim  zoo.cfg
dataDir=/opt/module/zookeeper-3.5.7/zkData
增加如下配置
#######################cluster##########################
server.2=hadoop102:2888:3888
server.3=hadoop103:2888:3888
server.4=hadoop104:2888:3888
xsync  zoo.cfg
```

### 启动

```
bin/zkServer.sh start
bin/zkServer.sh status
```

