```
202205
02 reidstemplate
03 flink练习
04 sourcefunction
04 es建表
04 写入数据
04 flink sql练习
05 spark 练习，主要是对比flink
06 spark sql
07 电商的五层级表
08 flink的es可视化
09 有条理，全面的覆盖。按照主题和分层进行整理
09 flume为离线数仓和实时数仓采集数据
09 logger
09 es数据类型
09 版本组件
09 线程池
10 labels错误，调试通过
10 输入有nan，成功运行整个程序

11 效果
11 指标计算
11 没有多粒度扫描阶段，后边再补充，先写吧。
11 拆分树
12 完成编写
12 保存训练结果
13 检查指标异常，数据是一致的，k_fold生成的数据一致 是训练的数据搞错了
13 多粒度扫描

14 单机的多粒度扫描
14 统计计算指标所需数据，各阶段花费时间，网络传输时间
14 不使用优化策略的多粒度扫描
14 spark ml的实现

15 如何统计的时间数据，多个节点呢。执行的时候每个节点单独统计出来。
15-25 部分传输广播变量，尝试rdd cache效果不好
15-25 加权分区，发现fit时间波动，尝试yarn资源隔离无效
15-25 尝试不同的kfold、扫描粒度等无效
15-25 只采用5节点，这个东西只能做到这个地步了
25  主要是文章，ti，大数据的代码
25  文章主要ner的和摸索的那些，ner的有一些大文件无法上传，以其他形式记录好，比如资源链接。
25  总结之前的article代码
25  整理项目代码、论文代码到github和移动磁盘,angr  实验室电脑、移动磁盘、远程仓库

11 独立编写完成flink sql。

202204
01 kafka理论知识博客
02  hbase
02  zk
02 javase并发
03  presto，superset，azkaban
04  redis,sparkstreaming那个项目总结
05  ti项目写完，算法以前的笔记总结，数据治理
06  写当前可展示部分
06 scala写法
06  数据库Scala写法
06  streaming api
06  jvm
06  算法
07  网络
07  操作系统
07  concurrent 容器,面经里的
09  clickhouse搭建
09  flink可视化，代码
11  讲清逻辑，深入细节，代码部分
11 搭建
12 基本架构
12 Flink Data Stream API，窗口函数，多流操作。
13 flink商品实时推荐,功能模块，技术
13 跑起来
13 详细过一遍，代码注释一遍
15 页面展示，博客
15 先在集群上弄，然后开虚拟机编写吧，把数据迁移过来。
15 并不是所有任务都适合todo，30分钟todo适合那种任务较为简单，困难可预估，30分钟有进展的。
16 mysql弄好，superset整理好，
16 组会ppt
17 es弄好
17 数据质量管理
17 推荐模块web代码
18 数据质量控制
19  组会
20  推荐web模块
22  es设计索引，不同索引类型。hadoop日志
24  kerberos ，ranger
24  spark理论
25  flink理论
26  flink理论
27  flink理论
28  flink练习题 目录
28  flink练习题1
28  spark练习题

202203
03 DW 66-
03 DW 67
09 八股hdfs,yarn，mr
09 dwd层，dws层，dwt层，ads层
10 每一层的sql模板
11 kettle场景
12 superset对应ads
13 文本相关
15 前端搜索框，后端环境变量,前端代码变动sampleList、cve页面的分页buildList空值。后端cve生成stix时，用cve.title填充stix的description，es_provider添加cve和ip的搜索，解决空值和统计错误问题。
vt爬虫
16 每个机器的启动脚本，及总体的ssh启动脚本。
17 mr用法及理论，yarn用法及理论。
17 spark用法及理论，spark-core pdf
17 spark用法及理论，spark-core pdf
17 hive sql项目用法
18 项目时间，绘制项目示意图，准备恶意样本
19 architecture只取前3个，去除导航栏的导入威胁情报选项，springboot单元测试。
19  log4j日志，读取application.yml错误。
19 lisa文件乱码，apt的批量读取。这个blob时celery写入的，可能使用了独特的编码方式，没法解码。
20组会
20 csdn博客写法
20 是否适合长文本，是否可以处理多标签的分类问题。
20 skearn gcforest 
21 hivesql尚硅谷
202202
19 检查vsphere平台  ok
19 假期文章  ok
20 源码个人电脑  ok
21 精简既有方法、精简第0、1节内容。
21 老旧文献更新为近5年文献
21 docker离线运行，docker离线安装镜像，pip，apt等离线安装库。  
21 mr crawler
22 填写发票，缴纳审稿费
22 改了哪些：down.py的时间加了8小时，服务器的时间改了时区，前端sampleList加了标签，PcapAnalyze导入功能加了解析，lisa改了celery的获取utc时间，java后端改了统计limit3，
24 hive八股文、用法、spark八股文、用法
25 spark core
27 sparkstreaming 日活
28 ss 首单
28 kafka用法与理论,博客
28 zookeeper用法与理论，博客

202201
03 DW13 ok
03 DW20 ok
04 DW23  ok
04 DW26  ok
04 DW29 ok
04 DW32  ok
04 DW35  ok
04 DW40  ok
05 DW43  OK
05 DW46 ok
05 DW49 ok
05 DW52  ok
05 DW55  ok
05 DW58  ok
05 DW60  ok
05 DW61  ok
05 DW62  ok
05 DW63  ok

10 pcapFeature github分支保存好 ok
10 pcapFeature classifyIotDevice 分支的功能修改，Flowgen只区分ip，不区分port。  ok
10 修改统计指标，统计指标只区分ip，不区分port  don`t need
10 选择设计需要的特征，写文档  Ok
11 实现特征统计代码,构造测试数据，编写前6个。
11 统计dns，协议有哪些，编写统计特征代码。

