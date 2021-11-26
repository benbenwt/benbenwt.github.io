



```
prorequest,dblp,google,hindawi,paperwithcode
```



```
fournumber
importBymodule
searches
cve

searchprovider
platformprovider
```



```
一行为一条记录，记为一次。
列数可以不固定，但是需要可分割的列。
fp输出的为所有项的频次。对应于ti就是所有列的组合频次。
```





### metrics

```
https://blog.csdn.net/hahajinbu/article/details/78629130
```

````
myModel.py
worker.py
to_simple_rdd.py
evaluate.py
````

```
Dow
```



|       | **0**  | 1    | **2**  |
| ----- | ------ | ---- | ------ |
| **0** | **00** | 01   | 02     |
| **1** | 10     | 11   | 12     |
| **2** | 20     | 21   | **22** |

```
https://blog.csdn.net/pearl8899/article/details/109987788
对角线为命中的数量。
macrof1,分类别计算f1值，然后平均。对于单个类别，例如0类：
召回率=（00）/（00+01+02）
精确度=(00)/(00+10+20)
计算单类别f1，然后平均三个指标得到全局。

对于整个数据集所有类别(micro)：
召回率=（00+11+22）/（00+01+02+10+11+12+20+21+22）
精确度=(00+11+22)/（00+01+02+10+11+12+20+21+22）
计算microf1即可。
microf1，直接使用计算后的精确度和召回率计算f1，一般三个值相等...,包括accuracy也相等。

```

```
k-cfsfdp，本来就有人提出了......
```



```
三个指标（单机，该方法），达到准确度的速度（history）（同步，异步，该方法），

```

```
已存方法，新领域，这个不算
(改变)，已存方法(改变)，新领域，
apt报告，lstm，并行，ti。

1.多个步骤，改了一个步骤。
apt报告,lstm，并行，ti。
2.集成

同步等待，异步陈旧参数，通信开销（压缩，通信窗口，参数服务器），收敛否及效率。通信行为和收敛的关系相伴相生
既然每个电脑配置不同，你给不同的数据不行嘛。（实验，，，）
-------1.提model并行方法,这不就相当于通用型了，那你不得超越所有....
1.通信开销 -J，moving rate α
1.（陈旧度）通信协议？？这不是又是泛用，而且是单独的点。异步陈旧度（这个可以自定义，参数版本），异步（对于长期处于trainging，这种不用等到提交再去评定陈旧，可丢弃），提交时和定时都要检测陈旧度。
worker过于陈旧（参数版本，），丢弃并拉取最新。worker提交的过于陈旧，加权削弱或丢弃。
1.对于快的电脑分配更多的数据集（计算能力）。
1.弥补spark同步能力的缺失，其rdd只能同步拉取
2.基于spark实现了参数服务器异步训练model，使用...策略增项了model。easgd,downpour
3.将model应用于ti，取得了...效果
```

```
parallel training, Spark cloud computing
bigdata, Hadoop, apache spark,
big data,ensemble techniques
```



# Apache Spark DL Book

### 1 setting up 

```
jupyter notebook 配置 pyspark
spark是运行在jvm中的，需要安装jdk。使用anaconda安装jupyter、pyspark。然后配置环境变量如下，运行sparknotebook会自动启动notebook，也可以不配置环境变量，在运行命令时指定即可。
配置环境变量如下,vim ~/.bashrc:
export PATH="/home/docker/minianaconda/bin:$PATH"
function sparknotebook()
{
export SPARK_HOME=/home/docker/minianaconda
export PYSPARK_PYTHON=python3
export PYSPARK_DRIVER_PYTHON=jupyter
export PYSPARK_DRIVER_PYTHON_OPTS="notebook"
$SPARK_HOME/bin/pyspark
}
等价于在terminal运行命令时指定：
$SPARK_HOME/bin/pyspark  SPARK_HOME=/home/docker/minianaconda PYSPARK_PYTHON=python3 ...省略
```



# 共享和可信度

**A Reputation-based Approach using Consortium Blockchain  for Cyber Threat Intelligence Sharing**

```
问题：cti中包含企业隐私信息，如何确保不被窃取,避免个人身份泄露，进一步利用风险，名誉损失。应对拜占庭攻击，使用区块链。
解决：分布式信誉系统，联盟区块链
指标：吞吐量，达成共识效率
优点：解决了拜占庭
缺点：
创新：分布式信誉
文献：24-26
```

**Attribute Based Sharing in Cybersecurity**
**Information Exchange Framework**

```
问题：避免个人身份泄露，进一步利用风险，名誉损失。共享组织可能访问隐私信息
解决：使用基于属性的半信任控制访问。
```

**Distributed security framework**

```
MISP:open-source threat intelligence sharing platform
```

**Info-Trust: A Multi-Criteria and Adaptive Trustworthiness Calculation Mechanism for Information Sources**

```
```



```
挖掘，可信，共享，知识推断，资产发现，资产加情报，web自定义sql
```



```
硕博论文集：ProQuest
计算机相关：dblp
论文集：arxiv
```



```
威胁情报源的可信性评估，威胁情报内容的可信性评估，基于....
```



