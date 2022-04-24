# 安装

```
#在服务端主机
yum install -y krb5-server
#在客户端主机
yum install -y krb5-workstation krb5-libs
```

```
#修改服务端配置文件，指定kerberos的realm域
vim /var/kerberos/krb5kdc/kdc.conf

[kdcdefaults]
 kdc_ports = 88
 kdc_tcp_ports = 88

[realms]
 EXAMPLE.COM = {
  #master_key_type = aes256-cts
  acl_file = /var/kerberos/krb5kdc/kadm5.acl
  dict_file = /usr/share/dict/words
  admin_keytab = /var/kerberos/krb5kdc/kadm5.keytab
  supported_enctypes = aes256-cts:normal aes128-cts:normal des3-hmac-sha1:normal arcfour-hmac:normal camellia256-cts:normal camellia128-cts:normal des-hmac-sha1:normal des-cbc-md5:normal des-cbc-crc:normal
 }
```

```
#修改客户端配置文件
vim /etc/krb5.conf

# Configuration snippets may be placed in this directory as well
includedir /etc/krb5.conf.d/

[logging]
 default = FILE:/var/log/krb5libs.log
 kdc = FILE:/var/log/krb5kdc.log
 admin_server = FILE:/var/log/kadmind.log

[libdefaults]
 dns_lookup_realm = false
 dns_lookup_kdc = false
 ticket_lifetime = 24h
 renew_lifetime = 7d
 forwardable = true
 rdns = false
 pkinit_anchors = FILE:/etc/pki/tls/certs/ca-bundle.crt
 default_realm = EXAMPLE.COM
 #default_ccache_name = KEYRING:persistent:%{uid}

[realms]
 EXAMPLE.COM = {
  kdc = hadoop102
  admin_server = hadoop102
 }
[domain_realm]
# .example.com = EXAMPLE.COM
# example.com = EXAMPLE.COM
```

```
#初始化KDC数据库，根据提示输入密码
kdb5_util create -s
#修改服务端管理员权限配置文件
vim /var/kerberos/krb5kdc/kadm5.acl
*/admin@EXAMPLE.COM     *
```

```
#启动kerberos服务
#启动kdc
systemctl start krb5kdc
systemctl enable krb5kdc

#启动Kadmin，时kdc数据库访问入口
systemctl start kadmin
systemctl enable kadmin

#创建管理员，在服务端主机执行如下命令
kadmin.local -q "addprinc admin/admin"
```

# Hadoop Kerberos

## 创建hadoop系统用户

>为hadoop开启kerberos，需要为不同服务准备不同的用户，启动服务时需要相应的用户。必须在所有节点创建以下用户和用户组。

| User：Group   | Daemons                                          |
| ------------- | ------------------------------------------------ |
| hdfs:hadoop   | NameNode,Secondary NameNode,JournalNode,DataNode |
| yarn:hadoop   | ResourceManager,NodeManager                      |
| mapred:hadoop | MapReduce JobHistory Server                      |

```
#在三台机器上都添加hadoop用户组
groupadd hadoop
```

```
#在三台机器上根据表格创建各用户并设置密码
useradd hdfs -g hadoop
useradd yarn -g hadoop
useradd mapred -g hadoop
```

## 创建kerberos主体principal

| **服务**               | **所在主机** | **主体（****Principal****）** |
| ---------------------- | ------------ | ----------------------------- |
| **NameNode**           | hadoop102    | nn/hadoop102                  |
| **DataNode**           | hadoop102    | dn/hadoop102                  |
| **DataNode**           | hadoop103    | dn/hadoop103                  |
| **DataNode**           | hadoop104    | dn/hadoop104                  |
| **Secondary NameNode** | hadoop104    | sn/hadoop104                  |
| **ResourceManager**    | hadoop103    | rm/hadoop103                  |
| **NodeManager**        | hadoop102    | nm/hadoop102                  |
| **NodeManager**        | hadoop103    | nm/hadoop103                  |
| **NodeManager**        | hadoop104    | nm/hadoop104                  |
| **JobHistory Server**  | hadoop102    | jhs/hadoop102                 |
| **Web UI**             | hadoop102    | HTTP/hadoop102                |
| **Web UI**             | hadoop103    | HTTP/hadoop103                |
| **Web UI**             | hadoop104    | HTTP/hadoop104                |

