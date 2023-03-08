## 可能问题
### 数据量，es和mysql对应的问题
#### es mapping设计
#### es 集群管理
>数据库实例之间、reindex reroute(allocate、move、cancel)
##### es集群状况分析
>使用es-head查看所有索引，索引分片分布，集群健康状态颜色,红色、黄色、绿色，分别表示不同的告警级别，对应主分片unsigned、副本分片unsigned、都已allocation。可能是内存不足、节点不可达导致的。
```
#查看分片分布
 curl -XGET 'http://172.17.161.205:9200/_cat/shards?h=index,shard,prirep,state,unassigned.reason'
 #修改分片配置
 curl -XPUT 'http://localhost:9200/_settings' -d'
{
  "number_of_replicas": 0
}'
```
##### 索引数据迁移
```
#使用reindex迁移到新索引
POST _reindex { "source": { "index": "my_index_name" }, "dest": { "index": "my_index_name_new" } } 
```
##### 数据库实例之间数据迁移
>使用logstash，编写conf文件，定义input、output，填写其ip和index以及scroll的size。

##### es结点扩展
```
将压缩包解压到新的机器，修改配置文件添加对应host，拷贝es/data的数据到新的机器
#通过禁用rebalance，可以避免花费很久时间
PUT /_cluster/settings
{
    "transient" : {
        "cluster.routing.allocation.enable" : "none"      //取消分片权衡
    }
}
#使用此命令引入新的机器ip，会将分片迁移到对应机器
PUT index_name/_settings
{
  "index.routing.allocation.include._ip": "10.124.105.5,10.124.105.6,10.124.105.7"
}

#手动强制迁移,需要先关闭自动分配。
PUT /_cluster/settings
{
    "transient" : {
        "cluster.routing.allocation.enable" : "none"      //取消分片权衡
    }
}
#然后手动迁移
# node-3分片1迁移到node-2
POST /_cluster/reroute
{
    "commands" : [
        {
            "move" : {
                "index" : "top_n_database_statement-20200519", "shard" : 1,
                "from_node" : "node-3", "to_node" : "node-1"
            }
        }
    ]
}
# node-3分片2迁移到node-1
POST /_cluster/reroute
{
    "commands" : [
        {
            "move" : {
                "index" : "top_n_database_statement-20200519", "shard" : 2,
                "from_node" : "node-3", "to_node" : "node-1"
            }
        }
    ]
}

```

##### es分片和段合并
>一个索引默认5个主分片，每个主分片可以设置副本分片，一个分片就是一个lucene实例，是一个完整的搜索引擎服务。
>一个分片包含多个segment段。
>数据分配到对应dataNode后，等待refresh，默认一秒，然后生成一个segment并写入磁盘translog。一段时间后，进行flush到磁盘并清空translog。
#### mysql带过滤条件的深度分页
>例如 select * from record where type='pcap' limit 10000,20
>带过滤条件的查询，自然是需要回表的，那么如上的回表数据量就为10020，可以通过在子查询中提前limit，再去回表，减少回表的数据量。
>select * from record where id in (select id from record where type='pcap' limit 10000,20),此时回表数据量为20。

>对于查询，尽量索引覆盖到过滤或查询的字段，当无法覆盖到查询字段时，致力于减少回表查询聚集索引的数据量。
#### grpc
>编写proto并生成java、python代码。
>server.start()
>client.getAptResult()
#### 数据开发的八股，总体性的流程、特性
### grpc
#### 理论
##### 基本概念
>该系统基于 HTTP/2 协议传输，可以实现多路复用的长连接，效率更高
>channel是对 特定grpc server的主机端口的连接
>stub是在channel基础上创建的，stub用于向server发送请求调用
>封装好request后，由stub调用方法并传入request，然后接收response
>serverBuilder用于根据ip和端口创建服务，并向server添加Service类，即可提供服务。
###### proto配置文件

#### helloworld
>1.编写proto文件，定义service和message
>2.编译proto生成service和message对应的java代码、python代码等。
>3.创建server
>3.创建client

### grpc-spring-boot-starter
>https://cloud.tencent.com/developer/article/1608265
>通过@GrpcService注册rpc service到注册中心。
>通过@GrpcClient("spring-boot-grpc-server")调用服务

## 用户登入登出
>https://www.zhihu.com/question/291135431
>前后端分离鉴权，分布式session登陆状态
>分开的tomcat服务怎么实现的登录验证

>令牌：https://blog.csdn.net/qq_33012981/article/details/108990530
>token可以实现登入，无法踢下线和登出。
>https://blog.csdn.net/weixin_42382291/article/details/106209359
### 单点登录
### 单点登出
>挤下线，踢下线
### 权限管理
>

## 数据安全验证
```
https://www.cnblogs.com/54chensongxia/p/14016179.html
https://blog.csdn.net/q343509740/article/details/80914939

#添加依赖
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>6.0.1.Final</version>
</dependency>

#使用注解
public class UserDTO {

    private Integer userId;

    @NotEmpty(message = "姓名不能为空")
    private String name;
    
    @Range(min = 18,max = 50,message = "年龄必须在18和50之间")
    private Integer age;
    
    @DecimalMin(value = "0.00", message = "费率格式不正确",groups = UpdateFeeRate.class)
    @DecimalMax(value = "100.00", message = "费率格式不正确",groups = UpdateFeeRate.class)
    private BigDecimal gongzi;
    //省略get和set方法
}

@Length
@Pattern()

#
@RestControllerAdvice
实现异常全局处理，切面
```

## 支付
>使用alipay api
>https://blog.csdn.net/ybsgsg/article/details/124348842

### 基本概念
>用户私钥：在向支付宝发送请求时，需要使用用户私钥加密，然后支付宝会使用用户公钥解密。
>用户公钥：在支付宝沙箱配置好，用于解密用户发出的数据
>支付宝公钥：用于解密支付宝返回的响应信息。

## 支付方式
### 网页付
>不通过接口异步通知，直接在支付完成后，在浏览器前端请求return_url，进行跳转。
>其检测付款结果的方式包括：
>1,前端请求后端接口
>2,轮询检测
### 当面付
>https://opendocs.alipay.com/open/194/106078?ref=api
>当面付异步通知参数:https://opendocs.alipay.com/pre-open/00a9jk
>通过扫描商家生成的对应于订单的二维码，进行付款操作。
>其检测付款结果的方式包括：
>1,异步通知接口，支付宝服务端通过请求接口notify_url，通知付款结果，必须提供公网地址 
>2,轮询请求alipay.trade.query。