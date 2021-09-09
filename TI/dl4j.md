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

