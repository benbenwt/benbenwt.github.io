### 安装

http://hbase.apache.org/book.html#quickstart

1下载安装包解压，在conf/hbase-env.sh中设置jdk路径

2快捷启动：运行bin/start-hbase.sh,访问http://localhost:16010

### hbase shell

./bin/hbase shell启动shell

### 与关系型数据库基本结构对比

传统数据库一个表的结构如下：

| 姓名       | 年龄 | 性别       | 成绩 |
| :--------- | :--- | :--------- | :--- |
| liujinghui | 18   | strong man | 100  |
| zhangsan   | 99   | Niangs     | -1   |

转换成HBase数据库的表结构就如下所示

|             | Info      | Score    |          |            |             |
| ----------- | --------- | -------- | -------- | ---------- | ----------- |
| **Row_key** | Info:name | Info:age | Info:sex | Score:name | Score:score |
|             |           |          |          |            |             |

通过行key，列族：列确定一个单一元素