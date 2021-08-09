```
spark on angel的文档：https://www.bookstack.cn/read/angel-v3.0/apis-AngelClient.md
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

### 确定技术

```
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
