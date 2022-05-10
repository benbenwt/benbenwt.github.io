### np和scipy matrix

```
https://blog.csdn.net/Scythe666/article/details/84623786
```

```
 my_matrix = scipy.sparse.csr_matrix((2,2))
 my_array = my_matrix.A
 sA = sparse.csr_matrix(A)
```

### np

```
https://www.cnblogs.com/moon1992/p/4946717.html
x=np.array[1,2,3,4]
x[0:3:2]
分号之间分别代表起始，终止，步长
```



### csv

```
f=open(os.path.join(base_directory,'attckDescription.csv'))
csvWriter=csv.writer(f)
csvWriter.writerow(['Text','Tacticid','techniqueid'])
csvWriter.writerows(rowList)
```

```
my_reader=csv.reader(fp,delimiter=',')
for row in my_reader
```



### 安装

>源码安装

>https://blog.csdn.net/enter89/article/details/99681716

# numpy

## nan值和inf值的检查及替换

>https://www.jianshu.com/p/a91246e6b83b

# venv

```
pip install virtualenv或apt-get install virtualenv
virtualenv venv
指定python解释器
virtualenv -p /usr/bin/python2.7 venv
激活虚拟环境
source venv/bin/activate
退出虚拟环境
deactivate
```

windows

```
cd  venv/Scripts
activate
```



# FLASK

### flask

```
flask run  -p 8888 -h 0.0.0.0
```



### flask sqlalchamy

```
SQLALCHEMY_DATABASE_URI = 'mysql+pymysql://root:root@172.42.0.102:3306/teacher' 
SQLALCHEMY_BINDS = {
    'lisa': 'mysql+pymysql://lisa:lisa@172.42.0.14:3306/lisadb',
    'platform':'mysql+pymysql://root:root@172.42.0.102:3306/platform'
}
app=Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = SQLALCHEMY_DATABASE_URI
app.config['SQLALCHEMY_BINDS'] = SQLALCHEMY_BINDS
app.config['SQLALCHEMY_TRACK_MODIFICATIONS']=False
db = SQLAlchemy(app,engine_options={"pool_size":20})
```

```
from app import db
class Malware(db.Model):
    __bind_key__ = "lisa"
    __tablename__ = 'celery_taskmeta'
    id = db.Column(db.Integer, primary_key=True)
    task_id = db.Column(db.String(155))
    status = db.Column(db.String(50))
    result = db.Column(db.BINARY)
    date_done = db.Column(db.DateTime)
    traceback = db.Column(db.Text)
    id_ = db.Column(db.String(155))
```

```
malwareids = db.session.query(Malware.id_).all()
malware = Malware(id_=md5, status='SUCCESS', traceback='importmodule')
            db.session.add(malware)
```

### slqalchamy

```
# from sqlalchemy import create_engine
# from sqlalchemy.orm import declarative_base, sessionmaker
#
# lisaEngine =create_engine('mysql+pymysql://lisa:lisa@172.18.65.185:33306/lisadb',
#                       encoding='utf-8',pool_size=20) #可以加echo=True显示数据
#
#
# lisaSessionClass=sessionmaker(bind=lisaEngine)
# lisaSession=lisaSessionClass() # 生成session实例相当于cursor游标
#
#
# platformEngine =create_engine('mysql+pymysql://root:root@172.18.65.185:3306/platform',
#                       encoding='utf-8',pool_size=20) #可以加echo=True显示数据
#
# platformSessionClass=sessionmaker(bind=platformEngine)
# platformSession=platformSessionClass() # 生成session实例相当于cursor游标

```

```
Base=declarative_base()

class Malware(Base):
    __bind_key__="lisa"
    __tablename__ = 'celery_taskmeta'
    id = Column(Integer, primary_key=True)
    task_id = Column(String(155))
    status=Column(String(50))
    result=Column(BINARY)
    date_done = Column(DateTime)
    traceback = Column(Text)
    id_=Column(String(155))
```

```
platformSession.session.add(pcap)
platformSession.commit()
pcapids = platformSession.query(Pcap.md5).all()
```



