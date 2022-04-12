```
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
13 iot flink


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

