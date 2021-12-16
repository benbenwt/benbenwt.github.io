##### beta6版本

```
自己的生成的pb文件使用旧的和samediff都工作正常。
SameDiff sd= TFGraphMapper.importGraph(f);
SameDiff sd = SameDiff.importFrozenTF(f);

fb文件可导入，工作失败，java.lang.NullPointerException
```

##### M1.1-1.0.0-SNAPSHOT

```
自己的生成的pb文件使用旧的和samediff都可导入，不可fit。新api直接无法导入
Exception in thread "main" java.lang.UnsatisfiedLinkError: org.nd4j.nativeblas.Nd4jCpu$Environment.isUseMKLDNN()Z
TensorflowFrameworkImporter tensorflowFrameworkImporter = new TensorflowFrameworkImporter();c创建报错...
fb文件导入可以，无法工作。Samediff output op named mask did not have any ops associated with it.
```



```
samedif 实体
input ：128个字对应的id ， label：类别对应的标签
input ：128个字符对应的id ，label：类别对应的标签
bert输入: tokenidx,mask,segmentidx     label
```



```
通用写法：导入，加层，创建数据集，训练，测试，保存模型。
```

##### 导入

```
File f=new File("D:\\deeplearning4j-examples\\mydownload\\chinese_L-12_H-768_A-12\\chinese_L-12_H-768_A-12\\frozen","bert_frozen_mb4_len128.pb");
SameDiff sd= TFGraphMapper.importGraph(f);
```

##### 加层

##### samediff

```
SDVariable labels = sd.placeHolder("label", DataType.FLOAT, 1, 2);

//        截止到close为添加的层
        NameScope my_transfer = sd.withNameScope("loss");
//        取之前model的最后一层
        SDVariable input = sd.getVariable("bert/pooler/dense/Tanh");

        SDVariable my_flatten_weights = sd.var("flatten_weights", new XavierInitScheme('c', 768, 2), DataType.FLOAT, 768, 2);
        SDVariable my_flatten_bias = sd.var("flatten_bias", new UniformInitScheme('c', 2), DataType.FLOAT, 2);
//        将之前model的最后一层乘以my_flatten_weights加上my_flatten_bias存储到
        SDVariable linear_output = input.mmul(my_flatten_weights).add("linear_output",my_flatten_bias);
        SDVariable softmax_output = sd.nn().softmax("softmax", linear_output);
        SDVariable loss = sd.loss().logLoss("Loss", labels, softmax_output);
        my_transfer.close();
//
        sd.setLossVariables(loss);
        sd.addListeners(new ScoreListener(1));
//        System.out.println(sd.summary());
```

```
```



##### NeuralNetConfiguration

```
     MultiLayerConfiguration conf = new NeuralNetConfiguration.Builder()
            .updater(new RmsProp(0.0018))
            .l2(1e-5)
            .weightInit(WeightInit.XAVIER)
            .gradientNormalization(GradientNormalization.ClipElementWiseAbsoluteValue).gradientNormalizationThreshold(1.0)
            .list()
            .layer( new LSTM.Builder().nIn(inputNeurons).nOut(200)
                .activation(Activation.TANH).build())
            .layer(new RnnOutputLayer.Builder().activation(Activation.SOFTMAX)
                .lossFunction(LossFunctions.LossFunction.MCXENT).nIn(200).nOut(outputs).build())
            .build();

        MultiLayerNetwork net = new MultiLayerNetwork(conf);
        net.init();
```



##### 创建数据集

**创建CollectionLabeledPairSentenceProvider数据集**



>CollectionLabeledPairSentenceProvider
>
>BertIterator
>
>BertIterator implements MultiDataSetIterator

