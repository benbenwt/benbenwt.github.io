### 安装

>官网quick start:https://phoenix.apache.org/Phoenix-in-15-minutes-or-less.html#

```
#解压压缩吧
#将其中的phoenix-server-hbase-2.3-5.1.2.jar拷贝到hbase/lib下，并重启hbase
#启动交互shell
bin/sqlline.py <your_zookeeper_quorum>
#从csv导入数据到hbase，sql文件为建表语句，csv为数据。
./psql.py <your_zookeeper_quorum> us_population.sql us_population.csv
./psql.py 172.18.65.187 us_population.sql us_population.csv
#登入shell
bin/sqlline.py 172.18.65.187
#查看从csv插入的数据
SELECT state as "State",count(city) as "City Count",sum(population) as "Population Sum"
FROM us_population
GROUP BY state
ORDER BY sum(population) DESC;
#如果需要在其他客户端访问phoenix，引入phoenix-client-hbase-2.3-5.1.2.jar 即可。
```

##### sql练习

```
select * from us_population where city like 'Los%';
select * from us_population where population>=1;
select * from  test where TO_DATE(ttime,'yyyyMMddHHmmss')=TO_DATE('20141125','yyyyMMdd')
```

