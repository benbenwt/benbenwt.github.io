### 常用命令

pwd

cat

### 网络

##### 防火墙

1、开放端口

**firewall-cmd --zone=public --add-port=5672/tcp --permanent**  # 开放5672端口

**firewall-cmd --zone=public --remove-port=5672/tcp --permanent** #关闭5672端口

**firewall-cmd --reload**  # 配置立即生效

 

2、查看防火墙所有开放的端口

**firewall-cmd - -zone=public - -list-ports**

 

3.、关闭防火墙

如果要开放的端口太多，嫌麻烦，可以关闭防火墙，安全性自行评估

**systemctl stop firewalld.service**

 

4、查看防火墙状态

 **firewall-cmd - -state**

 

5、查看监听的端口

**netstat -lnpt**

![img](https://img2018.cnblogs.com/blog/1336432/201903/1336432-20190302110949754-1765820036.png)

*PS:centos7默认没有 netstat 命令，需要安装 net-tools 工具，yum install -y net-tools*

 

 

6、检查端口被哪个进程占用

**netstat -lnpt |grep 5672**

![img](https://img2018.cnblogs.com/blog/1336432/201903/1336432-20190302104128381-1210567174.png)

 

7、查看进程的详细信息

**ps 6832**

![img](https://img2018.cnblogs.com/blog/1336432/201903/1336432-20190302104342651-779103690.png)

 

8、中止进程

**kill -9 6832**

##### 配置网络



### 常用目录

/etc/hosts

/etc/sysconfig/network

### ssh连接与ftp

ssh连接和ftp使用系统中已创建的用户名及密码，加上ip和端口实现访问。

ssh默认在22，ftp默认在21,sftp，默认在22端口。修改ssh在etc/ssh/ssh_config中修改。

notepad配置好sftp插件，填写参数即可上传。

### 服务器之间copy文件

yum install ssh-clients安装scp命令

scp /tm  @172.18.65.22:/root/将当前电脑tm拷贝到目标主机root目录下。随后输入密码即可。

例如：scp workers  root@hbase2:反单引号pwd反单引号

反单引号：1左边那个`



##### rsync

rsync     -rvl  /tmp   root@hbae:/tmp

##### xsync

复制到所有节点相同目录下。



### problem

##### 当前用户不是root用户，修改etc文件。

su root切换到root用户。chmod 777 /etc/sudoers开启修改权限，vim打开在其中，加上usr ALL=(ALL) ALL，为用户开启root。

##### 配置好ip，gateway，掩码，dns后，连接校园网无法访问百度

局域网内连通，连接校园网后无法访问百度。使用ip测试，可访问百度，dns有问题。更换dns，114改为1.2.4.8。

##### ssh远程无法连接

利用webconsole登陆进入后，使用su root可以切换到root目录下，也修改了密码。但是ssh仍无法连接。

解决：在web登陆界面用root用户进入，再用ssh即可连接，奇怪。

解决：上边的解决方案也不一定，我修改密码使用service sshd restart后又可以登陆了。

##### ECDSA host key for ip has been changed

在当前机器执行ssh-keygen -R 远程ip，刷新信息即可。

##### ssh秘钥公钥问题

将自己的公钥给别人，自己可以登陆别人。

使用ssh-keygen -t  rsa生成秘钥和公钥

使用ssh-copy-id ip将自己的秘钥放到目标主机的authorized_hosts

中。随后使用ssh ip即可无密码访问。

##### 关于slaves

hadoop3以后，slaves更名为workers。