```
List<String> sentences = new ArrayList<String>() {{add("这个菜很好吃");
            add("那个商品质量太差了");
            add("差评！太垃圾了！");
            add("非常喜欢这个品类");}};
        List<String> sentencesR = new ArrayList<String>() {{add("");add("");add("");add("");}};
        List<String> label = new ArrayList<String>() {{add("pos");add("neg");add("neg");add("pos");}};
        CollectionLabeledPairSentenceProvider labeledPairSentenceProvider = new CollectionLabeledPairSentenceProvider(sentences, sentencesR, label, new Random(123L));
        File wordPieceTokens = new File("D:\\deeplearning4j-examples\\mydownload\\chinese_L-12_H-768_A-12\\chinese_L-12_H-768_A-12\\vocab.txt");

        BertWordPieceTokenizerFactory t = new BertWordPieceTokenizerFactory(wordPieceTokens, true, true, StandardCharsets.UTF_8);
        BertIterator b = BertIterator.builder()
            .tokenizer(t)
            .lengthHandling(BertIterator.LengthHandling.FIXED_LENGTH, 128)
            .minibatchSize(1)
            .sentenceProvider(null)
            .sentencePairProvider(labeledPairSentenceProvider)
            .featureArrays(BertIterator.FeatureArrays.INDICES_MASK_SEGMENTID)
            .vocabMap(t.getVocab())
            .task(BertIterator.Task.SEQ_CLASSIFICATION)
            .prependToken("[CLS]")
            .appendToken("[SEP]")
            .build();
```

##### NewsIterator implements DataSetIterator

```
NewsIterator iTrain = new NewsIterator.Builder()
            .dataDirectory(DATA_PATH)
            .wordVectors(wordVectors)
            .batchSize(batchSize)
            .truncateLength(truncateReviewsToLength)
            .tokenizerFactory(tokenizerFactory)
            .train(true)
            .build();
```



##### 训练

##### samediff beta6

```
TrainingConfig c = TrainingConfig.builder()
            .updater(new Adam(0.01))
            .l2(1e-5)
            .dataSetFeatureMapping("Placeholder", "Placeholder_1")
            .dataSetFeatureMaskMapping("Placeholder_2")
            .dataSetLabelMapping("label")
            .build();
        sd.setTrainingConfig(c);
        System.out.println("Start Training...");
        long start = System.currentTimeMillis();
        for( int i = 0; i < 50; ++i ){
            sd.fit(datasetIter, 1);
            datasetIter = getSupervisedDataIterator();
        }
```

##### master 1.0.0-M1.1 example

```
History hist = sd.fit()
                .train(trainData, 1)
                .exec();
Evaluation e = hist.finalTrainingEvaluations().evaluation("out");
System.out.println("Accuracy: " + e.accuracy());
```

##### neturalNetConfiguration

```
net.setListeners(new ScoreIterationListener(1), new EvaluativeListener(iTest, 1, InvocationType.EPOCH_END));
net.fit(iTrain, 10);
```



##### 测试

```
Evaluation eval = net.evaluate(iTest);
```

##### samediff

```
 String outputVariable = "softmax";
 Evaluation evaluation = new Evaluation();
 sd.evaluate(testData, outputVariable, evaluation);
 System.out.println(evaluation.stats());
```



##### 保存模型

##### samediff

>https://github.com/eclipse/deeplearning4j-examples/blob/master/samediff-examples/src/main/java/org/nd4j/examples/samediff/quickstart/modeling/MNISTFeedforward.java

```
File saveFileForInference = new File("sameDiffExampleInference.fb");
sd.asFlatFile(saveFileForInference);
SameDiff loadedForInference = SameDiff.fromFlatFile(saveFileForInference);
```



```
net.save(new File(dataLocalPath,"NewsModel.net"), true);
```

**ComputationGraphConfiguration**

```
ComputationGraphConfiguration conf = new NeuralNetConfiguration.Builder()
            .weightInit(WeightInit.XAVIER)
            .updater(new Nesterovs(0.1, 0.9))
            .graphBuilder()
            .addInputs("in")
            .addLayer("layer0", new DenseLayer.Builder().nIn(4).nOut(3).activation(Activation.TANH).build(), "in")
            .addLayer("layer1", new OutputLayer.Builder(LossFunctions.LossFunction.NEGATIVELOGLIKELIHOOD).activation(Activation.SOFTMAX).nIn(3).nOut(3).build(), "layer0")
            .setOutputs("layer1")
            .build();

        ComputationGraph net = new ComputationGraph(conf);
        net.init();


        //Save the model
        File locationToSave = new File("MyComputationGraph.zip");       //Where to save the network. Note: the file is in .zip format - can be opened externally
        boolean saveUpdater = true;                                             //Updater: i.e., the state for Momentum, RMSProp, Adagrad etc. Save this if you want to train your network more in the future
        net.save(locationToSave, saveUpdater);

        //Load the model
        ComputationGraph restored = ComputationGraph.load(locationToSave, saveUpdater);

```







