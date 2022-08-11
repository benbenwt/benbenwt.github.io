[TOC]
# 安装

```
sudo yum install -y gcc gcc-c++ libffi-devel python-devel python-pip python-wheel python-setuptools openssl-devel cyrus-sasl-devel openldap-devel MySQL-python mysql-devel
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
#关于为什么可以直接使用superset命令，因为它将site-packages/superset/bin/superset软连接到了/usr/bin或添加了环境变量。有时候不是放在对应安装包的bin目录下，而是在python环境的目录，gunicorn就是这样，如conda/env/my_python/bin/gunicorn
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

###### no module named socketserver
>pip install -U werkzeug

# superset用法

>1创建databases，在选项中填写数据库连接即可，然后点击test connection，测试是否可以连接。
>
>2创建tables，这是为了指定你需要使用那些数据，在这里选择需要的表即可。
>
>3创建dashboard，dashboard用于容纳多个chart图形，也就是数据大屏。
>
>4创建charts，在左侧选择tables数据源，并选择图形效果，如饼状图、柱状图等。在左侧可以指定时间字段，分组的字段，分组的聚合函数等。

## superset迁移dashboard

>https://blog.csdn.net/weixin_39358657/article/details/104399895

```
superset export_dashboards -f dashboards.json
superset import_dashboards -f dashboards.json

#如果此方法无效，可以尝试直接复制superset的sqlite数据库文件，其在当前用户的.superset目录下，即~/.superset
```



