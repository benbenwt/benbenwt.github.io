### ClassifyIotDevice

##### pca降维后无列名原理

```
pca原理 :https://www.zhihu.com/question/38417101/answer/94338598   ok
教程:https://blog.csdn.net/missionnn/article/details/121191490
```

##### json包含的原始 信息

```
ip:
src_ip,dst_ip,是否局域网，地区，运营商，proto，ttl，rb,mf,df,
tcp:
src_port,dst_port,flags,windows_size,urgent_pointer,payload_len
```

##### 构造的特征

```
http://blog.sina.com.cn/s/blog_7fc4c23d0101fa0z.html
var1 窗口大小60
var2 移动距离10
Feature Name				Description
1，	包的个数和尺寸（19）
前向数据包总个数
反向的数据包总个数
正向包的总大小，caplen
反向包的总大小
正向包的最小尺寸，caplen
正向包的最大尺寸
正向包的平均大小，caplen
正向包的标准偏差大小
反向包的最小尺寸
反向包的最大尺寸，caplen
反向包的平均大小
反向包的标准偏差大小

每秒转发数据包数
每秒反向数据包数
数据包的最小长度
数据包的最大长度
数据包的平均长度
数据包的标准偏差长度
数据包的方差长度

2，包的发送间隔时间(14)
发送的两个数据包之间的平均时间  
发送的两个数据包之间的标准偏差时间
发送的两个数据包之间的最长时间
发送的两个数据包之间的最短时间

前向发送的两个数据包之间的最短时间
前向发送的两个数据包之间的最长时间
前向发送的两个数据包之间的平均时间
前向发送的两个数据包之间的标准偏差时间
前向发送的两个数据包之间的总时间

反向发送的两个数据包之间的最短时间
反向发送的两个数据包之间的最大时间
反向发送的两个数据包之间的平均时间
反向发送的两个数据包之间的标准偏差时间
反向发送的两个数据包之间的总时间


3，包的标志位信息(16)
每秒字节数,caplen/time
每秒包数 
用于前向发送的标头的总字节数
用于反向发送的标头的总字节数

在正向传输的数据包中设置 PSH 标志的次数（UDP 为 0）,push
在反向传输的数据包中设置 PSH 标志的次数（UDP 为 0）
在正向传输的数据包中设置 URG 标志的次数（UDP 为 0）,urg
在反向传输的数据包中设置 URG 标志的次数（UDP 为 0）

带有 FIN 的数据包数,fin
带有 SYN 的数据包数,syn
带有 RST 的数据包数，reset
PUSH 数据包数,push
带有 ACK 的数据包数,ack
带有 URG 的数据包数,urg
带有 CWR 的数据包数,cwr
带有 ECN 的数据包数,ecn



4包的dns信息、payload等信息
提交模块说明-2021-12-17.docx
这是一条dns示例数据
{"time":1637832980441434000,"caplen":92,"filename":"/opt/data/cap/ceshi2/xiao-01.pcap","data":[{"eth":{"dst_mac":"50:DA:00:42:10:B4","src_mac":"B8:C6:AA:79:36:8D","type":2048}},{"ip":{"v":4,"hdr_len":5,"len":78,"id":46983,"flag":16384,"flags":{"rb":0,"df":1,"mf":0,"offset":0},"ttl":64,"proto":17,"checksum":56255,"src_ip":"192.168.1.203","dst_ip":"114.114.114.114","src_country":"局域网","src_area":"对方和您在同一内部网","dst_country":"江苏省南京市","dst_area":"南京信风网络科技有限公司GreatbitDNS服务器"}},{"udp":{"src_port":28682,"dst_port":53,"len":58,"sum":17567,"payload_len":50}},{"dns":{"id":40342,"flag":256,"flags":{"response":0,"opcode":0,"authoritative":0,"truncated":0,"recdesired":1,"recavail":0,"z":0,"authenticated":0,"checkdisable":0,"rcode":0},"qry_count":1,"resp_count":0,"auth_count":0,"addr_count":0,"qry":[{"name":"spoon.skyworthdigital.sjxrbj.com","type":28,"class":1}]}}],"payload":"","proto":"eth.ip.udp.dns","src_mac":"B8:C6:AA:79:36:8D","dst_mac":"50:DA:00:42:10:B4","src_ip":"192.168.1.203","dst_ip":"114.114.114.114","src_country":"局域网","src_area":"对方和您在同一内部网","dst_country":"江苏省南京市","dst_area":"南京信风网络科技有限公司GreatbitDNS服务器","src_port":28682,"dst_port":53,"payload_len":50}
```

```
读取一个json文件，遍历单条记录。统计特定时间段内的数据。统计单个维度的计数，计数，结束阶段计算。统计单个维度的数值指标（均值，方差），添加值，结束阶段计算。其他类型的，添加对象或值，结束阶段计算。添加和结束的计算函数暴露即可。
```



### CICFlowMeter

