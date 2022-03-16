### 微服务

```
1外界其他服务可以依赖它，它必须提供封装良好的api
2自身可以水平扩展，使用docker等技术快速水平扩展，并注册到管理中心。
```



>拆分的模块，并不意味着他就是独立的服务，而是说它可以作为其他功能的拼图。作为其他模块的依赖被调用。拼图式的就是通过依赖调用，而不用拼图的就是可以运行的独立的服务。

# 创建方式

创建父模块，在depencencyManagement中管理版本依赖。通过new，module创建子级模块。在project structure中添加对其他模块比如api的依赖。

### spring-cloud-dependencies

管理spring-cloud相关的所有依赖，ctrl加左键点击查看。

