
flink在编码层面与原本的区别，把核心数据和操作抽离出来了，
包括keyBy，window，Aggregate，Process。
对数据处理流程进行了概念的划分，方便理清逻辑。代码的核心逻辑表达和实现细节进行了划分，比如聚合的细节是在单独的函数，而keyBy,window,aggregate相当于语义层，可以清楚地看到计算了什么，不会与细节混在一起。当需要扩展、兼容时，可以快速确认要修改的对应该概念，找到修改位置，不用到繁杂大量代码中取寻找应该修改的那一行。

编码分层，核心逻辑层-细节概念层(不涉及细节，直观控制和观察整体流程)，概念划分（清晰的将整个流程拆分成概念，由概念类operator控制细节，组成整体流程）

main(source.KeyBy().process())

ifpresent(consumer) ,consumer producer
spring复习

subline快捷键：
ctrl shift p
preferences 设置插件快捷键

彩云：https://www.xn--nosa.com/user

```
#summary
java,springboot,springcloud,maven,nginx,vue,elasticsearch,mysql,docker,git,shell。zk,kafka,flume,logstash,hdfs,mr,hive,hbase,spark,kylin,superset。
```

```
Iaas，提供存储，计算，内存资源。
Paas，提供部署运行软件的平台。
Saas，提供软件服务。
```

```
http://caiyun.fuligou.tk/
```

```
hdfs :hive:Ma*y 22 20:40  copy14  copy 119
```

```
group关闭后，无法重置offset
```

```
kafka乱码
来源此处中文乱码：LOC-['u7231u5c14u5170', 'u7f8eu56fd']
python的producer加入,json.dump(result,ensure_ascii=false)
java的consumer用utf解码，并用StringEscapeUtil解码字符串.
```

```
SELECT FROM_UNIXTIME(sm_time,'%Y') as years FROM sentiment GROUP BY years
```



```
apiKey :3103738dd023eb285405409cd17cf817b6207cf654ee5dac47a35ea538aa39fd
#malware source
https://zeltser.com/malware-sample-sources/
爬虫证书
keytool -list -keystore cacerts -alias vbooking
keytool -import -alias abc -keystore cacerts -file D://abc.cer
changeit
https://tracker.virusshare.com:7000/torrents/VirusShare_00000.zip.torrent?3B9193870FF50310C54EA415C2F21274A795B76C

删除path环境变量的:C:\Program Files (x86)\Common Files\Oracle\Java\javapath
java security的tsl,certpath
```



```
#tempory
e09ca031-c3a9-4be2-a082-9cf209b215e4
```

```
#daily
ccelery AttributeError
```



##### TI

```
hdfs延时，es不延时
hdfs输入只能为一个文件夹？
```

```
in:name React stars:>5000
 in:readme React
 in:description 
```

```
#ti
基于符号逻辑和统计推理方法
Peer-To-Peer，RDF，OWL EL。Gray XMT  RDFS推理。
```

```
固件：厂商官网
Metasploit
https://www.exploit-db.com/exploits/45909
http://support.netgear.cn/doucument/More.asp?id=2251
https://www.itdaan.com/so?q=NetGear%E5%A4%9A%E6%AC%BE%E8%B7%AF%E7%94%B1%E5%99%A8%E8%BF%9C%E7%A8%8B%E5%91%BD%E4%BB%A4%E6%B3%A8%E5%85%A5%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90+%E6%9B%B4%E6%96%B0%E8%A1%A5%E4%B8%81%E5%88%86%E6%9E%90+&page=8
FirmTool是CNCERT自主研发一款固件脆弱性分析工具，主要用于工控和物联网固件的安全性检测，
```



https://www.cvedetails.com/

```
扫描:漏洞（模拟攻击），恶意软件（静态，动态）
入侵检测：日志，流量->画像（收集数据，统计），态势（资产，风险，ids）
威胁情报
```



```
用一个较复杂的模式去拟合数据中的规律和特征，也就是说使用其蕴含的信息进行决策的。人要懂得其中的关键性知识，特征，拟人性。一个人类完成这个任务需要什么知识，你使用模型实现也无可避免，不然无法去优化。
```

```
malware class 2015 microsoft 
```

life

https://www.cvedetails.com/

virustotal

ibm x-force exchange

alienvultr

微步

virusshare