```
问题：情报共享社区中的威胁情报质量参差不齐，只有高可信的情报才有价值。
解决方案
指标
创新
优点
缺点
```



```
特点
理论部分都是自定义的评估方式，不同于直接用模型，自己的部分较多，本质是优化评估可信度的方式。实验部分可用仿真，但使用时又可以放到spark上。
如何仿真的？一个并行的算法如何仿真？spark的呢？
信息可信感知技术
```



### 谷歌学术

```
搜谷歌学术，进中文的那个谷歌搜索，点击搜索结果上的引号，赋值GB/T 标准的引用。
```



```
一个是http，一个是用rdd返回的结果直接处理的。
```

### spark

```
spark如何输出日志。
可用print，或和flask等框架一样使用日志框架。
如何使用flask服务的,post和get请求？
开flask服务，提供post和get接口。然后用urllib处理拉取和更新请求。
```



测试此版本

```
tensorflow 2.1.0, pyspark 3.0.2, jdk-8u281 and python 3.7 and elephas 1.4.2
```



### 搜索关键词

```
选择advcanced search：全文搜索"apache spark" AND "deep learning"   ,   "apache spark" AND "name entity"
```



### github库

```
https://github.com/cerndb/dist-keras
相关的可选库
https://blog.csdn.net/weixin_33849942/article/details/91609549
https://blog.csdn.net/weixin_39195527/article/details/76099206
```

# 并行

>厦大的一个lab: http://dblab.xmu.edu.cn/blog/2935/

```
dist-keras-:https://joerihermans.com/ramblings/distributed-deep-learning-part-1-an-introduction/
```

```
提出并行方法，使用异步downpour,spark实现并行了模型,
应用于领域
```

```
网络开销：参数压缩，通信行为(一堆指标参数)
参数服务器：模型分区
训练速度：异步
```

### 基本概念

```
GD（Gradient Descent）：就是没有利用Batch Size，用基于整个数据库得到梯度
BGD，每次迭代使用批量的数据计算梯度，再更新
SGD，每次使用<随机选取>一个数据计算梯度，然后执行更新。
mini-batch sgd,每次使用<随机选取>批量数据计算梯度，然后执行更新。
MBGD(mini-batch bgd),BGD和SGE的折中，每次使用数据集一部分计算，注意不是随机的。也就是我默认使用的rdd分区，但是我没有在一次迭代后更新参数。
1，如果我使用分为n块的全局数据集，再执行n次更新这是sgd嘛？如果我将这n个平均了再更新，是sgd嘛？如果是，那么划分了数据集就一定是sgd嘛？
不是sgd，是mbgd。不是sgd，是mgbd，只是步长不同。不是，主要区别在随机选取。
2原本单机的不就是按batch嘛，也是数据集的部分呀，这是bgd。并行了不还是一样？

算完了整个数据集才更新，是rdd默认模式，为bgd。
按步骤，每次迭代要更新，为mbgd。

blog:https://blog.csdn.net/qq_19672707/article/details/94056538
```

### 文献

##### Automatic and Accurate Extraction of Threat Actions  from Unstructured Text of CTI Sources

```
数据集来自att&ck和收集的apt报告，来自symantec security c
svm分类，然后Stanford typed dependencies词性依赖工具挑选action，最后BM25,tf-idf映射。
取threat-action，映射，频繁
hmm效果，映射，频繁点缀
stadnford:https://blog.csdn.net/guotong1988/article/details/50667335
stadnford manual script: https://downloads.cs.stanford.edu/nlp/software/dependencies_manual.pdf
stadnford gui: https://blog.csdn.net/sinat_41144773/article/details/102828718
frequent question:https://nlp.stanford.edu/software/parser-faq.html
stadnford csdn:https://blog.csdn.net/guotong1988/article/details/50667335
短语结构和依存句法：https://blog.csdn.net/qq_43428310/article/details/107290398
information retrieval ：information retrieval
```

##### stanford用法

```
查看dependencies_manual.pdf文件，the pratice 章节有具体用法。前边章节有理论介绍。
```

##### gui

```
load parser
load file 输入的句子
```

##### cmd

```
java -mx200m -cp "*"  edu.stanford.nlp.parser.lexparser.LexicalizedParser  -retainTmpSubcategories -originalDependencies -outputFormat "penn,typedDependencies" -outputFormatOptions "basicDependencies"  "edu/stanford/nlp/models/lexparser/englishPCFG.ser.gz"  test.txt
englishPCFG.ser.gz这种后缀的都在models.jar包中
LexicalizedParser这种功能一般在sources.jar包中
```

##### 流程

```
句子分类，句法依赖。信息相似度，信息检索，文本相似度，相似度计算
分句，然后提取依赖，然后筛选。
att&ck描述一般是threat-actor怎么怎么样，主语最好替换以下。不一定是一句，可能是一个长度的窗口。只选需要的部分，满足动名词的部分。
数据集格式，tactic  1->n technique   n<->n  (procedure,blogtxt)
```

##### 问题

