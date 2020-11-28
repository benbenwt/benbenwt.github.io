# spring boot

>使用基本流程：
>
>1加入starter依赖
>
>2配置properties或yml
>
>3创建主启动类
>
>4通过注解开启相关功能
>
>5运行主启动类

### 创建启动类

@SpringbootApplication

SpringApplication.run(mainApplication.class,args);

### 注解

##### @SpringbootApplication

>相当于，@SpringbootConfiguration,@EnableAutoConfiguration@ComponentScan等注解生效

##### @SpringbootConfiguration

相当于@Configuration注解的重新定义

##### @Configuration 

声明一个Spring配置类

##### @Bean

在@Configuration注解标记的类中将标记了@Bean的方法返回值对象加入IOC容器，可以对应XML配置文件中的bean标签来理解。

##### @EnableAutoConfiguration

启动自动配置

##### @AutoConfigurationPackage

当前包下包含需要扫描的类

##### @ComponentScan

指定要扫描的包，将此注解放在spring启动类上，括号内写上需要扫描的包。

springboot默认会扫描与启动类同级的文件夹。

### application配置文件

>配置好启动类后，我们还需要为特定的服务设置配置文件

##### yml文件

缩进表示层级，冒号和数据之间有一个空格。例如：

server:

  port:  300

  servlet:

​        context-path:/aaa

spring:

​        application:

​				

##### server

.port，设定tomcat开放端口。

.context-path，设定tomcat根路径。

### problem

##### spring intial失败

输入start.aliyun.com

# 



- 相关概念

  springboot将spring封装，实现自动配置等。
  ​

  - 微服务
    一个应用应该是一组小型服务，这些服务通过http进行沟通。
    与之对应的是单体应用，所有页面集中在一台主机上。

  - springboot结构

  - hello world探究

    starter-parent的父级依赖管理所有依赖的版本。
    starter-web导入web模块所需依赖。

    - 依赖
      starter-parent的父级依赖管理所有依赖的版本。
      starter-web导入web模块所需依赖。

    - 注解
      SpringBootApplication  启动类
      [SpringApplication.run](http://springapplication.run/)(HelloWorldMainApp.class,args); 运行启动类
      @Component  组件类
      @EnableAutoConfiguration 自动配置，即导入元数据的所有命名包。只扫描主配置类所在包下面的组件。

      

  ### Springboot集成Mybatis

  

  # 部署

  ### 打包
  导入spingboot-maven-plugin，随后点击mavenproject->Lifecycle->package生成jar包。

  ### 启动
  java  - jar 名称：执行程序。

  ### 发生的的错误

  ##### maven编译版本和本地版本不同

  解决方法：
  <properties>    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>    <maven.compiler.encoding>UTF-8</maven.compiler.encoding>    <java.version>11</java.version>    <maven.compiler.source>11</maven.compiler.source>    <[maven.compiler.target](http://maven.compiler.target/)>11</[maven.compiler.target](http://maven.compiler.target/)></properties>
  ​

  ##### springboot的maven打包出错,运行后显示无主清单

  解决方法：
  <build>   

   <plugins>      

    <plugin>         

     <groupId>org.springframework.boot</groupId>    

  ​        <artifactId>spring-boot-maven-plugin</artifactId>            <version>2.0.1.RELEASE</version>   

  ​         <executions>              

    <execution>            

  ​        <goals>              

  ​          <goal>repackage</goal>             

  ​       </goals>             

     </execution>         

     </executions>     

     </plugin>  

    </plugins></build>
  ​



