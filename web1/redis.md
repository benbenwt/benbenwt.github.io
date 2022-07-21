[TOC]

# 应用场景
1内存持久化
2效率高，高速缓存
3发布订阅系统
4地图信息分析
5计数器

## NoSQL

有四类

### 文档类

如moongodb。mongodb是一个基于分布式存储的数据库，c++编写。mongodb介于关系型和非关系型数据库之间。

### key，value类

如redis（remote Dictionary server）

### 图形数据库

 存储社交关系，应用在广告推荐。node4j

### 列存储数据库

 HBase  应用在分布式文件系统

## 特性
多样的数据类型，持久化，集群，事务。
redis是基于内存操作，性能与cpu无关，只受内存和网络宽带影响，所以使用单线程。

# redis用法
## 安装

>1解压后make编译，进入src 使用make install安装。redis-server  redis.conf启动
>2redis.cli 启动客户端，ping测试连接
>3基本操作：set，get，keys *
>redis.conf配置访问端口，能否远程等。

## Scala API
>scala的 object相当于 java class中的 static作用，如果你编写了一个object，你可以直接用object.func1()进行调用，也就是说这是属于类的方法，不用new对象出来，其没有构造函数，无法传递初始化参数。你也可以理解为这是一个单例对象。
```
import redis.clients.jedis.{Jedis, JedisPool, JedisPoolConfig}

private var jedisPool:JedisPool=new JedisPoolConfig
jedisPoolConfig.setMaxTotal(100)  //最大连接数
jedisPoolConfig.setMaxIdle(20)   //最大空闲
jedisPoolConfig.setMinIdle(20)     //最小空闲
jedisPoolConfig.setBlockWhenExhausted(true)  //忙碌时是否等待
jedisPoolConfig.setMaxWaitMillis(5000)//忙碌时等待时长 毫秒
jedisPoolConfig.setTestOnBorrow(true) //每次获得连接的进行测试

jedisPool=new JedisPool(jedisPoolConfig,host,port.toInt)
client=jedisPool.getResource
```

### List
### Hash
```
client.hset("test","21","21value")
client.hgetAll("test")
client.del("test")
```
## JAVA 原生API

>https://blog.csdn.net/qq_34826261/article/details/105977430

```
#添加依赖，连接服务
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>3.0.1</version>
</dependency>
```

```
#测试连接
public class RedisUtil {
    private static Jedis jedis;
    public static Jedis connectRedis(String host){
        jedis=new Jedis(host);
        return jedis;
    }

    public static void main(String[] args) {
        Jedis jedis= RedisUtil.connectRedis("172.18.65.186");
        System.out.println(jedis.ping());
    }
}
```

| 方法名                        | 功能                                    |
| ----------------------------- | --------------------------------------- |
| flushDB()                     | 清除数据库所有数据                      |
| exists(String key)            | 判断键key是否存在                       |
| expire(String key,int second) | 设置key的过期时间                       |
| ttl(String key)               | key的剩余生存时间，不存在或永远都返回-1 |
| type(String key)              | key所储存值得类型                       |

### String

```
#增加string
jedis.set("one","1")
#查询键所对应的值
jedis.get("two")
#在原来的基础上修改，也就是将值追加到oldValue的尾部
jedis.append("two","修改")
#覆盖修改，新值覆盖旧值
jedis.set("two","覆盖修改")
#删除String
jedis.del("two")
```

### List

```
#增删改查，排序
#清除列表中所有数据
jedis.del("listTest")
#增加
jedis.lpush("listTest","1");
#查询,-1表示所有，查询出来的元素和插入顺序相反
jedis.lrange("listTest",0,-1)
#获取指定坐标的值
jedis.lindex("listTest",2)
#出栈
jedis.lpop("listTest")
#删除
#删除指定元素，第二个参数为删除个数（当由重复元素时）
jedis.lrem("listTest",2,"java")
#删除某一范围以外的数据
jedis.ltrim("listTest",0,3)
#修改指定坐标数据
jedis.lset("listTest",1,"jedis")
#排序,存在字母必须使用SortingParams，若为纯数字可直接使用sort
SortingParams sortingParameters = new SortingParams();
sortingParameters.alpha();
sortingParameters.limit(0, 4);
jedis.sort("listTest",sortingParameters))
```