>https://github.com/ahlashkari/CICFlowMeter
>
>Dataset: https://www.unb.ca/cic/datasets/ids-2017.html
>
>Dataset and pcap: http://205.174.165.80/CICDataset/CIC-IDS-2017/Dataset/PCAPs/
>
>项目基本结构：
>
>使用自定义的类调用jnetpcap读取pcap，再进行pcap流的划分，然后进行统计并插入csv。
>
>ReadPcapFileWorker 读取pcap
>
>BasicPacketInfo 单个packet的封装类
>
>BasicFlow 
>
>FlowGenerator  封装的pcap流的类
>
>InsertCsvRow csv插入
>
>jnetpcap-1.4.1.jar通过project structure添加到项目，不使用maven中的索引。jnetpcap通过调用winpcap或libpcap(linux)实现pcap的读写和解析，将对应的软件(.exe,.so)到平台上配置好,再引用jnetpcap-1.4.1.jar即可。

```
关注这几个类ReadPcapFileWorker,PacketReader,BasicFlow,FlowGenerator
PacketReader负责从pcap读取信息。ReadPcapFileWorker负责将PacketReader读取的信息写入FlowGenerator中，FlowGenerator的currentFlows(HashTable<String,BasicFlow basicFlow>)负责存储多个BasicFlow。basicFlow负责统计单个流的属性。
如果重写维度，要重写FlowFeature枚举的头部和DumpFlowBasedFeatures提取的维度信息。
维度组成包的大小，时间，（max,min,mean,std）
数据集网站:https://www.unb.ca/cic/datasets/ids-2017.html
数据集及pcap下载连接:http://205.174.165.80/CICDataset/CIC-IDS-2017/Dataset/PCAPs/
```

##### jnetpcap安装

```
/usr/lib/jnetpcap-libpcap.so
```

##### disspcap安装

```
https://disspcap.readthedocs.io/en/latest/installation.html#install-build-requirements
PYPI网站
```

```
ReadPcapFileWorker 读取pcap
InsertCsvRow csv插入
FlowGenerator 
```

### 可捕获的恶意软件和pcap

```
find -name "*.pcap"  -size +3M
./bcc3050e-a401-4aab-92f2-527ee522154d/capture.pcap
./017835ca-787a-41e1-bc1e-d88c882b318b/capture.pcap
./9954a0b1-138f-4e5b-96c6-c654d671555e/capture.pcap
./27c2952b-b4ab-479f-b0d4-9188fbf10ed1/capture.pcap
./f9f29621-fd75-4317-a603-d0275891af7c/capture.pcap
./a8a86de5-ad53-473c-b66f-46a8e835995c/capture.pcap
```

```
find -name "*.pcap"  -size +4M
```

##### 重启Pcap的web服务

```
#!/bin/bash
export HADOOP_HOME=/root/module/hadoop-3.1.4
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
PATH=$PATH:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/bin:/sbin
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib

while :
do
(ps -aux|grep controller.py|grep -v 'grep'|awk  '{print $2}';  )|xargs kill -9
(ps -aux|grep controller.py|grep -v 'grep'|awk  '{print $2}';  )
nohup python controller.py  &
sleep 1800
done
```

### wireshark分割

```
https://jingyan.baidu.com/article/63f2362854be2e0208ab3d20.html
cd /pathtoWireshark
editcap -c  分割份数  待分割文件目录\文件名.pcap 分割后存储的文件目录\文件名.pcap
```

### pcap恶意

```
https://www.unb.ca/cic/datasets/ids-2017.html
pcap index:https://mcfp.felk.cvut.cz/publicDatasets/
MACCDC比赛中的流量:https://www.netresec.com/?page=MACCDC
netresec网站: https://www.netresec.com/?page=PcapFiles
相关博客: https://www.cnblogs.com/bonelee/p/11379587.html

说几个我经常用的，免费的：
1.  Malware  Traffic  Analysis:  http://www.malware-traffic-analysis.net/2018/index.html    这个网站每天更新，主要是欧美地区的新鲜流行木马样本，基本上当天更新的马都很新～
2.Virus  Bay：  https://beta.virusbay.io/    这个算是社区贡献吧

收费的：
1.Virustotal  Intelligence：https://www.virustotal.com/intelligence  这个是VT提供的，你所在的公司要付钱给VT，这样你可以去根据HASH和自定义YARA去找样本。
2.Abusix：恶意垃圾邮件提供商，每天提供大量的新鲜的垃圾邮件，80%内容是恶意的。
3.Support  Intelligence：收集各大反病毒厂商收集的样本，然后转手卖给各大IOC提取商～
4.Lexsi：同Support  Intelligence

 

 

你好，比如说，我想分析利用MS17_010漏洞的病毒，又或者我想分析某款病毒分变种，有没有什么网站能够跟你条件来查样本呢？
网站能够根据条件来查样本，一般你需要去各大在线沙盘的网站，例如  Hybird-Analysis，根据Tag来找，找到了根据HASH来找样本

曾经也遇到楼主的问题，收集了一些国外的样本下载网站：
1）https://www.hybrid-analysis.com/    这个网站可以下载，但是需要注册账号，个人注册需要提交三个以上博客或者原创技术文章链接，使用企业邮箱申请的通过的比较快一些

2）https://app.any.run/    这个网站是一个免费沙箱，可以浏览其他人跑的样本结果，也可以下载样本，不需要注册账号就能下载，注册也是免费的！

3）http://vxvault.net/ViriList.php  这个！！没下载过

4）http://malc0de.com/database/  每天更新最新样本

最后老外推荐的样本资源：https://zeltser.com/malware-sample-sources/
```

