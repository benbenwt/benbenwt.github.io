[TOC]

```
free -h
```

gpu

```
lspci | grep -i vga
```

# 镜像

>https://mirrors.bfsu.edu.cn/centos/7/isos/x86_64/

>https://blog.csdn.net/frank1998819/article/details/84774176
>
>minimal是精简版本，dvd是最完整的，体积达到了4G。

# cpu

```
https://blog.csdn.net/sflsgfs/article/details/9129307
more /proc/cpuinfo | grep "model name"  
grep "model name" /proc/cpuinfo  
grep "CPU" /proc/cpuinfo  

#查看物理cpu数量
cat /proc/cpuinfo | grep name | cut -f2 -d: | uniq -c

#查看逻辑cpu数量
cat /proc/cpuinfo | grep "processor" |wc -l

#查看物理cpu是几个核心的
cat /proc/cpuinfo | grep "cores"|uniq
```

# sshd

```
查看状态：

systemctl status sshd.service
启动服务：

systemctl start sshd.service
重启服务：

systemctl restart sshd.service
开机自启：

systemctl enable sshd.service
```

```
##### sshd

ssh-keygen -t rsa 

ssh-copy-id -i /root/.ssh/id_rsa.pub  "-p2222 root@192.168.10.31"

ssh -p 2222 root@192.168.10.31

less secure

cat /var/log/secure
```





# 磁盘



## 常用命令

```
df -h 查看磁盘容量
```

# centos shell功能
## 开机启动
```
# /etc/rc.d/rc.local负责管理开机自启的程序，该程序需要赋予可执行权限。
vim /etc/rc.d/rc.local
在其中追加shell命令，达到自启的目的。

#该命令用于引入环境变量
source /etc/profile
```

## 定时任务
```
#centos 的定时任务需要借助crontab命令。
#可以通过修改/etc/crontab 添加定时任务，然后systemctl restart crond重启crontab服务让配置生效
 * * * * * /home/test.sh
 从左向右五个*单位分别是，分钟，小时，日期，月份，星期

#如下例子表示每隔24小时执行一次
* */24 * * * /test.sh

#如下例子表示没小时的第1分钟执行一次
1 * * * * /test.sh
```
# 用户管理

```
useradd -d  /home/username  username
userdel -f username
passwd username
sudo -u
#查看所有用户
cut -d : -f 1 /etc/passwd
```



# 环境变量

##### 登录shell与非登录shell

```
非登录shell访问访问不到/etc/profile，将编写的sh脚本放到/etc/profile.d下面，可以让两者都能访问。
```



# 软件安装

##### 自动yes

```
https://blog.csdn.net/linyisonger/article/details/106469176

# 一次
echo yes|[命令] # 输入 yes
echo y|[命令] # 输入 y
# 多次
yes yes|[命令] # 输入 yes
yes y|[命令] # 输入 y

# 例
yes yes|docker exec -i gitlab gitlab-rake gitlab:backup:restore 
# 效果如图 第二次询问自动执行

```

对于编译型语言，需要编译后进行安装。rpm是提供给redhat系列系统的，底层包管理工具，他会安装编译好的软件，但要手动安装所需依赖。dpkg是deb系列系统的包管理工具，同rpm。yum和apt则会自动安装编译后软件，并处理其所需依赖，分别用于centos和ubuntu。

对于解释型语言，有对应解释器和所需依赖即可。例如java的虚拟机，可以直接运行字节码文件，无需针对不同平台编译最终的安装包。

##### yum

yum-config-manageer

```
yum-config-manager -h
yum repolist all
yum repolist enabled
yum-config-manager --disable mysql-connectors-community
sudo yum-config-manager --add-repo https://repo.grakn.ai/repository/meta/rpm.repo
sudo yum update

cd /etc/yum.repos.d/
#管理下载docker的源，将其替换为aliyun
vim docker-ce.repo
```

##### rpm

```
rpm -qa packagename
rpm -e name --nodeps
rpm -ivh name

```

yum安装

```
#yum三个rpm包yum,fastesmirror,metadata-parser，其中metadata和fastesmirror互相依赖，要同一条安装。
https://blog.csdn.net/m0_37886429/article/details/75009382
http://mirror.centos.org/centos/7/os/x86_64/Packages/
```



# 常用命令

