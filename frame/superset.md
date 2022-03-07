# 安装

```
sudo yum install -y gcc gcc-c++ libffi-devel python-devel python-pip python-wheel python-setuptools openssl-devel cyrus-sasl-devel openldap-devel
```



### 安装anaconda创建python环境

```
conda create -n superset python==3.7
conda activate superset
pip install --upgrade setuptools pip -i https://pypi.douban.com/simple/
pip install apache-superset -i https://pypi.douban.com/simple/
#这几个库版本不正确，需要重新安装
pip install markupsafe==2.0.1
pip install WTForms==2.3.3
pip install sqlalchemy==1.3.24
#初始化数据库
superset db upgrade
export FLASK_APP=superset
superset fab create-admin
superset init
```

### 配置gunicorn

```
pip install gunicorn -i https://pypi.douban.com/simple/
gunicorn --workers 5 --timeout 120 --bind hadoop102:8787  "superset.app:create_app()" --daemon 
访问http://hadoop102:8787
ps -ef | awk '/superset/ && !/awk/{print $2}' | xargs kill -9
```

### problem

###### 显示缺少模块，或者缺少模块里的函数

```
markupsafe==2.0.1
WTForms==2.3.3
sqlalchemy==1.3.24
```



# superset使用

```
创建datasource，创建table，创建dashboard，创建table并保存到dashboard。
table左侧选择指标，需要指定数据的时间范围，数据的query那些列，以及数据的group by和过滤条件。
```

