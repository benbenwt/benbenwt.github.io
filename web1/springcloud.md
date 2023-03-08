
# java、python微服务调用
>https://www.jianshu.com/p/a9355b2a4076
>grpc官方文档：https://doc.oschina.net/grpc?t=58009
# 理论知识
## 组件架构
>注册中心、负载均衡、熔断器断路器、服务网关、服务配置中心、事件消息总线、REST调用
### Eureka
>注册中心，各个服务启动时，Eureka Client会注册到Eureka Server，并且 Eureka Client还可以拉取Eureak Server的注册表，从而知道其他服务在哪。
>Eureka实现了服务的注册与发现，类似于Zookeeper的动态上下线，存储统一的各节点服务信息，提供给其他服务访问。
### Ribbon
>服务间发起请求时，基于Ribbon做负载均衡，从一个服务的多台机器中选择一台。
>在客户端获取到生产者服务列表后，就需要判断选择合适的节点，由于在客户端发生，又称为Ribbon客户端负载均衡器。
#### 负载均衡方式
>常用方式：1服务端负载均衡 2客户端负载均衡
>服务端负载均衡：在客户端和服务端之间建立一个负载均衡服务器，例如Nginx。该负载均衡服务器有一份可用服务清单，通过心跳机制维护，保证清单内服务节点可用。当客户端发送请求时，将其交给负载均衡服务器，然后其根据负载均衡算法分配请求到服务节点。
>客户端负载均衡：直接在客户端决定请求那个后端服务。需要先从注册中心获取可用的服务列表，然后在客户端通过逻辑代码决定发送到那个服务端。
#### Ribbon负载均衡策略
>Ribbon提供了7个默认实现类，即策略。包括 轮询、随机、加权、最小响应时间、最小并发量
>通过RestTemplate的请求拦截实现依赖服务的接口调用
### Feign
>服务接口调用，基于Feign的动态代理，根据注解和选择的机器，拼接请求URL地址，发起请求。
### Hystrix
>熔断器、断路器，可以保护微服务应用。发起请求是通过Hystrix的线程池来走的，不同的服务走不同的线程池，实现了不同服务调用的隔离，避免了服务雪崩问题。
### Zuul
>服务网关，如果前段、移动端需要调用后端系统，统一从Zuul网关进入，从Zuul网关转发请求给对应的服务。
### Config组件
>服务配置中心，
### Bus组件
>事件消息总线

### 微服务
```
划分的准则：业务流程步骤，业务的分块。同一块业务分为基础应用和核心业务，如基础的登录、用户等，核心的支付、下载等。
1外界其他服务可以依赖它，它必须提供封装良好的api
2自身可以水平扩展，使用docker等技术快速水平扩展，并注册到管理中心。
```
>拆分的模块，并不意味着他就是独立的服务，而是说它可以作为其他功能的拼图。作为其他模块的依赖被调用。拼图式的就是通过依赖调用，而不用拼图的就是可以运行的独立的服务。

# 使用方法
## Eureka
### 创建Eureka Server。
```
spring.application.name=eureka-server
#服务注册中心端口号
server.port=8761
#服务注册中心实例的主机名
eureka.instance.hostname=localhost
#是否向服务注册中心注册自己
eureka.client.register-with-eureka=false
#是否检索服务
eureka.client.fetch-registry=false
#服务注册中心的配置内容，指定服务注册中心的位置（下面最后的字符串只能是eureka，不能是别的）
eureka.client.serviceUrl.defaultZone=http://${eureka.instance.hostname}:${server.port}/eureka/

```
```
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}

//访问网页的localhost:8761
```
### 创建Eureka客户端
```
#自己的名字
spring.application.name=service-producer
#自身端口
server.port=8762
#注册到eureka服务器链接
eureka.client.serviceUrl.defaultZone=http://localhost:8761/eureka/

```

```
//创建Eureka客户端,链接注册中心。
//启动类需要加上@EnableEurekaClient或者@EnableDiscoveryClient
@SpringBootApplication
@EnableDiscoveryClient
@RestController
public class ServiceProducerApplication {
    @Value("${server.port}")
    String port;
    public static void main(String[] args) {
        SpringApplication.run(ServiceProducerApplication.class, args);
    }
    @RequestMapping("/hi")
    public String hi(@RequestParam(value = "name", defaultValue = "lbj") String name) {
        return "您好 " + name + " ,端口:" + port;
    }
}
```

## Ribbon
### 通过RestTemplate消费
```
#自己的名字
spring.application.name=service-consumer
#自身端口
server.port=8764
#注册到eureka服务器链接
eureka.client.serviceUrl.defaultZone=http://localhost:8761/eureka/
```

```
@SpringBootApplication
@EnableDiscoveryClient
@RestController
public class ServiceConsumerApplication {

    @Autowired
    private RestTemplate restTemplate;
    @Bean
    @LoadBalanced
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }
    public static void main(String[] args) {
        SpringApplication.run(ServiceConsumerApplication.class, args);
    }
    @RequestMapping("hello")
    public String hello(@RequestParam(value = "name",defaultValue = "詹姆斯") String name){
        return restTemplate.getForObject("http://service-producer/hi?name="+name,String.class);
    }
}

```
### 修改默认负载均衡策略
#### 通过配置文件配置
```
hello:
    ribbon:  
     NFLoadBalancerRuleClassName:com.netflix.loadbalancer.RandomRule
```
#### 通过java注解配置
```
// @Configuration -- 通过注解@RibbonClient为特定的服务配置负载均衡策略，
//                   这时候该注解需去掉，否则会执行两次RandomRule()
public class RibbonConfiguration{
      @Bean
      public IRule ribbonRule(){
          //随机负载
          return new RandomRule();
     }
}
```

```
@Configuration
@RibbonClient(name="hello", configuration=RibbonConfiguration.class)
    public class TestRibbonConfiguration{
}
```

## Hystrix

## Gateway
## Config
## Bus
## Feign
### 通过Feign消费
```
#自己的名字
spring.application.name=service-consumer-feign
#自身端口
server.port=8765
#注册到eureka服务器链接
eureka.client.serviceUrl.defaultZone=http://localhost:8761/eureka/

```

```
@SpringBootApplication
@RestController
@EnableFeignClients
@EnableDiscoveryClient
public class ServiceConsumerFeignApplication {
    @Autowired
    IProducer iProducer;
    public static void main(String[] args) {
        SpringApplication.run(ServiceConsumerFeignApplication.class, args);
    }
    @RequestMapping(value = "/hello")
    public String hello(@RequestParam(value = "name", defaultValue = "ddd") String name) {
        return iProducer.hello(name);
    }
}

```

```
@FeignClient(value = "service-producer")
public interface IProducer {
    @RequestMapping(value = "/hi",method = RequestMethod.GET)
    String hello(@RequestParam(value = "name") String name);
}

```
## 创建项目
创建父模块，在depencencyManagement中管理版本依赖。通过new，module创建子级模块。在project structure中添加对其他模块比如api的依赖。

### spring-cloud-dependencies
管理spring-cloud相关的所有依赖，ctrl加左键点击查看。