```
#认证为管理员并登录
kinit admin/admin
kadmin
#登陆后添加主体
addprinc -randkey test/test
#将主体的密钥吸入keytab文件
xst -k /etc/security/keytab/test.keytab test/test

#此命令可以在shell直接创建主体，写入keytab
kadmin -padmin/admin -wadmin -q"addprinc -randkey test/test"
kadmin -padmin/admin -wadmin -q"xst -k /etc/security/keytab/test.keytab test/test"
```

```
#创建hadoop主体
kadmin -padmin/admin -wadmin -q"addprinc -randkey nn/hadoop102"
kadmin -padmin/admin -wadmin -q"xst -k /etc/security/keytab/nn.service.keytab nn/hadoop102"
#在每台对应的机器，按照表格样式创建对应的principal主体

#修改keytab文件的所有者和访问权限
chown -R root:hadoop /etc/security/keytab/
chmod 660 /etc/security/keytab/*
```

## 修改hadoop配置文件

```
vim /opt/module/hadoop-3.1.3/etc/hadoop/core-site.xml

<!-- Kerberos主体到系统用户的映射机制 -->
<property>
  <name>hadoop.security.auth_to_local.mechanism</name>
  <value>MIT</value>
</property>

<!-- Kerberos主体到系统用户的具体映射规则 -->
<property>
  <name>hadoop.security.auth_to_local</name>
  <value>
    RULE:[2:$1/$2@$0]([ndj]n\/.*@EXAMPLE\.COM)s/.*/hdfs/
    RULE:[2:$1/$2@$0]([rn]m\/.*@EXAMPLE\.COM)s/.*/yarn/
    RULE:[2:$1/$2@$0](jhs\/.*@EXAMPLE\.COM)s/.*/mapred/
    DEFAULT
  </value>
</property>

<!-- 启用Hadoop集群Kerberos安全认证 -->
<property>
  <name>hadoop.security.authentication</name>
  <value>kerberos</value>
</property>

<!-- 启用Hadoop集群授权管理 -->
<property>
  <name>hadoop.security.authorization</name>
  <value>true</value>
</property>

<!-- Hadoop集群间RPC通讯设为仅认证模式 -->
<property>
  <name>hadoop.rpc.protection</name>
  <value>authentication</value>
</property>

```