### Set

```
#集合类型增删改查
jedis.sadd("setTest","1")
#查询
jedis.smemebers("setTest")
#删除
jedis.screm("setTest","1")
#交并差
System.out.println("两集合的交集" + jedis.sinter("setTest", "setTest2"));
System.out.println("两集合的并集" + jedis.sunion("setTest", "setTest2"));
// setTest中有，setTest2中没有的元素
System.out.println("两集合的差集" + jedis.sdiff("setTest", "setTest2"));
```

### Sorted Set

```
public class RedisSortedSetTest {
    public static void main(String[] args) {
        Jedis jedis = RedisUtil.connectRedis("localhost");
        // 增加
        jedis.zadd("sortedSetTest", 3.1, "key1");
        jedis.zadd("sortedSetTest", 7.0, "key2");
        jedis.zadd("sortedSetTest", 5.5, "key3");
        jedis.zadd("sortedSetTest", 1.0, "key4");
        // 按权重值进行排序
        System.out.println("查询所有元素：" + jedis.zrange("sortedSetTest", 0, -1));
        // 删除
        jedis.zrem("sortedSetTest", "key1");
        System.out.println("删除后，查询所有元素：" + jedis.zrange("sortedSetTest", 0, -1));
        // 其他查询
        System.out.println("查询集合中的元素中个数：" + jedis.zcard("sortedSetTest"));
        System.out.println("查询集合中权重某个范围内（1.0——5.0），元素的个数：" + jedis.zcount("sortedSetTest", 1.0, 5.0));
        System.out.println("查看集合中指定元素的权重：" + jedis.zscore("sortedSetTest", "key4"));
    }
}
```

## RedisTemplate

>https://blog.csdn.net/qq_34826261/article/details/105977430

>Jedis有很多缺点：
>
>1编码繁琐，开发者需要关注的太多  
>
>2connection缺乏管理  
>
>3数据操作需要关注序列化和反序列化
>
>RedisTemplate的优点：
>
>1.RedisTemplate高度封装，连接池自动管理；
>
>2.jedis客户端中大量api进行了封装，将同一类型操作封装为operation接口（ValueOperations-简单K-V操作、HashOperations、ListOperations、SetOperations、ZSetOperations）；
>
>3针对数据的“序列化和反序列化”提供了多种可选择测量（JdkSerializationRedisSerializer、StringRedisSerializer、JacksonJsonRedisSerializer、OxmSerializer）。也可自定义类进行序列化和反序列化。

### 添加依赖

```
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-redis</artifactId>
    <version>1.8.10.RELEASE</version>
</dependency>

<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.9.0</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>5.2.5.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.2.5.RELEASE</version>
</dependency>
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>compile</scope>
</dependency>
```

### 创建填写redis.properties

```
#ip地址
redis.hostName=127.0.0.1
#端口号
redis.port=6379
#如果有密码
redis.password=
#客户端超时时间单位是毫秒 默认是2000
redis.timeout=2000

#最大空闲数
redis.maxIdle=10
#连接池的最大数据库连接数。设为0表示无限制,如果是jedis 2.4以后用redis.maxTotal
redis.maxActive=10
#控制一个pool可分配多少个jedis实例,用来替换上面的redis.maxActive,如果是jedis 2.4以后用该属性
redis.maxTotal=10
#最大建立连接等待时间。如果超过此时间将接到异常。设为-1表示无限制。
redis.maxWaitMillis=1000
#连接的最小空闲时间 默认1800000毫秒(30分钟)
redis.minEvictableIdleTimeMillis=300000
#每次释放连接的最大数目,默认3
redis.numTestsPerEvictionRun=1024
#逐出扫描的时间间隔(毫秒) 如果为负数,则不运行逐出线程, 默认-1
redis.timeBetweenEvictionRunsMillis=30000
#是否在从池中取出连接前进行检验,如果检验失败,则从池中去除连接并尝试取出另一个
redis.testOnBorrow=false
#在空闲时检查有效性, 默认false
redis.testWhileIdle=false
```

