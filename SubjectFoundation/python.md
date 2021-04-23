##### EMBER

```
#ember安装和配置
Install after cloning the EMBER repository
Use pip or conda to install the required packages before installing ember itself:
#use pip
pip install -r requirements.txt
python setup.py install
#use conda
conda config --add channels conda-forge
conda install --file requirements_conda.txt
python setup.py install
#malconv安装
安装pytorch
```

```
#ember代码
#malconv
    def __init__(self, out_size=2, channels=128, window_size=512, embd_size=8):
        super(MalConv, self).__init__()
        #有257个数据，每个数据8维，补全为0，不进行补全。
        self.embd = nn.Embedding(257,embd_size, padding_idx=0)
        
        self.window_size = window_size
    	#输入向量的长度，词向量维度。卷积产生的通道，有多少个就需要多少个一维卷积。卷积核尺寸。卷积步长
        self.conv_1 = nn.Conv1d(embd_size, channels, window_size, stride=window_size, bias=True)
        self.conv_2 = nn.Conv1d(embd_size, channels, window_size, stride=window_size, bias=True)
        #自适应最大池化，输出尺寸为(1,1)
        self.pooling = nn.AdaptiveMaxPool1d(1)
        #线性结构，前一层神经元和当前层神经元。
        self.fc_1 = nn.Linear(channels, channels)
        self.fc_2 = nn.Linear(channels, out_size)
```



### Anaconda

```
#jupyter切换虚拟环境
conda install nb_conda
conda install -c conda-forge nb_conda_kernels
jupyter serverextension disable nb_conda
jupyter serverextension enable nb_conda
```

```
#jupyter ipprocess import error
1.pip install ipywidgets
2.jupyter nbextension enable --py widgetsnbextension
```



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
#安装pytorch
进入pytorch生成对应命令
conda install pytorch torchvision torchaudio cpuonly -c pytorch --channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch/
#安装tensofrflow
conda install tensorflow -c https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda install --offline 包名
#离线安装mkl
到https://mirrors.bfsu.edu.cn/anaconda/pkgs/free/win-64/下载tar包
到anaconda\Lib\site-packages\下创建tensorflow文件夹，放入tar包
使用conda进入该目录，执行conda install --offline  -f 包名

conda clean --all
conda clean --packages --tarballs
conda update --all
conda update --strict-channel-priority --all
conda clean -i 
ping mirror.tuna.tsinghua.edu.cn

conda config --show-sources
conda list
conda info
#创建新环境用conda创建，不要用pycharm，会创建失败。
```

```
#.condarc配置,使用命令添加后仍无法下载。删除channels下的defaults，https改为http，添加后缀win-64
channels:
  - conda-forge
  - http://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch/win-64
  - http://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/msys2/win-64
  - http://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/win-64
  - http://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/win-64
  - http://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/win-64
remote_read_timeout_secs: 1000.0
show_channel_urls: true

```



### 编译安装python

```
yum remove python3 python3.x
#python依赖
yum install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gcc make
./configure prefix=/usr/local/python3
make && make install
```



### pip

```
临时更换源
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple tensorflow
pip install pythonModuleName -i https://mirrors.aliyun.com/pypi/simple
pip install pythonModuleName --extra-index-url https://mirrors.aliyun.com/pypi/simple
阿里的：http://mirrors.aliyun.com/pypi/simple/
豆瓣的：http://pypi.douban.com/simple/
```

```
#pyinstaller
pyinstaller -F test.py
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

##### collecting package metadata卡住

```
删除.condarc
```