```
vim /opt/module/hadoop-3.1.3/etc/hadoop/hdfs-site.xml

<!-- 访问DataNode数据块时需通过Kerberos认证 -->
<property>
  <name>dfs.block.access.token.enable</name>
  <value>true</value>
</property>

<!-- NameNode服务的Kerberos主体,_HOST会自动解析为服务所在的主机名 -->
<property>
  <name>dfs.namenode.kerberos.principal</name>
  <value>nn/_HOST@EXAMPLE.COM</value>
</property>

<!-- NameNode服务的Kerberos密钥文件路径 -->
<property>
  <name>dfs.namenode.keytab.file</name>
  <value>/etc/security/keytab/nn.service.keytab</value>
</property>

<!-- Secondary NameNode服务的Kerberos主体 -->
<property>
  <name>dfs.secondary.namenode.keytab.file</name>
  <value>/etc/security/keytab/sn.service.keytab</value>
</property>

<!-- Secondary NameNode服务的Kerberos密钥文件路径 -->
<property>
  <name>dfs.secondary.namenode.kerberos.principal</name>
  <value>sn/_HOST@EXAMPLE.COM</value>
</property>

<!-- NameNode Web服务的Kerberos主体 -->
<property>
  <name>dfs.namenode.kerberos.internal.spnego.principal</name>
  <value>HTTP/_HOST@EXAMPLE.COM</value>
</property>

<!-- WebHDFS REST服务的Kerberos主体 -->
<property>
  <name>dfs.web.authentication.kerberos.principal</name>
  <value>HTTP/_HOST@EXAMPLE.COM</value>
</property>

<!-- Secondary NameNode Web UI服务的Kerberos主体 -->
<property>
  <name>dfs.secondary.namenode.kerberos.internal.spnego.principal</name>
  <value>HTTP/_HOST@EXAMPLE.COM</value>
</property>

<!-- Hadoop Web UI的Kerberos密钥文件路径 -->
<property>
  <name>dfs.web.authentication.kerberos.keytab</name>
  <value>/etc/security/keytab/spnego.service.keytab</value>
</property>

<!-- DataNode服务的Kerberos主体 -->
<property>
  <name>dfs.datanode.kerberos.principal</name>
  <value>dn/_HOST@EXAMPLE.COM</value>
</property>

<!-- DataNode服务的Kerberos密钥文件路径 -->
<property>
  <name>dfs.datanode.keytab.file</name>
  <value>/etc/security/keytab/dn.service.keytab</value>
</property>

<!-- 配置NameNode Web UI 使用HTTPS协议 -->
<property>
  <name>dfs.http.policy</name>
  <value>HTTPS_ONLY</value>
</property>

<!-- 配置DataNode数据传输保护策略为仅认证模式 -->
<property>
  <name>dfs.data.transfer.protection</name>
  <value>authentication</value>
</property>

```

```
vim /opt/module/hadoop-3.1.3/etc/hadoop/yarn-site.xml

<!-- Resource Manager 服务的Kerberos主体 -->
<property>
  <name>yarn.resourcemanager.principal</name>
  <value>rm/_HOST@EXAMPLE.COM</value>
</property>

<!-- Resource Manager 服务的Kerberos密钥文件 -->
<property>
  <name>yarn.resourcemanager.keytab</name>
  <value>/etc/security/keytab/rm.service.keytab</value>
</property>

<!-- Node Manager 服务的Kerberos主体 -->
<property>
  <name>yarn.nodemanager.principal</name>
  <value>nm/_HOST@EXAMPLE.COM</value>
</property>

<!-- Node Manager 服务的Kerberos密钥文件 -->
<property>
  <name>yarn.nodemanager.keytab</name>
  <value>/etc/security/keytab/nm.service.keytab</value>
</property>

```

```
vim /opt/module/hadoop-3.1.3/etc/hadoop/mapred-site.xml

<!-- 历史服务器的Kerberos主体 -->
<property>
  <name>mapreduce.jobhistory.keytab</name>
  <value>/etc/security/keytab/jhs.service.keytab</value>
</property>

<!-- 历史服务器的Kerberos密钥文件 -->
<property>
  <name>mapreduce.jobhistory.principal</name>
  <value>jhs/_HOST@EXAMPLE.COM</value>
</property>

```

```
xsync 以上修改的文件
```



## 配置hdfs使用https安全传输协议