### 配置类

>配置类用于从redis.properties中读取配置，并根据配置创建JedisConnectionFactory 对象

```
使用JavaConfig的方式来配置，主要是三个Bean，读取配置文件设置各种参数的RedisConnectionFactory以及预期的RedisTemplate和RedisUtil。
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cache.jcache.config.JCacheConfigurerSupport;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
import org.springframework.core.env.Environment;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.connection.jedis.JedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;

@Configuration
@PropertySource("classpath:redis.properties")
public class RedisConfig extends JCacheConfigurerSupport {
    @Autowired
    private Environment environment;
    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        JedisConnectionFactory fac = new JedisConnectionFactory();
        fac.setHostName(environment.getProperty("redis.hostName"));
        fac.setPort(Integer.parseInt(environment.getProperty("redis.port")));
        fac.setPassword(environment.getProperty("redis.password"));
        fac.setTimeout(Integer.parseInt(environment.getProperty("redis.timeout")));
        fac.getPoolConfig().setMaxIdle(Integer.parseInt(environment.getProperty("redis.maxIdle")));
        fac.getPoolConfig().setMaxTotal(Integer.parseInt(environment.getProperty("redis.maxTotal")));
        fac.getPoolConfig().setMaxWaitMillis(Integer.parseInt(environment.getProperty("redis.maxWaitMillis")));
        fac.getPoolConfig().setMinEvictableIdleTimeMillis(
                Integer.parseInt(environment.getProperty("redis.minEvictableIdleTimeMillis")));
        fac.getPoolConfig()
                .setNumTestsPerEvictionRun(Integer.parseInt(environment.getProperty("redis.numTestsPerEvictionRun")));
        fac.getPoolConfig().setTimeBetweenEvictionRunsMillis(
                Integer.parseInt(environment.getProperty("redis.timeBetweenEvictionRunsMillis")));
        fac.getPoolConfig().setTestOnBorrow(Boolean.parseBoolean(environment.getProperty("redis.testOnBorrow")));
        fac.getPoolConfig().setTestWhileIdle(Boolean.parseBoolean(environment.getProperty("redis.testWhileIdle")));
        return fac;
    }

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<String, Object> redis = new RedisTemplate<String, Object>();
        redis.setConnectionFactory(redisConnectionFactory);
        redis.afterPropertiesSet();
        return redis;
    }

    @Bean
    public RedisUtil redisUtil() {
        return new RedisUtil();
    }
}

```

### 测试类

```
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import java.util.HashMap;
import java.util.Map;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = {RedisConfig.class})
public class RedisTest {
    @Autowired
    private RedisUtil redisUtil;

    @Test
    public void test() {
        Map<String, Object> hashMap = new HashMap<String, Object>();
        hashMap.put("123", "hello");
        hashMap.put("abc", "456");

        redisUtil.hmset("hashTT", hashMap);
        System.out.println(redisUtil.hmget("hashTT"));

        redisUtil.set("a","123");
        System.out.println(redisUtil.get("a"));
    }
}

```

### 解决序列化问题

>自定义序列化类

```
import org.springframework.data.redis.serializer.RedisSerializer;
import org.springframework.data.redis.serializer.SerializationException; 
import org.springframework.util.Assert;

import java.nio.charset.Charset;

public class RedisSerializerUtil implements RedisSerializer<Object> {
    private final Charset charset;

    public RedisSerializerUtil() {
        this(Charset.forName("UTF8"));
    }

    public RedisSerializerUtil(Charset charset) {
        Assert.notNull(charset, "Charset must not be null!");
        this.charset = charset;
    }

    @Override
    public byte[] serialize(Object object) throws SerializationException {
        return object == null ? null : String.valueOf(object).getBytes(charset);
    }

    @Override
    public Object deserialize(byte[] bytes) throws SerializationException {
        return bytes == null ? null : new String(bytes, this.charset);
    }
}

```

