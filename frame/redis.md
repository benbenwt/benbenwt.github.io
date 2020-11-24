# NoSQL

有四类

## 文档类

如moongodb。mongodb是一个基于分布式存储的数据库，c++编写。mongodb介于关系型和非关系型数据库之间。

## key，value类

如redis（remote Dictionary server）

## 图形数据库  

 存储社交关系，应用在广告推荐。node4j

## 列存储数据库

 HBase  应用在分布式文件系统

# 应用场景
1内存持久化
2效率高，高速缓存
3发布订阅系统
4地图信息分析
5计数器

# 特性
多样的数据类型，持久化，集群，事务。
redis是基于内存操作，性能与cpu无关，只受内存和网络宽带影响，所以使用单线程。

# 使用

1解压后make编译，进入src 使用make install安装。redis-servr  redis.conf启动
2redis.cli 启动客户端，ping测试连接
3基本操作：set，get，keys *
redis.conf配置访问端口，能否远程等。

## 和springboot
导入依赖，在application.xml配置好host，端口

## redis-benchmark
redis-benchmark -h [localhost](http://localhost/) -p -6379 -c 100 -n 100000
c为并发客户端量，n为请求数量。
​
​测试并发量

## redis-cli
redis客户端

##  数据库
默认16个数据库
​select 7  切换数据库
DBSIZE 查看dbsize
flushdb 清空当前db
FLUSHALL 清空所有db

## 五大数据类型

消息队列，中间件，订阅消息，数据库，缓存。

## String

set name bob
 get name EXISTS name
 move name 1 
EXPIRE    name 10 
ttl name剩余生命
 type name 查看key的类型 
APPEND key1 ‘hello’

incr
 decr 
INCRBY count 10 
DECRBY count 20
 GETRANGE key1 0 3 [] 
GETRANGE key2 0 -1 获取所有，即get key2
 SETRANGE key  1 xx替换指定位置开始的字符串
 setex key3 30 设置过期时间 setnx mykey 'redis'若不存在
set mset k1 v1 k2 v2 
mget k1 k2 k3 
msetnx k1 v1 k4 v4

set user:1{name:zhangsan,age:1}
 mset user:1:name zhangsan user:1:age 
getset db redis  不存在返回nil，否则获取并更新 



## 场景
计数器  uid:9645454:follow 0 incr
​粉丝数
对象缓存存储
统计多单位的数量

## List
LPUSH list one
LRANGE list 0 -1先插入的下标大，后插入的小。
​Rpush list good 放置在list右边
​Lpop list
Rpop 
​Llen
lindex list  0
​lrem list 1 one
lrem list 2 three
​ltrim   list 1 2
rpoplpush mylist myotherlist
​exists list
lset   key index  value
lset key  before  value  insertvalue

## Set
sadd myset  world
sismember myset hello
​srandommember  myset
smembers myset
spop  myset随机删除
smove  myset  myset2  "world"
sdiff   set1 set2
sinter set1 set2
​sunion set1 set2

## Hash
key-map结构,map由hash值和所设定的value构成。
hset myhash name bob
hmset  myhash field1 hello field2 world
​hmget myhash field1 field2
​hgetall myhash
hdel myhash field1
hlen myhash
hexists
hkeys myhash
hincrby myhash field1 1
hsetnx myhash field1 hello
​

## Zset

# 持久化RDB

## 机制描述

>隔一段时间保存数据快照snapshot到硬盘

## 触发时机

### 基于默认配置

save 900 1

若900秒内发生了一次修改，则保存。

save 300 10

save 60 10000

### 手动保存

save ，bgsave

flushall 清空数据库，同时触发保存到硬盘。

shutdown推出redis，触发保存。

### 配置项

save    禁用RDB机制

dbfilename  文件名，如dump.rdb  设置RDB机制，数据存储文件的文件名。

dir  redis工作目录，存放持久化文件的目录，指定为目录不是文件名。

dump。rdb保存二进制数据，可以用来备份恢复。

# 持久化AOF

>保存数据操作命令，恢复时执行一遍命令。

根据aof文件内容决定保存策略。