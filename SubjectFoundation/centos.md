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

##### rsync

rsync     -rvl  /tmp   root@hbae:/tmp

##### xsync

复制到所有节点相同目录下。



### problem

##### 当前用户不是root用户，修改etc文件。

su root切换到root用户。chmod 777 /etc/sudoers开启修改权限，vim打开在其中，加上usr ALL=(ALL) ALL，为用户开启root。

##### 配置好ip，gateway，掩码，dns后，连接校园网无法访问百度

局域网内连通，连接校园网后无法访问百度。使用ip测试，可访问百度，dns有问题。更换dns，114改为1.2.4.8。