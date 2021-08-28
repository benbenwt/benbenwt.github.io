```
图片8，9不清晰
```



### install dl4j

```
maven可能不是最新版本，暂时不管这个。
报错{

}
```



```
spark on angel的文档：https://www.bookstack.cn/read/angel-v3.0/apis-AngelClient.md
```

```
可行性:
angel做ps,spark进行同步的聚合和实现。
angel learn，a start  demo learn

```



### 解决什么问题的

```
大量稀疏数据，超过单台极限，整合数据收集、特征工程和模型。
spark使用的意义
```

### 几种形式

```
多个节点是多个超参数，挑出好的
多个节点是多个数据并行，都执行完成后进行平均梯度。
模型并行，实现非常复杂，要控制cpu粒度的计算和节点通信。同时，可能模型并行也不适合深度学习，无法提高太多训练速度。将同一层过大网络进行拆分，并行计算。

saprk的计算是同步执行，等所有梯度计算完成。
ps的架构是异步执行，梯度不用互相等待。且ps由ps负责参数，而不是类似spark的单个driver，缓解网络压力。


```

### MXNet

```
链接： http://mxnet.io/  

MXNet（发音为 mix-net）起源于卡内基梅隆大学和华盛顿大学的实验室。MXNet 是一个全功能、可编程和可扩展的深度学习框架，支持最先进的深度学习模型。MXNet 支持混合编程模型（命令式和声明式编程）和多种编程语言的代码（包括 Python、C++、R、Scala、Julia、Matlab 和 JavaScript）。2017 年 1 月 30 日，MXNet 被列入 Apache Incubator 开源项目。

MXNet 支持深度学习架构，如卷积神经网络（CNN）、循环神经网络（RNN）和其包含的长短时间记忆网络（LTSM）。该框架为图像、手写文字和语音的识别和预测以及自然语言处理提供了出色的工具。有些人称 MXNet 是世界上最好的图像分类器。

MXNet 具有可扩展的强大技术能力，如 GPU 并行和内存镜像、快速编程器开发和可移植性。此外，MXNet 与 Apache Hadoop YARN（一种通用分布式应用程序管理框架）集成，使 MXNet 成为 TensorFlow 有力的竞争对手。

MXNet 不仅仅只是深度网络框架，它的区别在于支持生成对抗网络（GAN）模型。该模型启发自实验经济学方法的纳什均衡。
```



### Deeplearning4J

```
地址： https://deeplearning4j.org/  
maven :https://search.maven.org/search?q=deeplearning4j
dl4j spark:https://deeplearning4j.konduit.ai/spark/tutorials/dl4j-on-spark-quickstart
ml,dl的基础名词:https://wiki.pathmind.com/

Eclipse Deeplearning4j GitChat课程：https://gitbook.cn/gitchat/column/5bfb6741ae0e5f436e35cd9f
Eclipse Deeplearning4j 系列博客：https://blog.csdn.net/wangongxi
Eclipse Deeplearning4j Github：https://github.com/eclipse/deeplearning4j
官方文档：https://deeplearning4j.konduit.ai/
备用文档：https://blog.konduit.ai/tag/getting-started/
官方社区：https://community.konduit.ai/
相关博客介绍
https://www.zhihu.com/question/426358326/answer/1535430116
https://blog.csdn.net/weixin_40956627/article/details/115408049
https://www.shuzhiduo.com/A/KE5QrjLqdL/

Deeplearning4J（DL4J）是用 Java 和 Scala 编写的 Apache 2.0 协议下的开源、分布式神经网络库。DL4J 最初由 SkyMind 公司的 Adam Gibson 开发，是唯一集成了 Hadoop 和 Spark 的商业级深度学习网络，并通过 Hadoop 和 Spark 协调多个主机线程。DL4J 使用 Map-Reduce 来训练网络，同时依赖其它库来执行大型矩阵操作。

DL4J 框架支持任意芯片数的 GPU 并行运行（对训练过程至关重要），并支持 YARN（Hadoop 的分布式应用程序管理框架）。DL4J 支持多种深度网络架构：RBM、DBN、卷积神经网络（CNN）、循环神经网络（RNN）、RNTN 和长短时间记忆网络（LTSM）。DL4J 还对矢量化库 Canova 提供支持。

DL4J 使用 Java 语言实现，本质上比 Python 快。在用多个 GPU 解决非平凡图像（non-trivial image）识别任务时，它的速度与 Caffe 一样快。该框架在图像识别、欺诈检测和自然语言处理方面的表现出众。
```