```
send waves of spearphishing messages  ,只能取到send  waves，没有spearphishing影响后边的相似度计算
00 00 03 Malware downloads the URL that follows the command ID .数字影响提取
prep [ connected to ] 
pobj [ to 198.55.115.71 ] 
prep [ connected over ] 
pobj [ over port ] 
num [ port 1913 ] 
prep — 介词修饰这一种也只能丢失信息
```

att&ck

```
tactic -> technique -> procedure(group,description)
group -> software
group -> technique

关于enterprise和mobile：http://www.yidianzixun.com/article/0SBkITCh
Enterprise 领域主要面向的平台为 Linux、MacOSI 和 WindowsATT&CK Matrix 的解读
```



##### featureSmith

```
dataset from scientific papers and malware from Drebin dataset
ttp,然后检测恶意软件
```

##### Using Entropy and Mutual Information to Extract Threat Actions from Cyber Threat Intelligence

```
互信息(Mutual Information)是信息论中的概念，用来衡量两个随机变量之间的相互依赖程度。
csdn MI:https://www.jianshu.com/p/00254c4d0931
数据集来自att&ck，wiki
改进了取threat-action部分，让vo自由组合，而不是按照编写的规则。自由组合会有很多不合法的组合，通过MI进行筛选。但是没有映射为ttp，可能是因为映射为ttp并没有提升效果，如send data，send info等可能重复的threat-action。提升的方面体现在，获取了更多组合方式。
相似度计算：https://blog.csdn.net/circle2015/article/details/107952424
这种挑选方式，增加了vo对，但是使用相似度过滤仍然会保留不合法的vo对。提取出不正确的threat-ation，例如send,download,crate,modify,明明是download dll， exe或sh但会将其与send结合。
```

##### EX-Action

```
改了第二部
```

```
上边三篇都是使用了句法依存、机器学期、MI、词向量等，无改进部分。但分别解决了部分问题，如ttpdrill解决的抽取宏观信息，ttp，而不是ioc。用句法依存和MI解决。ActionMiner解决抽取不全，用MI解决。第三个是改了第一步的标签，第二步加权了一些指标，解决本体的使用问题。那如此类比，标签的动作和谁组合，合并词语，指代，词性，分类，hmm威胁动作，使用本体是为了从大量的组合中筛选有用的，替换dll、exe等进行映射，spark提升效率，相似度和MI会漏掉部分action，聚类确定分类。crf边界问题，标签bi倒置问题，长短距离问题，词向量表达问题。领域问题，前人的方法，仍存在问题，你的方法。
具体步骤确定action，确定threat-action，确定映射的threat-action分类。避免多个分类器，避免定义和使用本体，避免使用本体的多个属性进行相似度计算，避免使用此方法筛选action。
无法针对安全领域的语法，所以用hmm。然后再筛选，并且并行了。只识别需要的动名词对。
attck:https://attack.mitre.org/tactics/TA0006/
capec:https://capec.mitre.org/documents/documentation/CAPEC_Schema_Description_v1.3.pdf
动名词对标签以及hmm模型取动名词对，然后聚类加文档相似度加频繁项集。数据集用之前那个，分词后标记。
```



##### Threat Action Extraction using  Information Retrieval

```
直接标签标记需要的threat-action，提取threat-action时就不拿无用的threat-action（vo组合）
```

##### Automated Threat Report Classification Over Multi-Source Data

```
分步骤分别分类tactic，techique，action，偏移调整权重
```





##### A New Improved Baum-Welch Algorithm for Unsupervised Learning for Continuous-Time HMM Using Spark 

```
提了一个并行版本的Baum-Welch和一个用于无监督版本的Baum-Welch
```



```
http://blog.sina.com.cn/s/blog_8267db980102wpvz.html
hmm:https://www.cnblogs.com/jacklu/p/7753471.html
https://www.cnblogs.com/lokvahkoor/p/12781056.html
https://zhuanlan.zhihu.com/p/248184140
crf:https://www.lookfor404.com/%E5%91%BD%E5%90%8D%E5%AE%9E%E4%BD%93%E8%AF%86%E5%88%AB%E7%9A%84%E8%AF%AD%E6%96%99%E5%92%8C%E4%BB%A3%E7%A0%81/
crf:https://github.com/lipengfei-558/hmm_ner_organization


```





##### association rules mining

```
这种针对多条记录，找到在很多条记录都出现的组合。在cti领域很奇怪，难道说很多分威胁情报都重复出现了一个组合信息，你才能挖掘出来嘛，单个情报就已经有价值了。所以说关联你获得的是什么结果呢？获得的是重复出现的信息，即趋势...也不对....,什么东西可能会在很多cti中出现？哪些不针对特定事件的，比如地点，攻击类别等固定的属性值，每个人都具有的，如端口，地点，操作系统，语言，时间等信息，这不就类似于统计了嘛，不过是统计了多个列而已。像文献中所提的ip，hash怎么可能呢？难道有多个cti中包含此ip，hash...。它并不是对关联的东西进行链接，而是统计出高频出现的组合。
那如果我需要统计特定过滤条件下的整体的趋势，怎么用。如我只要特定时间、地点、组织。并不只是统计，比如你用es能直接统计出s，c，l要求的内容嘛，不能。如果能限定关注的点下的置信度，也会有价值，相当于用where限制统计区域。
评估指标呢，速度，内容，兴趣度。
```

