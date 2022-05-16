##### 虚拟机上传到vSphere client,esix上

##### 打包上传

打包方式两种：

ova打包整个

ovf配置信息+vmdk

这种方式错误错误很多，不建议

##### vmware workstation上传

将虚拟机关机，选中虚拟机。先要更改兼容版本，点击兼容性管理，选择合适版本，然后克隆此虚拟机。此后，点击虚拟机->管理->上传,输入vSphere的主机地址及用户和密码。点击上传克隆的兼容版本即可。

#### problem

>无法连接外部网络，连宿主机都无法ping通，进入vmware->编辑->网络编辑器->更改设置->恢复默认设置.

# 创建虚拟机

>https://blog.csdn.net/u010483330/article/details/122923612
>
>我们可以将iso文件上传到他的存储介质中，然后创建虚拟机时选中存储介质对应的文件即可。
>
>在CD/DVD驱动器中指定数据存储ISO文件
>
>开机后选择 Test this media & install Centos,然后跟随指引。
>
>Vsphere 6.5

## 上传文件到vsphere存储

>vspherer首页有四栏，分别是主机与集群、虚拟机与集群、存储、网络
>
>点击存储，选中一个存储介质，然后点击查看摘要中的剩余空间，再点击文件，可以查看介质中存储了那些文件，我们可以看到创建的虚拟机磁盘存储在其中。然后点击上载文件，将本地的iso文件上载到vsphere的存储中。

## 配置虚拟机iso

>右键创建虚拟机中直接选择创建，就是第一个选项，不使用模板和复制。在硬件配置中，选中CD/DVD驱动器，然后选择数据存储ISO文件，找到我们之前上传的iso文件。最后勾选开机时链接此iso文件。

## 开启虚拟机安装iso

>这里边基本都是默认操作，但是分区需要操作。
>
>我们点击带警告标记的INSTALLATION DESTINATION(Custom partitioning selected)，点击I will configure partitioning，点击 click here to create them automatically，然后将其中的/home分区删除、/分区也删除，再自己创建一个/分区，最后保存。
>
>我们也可以在这一步配置hostname，然后就可以点击done，让iso开始安装了。



# vmware网络配置

>vmware->编辑->网卡设置，这是用于在windows宿主机上创建一张网卡，这张网卡的ip、网关和子网掩码可以自行指定。然后我们我们的虚拟机在该网卡规定的网段内设置ip，若开启dhcp，他就会自动获取ip。如果未开启，可以手动设置该网段的一个合法ip，配置网关和掩码后，就可以与宿主机通信，配置好dns后就可以连通外界，连接互联网。
>
>https://zhuanlan.zhihu.com/p/130984945