```
说明
python中时间日期格式化符号：

%y 两位数的年份表示（00-99）
%Y 四位数的年份表示（000-9999）
%m 月份（01-12）
%d 月内中的一天（0-31）
%H 24小时制小时数（0-23）
%I 12小时制小时数（01-12）
%M 分钟数（00=59）
%S 秒（00-59）
%a 本地简化星期名称
%A 本地完整星期名称
%b 本地简化的月份名称
%B 本地完整的月份名称
%c 本地相应的日期表示和时间表示
%j 年内的一天（001-366）
%p 本地A.M.或P.M.的等价符
%U 一年中的星期数（00-53）星期天为星期的开始
%w 星期（0-6），星期天为星期的开始
%W 一年中的星期数（00-53）星期一为星期的开始
%x 本地相应的日期表示
%X 本地相应的时间表示
%Z 当前时区的名称
%% %号本身
```



# mysql

>python使用传统的mysqlcient，必须安装对应的平台依赖，不然无法安装，按照如下步骤安装。

```
yum install mysql-devel python-devel
pip install mysqlclient
```



# pycharm调试快捷键

```
浏览器能访问翻墙的网站，requests请求不到。
用pycharm编写，import目录从项目根开始写，但是分开执行时，不能从根开始写。只能从当前py文件所处目录开始写。
```



```
F7 ，下一步
shift+F8，跳出此函数
```

```
控制台乱码:setting->editor->三个coding
```





### pycurl

```
https://www.cnblogs.com/angle6-liu/p/12401217.html
```

### sklearn

```
y_true = [0, 1, 2, 0, 1, 2,1, 2, 0, 1, 2]
y_pred = [0, 2, 1, 0, 0, 0,1, 0, 1, 1, 2]
p=metrics.precision_score(y_true, y_pred, average='macro')
r=metrics.recall_score(y_true, y_pred, average='macro')
f1=metrics.f1_score(y_true, y_pred, average='macro')
a = metrics.accuracy_score(y_true, y_pred)

print(f'macro: {p},{r},{f1},{a}')

p = metrics.precision_score(y_true, y_pred, labels=[0],average='micro')
r = metrics.recall_score(y_true, y_pred,labels=[0], average='micro')
f1 = metrics.f1_score(y_true, y_pred,labels=[0], average='micro')
a = metrics.accuracy_score(y_true, y_pred)

print(f'micro: {p},{r},{f1},{a}')

labels参数表示要使用的类别有哪些
```



# miniconda

```
https://blog.csdn.net/weixin_43141320/article/details/108343528
```



# selenium

```
教程：https://www.jianshu.com/p/1531e12f8852
驱动地址及版本选择：https://sites.google.com/a/chromium.org/chromedriver/downloads/version-selection
```

# PYTHON

## pymysql

```
conn = pymysql.connect(
        host='******.com',
        user = 'test',
        password = 'test',
        db = 'market_test',
        charset = 'utf8'
    )
cur = conn.cursor()
sql_countAll = "select count(*) from record where createtime>'%s' and createtime<'%s';" %(timeStart, timeEnd)
cur.execute(sql_countAll)
countAll = cur.fetchall()[0][0]
print("订单数：",countAll)
```



## requests库

```
requests.post(url,body)
requests.get(url,param)
```



## python语法糖或常用函数

```
__call__   person（）
__repr__
__getitem__ person[0:3]
```



## python替换换行符

```
str.replace("\r","").replace("\n",""),必须两个都替换
```

## matplot绘图

```
https://blog.csdn.net/qq_41262248/article/details/79839998
```



## httpServer

```
python -m http.server 端口号
```

## subprocess

```
https://www.cnblogs.com/xiugeng/p/8874227.html
import subprocess
subprocess.Popen(['cd '], stdout=subprocess.PIPE, shell=True)
```



```
pip install ipykernel
jupyter kernelspec list
jupyter kernelspec remove kernelname
#添加环境到jupyter
python -m ipykernel install --user --name 虚拟环境名 --display-name Jupyter中要显示的名字
#打开jupyter
Jupyter notebook
#jupyter自动补全
pip install jupyter_contrib_nbextensions
jupyter contrib nbextension install --user
jupyter contrib nbextension install --user --skip-running-check
```





# Anaconda

##### anaconda pip 错误

```
error : ImportError: cannot import name 'InvalidSchemeCombination' from 'pip._internal.exceptions'

https://stackoverflow.com/questions/67446140/importerror-cannot-import-name-invalidschemecombination-from-pip-internal-e
```



##### 安装jupyter

```
https://www.cnblogs.com/fwl8888/p/9586211.html
conda install jupyter notebook
```

##### 将python kernel安装到jupyter