##### Intrusion Detection Using Big Data and Deep Learning Techniques

```
随机森林和 GBT
分类器是使用 Apache Spark 的内置机器实现的
学习库 MLlib。 深度学习模型实现
使用分布式 Keras[9]

特征挑选

三个model：
Deep neural network 99.19
Random Forest 98.86
Gradient Boosted Tree 97.92

code repository：https://github.com/OsamaFaker/INTRUSION-DETECTION-BIG-DATA
```

##### A Scalable and Hybrid Intrusion Detection System Based on the Convolutional-LSTM Network

```
第一阶段采用基于 Spark ML 的异常检测模块。第二阶段
作为一个误用检测模块，它基于 Conv-LSTM 网络，这样既全局
```

##### **Traffic Network Flow Prediction Using Parallel Training for Deep Convolutional Neural_Networks_on_Spark_Cloud**

```
这个就是把平均梯度理论给推导了一遍，然后做了个实验，比对三个指标。然后对比加速比、不同节点的扩展性。
本文的主要贡献总结如下。(1)为基于自适应梯度下降原理的离散余弦神经网络模型建立了并行训练算法的理论基础。通过局部学习，保证了所提出的并行训练算法能够像串行训练算法一样，以良好的收敛性学习整个数据集的全局特征(把具体怎么算的说明了一遍,如由局部参数到全局参数，局部损失、局部参数计算，这不是已有理论嘛，只不过不是针对此dcnn模型)
说明了目标函数、训练过程（前向，后向（如由局部参数到全局参数，局部损失、局部参数计算））
l
j
i
f
前向复述了一遍，后向全连接层的kernel matrix和bias matrix按照平均梯度更新，
(2)将DCNN模型应用于交通网络流量预测，以捕捉时空数据的特征。然而，这是使用并行训练算法实现的，并且该模型首先在Spark云计算平台上实现，以解决交通网络流的大数据所面临的计算复杂性。通过实际交通网络流量数据验证了基于云的并行训练算法的渐近收敛性和加速优越性。（spark实现）

```

##### hadoop entity recongnition

```
强行用crf+规则，用规则提取本来就是最好的了。
1提取的类型过于简单，都是格式固定的，可用正则处理。
2对比过少，如不同数量机器的指标和时间。
```

##### Adaptive‑Miner: an efcient distributed  association rule mining algorithm on Spark

```
https://github.com/sanjaysinghrathi/Adaptive-Miner
```



##### An improved approach for mining association rules in parallel using Spark Streaming

```
http://fimi.uantwerpen.be/data/
```

##### chainsmith

```
https://ioc-chainsmith.org/
```

##### An Association Rule Mining-Based Framework for  Profiling Regularities in Tactics Techniques and 
Procedures of Cyber Threat Actors 

```
TAXXI FORMAT:http://hailataxii.com/
MITRE:https://medium.com/mitre-attack/attack-subs-what-you-need-to-know-99bce414ae0b
IOC:https://www.threatminer.org/
CSDN APRIORI：https://blog.csdn.net/qq_45083791/article/details/111408587
IBM:https://www.ibm.com/support/producthub/icpdata/docs/content/SSQNUZ_latest/wsd/nodes/associationrules.html
CTI DATASET:https://ieee-dataport.org/open-access/cyber-threat-intelligent-cti-dataset-generated-public-security-reports-and-malware
Feed:https://logz.io/blog/open-source-threat-intelligence-feeds/
GITHUB-FEED:https://github.com/hslatman/awesome-threat-intelligence
GROUP:https://docs.google.com/spreadsheets/u/1/d/1H9_xaxQHpWaa4O_Son4Gx0YOIzlcBWMsdvePFX68EKU/pubhtml
GITHUB-IOCS:https://github.com/davidonzo/Threat-Intel/
MISP FEED(real-time):https://www.circl.lu/doc/misp/feed-osint/
```

```
hash,ip,port,url,timestamp,type;geoIP;ttp
malware,network,location,ttp
```

```
gspan,apriori-based
scan,dbscan
```



### 数据并行

```
1同步方法 vs. 异步方法
2参数平均法 vs. 更新式方法
3中心化同步 vs. 分布式同步
1怎么收集？2收集后计算规则？3中心化同步 vs. 分布式同步？
平均与否，就是步长的区别，不平均步长是n，平均为1。
```

# elephas+keras_bert

正确运行版本:

```
python==3.6.8
tensorflow==1.14.0
Keras==2.2.5
keras_bert==0.83.0
keras_contrib==2.0.8
Flask==1.1.2
pyspark==3.1.2
sklearn
spacy
```



```
通信行为间隔
worker端梯度的形式
梯度的阿尔法
```



##### 主要错误

```
1.keras库的engine/saving.py有decode的错误
2.keras_contrib类型错误，到keras_contrib的issue可以找到。
```



```
spark-submit  --master spark://172.18.65.187:7077 --py-files /root/software/spark-elephas/spark_elephas.zip --conf "spark.pyhspark.driver.python=/root/miniconda3/envs/elephas1/bin/python" --conf "spark.pyspark.python=/root/miniconda3/envs/elephas1/bin/python"  /root/software/spark-elephas/myModel.py
```