### DJL

```
djl基于java强大的抽象能力,将各ai框架抽象成一个一个的算法引擎,类似于在java中操作数据库都可以通过jdbc一样,对各种ai引擎提供了统一的访问接口,可以方便的部署和推理,同时也可以用于训练,其本身不实现算法。
aws
包装多层引擎不方便修改
```

### Tribuo 

```
oracle only maching learning
```



### 确定技术

```
一个拥有java接口的库，如何与spark、hadoop集成，实现ps和分布式。
deeplearning4j,mxnet
相关博客:https://blog.csdn.net/demm868/article/details/103052471

需要一个现存的框架，支持编写自定义的算法结构和并行结构，提供基础数学计算和通信功能。
直接使用ts，py导致spark实际只有ps的作用，而且不易于扩展自定义的功能。
通过mapreduce并行的操作，会并行map函数内的步骤（多数据划分后进行分split执行）。
训练，载入，保存。  单机->分布
算法：ts，py，
ps：spark，
通信：spark，
梯度：A-SGD，
```



### 论文

| name             | code | ts,py,caffe | ps   | async         | centralized   | company |
| ---------------- | ---- | ----------- | ---- | ------------- | ------------- | :------ |
| bigdl on spark   | yes  | no          |      |               | centralized   | intel   |
| spark on angel   | yes  | no          | no   |               |               | tencent |
| caffeonspark     | yes  | yes         |      |               |               | yahoo   |
| firecaffe        | code | yes         | yes  | nocentralized |               |         |
| sparknet         | yes  | yes         |      | no            | decentralized |         |
| mpc sgd on spark | yes  | no          |      | yes           |               |         |
| sparknlp         | yes  | yes         |      |               |               |         |

```
parameter和mapreduce结构都会有网络瓶颈。
```



```
并行流水线(ts,py):https://paperswithcode.com/paper/gpipe-efficient-training-of-giant-neural#
```

```
关于batch_size,iteration,epoch:https://blog.csdn.net/stay_foolish12/article/details/107386434
```

```
sparkonangel :ps
sparknlp:caffe,ps
mpc:ps
```



```
算法并行，分布式计算
opencl，sparkcl,Aparapi
java djl，sarpk nlp，sparkdeeplearning
```

```
pettum：ssp模型
sparknetx:https://blog.csdn.net/chang_ge/article/details/52687461
知乎介绍：https://zhuanlan.zhihu.com/p/81784947
Intel的一个框架:BigDL
腾讯的一个参数服务器框架:Spark on Angel
模型在框架间迁移：MLeap，https://www.dazhuanlan.com/liyan910117/topics/978744
微软框架：mmlspark
最有名的框架：deeplearning4j，http://link.zhihu.com/?target=https%3A//github.com/eclipse/deeplea rning4j
雅虎： tensorflow on spark，http://link.zhihu.com/?target=https%3A//github.com/yahoo/ TensorFlowOnSpark
ps框架：ps-lite
ps allreduce：https://zhuanlan.zhihu.com/p/100012827
框架
ps框架：https://github.com/rjagerman/glint
框架：https://github.com/databricks/spark-deep-learning
谷歌：DistBelief，实现了A-SGD和ps架构。
```

**Intel的一个框架:BigDL**

**针对nlp的：https://github.com/JohnSnowLabs/spark-nlp**

**腾讯angel的介绍：https://segmentfault.com/a/1190000010471835**
