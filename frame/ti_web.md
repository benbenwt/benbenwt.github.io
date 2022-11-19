## spring-cloud、grpc
### grpc
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