```
these are Deep Feed-Forward Neural Network (DNN) and two ensemble techniques, Random Forest and Gradient Boosting Tree
(GBT). 
```

### 碰到的问题

keras有一个f.attr['keras_version'].decode('utf-8')报错，无法对str解码，我将decode删除了。

##### elephas改了很多

elephas主要更改了spark_model,spark_worker,worker.train,to_simple_rdd.

数据流动顺序:data->to_simple_rdd->spark_model.fit->mapPartion->worker.train->substract_params

##### 关于keras和keras_bert

```
model=model_from_json(json,custom_objects)
keras_bert 有get_custom_objects,为一个字典，键值分别为层的名称和来自哪个包哪个类。
```

存储model

```
https://blog.csdn.net/linmingan/article/details/50736615
```



# elephas

>要求keras 2.2.5,但为了统一，不再使用tensorflow.keras,tensorflow版本随意。
>
>keras-bert-keras2.2.5-tensorflow1

### 异步worker

```
在分区上执行了model.fit后，请求flask服务，更新master_urld
```



### 正确版本

```
https://github.com/maxpumperla/elephas/issues/82
https://github.com/maxpumperla/elephas/issues/146
使用elephas==1.0.0版本,会自动安装其他依赖。
spark 3.1.2
java 1.8
Python 3.7.9
tensorflow==2.1.3
pyspark==3.0.1
pyspark==3.1.2
keras==2.2.5
```

##### Implementing a Deep Learning Model for Intrusion Detection on Apache Spark Platform

```
两层lstm，展示了很多实验。有如下，不同层数（1，2，3），不同模型（mlp,rnn,lstm），不同集群configuration，排列组合共3*3*3=27种数据。还对比了有无smote采样，两分类和多分类。
```



```
elephas
"apache spark" AND "deep learning" 
```

##### 修改记录

```
AttributeError: 'Sequential' object has no attribute 'compiled_metrics'
spark_model.py , line 47

metrics=model.metrics
```

```
Caused by: java.io.EOFException
	at java.io.DataInputStream.readInt(DataInputStream.java:392)
	at org.apache.spark.api.python.PythonRunner$$anon$3.read(PythonRunner.scala:642)
	... 29 more
应该是序列化和反序列化的问题，如传递json文件，weights等。
```



##### spark

```
spark更多是一个方法的实现，文中都没有对其进行性能的调优，只是实现了功能。
文章1：文本情感，集成了svm，lr，crf做分类，实验对比了单机和节点f1和速度。
文章2：舆情情感：加了自己的数据处理和输入特征处理，用stacking加权集成rf,gbdt,xgboost,knn,svm，lr做次级学习器。实验对比了，TF-IDF,Word2vec等不同输入特征处理的f1，及不同stacking策略的不同节点速度。
```

```
表4  不同模型实验结果对比
模型	结果
	精确率	召回率	F1值
1  BERT-Softmax	68.67	52.53	58.41
2  BERT-CRF	72.53	51.21	58.81
3  LSTM-CRF	73.42	56.10	62.37
4  CNN-LSTM-CRF	70.16	63.67	66.41
BERT-BiLSTM-CRF	75.83	64.16	68.84

```

```
针对什么难题问题，提出什么方法，进行了什么实验，结果如何。
但是方法不能对比，实验部分还是突出不了方法。
```



##### 要改的

```
摘要
修订
文献

IOC翻译为攻击指示器
格式，计算机应用

逻辑上?
```



```
lisa生成pcap，pcap逆向到83维特征。
```

##### 南京大学实验室

```
http://pasa-bigdata.nju.edu.cn/links.html
```



```
自己之前的确忽略了一些东西，没有进行科学理性的评估。
可选：在hadoop上实现es的功能，再并行化点东西。或不使用dl4j直接用简易的算法。
```

# keras-bert错误更改

##### AttributeError: module 'tensorflow' has no attribute 'name_scope' 

```
https://stackoverflow.com/questions/51724309/attributeerror-module-tensorflow-has-no-attribute-name-scope-with-keras
pip install -I tensorflow
pip install -I keras
```



```
keras_bert==0.83.0
Keras==2.2.4
seqeval==0.0.10
keras_contrib==2.0.8
matplotlib==3.3.1
numpy==1.16.4
Flask==1.1.2
tensorflow==1.14.0
```

```
使用elephas==1.0.0版本,会自动安装其他依赖。
spark 3.1.2
java 1.8
Python 3.7.9
pyspark==3.0.1
tensorflow==2.1.3
keras==2.2.5
```



##### bert-lstm-crf教程

```
csdn教程:https://blog.csdn.net/jclian91/article/details/111728692
github资源:https://github.com/percent4/keras_bert_sequence_labeling
```

##### AttributeError: module 'tensorflow' has no attribute 'placeholder'

```
修改tensorflow_backend文件,optimizers.py
import tensorflow.compat.v1 as tf
tf.disable_v2_behavior()


```

##### 添加头尾多了

```
```



