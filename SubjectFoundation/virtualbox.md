# problem

### 桥接模式设置后无网络

桥接模式，虚拟机和宿主机通过某一网卡建立通道，虚拟机拥有独立ip与真机无异。点击宿主机右下角网络，进入网络与共享中心查看当前使用的网络对应网卡。在虚拟机中选择桥接模式，并选择正确的网卡。

### 添加共享文件夹

在设置-设备中开启共享文件夹，还要使用此命令sudo` `usermod` `-a -G vboxsf 用户名才可。

### 创建virtualboxclient  COM对象失败，没有注册类

几天前可用，今天显示此错误。

换成6.x版本有：please try reinstalling virtualbox where:supr3harden edwinres pawn what 5...的错误.

解决：卸载 Microsoft visual C++ 2015-2019 Redistributable。

这几天之间只安装了几个软件，控制面板排序方式按安装时间，把他们都卸载即可。或者还原点。c++的插件为vmware remote client携带的，将其及virtualbox卸载后，重启机器，在安装virtualbox5.2.44即可。

##### 无法创建任务的错误

此错误由于遁地模拟器和virtualbox冲突，只能开一个。