##### 时间设定

```
#系统时间
date 
date -R
#硬件时间
hwclock --show
hwclock --hctosys 
CMOS：clock -w
timedatectl
#联网修改
ntpdate  pool.ntp.org
hwclock -W
hwclock
```

```
修改时间

sudo date -s MM/DD/YY //修改日期
sudo date -s hh:mm:ss //修改时间
在修改时间以后，修改硬件CMOS的时间

sudo hwclock --systohc //非常重要，如果没有这一步的话，后面时间还是不准

```

# 修改时区

>###### https://www.jianshu.com/p/b67f3f3c6926

```
timedatectl list-timezones |grep Shanghai    #查找中国时区的完整名称
timedatectl set-timezone Asia/Shanghai    #其他时区以此类推
timedatectl status
#设定时间
timedatectl set-time 15:58:30
#或者直接手动创建软链接
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

#暴力方法，直接复制文件到localtime
```



pwd

cat

# 图形界面

#### gnome

##### 安装

>https://www.jianshu.com/p/b8ac516074c7

##### gnome-shell

登录之后，桌面卡住时找到pid杀死即可。系统会自动重启gnome，类似windows的explorer。

##### 开机时图形界面卡死

```
命令行手动startx，启动图形界面。确保开启了sshd自启，不然ssh都连接不上。
ps -ef|grep gnome-shell
kill -9 
```

##### 开机进入命令行

```
开机进入命令行
systemctl set-default multi-user.target
开机进入图像界面
systemctl set-default graphical.target
```



# 网络

##### 查看网卡

```
netstat -i
ip addr
route -n
ip route show 
traceroute www.baidu.com -s 100
netstat -r
```

##### 查看端口占用

```
lsof -i:8080
```

###### netstat -lnpt

>*PS:centos7默认没有 netstat 命令，需要安装 net-tools 工具，yum install -y net-tools*

```
#检查端口被哪个进程占用,其中l代表展示listen的端口，p代表展示对应的process，t代表tcp（使用u就只查看udp），n代表not resolve names，不懂什么意思。
netstat -lnpt |grep 5672
#查看进程的详细信息
ps 6832
#中止进程
kill -9 6832
```

##### 配置网络

##### ifconfig管理网卡和网络

```
sudo ifconfig ens33:0  192.169.0.100 up
sudo ifconfig ens33:0  down
```

##### ssh连接与ftp

ssh连接和ftp使用系统中已创建的用户名及密码，加上ip和端口实现访问。

ssh默认在22，ftp默认在21,sftp，默认在22端口。修改ssh在etc/ssh/ssh_config中修改。

notepad配置好sftp插件，填写参数即可上传。

##### 网卡

brctl命令

##### 参数说明和示例

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

##### iptables



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









##### 服务器之间copy文件

yum install ssh-clients安装scp命令

scp /tm  @172.18.65.22:/root/将当前电脑tm拷贝到目标主机root目录下。随后输入密码即可。

例如：scp workers  root@hbase2:反单引号pwd反单引号

反单引号：1左边那个`



##### rsync

rsync     -rvl  /tmp   root@hbae:/tmp

##### xsync

复制到所有节点相同目录下。

## 网卡文件

>详细的参数解析：https://blog.csdn.net/zhangchao_cn/article/details/85246558
>
>网卡文件一般在/etc/sysconfig/network-scripts下边，其中的ens33或ensxx表示其默认网卡，我们可以在其中配置网卡信息。

```
#其中关键的参数
NM_CONTROLLED=no #这个参数表示不使用Networkmanager管理网络，使用此配置文件。当我们关闭NetworkManager后，必须修改此选项。
ONBOOT=yes #是否开机自启
BOOTPROTO=static #是dhcp还是静态static
```

## 配置一个静态ip网卡

```
参数解释
DEVICE     接口名（设备,网卡）
USERCTL    [yes|no]（非root用户是否可以控制该设备）
BOOTPROTO  IP的配置方法[none|static|bootp|dhcp]（引导时不使用协议|静态分配IP|BOOTP协议|DHCP协议）
HWADDR     MAC地址   
ONBOOT     系统启动的时候网络接口是否有效（yes/no）   
TYPE       网络类型（通常是Ethemet）   
NETMASK    网络掩码   
IPADDR     IP地址   
IPV6INIT   IPV6是否有效（yes/no）   
GATEWAY    默认网关IP地址
BROADCAST  广播地址
NETWORK    网络地址

