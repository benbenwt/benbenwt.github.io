### Anaconda

```
#安装

#命令
conda create -n lisa --clone base
conda create -n pytorch python=3.6
activate pytorch
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge 
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/msys2/

conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch/



conda config --set remote_read_timeout_secs 1000.0
进入pytorch生成对应命令
conda install pytorch torchvision torchaudio cpuonly -c pytorch --channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch/

conda install --offline 包名
conda clean --all
conda clean --packages --tarballs
conda update --all
conda update --strict-channel-priority --all
conda clean -i 
ping mirror.tuna.tsinghua.edu.cn
#创建新环境用conda创建，不要用pycharm，会创建失败。
```

```
#.condar
channels:
show_channel_urls: true
channel_alias: https://mirrors.tuna.tsinghua.edu.cn/anaconda
default_channels:
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/pro
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/msys2
custom_channels:
  conda-forge: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  msys2: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  bioconda: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  menpo: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  pytorch: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  simpleitk: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
ssl_verify: false
```





### 编译安装python

```
yum remove python3 python3.x
#python依赖
yum install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gcc make
./configure prefix=/usr/local/python3
make && make install
```

```
#下载卡住快照
删除channels下的defaults，https改为http，添加后缀win-64

```



### pip

```
临时更换源
pip install pythonModuleName -i https://mirrors.aliyun.com/pypi/simple
pip install pythonModuleName --extra-index-url https://mirrors.aliyun.com/pypi/simple
阿里的：http://mirrors.aliyun.com/pypi/simple/
豆瓣的：http://pypi.douban.com/simple/
```

### problem

##### pycountry不可使用

>安装Pycountry,首先用pyi-makespec --onefile [temp2.py](http://temp2.py)创建一个spec，然后修改spec，再PyInstaller --clean temp2.spec生成程序。

```
#原文
Presumably pycountry uses pkg_resources.get_distribution("pycountry") somewhere to access its own metadata (usually to set a __version__ attribute). But PyInstaller doesn’t collect that by default. To include it you need to use the spec file. It looks like your code is named [temp2.py](http://temp2.py) so your spec file will be called temp2.spec. 

1.Open temp2.spec (it’s just a Python script so you can use your Python editor). Then put the following at the top:
from PyInstaller.utils.hooks import copy_metadata

2.And in the a = Analysis(...) section change:
    datas = [],
to
    datas = copy_metadata("pycountry"),

3.Then rebuild using:
PyInstaller --clean temp2.spec
```