```
keytool -keystore /etc/security/keytab/keystore -alias jetty -genkey -keyalg RSA
chown -R root:hadoop /etc/security/keytab/keystore
chmod 660 /etc/security/keytab/keystore
xsync /etc/security/keytab/keystore

mv $HADOOP_HOME/etc/hadoop/ssl-server.xml.example $HADOOP_HOME/etc/hadoop/ssl-server.xml
vim $HADOOP_HOME/etc/hadoop/ssl-server.xml
<!-- SSL密钥库路径 -->
<property>
  <name>ssl.server.keystore.location</name>
  <value>/etc/security/keytab/keystore</value>
</property>

<!-- SSL密钥库密码 -->
<property>
  <name>ssl.server.keystore.password</name>
  <value>123456</value>
</property>

<!-- SSL可信任密钥库路径 -->
<property>
  <name>ssl.server.truststore.location</name>
  <value>/etc/security/keytab/keystore</value>
</property>

<!-- SSL密钥库中密钥的密码 -->
<property>
  <name>ssl.server.keystore.keypassword</name>
  <value>123456</value>
</property>

<!-- SSL可信任密钥库密码 -->
<property>
  <name>ssl.server.truststore.password</name>
  <value>123456</value>
</property>


xsync $HADOOP_HOME/etc/hadoop/ssl-server.xml
```



## 配置yarn使用LinuxContainerExecutor

```
chown root:hadoop /opt/module/hadoop-3.1.3/bin/container-executor
chmod 6050 /opt/module/hadoop-3.1.3/bin/container-executor
#三台机器都执行如上

#三台机器都执行如下
chown root:hadoop /opt/module/hadoop-3.1.3/etc/hadoop/container-executor.cfg
chown root:hadoop /opt/module/hadoop-3.1.3/etc/hadoop
chown root:hadoop /opt/module/hadoop-3.1.3/etc
chown root:hadoop /opt/module/hadoop-3.1.3
chown root:hadoop /opt/module
chmod 400 /opt/module/hadoop-3.1.3/etc/hadoop/container-executor.cfg
```

```
vim $HADOOP_HOME/etc/hadoop/container-executor.cfg
yarn.nodemanager.linux-container-executor.group=hadoop
banned.users=hdfs,yarn,mapred
min.user.id=1000
allowed.system.users=
feature.tc.enabled=false
```

```
vim $HADOOP_HOME/etc/hadoop/yarn-site.xml

<!-- 配置Node Manager使用LinuxContainerExecutor管理Container -->
<property>
  <name>yarn.nodemanager.container-executor.class</name>
  <value>org.apache.hadoop.yarn.server.nodemanager.LinuxContainerExecutor</value>
</property>

<!-- 配置Node Manager的启动用户的所属组 -->
<property>
  <name>yarn.nodemanager.linux-container-executor.group</name>
  <value>hadoop</value>
</property>

<!-- LinuxContainerExecutor脚本路径 -->
<property>
  <name>yarn.nodemanager.linux-container-executor.path</name>
  <value>/opt/module/hadoop-3.1.3/bin/container-executor</value>
</property>

xsync $HADOOP_HOME/etc/hadoop/container-executor.cfg
xsync $HADOOP_HOME/etc/hadoop/yarn-site.xml
```

## 安全模式启动hadoop集群

```
#三台机器执行 hadoop-env.sh  默认值为 ${HADOOP_HOME}/logs
chown hdfs:hadoop /opt/module/hadoop-3.1.3/logs/
chmod 775 /opt/module/hadoop-3.1.3/logs/

#dfs.namenode.name.dir
chown -R hdfs:hadoop /opt/module/hadoop-3.1.3/data/dfs/name/
chmod 700 /opt/module/hadoop-3.1.3/data/dfs/name/

#dfs.datanode.data.dir
chown -R hdfs:hadoop /opt/module/hadoop-3.1.3/data/dfs/data/
chmod 700 /opt/module/hadoop-3.1.3/data/dfs/data/

#SecondaryNameNode  dfs.namenode.checkpoint.dir
chown -R hdfs:hadoop /opt/module/hadoop-3.1.3/data/dfs/namesecondary/
chmod 700 /opt/module/hadoop-3.1.3/data/dfs/namesecondary/

#yarn.nodemanager.local-dirs（NodeManager节点）
chown -R yarn:hadoop /opt/module/hadoop-3.1.3/data/nm-local-dir/
chmod -R 775 /opt/module/hadoop-3.1.3/data/nm-local-dir/

#yarn.nodemanager.log-dirs（NodeManager节点）
chown yarn:hadoop /opt/module/hadoop-3.1.3/logs/userlogs/
 chmod 775 /opt/module/hadoop-3.1.3/logs/userlogs/
```