```

```
 #########start设置静态地址例子 这一段在文件中一般默认就有#########
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
#BOOTPROTO="dhcp"
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="ens33"
UUID="ac9b66bf-74fb-4bda-b89f-c66ff84c9571"
DEVICE="ens33"

 
##static assignment 这一段是需要自己修改的，后边的静态ip配置也需要自己追加，特别是新安装的centos系统，没有这些参数的模板##
NM_CONTROLLED=no #表示该接口将通过该配置文件进行设置，而不是通过网络管理器进行管理
ONBOOT=yes #开机启动

BOOTPROTO=static #静态IP
IPADDR=192.168.59.134 #本机地址
NETMASK=255.255.255.0 #子网掩码
GATEWAY=192.168.59.2 #默认网关
DNS1=8.8.8.8
DNS2=8.8.4.4
 
#########end设置静态地址例子#########
service network restart
```



# 服务进程

```
#
ps -aux|grep "hadoop"
#e表示所有进程，f表示完整信息
ps -ef|grep "hadoop"
```



# 文件

##### 查看磁盘和文件夹

```
#disk free，磁盘
df -h
#disk usage，文件目录
du -h
```



```
1、 统计当前文件夹下文件的个数

　　ls -l |grep "^-"|wc -l

2、 统计当前文件夹下目录的个数

　　ls -l |grep "^d"|wc -l

3、统计当前文件夹下文件的个数，包括子文件夹里的 

　　ls -lR|grep "^-"|wc -l

4、统计文件夹下目录的个数，包括子文件夹里的

　　ls -lR|grep "^d"|wc -l

grep "^-" 

　　这里将长列表输出信息过滤一部分，只保留一般文件，如果只保留目录就是 ^d

wc -l 

　　统计输出信息的行数，因为已经过滤得只剩一般文件了，所以统计结果就是一般文件信息的行数，又由于一行信息对应一个文件，所以也就是文件的个数。
```

```
遍历文件夹删除特定文件，清除lisa的所有存储，保留report.json。
#! /bin/bash
function read_dir(){
for file in `ls $1` #注意此处这是两个反引号，表示运行系统命令
do
 if [ -d $1"/"$file ] #注意此处之间一定要加上空格，否则会报错
 then
 read_dir $1"/"$file
 else
 if [ $file != 'report.json'];then
 rm -rf $1"/"$file
 echo $1"/"$file #在此处处理文件即可
 fi
done
} 
#读取第一个参数
read_dir $1
```



##### 常用目录

/etc/hosts

/etc/sysconfig/network

```
#centos7
hostnamectl查看信息
/etc/hostname修改主机名
systemctl restart systemd-hostnamed重启服务或重启电脑
```

##### tar解压缩

```
tar
-c: 建立压缩档案,在解压的包后面使用，-c /解压的文件名
-x：解压
-t：查看内容
-r：向压缩归档文件末尾追加文件
-u：更新原压缩包中的文件

这五个是独立的命令，压缩解压都要用到其中一个，可以和别的命令连用但只能用其中一个。下面的参数是根据需要在压缩或解压档案时可选的。

-z：有gzip属性的
-j：有bz2属性的
-Z：有compress属性的
-v：显示所有过程
-O：将文件解开到标准输出

下面的参数-f是必须的

-f: 使用档案名字，切记，这个参数是最后一个参数，后面只能接档案名。
```



# problem

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

如果还不行，删除know_hosts即可。

##### ssh秘钥公钥问题

将自己的公钥给别人，自己可以登陆别人。

使用ssh-keygen -t  rsa生成秘钥和公钥

使用ssh-copy-id ip将自己的秘钥放到目标主机的authorized_hosts

中。随后使用ssh ip即可无密码访问。

>authorized_hosts，存储哪些主机在我这注册了，我拥有它的公钥。
>
>known_hosts,存储连接过的机器的公钥，当对用同一host对比公钥不同时，发出警告，防止黑客攻击。

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

sudo vi /etc/network/interfaces

ip addr查看网卡

单节点配置文件

##### Error: requested datatype primary not available

```
yum clean all
rm -rf /var/cache/yum
yum list iostat
```