>设置序列化方式

```
@Bean
public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
    RedisTemplate<String, Object> redis = new RedisTemplate<String, Object>();
    redis.setConnectionFactory(redisConnectionFactory);
    redis.afterPropertiesSet();

    // 设置redis的String/Value的默认序列化方式
    RedisSerializerUtil redisSerializerUtil = new RedisSerializerUtil();
    redis.setKeySerializer(redisSerializerUtil);
    redis.setHashKeySerializer(redisSerializerUtil);
    redis.setValueSerializer(redisSerializerUtil);
    redis.setHashValueSerializer(redisSerializerUtil);
    return redis;
}
```

### 工具类

>封装常用的redis操作

```
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Component;
import org.springframework.util.CollectionUtils;

import java.util.List;
import java.util.Map;
import java.util.Set;
import java.util.concurrent.TimeUnit;

/**
 * 基于spring和redis的redisTemplate工具类
 */
@Component
public class RedisUtil {
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;

    public void setRedisTemplate(RedisTemplate<String, Object> redisTemplate) {
        this.redisTemplate = redisTemplate;
    }
    //=============================common============================
    /**
     * 指定缓存失效时间
     * @param key 键
     * @param time 时间(秒)
     * @return
     */
    public boolean expire(String key,long time){
        try {
            if(time>0){
                redisTemplate.expire(key, time, TimeUnit.SECONDS);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 根据key 获取过期时间
     * @param key 键 不能为null
     * @return 时间(秒) 返回0代表为永久有效
     */
    public long getExpire(String key){
        return redisTemplate.getExpire(key,TimeUnit.SECONDS);
    }

    /**
     * 判断key是否存在
     * @param key 键
     * @return true 存在 false不存在
     */
    public boolean hasKey(String key){
        try {
            return redisTemplate.hasKey(key);
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 删除缓存
     * @param key 可以传一个值 或多个
     */
    @SuppressWarnings("unchecked")
    public void del(String ... key){
        if(key!=null&&key.length>0){
            if(key.length==1){
                redisTemplate.delete(key[0]);
            }else{
                redisTemplate.delete(CollectionUtils.arrayToList(key));
            }
        }
    }

    //============================String=============================
    /**
     * 普通缓存获取
     * @param key 键
     * @return 值
     */
    public Object get(String key){
        return key==null?null:redisTemplate.opsForValue().get(key);
    }

    /**
     * 普通缓存放入
     * @param key 键
     * @param value 值
     * @return true成功 false失败
     */
    public boolean set(String key,Object value) {
        try {
            redisTemplate.opsForValue().set(key, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }

    }

    /**
     * 普通缓存放入并设置时间
     * @param key 键
     * @param value 值
     * @param time 时间(秒) time要大于0 如果time小于等于0 将设置无限期
     * @return true成功 false 失败
     */
    public boolean set(String key,Object value,long time){
        try {
            if(time>0){
                redisTemplate.opsForValue().set(key, value, time, TimeUnit.SECONDS);
            }else{
                set(key, value);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 递增
     * @param key 键
     * @param delta 要增加几(大于0)
     * @return
     */
    public long incr(String key, long delta){
        if(delta<0){
            throw new RuntimeException("递增因子必须大于0");
        }
        return redisTemplate.opsForValue().increment(key, delta);
    }

    /**
     * 递减
     * @param key 键
     * @param delta 要减少几(小于0)
     * @return
     */
    public long decr(String key, long delta){
        if(delta<0){
            throw new RuntimeException("递减因子必须大于0");
        }
        return redisTemplate.opsForValue().increment(key, -delta);
    }

    //================================Map=================================
    /**
     * HashGet
     * @param key 键 不能为null
     * @param item 项 不能为null
     * @return 值
     */
    public Object hget(String key,String item){
        return redisTemplate.opsForHash().get(key, item);
    }

    /**
     * 获取hashKey对应的所有键值
     * @param key 键
     * @return 对应的多个键值
     */
    public Map<Object,Object> hmget(String key){
        return redisTemplate.opsForHash().entries(key);
    }

    /**
     * HashSet
     * @param key 键
     * @param map 对应多个键值
     * @return true 成功 false 失败
     */
    public boolean hmset(String key, Map<String,Object> map){
        try {
            redisTemplate.opsForHash().putAll(key, map);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * HashSet 并设置时间
     * @param key 键
     * @param map 对应多个键值
     * @param time 时间(秒)
     * @return true成功 false失败
     */
    public boolean hmset(String key, Map<String,Object> map, long time){
        try {
            redisTemplate.opsForHash().putAll(key, map);
            if(time>0){
                expire(key, time);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 向一张hash表中放入数据,如果不存在将创建
     * @param key 键
     * @param item 项
     * @param value 值
     * @return true 成功 false失败
     */
    public boolean hset(String key,String item,Object value) {
        try {
            redisTemplate.opsForHash().put(key, item, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 向一张hash表中放入数据,如果不存在将创建
     * @param key 键
     * @param item 项
     * @param value 值
     * @param time 时间(秒)  注意:如果已存在的hash表有时间,这里将会替换原有的时间
     * @return true 成功 false失败
     */
    public boolean hset(String key,String item,Object value,long time) {
        try {
            redisTemplate.opsForHash().put(key, item, value);
            if(time>0){
                expire(key, time);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 删除hash表中的值
     * @param key 键 不能为null
     * @param item 项 可以使多个 不能为null
     */
    public void hdel(String key, Object... item){
        redisTemplate.opsForHash().delete(key,item);
    }

    /**
     * 判断hash表中是否有该项的值
     * @param key 键 不能为null
     * @param item 项 不能为null
     * @return true 存在 false不存在
     */
    public boolean hHasKey(String key, String item){
        return redisTemplate.opsForHash().hasKey(key, item);
    }

    /**
     * hash递增 如果不存在,就会创建一个 并把新增后的值返回
     * @param key 键
     * @param item 项
     * @param by 要增加几(大于0)
     * @return
     */
    public double hincr(String key, String item, double by){
        return redisTemplate.opsForHash().increment(key, item, by);
    }

    /**
     * hash递减
     * @param key 键
     * @param item 项
     * @param by 要减少记(小于0)
     * @return
     */
    public double hdecr(String key, String item, double by){
        return redisTemplate.opsForHash().increment(key, item,-by);
    }

    //============================set=============================
    /**
     * 根据key获取Set中的所有值
     * @param key 键
     * @return
     */
    public Set<Object> sGet(String key){
        try {
            return redisTemplate.opsForSet().members(key);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

    /**
     * 根据value从一个set中查询,是否存在
     * @param key 键
     * @param value 值
     * @return true 存在 false不存在
     */
    public boolean sHasKey(String key,Object value){
        try {
            return redisTemplate.opsForSet().isMember(key, value);
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 将数据放入set缓存
     * @param key 键
     * @param values 值 可以是多个
     * @return 成功个数
     */
    public long sSet(String key, Object...values) {
        try {
            return redisTemplate.opsForSet().add(key, values);
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }

    /**
     * 将set数据放入缓存
     * @param key 键
     * @param time 时间(秒)
     * @param values 值 可以是多个
     * @return 成功个数
     */
    public long sSetAndTime(String key,long time,Object...values) {
        try {
            Long count = redisTemplate.opsForSet().add(key, values);
            if(time>0) {
                expire(key, time);
            }
            return count;
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }

    /**
     * 获取set缓存的长度
     * @param key 键
     * @return
     */
    public long sGetSetSize(String key){
        try {
            return redisTemplate.opsForSet().size(key);
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }

    /**
     * 移除值为value的
     * @param key 键
     * @param values 值 可以是多个
     * @return 移除的个数
     */
    public long setRemove(String key, Object ...values) {
        try {
            Long count = redisTemplate.opsForSet().remove(key, values);
            return count;
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }
    //===============================list=================================

    /**
     * 获取list缓存的内容
     * @param key 键
     * @param start 开始
     * @param end 结束  0 到 -1代表所有值
     * @return
     */
    public List<Object> lGet(String key,long start, long end){
        try {
            return redisTemplate.opsForList().range(key, start, end);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

    /**
     * 获取list缓存的长度
     * @param key 键
     * @return
     */
    public long lGetListSize(String key){
        try {
            return redisTemplate.opsForList().size(key);
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }

    /**
     * 通过索引 获取list中的值
     * @param key 键
     * @param index 索引  index>=0时， 0 表头，1 第二个元素，依次类推；index<0时，-1，表尾，-2倒数第二个元素，依次类推
     * @return
     */
    public Object lGetIndex(String key,long index){
        try {
            return redisTemplate.opsForList().index(key, index);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

    /**
     * 将list放入缓存
     * @param key 键
     * @param value 值
     * @return
     */
    public boolean lSet(String key, Object value) {
        try {
            redisTemplate.opsForList().rightPush(key, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 将list放入缓存
     * @param key 键
     * @param value 值
     * @param time 时间(秒)
     * @return
     */
    public boolean lSet(String key, Object value, long time) {
        try {
            redisTemplate.opsForList().rightPush(key, value);
            if (time > 0) {
                expire(key, time);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 将list放入缓存
     * @param key 键
     * @param value 值
     * @return
     */
    public boolean lSet(String key, List<Object> value) {
        try {
            redisTemplate.opsForList().rightPushAll(key, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 将list放入缓存
     * @param key 键
     * @param value 值
     * @param time 时间(秒)
     * @return
     */
    public boolean lSet(String key, List<Object> value, long time) {
        try {
            redisTemplate.opsForList().rightPushAll(key, value);
            if (time > 0) {
                expire(key, time);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 根据索引修改list中的某条数据
     * @param key 键
     * @param index 索引
     * @param value 值
     * @return
     */
    public boolean lUpdateIndex(String key, long index,Object value) {
        try {
            redisTemplate.opsForList().set(key, index, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 移除N个值为value
     * @param key 键
     * @param count 移除多少个
     * @param value 值
     * @return 移除的个数
     */
    public long lRemove(String key,long count,Object value) {
        try {
            Long remove = redisTemplate.opsForList().remove(key, count, value);
            return remove;
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }
}

```

## 和springboot

导入依赖，在application.xml配置好host，端口

## redis-benchmark
redis-benchmark -h [localhost](http://localhost/) -p -6379 -c 100 -n 100000
c为并发客户端量，n为请求数量。
​
​测试并发量

## redis-cli
redis客户端

## 数据库
默认16个数据库
​select 7  切换数据库
DBSIZE 查看dbsize
flushdb 清空当前db
FLUSHALL 清空所有db

# redis理论

>在入门级linux服务器上，每秒数10万次读写。

## 基本架构

>

## 五大数据类型

消息队列，中间件，订阅消息，数据库，缓存。

### String

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



#### 场景

计数器  uid:9645454:follow 0 incr
​粉丝数
对象缓存存储
统计多单位的数量

### List

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

### Set

sadd myset  world
sismember myset hello
​srandommember  myset
smembers myset
spop  myset随机删除
smove  myset  myset2  "world"
sdiff   set1 set2
sinter set1 set2
​sunion set1 set2

### Hash

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

### Zset

>有序set

## 持久化RDB

### 机制描述

>隔一段时间保存数据快照snapshot到硬盘

### 触发时机

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

## 持久化AOF

>保存数据操作命令，恢复时执行一遍命令。

根据aof文件内容决定保存策略。