```
与论文已有实验做对比，比较结果。
自己做实验对比，对比不同层。
```



```
最终版本
python3.6,tensorflow1.15.0,keras
```

the truth value of array is ambigous,use arr.all() or arr.any()

```
D:\DevInstall\anaconda\envs\tensorflow_1_1\Lib\site-packages\keras\engine
修改了load_model函数调用的saving.py，注释的为原始代码
# if weight_names:
        if weight_names.any():
            filtered_layer_names.append(name)
```

keras load_model 报错str不可decode

```
https://blog.csdn.net/guotianqing/article/details/115253163
pip install h5py==2.10
```



keras_contrib:

```
https://zhuanlan.zhihu.com/p/267689558
pip install git+https://www.github.com/keras-team/keras-contrib.git
```



```
所用版本问题:
好像是降低某个库的版本，一个小的库，keras依赖于它。
相关链接：
absl-py==0.13.0
antlr4-python3-runtime==4.8
argon2-cffi @ file:///C:/ci/argon2-cffi_1613037829118/work
astor==0.8.1
async-generator==1.10
attrs @ file:///tmp/build/80754af9/attrs_1620827162558/work
bleach @ file:///tmp/build/80754af9/bleach_1628110601003/work
blis==0.7.4
cached-property==1.5.2
cachetools==4.2.2
catalogue==2.0.5
certifi==2021.5.30
cffi @ file:///C:/ci/cffi_1625831763871/work
charset-normalizer==2.0.4
click==7.1.2
colorama @ file:///tmp/build/80754af9/colorama_1607707115595/work
contextvars==2.4
cycler==0.10.0
cymem==2.0.5
Cython==0.29.21
dataclasses==0.8
decorator @ file:///tmp/build/80754af9/decorator_1621259047763/work
defusedxml @ file:///tmp/build/80754af9/defusedxml_1615228127516/work
entrypoints==0.3
fuzzywuzzy==0.18.0
gast==0.2.2
gensim==4.0.1
google-auth==1.35.0
google-auth-oauthlib==0.4.5
google-pasta==0.2.0
grpcio==1.39.0
h5py==2.10.0
idna==3.2
immutables==0.16
importlib-metadata @ file:///C:/ci/importlib-metadata_1617877476772/work
ipykernel @ file:///C:/ci/ipykernel_1596206775157/work/dist/ipykernel-5.3.4-py3-none-any.whl
ipython==6.1.0
ipython-genutils @ file:///tmp/build/80754af9/ipython_genutils_1606773439826/work
ipywidgets @ file:///tmp/build/80754af9/ipywidgets_1610481889018/work
jedi @ file:///C:/ci/jedi_1611341129952/work
Jinja2 @ file:///tmp/build/80754af9/jinja2_1624781299557/work
joblib==1.0.1
jsonschema @ file:///tmp/build/80754af9/jsonschema_1602607155483/work
jupyter==1.0.0
jupyter-client @ file:///tmp/build/80754af9/jupyter_client_1616770841739/work
jupyter-console==5.2.0
jupyter-core @ file:///C:/ci/jupyter_core_1612213536848/work
jupyterlab-pygments @ file:///tmp/build/80754af9/jupyterlab_pygments_1601490720602/work
jupyterlab-widgets @ file:///tmp/build/80754af9/jupyterlab_widgets_1609884341231/work
keras==2.6.0
Keras-Applications==1.0.8
keras-bert==0.88.0
keras-contrib @ git+https://www.github.com/keras-team/keras-contrib.git@3fc5ef709e061416f4bc8a92ca3750c824b5d2b0
keras-embed-sim==0.9.0
keras-layer-normalization==0.15.0
keras-multi-head==0.28.0
keras-pos-embd==0.12.0
keras-position-wise-feed-forward==0.7.0
Keras-Preprocessing==1.1.2
keras-self-attention==0.50.0
keras-transformer==0.39.0
kiwisolver==1.3.1
Markdown==3.3.4
MarkupSafe @ file:///C:/ci/markupsafe_1621528313524/work
matplotlib==3.3.4
mistune==0.8.4
murmurhash==1.0.5
nbclient @ file:///tmp/build/80754af9/nbclient_1614364831625/work
nbconvert @ file:///C:/ci/nbconvert_1601896775581/work
nbformat @ file:///tmp/build/80754af9/nbformat_1617383369282/work
nest-asyncio @ file:///tmp/build/80754af9/nest-asyncio_1613680548246/work
notebook @ file:///C:/ci/notebook_1629205950149/work
numpy==1.19.5
oauthlib==3.1.1
opt-einsum==3.3.0
packaging @ file:///tmp/build/80754af9/packaging_1625611678980/work
pandocfilters @ file:///C:/ci/pandocfilters_1605120535071/work
parso @ file:///tmp/build/80754af9/parso_1617223946239/work
pathy==0.6.0
pickleshare @ file:///tmp/build/80754af9/pickleshare_1606932040724/work
Pillow==8.3.1
preshed==3.0.5
prometheus-client @ file:///tmp/build/80754af9/prometheus_client_1623189609245/work
prompt-toolkit==1.0.15
protobuf==3.17.3
pyasn1==0.4.8
pyasn1-modules==0.2.8
pycparser @ file:///tmp/build/80754af9/pycparser_1594388511720/work
pydantic==1.8.2
Pygments @ file:///tmp/build/80754af9/pygments_1629234116488/work
pyparsing @ file:///home/linux1/recipes/ci/pyparsing_1610983426697/work
pyrsistent @ file:///C:/ci/pyrsistent_1600141799440/work
python-dateutil @ file:///tmp/build/80754af9/python-dateutil_1626374649649/work
pytz==2021.1
pywin32==228
pywinpty==0.5.7
PyYAML==5.4.1
pyzmq @ file:///C:/ci/pyzmq_1628276569449/work
qtconsole @ file:///tmp/build/80754af9/qtconsole_1623278325812/work
QtPy==1.9.0
requests==2.26.0
requests-oauthlib==1.3.0
rsa==4.7.2
scikit-learn==0.24.2
scipy==1.4.1
Send2Trash @ file:///tmp/build/80754af9/send2trash_1607525499227/work
simplegeneric==0.8.1
simplejson==3.17.3
six @ file:///tmp/build/80754af9/six_1623709665295/work
sklearn==0.0
smart-open==5.2.0
spacy==3.1.1
spacy-legacy==3.0.8
srsly==2.4.1
stix2==3.0.0
stix2-patterns==1.3.2
tensorboard==2.1.1
tensorflow-gpu==2.1.0
tensorflow-gpu-estimator==2.1.0
termcolor==1.1.0
terminado==0.9.4
testpath @ file:///tmp/build/80754af9/testpath_1624638946665/work
thinc==8.0.8
threadpoolctl==2.2.0
tornado @ file:///C:/ci/tornado_1606942379977/work
tqdm==4.62.1
traitlets==4.3.3
typer==0.3.2
typing-extensions @ file:///tmp/build/80754af9/typing_extensions_1624965014186/work
urllib3==1.26.6
wasabi==0.8.2
wcwidth @ file:///tmp/build/80754af9/wcwidth_1593447189090/work
webencodings==0.5.1
Werkzeug==2.0.1
widgetsnbextension==3.5.1
wincertstore==0.2
wrapt==1.12.1
zipp @ file:///tmp/build/80754af9/zipp_1625570634446/work

```