12pca原理 :https://www.zhihu.com/question/38417101/answer/94338598   ok
教程:https://blog.csdn.net/missionnn/article/details/121191490
13 文章退修，格式修订，主要是文献格式等   ok
14 基金表格，退修说明填写，全文检查英文缩写等  ok
14 数据清洗完成了统计了45个特征，pca帮助维度减少到了20个。一起做的无明确分工。

202112
01，多标签分类机器学习例子   ok
01，多标签分类tf-idf例子  ok
02，跑通rcAtt，可能版本sklearn，环境有问题，不对    ok
02，跑通rcAtt，可能joblic,pickle,sklearn兼容问题...github issue里写了  ok
10，tf-idf 处理csv,读csv，取tfidf       ok
13,看ml-knn代码，MLKNN，evaluation两个点，结合理论看。 ok
14,看ml-knn代码  ok
15,看pymlknn这个仓库的代码  ok
15,slearn为什么较快 ok
15，pymlknn和MLKNN的先验都计算的错误的。  ok


4
15，看较新的文献   ok
15，mlknn跑ti的数据，评估效果  ok
16，确定步骤及可操作部分   ok
16,看文献              ok
16,文献code实现，如何编写mapreduce ok
17，github上传之前的文章代码  ok
17，文献code实现，先写knn部分，计算距离，排序 。  ok
23,origin版本mlknn的train   ok
27,cnn-lstm    ok
27,格式,    ok
27,修订内容，可升可降，整体上升或不动。CRF效果好  ok   
29,正则的统计数据，代码看一下  ok
31,绘制柱状图   ok
31,参考文献  ok

202111
22预估的备份任务添加了docker及docker-compose部分，故超时。   ok timeout
26docker构建软件园版本代码 ,挂载es logs，解决挂载目录权限问题。 ok
26解决挂载目录权限问题，cti——server挂载目录问题，覆盖了原来的文件，取消挂载即可,将size修改为10000避免max——result——window错误。影响的功能有cti_txt分页（所有数据分页），cti的备份功能（根据id查询批量stix）。  ok  
26java（不同项目，不同版本）   ok
26python（不同项目，不同版本） ok
26,docker管理版本    ok
26，先备份，然后直接替换整个container文件，然后进入容器，替换整个container的文件。  ok
26，attck爬虫git上去  ok
27，game
28, 取technique   nook
28， 同上，无法从findall结果在findall，直接find调用即可。      ok
28,填入technique，填入tactic           ok 
28，写序列化为文件的代码，构建文件夹目录。  nook  调试看错属性了
28，同上，                    ok
28，访问technique页面获取具体html内容并下载，解析填充pdfurl属性。  nook
29,下载technique的html页面，tf-idf示例,推送到github  ok
29,下载所有pdf，requests无法请求到      nook  
29，selenium下载pdf   nook
30，selenium下载pdf   ok
30,处理html和pdf到text  ok
30，导入功能后端读取缺失字段报错，主要是cve的sqlite取值报错。
30，将代码复制到笔记本编译一下。
30，冗余数据删除，使用json存储所需数据。
```

```
来源于目标的完成，可是有些东西无法称之为目标，因为其没有任何难度，而且在执行之前内容是模糊的。例如刷小说、电影，你不知道将看到的内容的质量，也没有任何难度，多巴胺来自于无法预料的精彩情节，其中无法预料不重要，只要是没剧透的即可，多巴胺并不是来自于你看完了这本书，而来自于其中的过程，这与目标是完全不同的。可以称为过程性的快乐，和结果目标性的快乐。过程性快乐具有轻松性，没有目标的压迫，但是也没有太多意义。
计划或者说目标，应该有可评估的产出和评估，产出例如记录的笔记，评估例如能够独立完成该部分的编码，两者同时达到才表示目标完成。因此写目标时要具体到行为，todo具体写明行为。

长

中：洗漱，工作学习，跑步

短：todo： 洗漱，flink练习题，跑步

1：文字，音乐，影视，基金
1：微信阅读，咪咕

2：户内，户外
2：

```



