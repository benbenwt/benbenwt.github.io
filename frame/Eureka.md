# problem

### eureka之间互相不注册

域名即hostname不能相同，必须在hosts中设置不同域名指向127.0.0.1。

如果访问不未启动或不存在的eureka中心，提示'500 Internal Privoxy Error'。

##### 删除yml中配置及启动类注释后，只启动provider。仍会请求eureka中心并报错。

删除pom中config和eurekaClient