### 启动hdfs

```
sudo -i -u hdfs hdfs --daemon start namenode
#三台机器
sudo -i -u hdfs hdfs --daemon start datanode
#SecondaryNameNode
sudo -i -u hdfs hdfs --daemon start secondarynamenode
```

```
#群起
vim $HADOOP_HOME/sbin/start-dfs.sh

HDFS_DATANODE_USER=hdfs
HDFS_NAMENODE_USER=hdfs
HDFS_SECONDARYNAMENODE_USER=hdfs

start-dfs.sh
```

## 修改HDFS特定路径访问权限

>参数mapreduce.jobhistory.intermediate-done-dir位于mapred-site.xml文件，默认值为/tmp/hadoop-yarn/staging/history/done_intermediate，需保证该路径的所有上级目录（除/tmp）的所有者均为mapred，所属组为hadoop，权限为770

```
[root@hadoop102 ~]# hadoop fs -chown -R mapred:hadoop /tmp/hadoop-yarn/staging/history/done_intermediate
[root@hadoop102 ~]# hadoop fs -chmod -R 1777 /tmp/hadoop-yarn/staging/history/done_intermediate

[root@hadoop102 ~]# hadoop fs -chown mapred:hadoop /tmp/hadoop-yarn/staging/history/
[root@hadoop102 ~]# hadoop fs -chown mapred:hadoop /tmp/hadoop-yarn/staging/
[root@hadoop102 ~]# hadoop fs -chown mapred:hadoop /tmp/hadoop-yarn/

[root@hadoop102 ~]# hadoop fs -chmod 770 /tmp/hadoop-yarn/staging/history/
[root@hadoop102 ~]# hadoop fs -chmod 770 /tmp/hadoop-yarn/staging/
[root@hadoop102 ~]# hadoop fs -chmod 770 /tmp/hadoop-yarn/

```

>参数mapreduce.jobhistory.done-dir位于mapred-site.xml文件，默认值为/tmp/hadoop-yarn/staging/history/done，需保证该路径的所有上级目录（除/tmp）的所有者均为mapred，所属组为hadoop，权限为770

```
[root@hadoop102 ~]# hadoop fs -chown -R mapred:hadoop /tmp/hadoop-yarn/staging/history/done
[root@hadoop102 ~]# hadoop fs -chmod -R 750 /tmp/hadoop-yarn/staging/history/done

[root@hadoop102 ~]# hadoop fs -chown mapred:hadoop /tmp/hadoop-yarn/staging/history/
[root@hadoop102 ~]# hadoop fs -chown mapred:hadoop /tmp/hadoop-yarn/staging/
[root@hadoop102 ~]# hadoop fs -chown mapred:hadoop /tmp/hadoop-yarn/

[root@hadoop102 ~]# hadoop fs -chmod 770 /tmp/hadoop-yarn/staging/history/
[root@hadoop102 ~]# hadoop fs -chmod 770 /tmp/hadoop-yarn/staging/
[root@hadoop102 ~]# hadoop fs -chmod 770 /tmp/hadoop-yarn/

```

## 启动yarn

```
sudo -i -u yarn yarn --daemon start resourcemanager
#三台机器
sudo -i -u yarn yarn --daemon start nodemanager
```

```
#群起
vim $HADOOP_HOME/sbin/start-yarn.sh

在顶部增加如下内容

YARN_RESOURCEMANAGER_USER=yarn
YARN_NODEMANAGER_USER=yarn

start-yarn.sh
```

## 启动历史服务器

```
 sudo -i -u mapred mapred --daemon start historyserver
```

## 使用

### 创建

