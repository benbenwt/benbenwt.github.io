| *compute the checksum of your file ...* |          *compare with*          |                  |                      |               |
| :-------------------------------------: | :------------------------------: | :--------------: | -------------------- | ------------- |
|                                         |             Windows              |      Linux       | Mac                  |               |
|                  SHA-1                  |  certUtil -hashfile *file* SHA1  |  sha1sum *file*  | shasum -a 1 *file*   | *file*.sha1   |
|                 SHA-256                 | certUtil -hashfile *file* SHA256 | sha256sum *file* | shasum -a 256 *file* | *file*.sha256 |
|                 SHA-512                 | certUtil -hashfile *file* SHA512 | sha512sum *file* | shasum -a 512 *file* | *file*.sha512 |
|                   MD5                   |  certUtil -hashfile *file* MD5   |  md5sum *file*   | md5 *file*           | *file*.md5    |

# ubuntu

##### 换源

```
RUN cp /etc/apt/sources.list /etc/apt/sources.list.bak  && \
echo 'deb http://mirrors.ustc.edu.cn/debian stable main contrib non-free' >>/etc/apt/sources.list  && \
echo 'deb http://mirrors.ustc.edu.cn/debian stable-updates main contrib non-free' >>/etc/apt/sources.list
```



### vmware

```
无法复制文件
卸载重装open-vm-tools
sudo apt-get autoremove open-vm-tools
sudo apt-get install open-vm-tools
sudo apt-get install open-vm-tools-desktop
使用vmware管理工具安装vmtools。
```

### shell

```
跳过请输入某个键的交互命令
y|qbittorrent-nox
yes|apt-get install test
```

### 系统

##### 权限管理

```
vim /etc/sudoers
```

##### crontab

```
#开启crontab日志
vim /etc/rsyslog.d/50-default.conf
取消cron*注释
service rsyslog  restart
service cron restart
```



cat /proc/version
lsb_release -a
uname -a  架构
dpkg  多少位
getconf 系统变量配置
arch 架构

### 网络及服务

```
ps -aux|grep qemu|awk '{print $2}'|xargs kill -9
```

```
#ens33 查看不到了
ifconfig ens33  up
sudo /sbin/dhclient
sudo systemctl restart network-manager.service
```



netstat -tlnp 查看服务端口号

ss -ntl

service nginx reload 重新加载配置
ps -ef | grep redis
ctrl+z 推出当前进程
pwd 打印工作路径，print work directory
mv redis-5.0.5 /usr/local/redis

make test
make install
redis-server
ps -ef
ps -aux|grep redis 查看端口
netstat -nlt|grep 6379
weget  url  下载文件
exit
wget http://download.redis.io/releases/reids-5.0.5.tar.gz
​curl

##### ufw

ufw default 
ufw enable

ufw status numbered
ufw delete 数字

ufw allow ip

ufw deny ip

ufw allow from ip to any port 端口号





### 文件管理

##### 压缩

```
zip -r lisa.tar lisa
unzip lisa.tar
tar -zxvf lisa.tar
tar -cvf lisa.tar lisa
```



##### trash管理

```
所在路径
/home/yourname/.local/share/Trash/files
对应挂载磁盘的垃圾箱路径./.Trash*/files
```



```
查看文件夹容量
du -ah --max-depth=1 
du -sh
按大小排序
ls -lS
ls -lSr
ls  -lH
```



### 查看日志

who /var/log/wtmp
tail/more    /var/log/secure

##### 查找文件

whereis 文件名   模糊查找
find /-name 文件名
locate  文件名

cat文件名

##### 权限管理

chmod使用的数字的意思： 读（r=4），写（w=2），执行（x=1）可读可写为4+2=6 依次内存 755表示的是文件所有者权限7（三者权限之和），与所有者同组用户5（读+执行），其他用户同前一个5，这里的4的意思是（其他）用户执行拥有所有者相同的文件权限（对于其他要使用的文件）

##### 文件权限

参考博客：https://blog.csdn.net/BjarneCpp/article/details/79912495

一个文件可以由10位字符表示，0位是表示文件类型，123位表示user用户权限，456位表示用户组的权限，789位表示其他用户的权限。r表示读，w表示写，x表示写。-在0位表示普通文件，在其他位表示无此权限。d在0位表示文件夹类型。