```
threatActor  vulnerability_cve  malware software location  identity
word2vec,bert...?
ann,bio...?如果需要，brat可以转
```

# 1

```
ROOT：要处理文本的语句
IP：简单从句
NP：名词短语
VP：动词短语
PU：断句符，通常是句号、问号、感叹号等标点符号
LCP：方位词短语
PP：介词短语
CP：由‘的’构成的表示修饰性关系的短语
DNP：由‘的’构成的表示所属关系的短语
ADVP：副词短语
ADJP：形容词短语
DP：限定词短语
QP：量词短语
NN：常用名词
NR：固有名词
NT：时间名词
PN：代词
VV：动词
VC：是
CC：表示连词
VE：有
VA：表语形容词
AS：内容标记（如：了）
VRD：动补复合词
CD: 表示基数词
DT: determiner 表示限定词
EX: existential there 存在句
FW: foreign word 外来词
IN: preposition or conjunction, subordinating 介词或从属连词
JJ: adjective or numeral, ordinal 形容词或序数词
JJR: adjective, comparative 形容词比较级
JJS: adjective, superlative 形容词最高级
LS: list item marker 列表标识
MD: modal auxiliary 情态助动词
PDT: pre-determiner 前位限定词
POS: genitive marker 所有格标记
PRP: pronoun, personal 人称代词
RB: adverb 副词
RBR: adverb, comparative 副词比较级
RBS: adverb, superlative 副词最高级
RP: particle 小品词
SYM: symbol 符号
TO:”to” as preposition or infinitive marker 作为介词或不定式标记
WDT: WH-determiner WH限定词
WP: WH-pronoun WH代词
WP$: WH-pronoun, possessive WH所有格代词
WRB:Wh-adverb WH副词
```

**关系表示**