```
#三台机器
useradd atguigu
echo atguigu | passwd --stdin atguigu
usermod -a -G hadoop atguigu
kadmin -p admin/admin -wadmin -q"addprinc -pw atguigu atguigu"
```

### 认证并使用

```
kinit atguigu
klist
hadoop fs -ls /
#注销认证
kdestroy
hadoop fs -ls /
```

# 理论知识

>Kerberos 是一种基于加密 Ticket 的身份认证协议。kerberos是一种计算机网络认证协议，能为网络通信中的双方提供严格的身份验证服务，确保通信双方身份的真实性和安全性。
>
>https://zhuanlan.zhihu.com/p/266491528

## 基本概念

### 客户端client

>发送请求的一方

### 服务端Server

>接受请求的一方

### 密钥分发中心Key Distribution Center，KDC

>密钥分发中心又分为两部分，分别是AS和TGS

#### AS Authentication Server

>认证服务器，专门用来认证客户端的身份并发放客户用于访问TGS的TGT

#### TGS Ticket Granting Server

>票据授予服务器，用来发放整个认证过程以及客户端访问服务端所需的服务授予票据

### Principal

>kerberos中的用户名，用于标识身份。principal主要由三部分构成：primary，instance和realm。包含instance的principal，一般会作为server端的principal，如namenode，hiveserver2，presto Coordinator等；不含有instance的principal，一般作为客户端的principal，用于身份验证。例如：
>
>hiveserver2/alibaba.com@EXAMPLE.COM:表示服务为hiveser2，部署在alibaba.com上，realm域为@EXAMPLE.COM。
>
>xiaoming@EXAMPLE.COM: 用户名为小明，realm域为@EAXMPLE.COM

### Keytab

>密码本，包含了多个principal与密码的文件，用户可以利用该文件进行身份认证。

### Ticket Cache

>客户端与KDC交互完成后，包含身份证信息的文件，短期有效，需要不断renew。

### Realm

>Kerberos系统中的一个namespace。不同kerberos环境，可以通过realm进行分区。

## 认证过程

>1.kerberos基于Ticket实现身份认证，而非密码。如果客户端无法利用本地密钥，解密出KDC返回地加密Ticket，认证将无法通过。
>
>2客户端依次与AS,TGS以及目标service进行交互，共交互三次。
>
>3客户端与其他组件交互时，都将获得两条信息，其中一条可以通过本地密钥解密出，另外一条无法解密出。
>
>4客户端想要访问的目标服务，不会直接与KDC交互，而是通过是否正确解密出客户端的请求来认证。
>
>5KDC Database包含所有principal对应的密码
>
>6kerberos中信息加密一般是对称加密，可配置成非对称加密。
>
>优点：1密码无需传输，基于ticket实现身份认证
>
>2双向认证
>
>3高性能，一旦client获得访问某个server的ticket，该server就能根据这个ticket实现对client的验证，无需KDC再次参与。

### 获取票据

>客户端会访问KDC两次，获取到访问服务票据后，再访问对应的服务。
>
>第一次访问KDC的AS获取到TGS的访问票据，第二次访问TGS获取到所需访问服务的票据。

### 访问服务

>根据获取到的访问服务票据取访问对应的服务

# 用法

## kerberos数据库操作

```
#本地登录
kadmin.local
#远程登录
kadmin
```

```
#创建kerberos主体,登陆后输入如下命令
addprinc test
#也可以shell直接执行
kadmin.local -q"addprinc test"
#修改主体密码
cpw test
#查看所有主体
list_principals
```

## kerberos认证操作

```
#使用kinit认证主体
kinit test
#查看认证凭证
klist
```

```
#密钥文件认证
kadmin.local -q "xst -norandkey -k  /root/test.keytab test@EXAMPLE.COM"
#使用生成的keytab
kinit -kt /root/test.keytab test
#查看认证凭证
klist 
#销毁凭证
kdestroy
klist
```

