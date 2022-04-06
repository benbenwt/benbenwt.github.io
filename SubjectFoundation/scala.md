# 基于语法

### 基本类型

>Byte                8位符号补码整数
>
>Short              16位
>
>Int                   32位
>
>Long                64位
>
>Float                 32位
>
>Double              64位
>
>Char                  16位
>
>String                字符序列
>
>Boolean            true或false
>
>Unit                   表示无值，和其他语言中void等同。用作不返回任何结果的方法的结果类型。Unit只有一个实例值，写成()
>
>Null                null或空引用
>
>Nothing          Nothing在Scala的类层级的最低端；是任何其他类型的子类型
>
>Any                   Any是所有其他类的超类
>
>AnyRef             是scala里所有引用类的基类
>
>scala中没有java中的原生类型，上述是对象，scala可以对数字等基础类型调用方法。

### 分支控制及循环

>if else 选择结构
>
>if else if  else
>
>while 循环

### 方法与函数

>方法就是定义在类中的函数
>
>def m(x:Int) =x+3
>
>vla f=(x:Int)=>x+3

### 闭包

>闭包是指可以访问一个函数里边的局部变量的另一个函数，例如
>
>var factor=3
>
>val multiplier=(i:Int)=>i*factor
>
>其中factor就是定义在函数外的自由变量，创建函数的过程就是将这个自由变量捕获而构成的一个封闭函数

### 类

>case class用于模式匹配，如定义了person类
>
>case Person("Alice")  => println("hi alice")
>
>case Person("bob")  => println("hi bob")

### 文件IO

```
val writer=new PrintWrite(new File("test.txt"))
writer.write("hi scala")
write.close()

Source.fromFile("test.txt").foreach{
    print
}
```

### 网络编程

### 多线程编程

### 函数式编程

>通过定义函数式编程的特性，我们可以知道函数式编程是什么。
>
>1只用表达式，不用语句。
>
>2没有副作用
>
>3函数是一等公民    闭包和高阶函数    惰性计算

### Kafka Spark Streaming API

```
# Kafka配置
kafka.broker.list=172.18.65.187:9092,172.18.65.186:9092,172.18.65.185:9092
# Redis配置
redis.host=172.18.65.186
redis.port=6379
```

```
private val properties: Properties = MyPropertiesUtil.load("config.properties")
val broker_list = properties.getProperty("kafka.broker.list")

  // kafka消费者配置
  var kafkaParam = collection.mutable.Map(
    "bootstrap.servers" -> broker_list,//用于初始化链接到集群的地址
    "key.deserializer" -> classOf[StringDeserializer],
    "value.deserializer" -> classOf[StringDeserializer],
    //用于标识这个消费者属于哪个消费团体
    "group.id" -> "gmall0523_group",
    //latest自动重置偏移量为最新的偏移量
    "auto.offset.reset" -> "latest",
    //如果是true，则这个消费者的偏移量会在后台自动提交,但是kafka宕机容易丢失数据
    //如果是false，会需要手动维护kafka偏移量
    "enable.auto.commit" -> (false: java.lang.Boolean)
  )
  
val dStream = KafkaUtils.createDirectStream[String,String](
      ssc,
      LocationStrategies.PreferConsistent,
      ConsumerStrategies.Subscribe[String,String](Array(topic), kafkaParam )
    )
    dStream
```

### Elasticsearch API

```
import io.searchbox.client.{JestClient, JestClientFactory}
jestFactory = new JestClientFactory
    jestFactory.setHttpClientConfig(new HttpClientConfig
        .Builder("http://ip:9200")
        .multiThreaded(true)
        .maxTotalConnection(20)
        .connTimeout(10000)
        .readTimeout(1000).build())
val jestClient:JestClient=jestFactory.getObject
val index:Index=new Index.Builder(source).index("my_index").`type`("movie").id("1").build()
jestClient.execute(index)
jestClient.close()
```

### Phoenix API

```
Class.forName("org.apache.phoenix.jdbc.PhoenixDriver")
    //建立连接
    val conn: Connection = DriverManager.getConnection("jdbc:phoenix:hbase,hbase1,hbase2:2181")
    //创建数据库操作对象
    val ps: PreparedStatement = conn.prepareStatement(sql)
    //执行SQL语句
    val rs: ResultSet = ps.executeQuery()
    val rsMetaData: ResultSetMetaData = rs.getMetaData
```

### Redis API

```
val prop = MyPropertiesUtil.load("config.properties")
val host = prop.getProperty("redis.host")
val port = prop.getProperty("redis.port")

val jedisPoolConfig: JedisPoolConfig = new JedisPoolConfig
jedisPoolConfig.setMaxTotal(100)  //最大连接数
jedisPoolConfig.setMaxIdle(20)   //最大空闲
jedisPoolConfig.setMinIdle(20)     //最小空闲
jedisPoolConfig.setBlockWhenExhausted(true)  //忙碌时是否等待
jedisPoolConfig.setMaxWaitMillis(5000)//忙碌时等待时长 毫秒
jedisPoolConfig.setTestOnBorrow(true) //每次获得连接的进行测试

jedisPool=new JedisPool(jedisPoolConfig,host,port.toInt)
val jedis=jedisPool.getResource
print(jedis.ping())
```



# problem

##### idea无法创建scala类

```
<orderEntry type="library" name="scala-sdk-2.11.8" level="application" />
```