如下例子表示，该普通文件，用户可读可写可执行。用户组可读,可写。其他只可读。

-rwxrw-r--

chmod使用数字表示权限，1为执行，2为写，4为读。共三位表示用户，用户组，其他用户。最高权限为7，即三者之和。

用法如下,将上边的10位例子写成chmod形式，如下：chmod   731  1.txt

chown使用字母修改权限：chmod g+w 1.txt,为用户组添加1.txt的写权限。g表示用户组，u表示用户，

**用户与组**

cat /etc/group

cat /etc/shadow

**rarlab**

```
#此版本centos7和ubuntu16.04无问题
wget http://www.rarlab.com/rar/rarlinux-x64-5.3.0.tar.gz
```

##### 软连接

```
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

find

```
find -size -2kb
向下取整
find ./ -name '.py' -exec rm -rf {} ;
find ./ -name "*" -type f -size 0c | xargs -n 1 rm -f
find ./ |wc -l
```

linux计算hash

```
md5sum
sha1sum       
sha256sum     
sha512sum    
shasum        
sha224sum     
sha384sum
shasum -a 512 -c 
```

# 



误解压文件，导致多了很多小文件。撤销操作

```
tar -tf 误解压文件 | xargs rm -rf
jar -tf 误解压文件 | xargs rm -rf
```

tar

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

### 软件管理

##### **apt常用命令**

```
#查看可用版本
apt-cache search ffmpeg | grep Version
apt-get install qbittorrent-nox=4.3
```





##### apt

sudo apt-get update刷新仓库的索引

apt源文件:/etc/apt/sources.list

```
apt-get update
apt-get upgrade
apt-get source git
apt-get  remove name
apt-get install --reinstall zlibc zliblg zliblg-dev
#配置源
sudo vim /etc/apt/sources.list
deb http://mirrors.ustc.edu.cn/ubuntu/ xenial main restricted universe multiverse
deb http://mirrors.ustc.edu.cn/ubuntu/ xenial-security main restricted universe multiverse
deb http://mirrors.ustc.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse
deb http://mirrors.ustc.edu.cn/ubuntu/ xenial-proposed main restricted universe multiverse
deb http://mirrors.ustc.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
deb-src http://mirrors.ustc.edu.cn/ubuntu/ xenial main restricted universe multiverse
deb-src http://mirrors.ustc.edu.cn/ubuntu/ xenial-security main restricted universe multiverse
deb-src http://mirrors.ustc.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse
deb-src http://mirrors.ustc.edu.cn/ubuntu/ xenial-proposed main restricted universe multiverse
deb-src http://mirrors.ustc.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
```

给git配置代理

```
git config --global http.proxy 'socks5://127.0.0.1：1080'
git config --global https.proxy 'socks5://127.0.0.1：1080'
git config --global http.proxy 'http://127.0.0.1：1080'
git config --global https.proxy 'http://127.0.0.1：1080'
```

### 磁盘管理

uname -a

cat  /etc/issue ，适用所有

cat /proc/version，使用所有

cat /etc/redhat-release，只适用于RedHat，SUSE，Debian等发行版本

lsb_release -a 所有发行版本

分区，格式化，挂载
​fdisk  -l  查看分区  n,p,w

**删除分区**

```
fdisk /dev/sda1
m
d
w
```

```
#格式化，并载磁盘到文件系统
mkfs  -t ext4  /dev/sdb1
mkdir /home/yoki/sdb1
mount  /dev/sdb1  /home/yoki/sdb1
#修改，使得开机自动挂载
gedit etc/fstab
/deb/sdb1  /home/yoki/sdb1 ext4  defaults 0 1
```



df -H 查看容量，挂载
blkid  查看
sudo chmod -R 4755 /home/sda1
sudo chmod -R 777 /home/sda1





### vultr搭建

```
#安装sshd服务。
sudo apt install openssh-serve
```



vultr使用心得
http://blog.51cto.com/13940125/2165848
1服务器搭建
购买服务器
检测服务器（能否ping通？,一把都开启了22，port 22开启？） 
摧毁服务器or进入第二步
2x贝壳连接
连接服务器(使用vultr提供的root，和密码)
(yum install git -y)
ss搭建
   git clone   https://github.com/Flyzy2005/ 

ss-fly
   ss-fly/ss-fly.sh -i password 1024
bbr加速
    ss-fly/ss-fly.sh -bbr
（firewall set）
3酸酸连接
切换为系统代理

ssh使用用户名和密码登录，共享数据，登录主机。
ssl为了数据加密和交流。
问题多出在服务器搭建。
https://medium.com/@antiless.dev/超详细-vultr-vps-搭建-ss-新手图文指导教程-4d6b33e411b6
小站:![img](file:///C:\Users\guo\AppData\Roaming\Tencent\QQTempSys\`7_{~]GF$3{MOQ4V_}PH]YC.png)www.flyzy2005.com

##### ssh服务ip限制
etc/ssh/sshd_config配置端口
在etc/hosts.allow  /etc/hosts.deny
控制ip访问服务。
grep sshd /var/log/auth.log | less

##### 修改sshd端口并开放

>sshd为服务器端程序
>
>ssh为客户端程序

修改ssh客户端默认连接端口

```
vim /etc/ssh/sshd_config
将port改为希望默认请求的ssh端口
```



https://medium.com/@antiless.dev/%E8%B6%85%E8%AF%A6%E7%BB%86-vultr-vps-%E6%90%AD%E5%BB%BA-ss-%E6%96%B0%E6%89%8B%E5%9B%BE%E6%96%87%E6%8C%87%E5%AF%BC%E6%95%99%E7%A8%8B-4d6b33e411b6

查看是否安装过

ufwsudo dpkg --get-selections | grep ufw
安装

ufwsudo apt-get install ufw
默认规则为拒绝

ufw default deny
防火墙50022端口的tcp访问

sudo ufw allow 50022
vnc服务端口

sudo ufw allow 5901
sudo ufw allow 5902
将22端口改为50022

##### sudo   vi /etc/ssh/sshd_config   

注意配置远程客户端修改sshd_config即可。找到port所在行取消注释，修改端口号即可。

##### 重启ssh服务

##### service sshd restart   

##### 查看更改是否生效

##### netstat -tlnp  

防火墙生效并开机启动
ufw enable     

检查设置规则
sudo ufw status

##### 服务端安装shadowsocks

wget –no-check-certificate  https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks.sh

chmod +x shadowsocks.sh

./shadowsocks.sh 2>&1 | tee shadowsocks.log

##### 客户端ssr设置

linux使用electron-ssr界面插件，进入该软件后会自动下载python-ssr。也可自己下载然后指定根目录。

https://github.com/qingshuisiyuan/electron-ssr-backu

#### 忘记密码设置等

登录服务器，一般在/etc/shadowsocks.json或etc/shadowsocks/shadowsacks.json中有信息。

##### bbr服务

>bbr时谷歌推出的一种提高网络利用率的算法，改变了tcp传统的拥塞控制策略。

ubuntu4.9以后内置bbr算法，uname -a查看内核版本。

若过低，下载内核升级。然后修改sysct.conf,命令如下。

```
sudo bash -c 'echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf'
sudo bash -c 'echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf'
```

sysctl -p保存设置

sysctl net.ipv4.tcp_available_congestion_control查看使用的算法。

lsmod |grep bbr查看是否开启bbr.

### problem

##### can not get lock
ps aux|grep apt
sudo kill -9 pid

##### 截屏黑屏

关闭virtualbox3D加速

***libappindicator3.so.1()(64bit) 被 google-chrome-stable-81.0.4044.138-1.x86_64 需要\***

已安装此libappindicator3.so.1依赖，仍有此提示.

```
#查看下层的依赖，将其安装即可
yum provides */libappindicator3.so.1
```

##### ssh problem

配置ufw特定ip访问特定端口，总是配完了无法访问，是因为实际ip不是ipconfig的显示ip。可以查看登录记录，知道实际ip.

没有apt命令

```
从正常机器上复制以下文件，即可使用apt，*表示正则，可用find命令查看对应文件。
/usr/bin/apt
/usr/lib//libapt-*
教程：https://blog.csdn.net/w294954902/article/details/84998797
```

