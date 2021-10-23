```
挖掘，可信，共享，推断，资产发现，资产加情报
```



```
硕博论文集：ProQuest
计算机相关：dblp
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

### elephas

```
一个是http，一个是用rdd返回的结果直接处理的。
```

### 正确版本

```
https://github.com/maxpumperla/elephas/issues/82
https://github.com/maxpumperla/elephas/issues/146
使用elephas==1.0.0版本,会自动安装其他依赖。
spark
java
Python 3.7.9
tensorflow==2.1.3
pyspark==3.0.1
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
```



### elephas

```
elephas
"apache spark" AND "deep learning" 
```

##### 修改记录

```
AttributeError: 'Sequential' object has no attribute 'compiled_metrics'
spark_model.py , line 47
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



bert-lstm-crf教程

```
csdn教程:https://blog.csdn.net/jclian91/article/details/111728692
github资源:https://github.com/percent4/keras_bert_sequence_labeling
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



```
AttributeError: module 'tensorflow' has no attribute 'placeholder'
修改tensorflow_backend文件,optimizers.py
import tensorflow.compat.v1 as tf
tf.disable_v2_behavior()
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