```
Malware analysis ATT&CK framework Classification Cyber threat intelligence Advanced persistent threats 
situational awareness, threat intelligence, game theory, network security, nash equilibrium
malware image
attacker behaviour modelling, cyber security,
fusion hidden markov models, hidden markov models, honeypot,
markov chain, sequence models, threat intelligence.
big data; network threats; feature graph clustering; community discovery; attacker discovery; attacker portrait
Data mining, Malware detection, Classification, Behavior-based, Signature-
based
security system state, security visualization, situational awareness
network security awareness; security situational awareness;
big data; network threats; feature graph clustering; community discovery; attacker discovery; attac[…]
摘自：基于大数据和图社群聚类算法的攻击者画像构建
在中国知网查看：https://kns.cnki.net/KXReader/Detail?TIMESTAMP=637536740860234375&DBCODE=CJFD&TABLEName=CJFDLAST2021&FileName=JSYJ202101047&RESULT=1&SIGN=2FttJ5GgbSj9fGmOZBurQZDRi3k%3d
本作品由中国知网负责全球范围内电子版制作与发行。版权所有，侵权必究。
Information security Intrusion detection Alert correlation Attacker groups
network security,intrusion detection,alert  correlation, attack behavior, behavior analysis
attack recognition
它们可以减少误报、消除重复条目、关联事件、分析攻击者策略并找到攻击者组。
it is reasonable for us to believe
that analyzing the IDS alerts is actually analyzing the attack
behaviors behind the alerts.
网络攻击行为分析大致可分为“网络中心”和“攻击者中心”两种方法。
#other
In general, a group of attackers can be definedasanorganizationconsistingofrelated
attackers, united by tools, the purpose and status of attacks, and information obtained
duringtheattacks.
Several methods of group identification are able to
determine those resources of the attackers that were not involved in known attacks, thus,
it is possible to identify or destroy attackers using such methods.
#attributed method
In order to speed up some of the algorithms in this category combine similar events
into meta-events containing the main characteristics of the group, after which new events
are compared with them. A graph model is usually used to represent data, so the task is
to search for clusters inside the graph by a certain weight function, which determines
the differences in the algorithms.
#attributed algrothim divided into 
This group can be divided into the following subcategories:
– Property-based
– Timestamp-based
– Based on statistical relationships
– Filtering
– Machine Learning-based
#1
RealSecure,
snort,shadow,ids
darpa,CIDF
Netpoketool
```

```
#dataset
malware,pcap,log,
https://archive.ics.uci.edu/ml/index.php
#site
http://www.covert.io/
https://www.researchgate.net/
https://link.springer.com/chapter/10.1007%2F978-3-030-65726-0_4
https://www.zhihu.com/question/30111950
https://dblp.uni-trier.de/search?q=attacker%20%20recognition
https://www.eurecom.fr/en/publication/1271
https://www.semanticscholar.org/paper/A-Review-of-Intrusion-Alerts-Correlation-Frameworks-Chahira-Kiruki/9eb3224e6a94c8e906f5f393702cc3f4114746f1?p2df
https://www.sciencedirect.com/science/article/abs/pii/S0951832010000232?via%3Dihub
https://arxiv.org/abs/2003.05822
```

```
ida,jeb,ghidra
```



##### 信息网站

```
#keyword
symbolic execution，malware analysis，threat intelligence，
```

```
知乎,图书馆(iee,知网),谷歌,sci-hub
```

```
github,kaggle,paperswithcode
```

```
情报学报,dblp
```

```
Fuzzers, Analysis, Backdoors, DoS, Exploits, Generic, Reconnaissance, Shellcode and Worms.
```

##### code查找

```
论文内容，搜索http,github等关键字
google程序算法,例：knn in python，谷歌作者名进学术主页
github搜索论文名，作者名，算法名
paperswithcode
arXiv网站
Find Code for Research Papers插件
Code Ocean: Professional tools for researches
https://researchcode.com/
https://www.semanticscholar.org/
看雪论坛
```

### 论文记录

##### 符号执行

```
Optimizing Symbolic Execution for Malware Behavior
Classification
生成系统调用依赖图计算相似性
使用的工具：angr，radare2

Mining control flow graph as API call-grams to detect portable executable malware

Higher-Order Symbolic Execution via Contracts
```



peer j

connected paper



site:github:awesome list





##### 下载

sourceforge

slideshare

https://notepad-plus.en.softonic.com/download

##### 镜像

https://mirrors.bfsu.edu.cn/

阿里，清华http://isoredirect.centos.org/centos/7/isos/x86_64/

博客园：https://www.cnblogs.com/jingch/p/11368982.html

遁地模拟器，飞天辅助，虚拟大师

210.28.18.6

190   				189				188

hbase  			hbase1  	  hbase2

hadoop102    hadoop103  hadoop104

Minio 存储服务 thumbor 图片裁剪等

openCTI 情报下载

CTI 

grakn ai:图数据库



