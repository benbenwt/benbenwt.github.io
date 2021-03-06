# 背景

​		在分布式系统中，我们会把功能互相分离，作为单独的服务。在单体应用中，我们通过new直接创建实现类，然后使用其功能，在分布式系统中我们无法这样做。

​		那么如何解决呢，方案1便是通过http请求进行数据交互，访问特定的接口获得对应的数据，也就是restful的由来。这种方式通过单纯数据交流访问其他服务，是一种面向资源的服务。方法2就是使用代理模式，最好通过spring ioc实现，通过spring注入对象。这里涉及到对象的创建方式[1](#1)。在注入时扫描到了reference注解，则生成代理类，将这个代理对象放进容器中。这个代理对象内部是通过httpclient实现的，将java对象序列化后传输在反序列化生成代理类。RPC就是要解决两个问题，服务之间的调用问题，让服务像本地调用一样方便。

​		restful和RPC对比：

​		/query？id=1		Get/query?id=1

​		RPC是面向过程的，它请求所需对象的序列化数据，然后创建以供使用。

​		restful使用http，获取所需数据。端与端使用统一的格式数据通信，无视端与端技术架构的不同。



<span id="1">[1].java创建对象的方式有：new关键字，通过反射new instance方法，通过clone方法，通过反序列化手段</span>



# 分布式系统基础架构

>一个分布式系统组件大致包括：
>
>- 服务注册中心，管理元数据
>- 服务提供方
>- 服务调用方

流行的分布式系统：

- HBase
- Kafaka
- Dubbo



# 参考文章

[什么是RPC](https://www.jianshu.com/p/052913a386b7)