```
abbrev: abbreviation modifier，缩写
acomp: adjectival complement，形容词的补充；
advcl : adverbial clause modifier，状语从句修饰词
advmod: adverbial modifier状语
agent: agent，代理，一般有by的时候会出现这个
amod: adjectival modifier形容词
appos: appositional modifier,同位词
attr: attributive，属性
aux: auxiliary，非主要动词和助词，如BE,HAVE SHOULD/COULD等到
auxpass: passive auxiliary 被动词
cc: coordination，并列关系，一般取第一个词
ccomp: clausal complement从句补充
complm: complementizer，引导从句的词好重聚中的主要动词
conj : conjunct，连接两个并列的词。
cop: copula。系动词（如be,seem,appear等），（命题主词与谓词间的）连系
csubj : clausal subject，从主关系
csubjpass: clausal passive subject 主从被动关系
dep: dependent依赖关系
det: determiner决定词，如冠词等
dobj : direct object直接宾语
expl: expletive，主要是抓取there
infmod: infinitival modifier，动词不定式
iobj : indirect object，非直接宾语，也就是所以的间接宾语；
mark: marker，主要出现在有“that” or “whether”“because”, “when”,
mwe: multi-word expression，多个词的表示
neg: negation modifier否定词
nn: noun compound modifier名词组合形式
npadvmod: noun phrase as adverbial modifier名词作状语
nsubj : nominal subject，名词主语
nsubjpass: passive nominal subject，被动的名词主语
num: numeric modifier，数值修饰
number: element of compound number，组合数字
parataxis: parataxis: parataxis，并列关系
partmod: participial modifier动词形式的修饰
pcomp: prepositional complement，介词补充
pobj : object of a preposition，介词的宾语
poss: possession modifier，所有形式，所有格，所属
possessive: possessive modifier，这个表示所有者和那个’S的关系
preconj : preconjunct，常常是出现在 “either”, “both”, “neither”的情况下
predet: predeterminer，前缀决定，常常是表示所有
prep: prepositional modifier
prepc: prepositional clausal modifier
prt: phrasal verb particle，动词短语
punct: punctuation，这个很少见，但是保留下来了，结果当中不会出现这个
purpcl : purpose clause modifier，目的从句
quantmod: quantifier phrase modifier，数量短语
rcmod: relative clause modifier相关关系
ref : referent，指示物，指代
rel : relative
root: root，最重要的词，从它开始，根节点
tmod: temporal modifier
xcomp: open clausal complement
xsubj : controlling subject 掌控者
中心语为谓词
subj — 主语
nsubj — 名词性主语（nominal subject） （同步，建设）
top — 主题（topic） （是，建筑）
npsubj — 被动型主语（nominal passive subject），专指由“被”引导的被动句中的主语，一般是谓词语义上的受事 （称作，镍）
csubj — 从句主语（clausal subject），中文不存在
xsubj — x主语，一般是一个主语下面含多个从句 （完善，有些）

```

```
中心语为谓词
subj — 主语
nsubj — 名词性主语（nominal subject） （同步，建设）
top — 主题（topic） （是，建筑）
npsubj — 被动型主语（nominal passive subject），专指由“被”引导的被动句中的主语，一般是谓词语义上的受事 （称作，镍）
csubj — 从句主语（clausal subject），中文不存在
xsubj — x主语，一般是一个主语下面含多个从句 （完善，有些）
中心语为谓词或介词
obj — 宾语
dobj — 直接宾语 （颁布，文件）
iobj — 间接宾语（indirect object），基本不存在
range — 间接宾语为数量词，又称为与格 （成交，元）
pobj — 介词宾语 （根据，要求）
lobj — 时间介词 （来，近年）
中心语为谓词
comp — 补语
ccomp — 从句补语，一般由两个动词构成，中心语引导后一个动词所在的从句(IP) （出现，纳入）
xcomp — x从句补语（xclausal complement），不存在
acomp — 形容词补语（adjectival complement）
tcomp — 时间补语（temporal complement） （遇到，以前）
lccomp — 位置补语（localizer complement） （占，以上）
— 结果补语（resultative complement）
中心语为名词
mod — 修饰语（modifier）
pass — 被动修饰（passive）
tmod — 时间修饰（temporal modifier）
rcmod — 关系从句修饰（relative clause modifier） （问题，遇到）
numod — 数量修饰（numeric modifier） （规定，若干）
ornmod — 序数修饰（numeric modifier）
clf — 类别修饰（classifier modifier） （文件，件）
nmod — 复合名词修饰（noun compound modifier） （浦东，上海）
amod — 形容词修饰（adjetive modifier） （情况，新）
advmod — 副词修饰（adverbial modifier） （做到，基本）
vmod — 动词修饰（verb modifier，participle modifier）
prnmod — 插入词修饰（parenthetical modifier）
neg — 不定修饰（negative modifier） (遇到，不)
det — 限定词修饰（determiner modifier） （活动，这些）
possm — 所属标记（possessive marker），NP
poss — 所属修饰（possessive modifier），NP
dvpm — DVP标记（dvp marker），DVP （简单，的）
dvpmod — DVP修饰（dvp modifier），DVP （采取，简单）
assm — 关联标记（associative marker），DNP （开发，的）
assmod — 关联修饰（associative modifier），NP|QP （教训，特区）
prep — 介词修饰（prepositional modifier） NP|VP|IP（采取，对）
clmod — 从句修饰（clause modifier） （因为，开始）
plmod — 介词性地点修饰（prepositional localizer modifier） （在，上）
asp — 时态标词（aspect marker） （做到，了）
partmod– 分词修饰（participial modifier） 不存在
etc — 等关系（etc） （办法，等）
中心语为实词
conj — 联合(conjunct)
cop — 系动(copula) 双指助动词？？？？
cc — 连接(coordination)，指中心词与连词 （开发，与）
其它
attr — 属性关系 （是，工程）
cordmod– 并列联合动词（coordinated verb compound） （颁布，实行）
mmod — 情态动词（modal verb） （得到，能）
ba — 把字关系
tclaus — 时间从句 （以后，积累）
— semantic dependent
cpm — 补语化成分（complementizer），一般指“的”引导的CP （振兴，的）
```

