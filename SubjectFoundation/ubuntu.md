# ubuntu系统

## 增加磁盘

### 基本命令

分区，格式化，挂载
​fdisk  -l  查看分区  n,p,w
mkfs  -t ext4  /dev/sdb1
mkdir /home/yoki/sdb1
mount  /dev/sdb1  /home/yoki/sdb1
gedit etc/fstab
/deb/sdb1  /home/yoki/sdb1 ext4  defaults 0 1
​
​df -H 查看容量，挂载
blkid  查看
sudo chmod -R 4755 /home/sda1
sudo chmod -R 777 /home/sda1
chmod使用的数字的意思： 读（r=4），写（w=2），执行（x=1）可读可写为4+2=6 依次内存 755表示的是文件所有者权限7（三者权限之和），与所有者同组用户5（读+执行），其他用户同前一个5，这里的4的意思是（其他）用户执行拥有所有者相同的文件权限（对于其他要使用的文件）
​

### 其他
ctrl+c 推出当前
top 查看进程
kill pid清楚

###  步骤
扩充磁盘使用此命令，注意磁盘内的数据丢失
VBoxManage list hdds
VBoxManage modifyhd 0bd9c696-1735-48ce-81cf-04e9f64c2418 –resize 51200 
新建磁盘直接软件新建
​1虚拟机新建磁盘，真机插入磁盘。
2分区
3格式化
4挂载及修改挂载文件



## VirtualBox迁移虚拟机
1将vdi赋值到对应目录
2到virtualbox软件的安装目录，执行VBoxManage internalcommands sethduuid “G:\新建文件夹\ubuntu16-02\ubuntu16-02.vdi” 
3进入virtualbox进入设置，存储，删除旧的磁盘，添加新磁盘。
或将原虚拟机复制到新目录，控制，注册导入。

## 文件

解压 tar -zxvf filename
unzip
cd-  跳回上次目录
df -h 查看磁盘容量
touch 1.txt创建文件
​ls-lrt 查看软连接

ls -l   查看软连接

### 查找文件
whereis 文件名   模糊查找
find /-name 文件名
locate  文件名

cat文件名

## 网络及进程

netstat -tlnp 查看服务端口号

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

## ufw
ufw default 
ufw enable

ufw status numbered
ufw delete 数字

ufw allow ip

ufw deny ip

ufw allow from ip to any port 端口号

## 修改ssh端口并开放

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

sudo   vi /etc/ssh/sshd_config   

重启ssh服务
service sshd restart   

查看更改是否生效
netstat -tlnp  

防火墙生效并开机启动
ufw enable     

检查设置规则
sudo ufw status

### vultr搭建

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

## ssh服务ip限制
etc/ssh/sshd_config配置端口
在etc/hosts.allow  /etc/hosts.deny
控制ip访问服务。
grep sshd /var/log/auth.log | less

### 安装shadowsocks

wget –no-check-certificate  https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks.sh

chmod +x shadowsocks.sh

./shadowsocks.sh 2>&1 | tee shadowsocks.log

### 使用端设置

ip，端口，密码，加密方式。

协议：origin，混淆：plain.

#### 忘记密码设置等

登录服务器，在/etc/shadowsocks.json或etc/shadowsocks/shadowsacks.json中有信息。

## bbr服务

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



### 查看日志

who /var/log/wtmp
tail/more    /var/log/secure

### ssh problem
配置ufw特定ip访问特定端口，总是配完了无法访问，是因为实际ip不是ipconfig的显示ip。可以查看登录记录，知道实际ip.

### 其他

cat /proc/version
lsb_release -a
uname -a  架构
dpkg  多少位
getconf 系统变量配置
arch 架构



### can not get lock
ps aux|grep apt
sudo kill -9 pid

##### 截屏黑屏

关闭virtualbox3D加速

# vmware

- win10 1903安装黑屏
  win10内置沙盒与vmware冲突。
  解决，升级vmware。

# git

- 基本
  git clone git://远程Git库地址  filename

