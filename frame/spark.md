### spark安装

>spark-3.1.2-bin-hadoop3.2,默认we端口8080,绑定失败会用8081端口。

````
安装教程:http://spark.apache.org/docs/latest/
````

##### standalone模式

```
复制压缩包解压即可
sbin/start-master.sh
netstat -lntp
curl localhost:8081
```



```
P17 sprk yanr部署模式

```



```
下载scala包或在idea的project structure中指定自动下载，在setting的plugins中下载scala插件。
创建maven项目，在project structure中添加scala。
sprk3.0.0,scala-2.12
```



##### RDD

```
RDD中操作分为Transformation,Action。对于Transformation是累积的，当有Action执行时才会执行累积的Transformation。Action常见的有collect，map，reduce，groupby等。
map是对一个数组的单个对象重复调用map函数内容。
reduce是将多个对象递归调用，最终归约为一个值。
```