```
切换jupyter python环境
https://www.jianshu.com/p/b34866f7334a
conda activate angr
conda install ipykernel
#安装内核
python -m ipykernel install  --name your_env
python -m ipykernel install --user --name your_env --display-name "your_display"
```



```
#jupyter切换虚拟环境,仅供
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

### conda操作命令

##### 安装

```
参考官网
```

##### 创建新环境

```
#命令
conda create -n lisa --clone base
conda create -n pytorch python=3.6
activate pytorch
#源管理
conda config --show-sources
conda config --remove channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/
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
#删除环境

conda remove -n env_name --all
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

  1）conda list 查看安装了哪些包。

    2）conda env list 或 conda info -e 查看当前存在哪些虚拟环境

    3）conda update conda 检查更新当前conda
```

##### 常用

```
conda remove -n env_name --all
delete the envs directory.......
conda create -n pytorch python=3.6
conda activate tensorflow_2
conda install jupyter notebook
conda install ipykernel
#安装内核
python -m ipykernel install  --name your_env
python -m ipykernel install --user --name your_env --display-name "your_display"


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
官网下载centos平台的压缩包，解压到目录。
cd Python.x.x
yum remove python3 python3.x
#python依赖
yum install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gcc make
./configure prefix=/usr/local/python3
make && make install
```



# pip

##### 查看版本

```
pip list
pip show 库名
```



##### 导出requiments

```
pip3.exe freeze > requirements.txt
pip3 freeze > requirements.txt
```



##### 安装keras_contrib

```
https://blog.csdn.net/qq_32863339/article/details/102791024
```



```
python -m pip install --upgrade pip
安装离线whl
pip install 1.whl
#read timeout
--default-timeout=1000
```

```
pip config unset global.index-url
pip config list
```

##### 使用源

```
#使用清华源
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple some-package
设为默认
pip install pip -U
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
升级pip
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple pip -U
临时更换源
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple tensorflow
pip install pythonModuleName -i https://mirrors.aliyun.com/pypi/simple
pip install pythonModuleName --extra-index-url https://mirrors.aliyun.com/pypi/simple
阿里的：https://mirrors.aliyun.com/pypi/simple
豆瓣的：http://pypi.douban.com/simple/
```

```
#pyinstaller
pyinstaller -F test.py
#重装pip
python3 -m pip install --upgrade --force-reinstall pip
```



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



##### 安装tensorflow

```
pip install --upgrade tensorflow
ts官网:https://tensorflow.google.cn/install/pip
```

##### 关于命令行调用

```
#关于为什么可以直接使用superset命令，因为它将site-packages/superset/bin/superset软连接到了/usr/bin或添加了环境变量。有时候不是放在对应安装包的bin目录下，而是在python环境的目录，gunicorn就是这样，如conda/env/my_python/bin/gunicorn
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

##### AttributeError: 'Tensor' object has no attribute '_keras_history' ****

```
https://blog.csdn.net/m0_49621298/article/details/115535976
通过keras_contrib导入crf_losses，在使用y_pred,y_true计算损失值时，y_pred._keras_history[:2]报错，表示tensor无此属性。
版本问题，使用 tensorflow-gpu==2.1.0，keras==2.3.1，keras-contrib==2.0.8主要是keras和tensorflow的版本可能过高。
```

##### load_weights报错AttributeError: ‘str‘ object has no attribute ‘decode‘

```
https://blog.csdn.net/canpian7/article/details/114883999
pip install h5py==2.10 -i https://pypi.douban.com/simple
然后从其Jupyter的内核即生效
```

#### requests库解码失败

```
python3.6.此代码在windows正常执行，centos7报错。
response=requests.post('redqueen')
print(response.text)
#输出为乱码
print(response.content.decode('utf-8'))
#decode 解码失败，有无法识别字符

观察response的headers如下：
content-encoding: br
content-type: text/html;charset=utf-8
request的headers如下:
accept-encoding: gzip, deflate, br

可见是由于br压缩格式导致的，可将request的br去掉，不接受br格式即可。或使用python库解压缩br格式，然后再解码打印。
```

##### ImportError: DLL load failed while importing _sqlite3

```
https://www.sqlite.org/download.html，下好了移动到anaconda的DLLS目录
```

```
http://www.nltk.org/nltk_data/，浏览器进入，然后点击"不安全"->"网站设置",允许自动下载和弹窗
```

