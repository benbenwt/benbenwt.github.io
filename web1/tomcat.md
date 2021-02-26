- web服务器与应用服务器
  Web服务器专门处理HTTP请求(request)，但是应用程序服务器是通过很多协议来为应用程序提供(serves)商业逻辑(business logic)。

- 常见java应用服务器
  BEA WebLogic server,
  IBM WebSphere Application Server,
  Oracle9i Application Server,
  jBoss,
  Tomcat​​

- 常见web服务器
  Apache
  Nginx
  Tomcat
  Lightttpd
  IBM WebSphere
  Microsoft IIS​​

- tomcat

  - tomcat顶层架构

    

    tomcat有两个核心组件：Connector和container
    多个connector和一个container组成一个service，而serice包括整个tomcat的生命周期由server控制。connector负责接收请求，container负责处理请求，sevice负责关联两者，同时初始化组件。所有组件声名在一个lifecycle的接口中控制。
    server提供接口让他人访问service集合。

    - connector
      connector使用protocolhandler

    - container