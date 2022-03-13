# 安装

```
#三台机器都添加solr用户
useradd solr
echo solr | passwd --stdin solr
#将solr压缩包解压
tar -zxvf solr-7.7.3.tgz -C /opt/module/
mv solr-7.7.3/ solr
chown -R solr:solr /opt/module/solr
#修改配置文件
vim /solr/bin/solr.in.sh
ZK_HOST="hadoop102:2181,hadoop103:2181,hadoop104:2181"
xsync /opt/module/solr
zk.sh start
#每个机器上执行此命令
sudo -i -u solr /opt/module/solr/bin/solr start
#修改系统的配置，如文件数目限制，进程数限制
修改/etc/security/limits.conf文件，增加以下内容
* soft nofile 65000
* hard nofile 65000
修改/etc/security/limits.d/20-nproc.conf文件
*          soft    nproc     65000
访问http://localhost：8983
```

