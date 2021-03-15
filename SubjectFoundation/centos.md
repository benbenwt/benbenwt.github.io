### sshd

ssh-keygen -t rsa 

ssh-copy-id -i /root/.ssh/id_rsa.pub  "-p2222 root@192.168.10.31"

ssh -p 2222 root@192.168.10.31

less secure

cat /var/log/secure

### 常用命令

pwd

cat

### 网络

##### 网卡

brctl命令

### 参数说明和示例

| 参数                      | 说明                   | 示例                  |
| ------------------------- | ---------------------- | --------------------- |
| `addbr <bridge>`          | 创建网桥               | brctl addbr br10      |
| `delbr <bridge>`          | 删除网桥               | brctl delbr br10      |
| `addif <bridge> <device>` | 将网卡接口接入网桥     | brctl addif br10 eth0 |
| `delif <bridge> <device>` | 删除网桥接入的网卡接口 | brctl delif br10 eth0 |
| `show <bridge>`           | 查询网桥信息           | brctl show br10       |
| `stp <bridge> {on|off}`   | 启用禁用 STP           | brctl stp br10 off/on |
| `showstp <bridge>`        | 查看网桥 STP 信息      | brctl showstp br10    |
| `setfd <bridge> <time>`   | 设置网桥延迟           | brctl setfd br10 10   |
| `showmacs <bridge>`       | 查看 mac 信息          | brctl showmacs br10   |



作者：河码匠
链接：https://www.jianshu.com/p/665382d70ab1
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

### iptables



##### 防火墙

>添加规则后使用reload，重启没用。firewall-cmd --reload



```
# 查询端口是否开放
firewall-cmd --query-port=3306/tcp
# 开放80端口
firewall-cmd --permanent --add-port=80/tcp
# 移除端口
firewall-cmd --permanent --remove-port=8080/tcp
```

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

```
firewall-cmd --list-all 
```

##### netstat -lnpt



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

### 文件



### problem

##### 当前用户不是root用户，修改etc文件。

su root切换到root用户。chmod 777 /etc/sudoers开启修改权限，vim打开在其中，加上usr ALL=(ALL) ALL，为用户开启root。

##### 配置好ip，gateway，掩码，dns后，连接校园网无法访问百度

局域网内连通，连接校园网后无法访问百度。使用ip测试，可访问百度，dns有问题。更换dns，114改为1.2.4.8。

##### ssh远程无法连接

利用webconsole登陆进入后，使用su root可以切换到root目录下，也修改了密码。但是ssh仍无法连接。

解决：修改密码使用service sshd restart后又可以登陆了。但只能从内网机器登陆，其他不行，开vpn也没用。

##### ECDSA host key for ip has been changed

在当前机器执行ssh-keygen -R 远程ip，刷新信息即可。

例子：ssh-keygen -R 192.168.0.100

##### ssh秘钥公钥问题

将自己的公钥给别人，自己可以登陆别人。

使用ssh-keygen -t  rsa生成秘钥和公钥

使用ssh-copy-id ip将自己的秘钥放到目标主机的authorized_hosts

中。随后使用ssh ip即可无密码访问。

##### 关于slaves

hadoop3以后，slaves更名为workers。

##### 关闭多个ssh连接

```
查看当前所有连接
w
who
查看自己登陆的用户
whoami查看自己所用连接
who am i
杀死ssh连接
pkill -kill -t pts/1
```

##### SSH连接时出现Host key verification failed

/etc/ssh/ssh_config中改为StrictHostKeyChecking=no 

##### sshd登录密码正确，提示permission denied

vim /etc/ssh/ssh_config

修改port，再改回去

putty选择connection->ssh->rsa

ssh-copy-id无效

     chmod g-w /home/your_user # 或　chmod 0755 /home/your_user
     chmod 700 /home/your_user/.ssh
     
     chmod 600 /home/your_user/.ssh/authorized_keys

 然后重启ssh服务，就可以免密码登陆了。原因在于ssh服务会检查文件权限码，如777这种，会被认为不安全。

##### 克隆后删除旧网卡

>关于服务已开启，端口已开放。但是ssh或其他服务总是断开连接的问题，最后意识到可能是ip的问题，换了一个ip发现好了。最近发现是由于188这个地址已经被其他人开启的机器占用了，当发现ssh端口为22时就应该意识到，我登陆的不是自己的机器，有另一台机器也为188。

/etc/sysconfig/network-scripts/ifcfg-exxx，

ip addr查看网卡

单节点配置文件