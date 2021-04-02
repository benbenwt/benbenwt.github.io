csdn:https://blog.csdn.net/delevindata/article/details/97385258

官网:https://dev.grakn.ai/docs/workbase/overview

```
grakn server start/stop
grakn console 
./grakn console --keyspace grakn_example --file /home/schema.gql
./grakn console -k grakn_example

// 查询person这个实体
match $p isa person; get;
// 查询person中实体有full-name属性的
match $p isa person, has full-name $n; get;
// 查询有employer,employee这两个关系的
match $emp (employer: $x, employee: $y) isa employment; get;
// 查询person实体中nickname属性为 "Mitzi",并且phone-number中带"+44"的.
match $p isa person, has nickname "Mitzi", has phone-number $pn; $pn contains "+44"; get;
#删除该person
match $p isa person, has email "raphael.santos@gmail.com"; delete $p;
#删除关系
match $t isa travel, has start-date 2013-12-22 via $r; delete $r;
```

```
#版本
grakn-core 1.8.4  grakn-workbase 1.3.5
```

### problem

grakn-workerbase无法连接，使用的workerbase版本不兼容。