```
temp:
Failed to execute goal org.bytedeco:javacpp:1.5.6:build (javacpp-compiler) on project nd4j-native: Execution javacpp-compiler of goal org.bytedeco:javacpp:1.5.6:build failed: Process exited with an error: 1 -> [Help 1]
```



### 脚本

```
check_cuda.sh 改版本
update.sh 转换版本
buildops.sh 构建libn
can not find openblas:https://community.konduit.ai/t/nd4j-compile-from-source-fails-with-snapshot/1349/3
```



### build from source

```
参考libnd4j下边的readme.md和windows.md,主要是安装msys,然后在msys安装一些环境依赖(gcc,cmake)，还有配置程序内外的环境变量（export path=/c/test），还有程序的依赖(openblas,maven,git)。
github libnd4j：https://github.com/eclipse/deeplearning4j/issues/9265
论坛链接:https://community.konduit.ai/t/build-documentation-out-of-date/180
build cuda:链接:https://community.konduit.ai/t/nd4j-compile-from-source-fails-with-snapshot/1349/4
mvn -B -V -U clean install -pl '!jumpy,!pydatavec,!pydl4j' -DskipTests=true -Dmaven.test.skip=true
build cuda:mvn  -Possrh Djavacpp.platform=linux-x86_64   -Dlibnd4j.chip=cuda clean install -DskipTests

```

```
mvn clean install -DskipTests -Dmaven.javadoc.skip=true -pl ‘!:nd4j-tests’
```

libnd4j编译后要设置LIBND4J_HOME ,让java能引用c程序。

```
github关键词:deeplearning4j实,deeplearning,deeplearning4j nlp
https://github.com/iromu/deeplearning4j-nlp
```



### 资料链接

```
从tensorflow生成pb文件
教程：https://blog.csdn.net/wangongxi/article/details/106718748
该教程使用的tensorflow 1.15.0
依赖如下：
absl-py==0.13.0
argon2-cffi==20.1.0
astor==0.8.1
async-generator==1.10
attrs==21.2.0
backcall==0.2.0
bleach==4.0.0
cached-property==1.5.2
certifi==2021.5.30
cffi==1.14.6
colorama==0.4.4
dataclasses==0.8
decorator==5.0.9
defusedxml==0.7.1
entrypoints==0.3
gast==0.2.2
google-pasta==0.2.0
grpcio==1.39.0
h5py==3.1.0
importlib-metadata==4.6.3
ipykernel==5.5.5
ipython==7.16.1
ipython-genutils==0.2.0
ipywidgets==7.6.3
jedi==0.18.0
Jinja2==3.0.1
jsonschema==3.2.0
jupyter==1.0.0
jupyter-client==6.1.13
jupyter-console==6.4.0
jupyter-core==4.7.1
jupyterlab-pygments==0.1.2
jupyterlab-widgets==1.0.0
Keras-Applications==1.0.8
Keras-Preprocessing==1.1.2
Markdown==3.3.4
MarkupSafe==2.0.1
mistune==0.8.4
nbclient==0.5.1
nbconvert==6.0.7
nbformat==5.1.3
nest-asyncio==1.5.1
notebook==6.4.3
numpy==1.19.5
opt-einsum==3.3.0
packaging==21.0
pandocfilters==1.4.3
parso==0.8.2
pickleshare==0.7.5
prometheus-client==0.11.0
prompt-toolkit==3.0.3
protobuf==3.17.3
pycparser==2.20
Pygments==2.9.0
pyparsing==2.4.7
pyrsistent==0.18.0
python-dateutil==2.8.2
pywin32==301
pywinpty==1.1.3
pyzmq==22.2.1
qtconsole==5.1.1
QtPy==1.9.0
Send2Trash==1.8.0
six==1.16.0
tensorboard==1.15.0
tensorflow==1.15.0
tensorflow-estimator==1.15.1
termcolor==1.1.0
terminado==0.11.0
testpath==0.5.0
tornado==6.1
traitlets==4.3.3
typing-extensions==3.10.0.0
wcwidth==0.2.5
webencodings==0.5.1
Werkzeug==2.0.1
widgetsnbextension==3.5.1
wincertstore==0.2
wrapt==1.12.1
zipp==3.5.0

```

