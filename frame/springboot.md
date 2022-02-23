



```
异常页面，权限管理
```





```
pageHelper使用后再在sql中使用LIMIT会报错，。
```



```
空指针异常不指定谁为空指针，这种报错很麻烦。
```



### jar包外部配置文件

```
通过此种方式来指定配置文件，可以覆盖jar包内的配置文件。
java  -jar  app.jar  --spring.config.location=application.yml
或直接在jar包的同级目录放置application.yml也可覆盖jar包内的配置文件。
```

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

##### SpringBootApplication  

启动类

[SpringApplication.run](http://springapplication.run/)(HelloWorldMainApp.class,args); 运行启动类

##### @Component  

##### 组件类

##### @EnableAutoConfiguration 

##### 自动配置，即导入元数据的所有命名包。只扫描主配置类所在包下面的组件。

@RequestBody

使用此注解将json数据直接封装好。不使用任何注解，可以自动将form-data封装到对象中。

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

server

   .port，设定tomcat开放端口。

   .context-path，设定tomcat根路径。

##### 

### 版本问题

##### 查看spring-boot-parent预定义版本

点击spring-boot-parent依赖的版本号，进入其pom文件。

再点击文件中的spring-boot-dependencies依赖的版本号进入其pom。

搜索所需依赖的关键字。

  ### Springboot集成Mybatis

  >1导入maven依赖，若有错误查看仓库，验证sha1值，若错误删除整个文件夹。删除pom中依赖再粘贴，重新下载。
  >
  >2配置yml文件，如下
  >
  >```yaml
  >server:
  >  port: 8001
  >spring:
  >  application:
  >    name: provider-dept8001
  >  datasource:
  >    type: org.apache.commons.dbcp2.BasicDataSource
  >    username: root
  >    password: root
  >    url: jdbc:mysql://localhost:3306/cloud_db_one?useUnicode=true&characterEncoding=utf-8&useSSL=true&serverTimezone=UTC
  >    driver-class-name: com.mysql.cj.jdbc.Driver
  >    dbcp2:
  >      min-idle: 10
  >      initial-size: 10
  >      max-total: 300
  >      max-wait-millis: 10000
  >      default-auto-commit: true
  >mybatis:
  >  mapper-locations: classpath:mybatis/mapper/*Mapper.xml
  >  type-aliases-package: com.weitao.api.entity
  >
  >#showSql
  >logging:
  >  level:
  >    com:
  >      example:
  >        mapper: debug
  >
  >eureka:
  >  client:
  >    service-url:
  >      defaultZone: http://localhost:7000/eureka,http://localhost:7001/eureka,http://localhost:7002/eureka
  >
  >```
  >
  >3编写三层：编写entity类，mapper接口，mapper对应的xml文件。添加mapper注解，或在启动类上添加mapperscan指定mapper所在文件夹。mapper，mapper.xml如下：
  >
  >```
  >@Mapper
  >public interface DeptMapper {
  >    Dept findById(Integer deptNo);
  >    List<Dept> findAll();
  >    boolean addDept(Dept dept);
  >}
  >```
  >
  >```java
  ><?xml version="1.0" encoding="UTF-8"?>
  ><!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
  >
  ><mapper namespace="com.weitao.provider.mapper.DeptMapper">
  >    <select id="findById" resultType="Dept" parameterType="Integer">
  >        select dept_no deptNo,dept_name deptName,db_source dbSource from dept where dept_no=#{deptNo}
  >    </select>
  >
  >    <select id="findAll" resultType="Dept">
  >         select dept_no deptNo,dept_name deptName,db_source dbSource from dept
  >    </select>
  >
  >    <insert id="addDept" parameterType="Dept">
  >        INSERT INTO dept(dept_name,db_source) VALUES(#{deptName},DATABASE())
  >    </insert>
  ></mapper>
  >
  >```
  >
  >4启动启动类并验证。

### springboot集成dbcp

>yml配置文件如下：需要引入commons-io和commons-dbcp2的依赖。
>
>```
>server:
>port: 8001
>
>spring:
>application:
>name: provider-dept8001
>datasource:
>type: org.apache.commons.dbcp2.BasicDataSource
>username: root
>password: root
>url: jdbc:mysql://localhost:3306/cloud_db_one?useUnicode=true&characterEncoding=utf-8&useSSL=true&serverTimezone=UTC
>driver-class-name: com.mysql.jdbc.Driver
>dbcp2:
> min-idle: 10
> initial-size: 10
> max-total: 300
> max-wait-millis: 10000
> default-auto-commit: true
>
>mybatis:
>mapper-locations: classpath:mybatis/mapper/*Mapper.xml
>type-aliases-package: com.weitao.api.entity
>
>#showSql
>logging:
>level:
>com:
> example:
>   mapper: debug
>```

### Springboot的jsp技术

配置yml文件如下：

```
spring:
 mvc:
  view:
   prefix:/WEB-INF/
   suffix:.jsp
```

同ssm一样，将controller结果解析至prefix目录下试图.jsp。

jsp使用很少，一般使用如Thymelef等。

### 测试

使用@SpringBootTest注解标记类，使用Test标记测试方法即可。

  ### 打包部署

>https://blog.csdn.net/aiyowei1106/article/details/85273705

### 打包修改配置文件

##### 自定义属性

```
application.ylm
user:
	sample_path: /home
java program
@Value("${user.sample_path}")
String path
```

##### 替换jar包内文件

```
jar tf file-upload-1.0-SNAPSHOT.jar 查看清单
jar xf file-upload-1.0-SNAPSHOT.jar BOOT-INF/classes/config.properties  解压指定文件
jar uf file-upload-1.0-SNAPSHOT.jar BOOT-INF/classes/config.properties  替换文件
```



##### 基本流程

1，将当前模块依赖的公共模块在project structure中引入。并将公共模块maven install到本地仓库，然后在当前模块pom中引入功能模块坐标。这样，在打包时会将公共模块加入jar内。

2，springboo项目打包时，应在pom内引入springboot-maven插件，设置主类名。若设置错误，会出现Exception in thread "main" java. lang.ClassNotFound Exception。



##### 打包时的pom文件设置如下

下方为两个插件，所有模块都使用了maven管理，故都添加maven-compiler。但，只在使用springboot的子模块中添加springboot-maven插件。

```
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.8.1</version>
            <configuration>
                <source>11</source>
                <target>11</target>
                <encoding>UTF-8</encoding>
            </configuration>
        </plugin>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <version>2.3.4.RELEASE</version>
            <configuration>
                <mainClass>com.weitao.parent.ParentApplication</mainClass>
            </configuration>
            <executions>
                <execution>
                    <id>repackage</id>
                    <goals>
                        <goal>repackage</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```



##### 打包

 随后点击mavenproject->Lifecycle->package生成jar包。

package：只打包

install：打包发布到本地仓库

deploy：打包发布到远程及本地

##### 启动
  java  - jar jar包名称。

  ### problem

##### maven编译版本和本地版本不同

解决方法：
  <properties>    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>    <maven.compiler.encoding>UTF-8</maven.compiler.encoding>    <java.version>11</java.version>    <maven.compiler.source>11</maven.compiler.source>    <maven.compiler.target>11<maven.compiler.target></properties>
​

##### springboot的maven打包出错,java  -jar 运行后显示无主清单

解决方法：


```
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.8.1</version>
            <configuration>
                <source>11</source>
                <target>11</target>
                <encoding>UTF-8</encoding>
            </configuration>
        </plugin>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <version>2.3.4.RELEASE</version>
            <configuration>
                <mainClass>com.weitao.parent.ParentApplication</mainClass>
            </configuration>
            <executions>
                <execution>
                    <id>repackage</id>
                    <goals>
                        <goal>repackage</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

# <span id = "jump">problem</span>

##### spring intial失败

解决：输入start.aliyun.com

##### Field injection is not recommended

推荐使用set方法和构造方法注入，不要使用变量注入。

https://blog.csdn.net/zhangjingao/article/details/81094529

##### No serializer found for class com.weitao.api.entity

序列化的类还需要添加get，set方法。

##### 版本问题

ctrl左键点击依赖，查看spring-boot-dependeccies或spriing-cloud-dependencies具体定义。

##### server.port失效问题

添加子模块后，会让父级模块打包方式变为pom，所以无法让设置生效。

可以在配置服务的地方设置override parameter，或改变打包方式,将pom.xml中的package改为jar，但是最好不要，因为他是作为父级模块。

##### 打包后时提醒无某依赖的模块

例如依赖api基础模块，则在project structure中添加moudles依赖。并把api模块maven install的本地仓库，在当前模块中使用pom引入api模块。

##### 打包成功，运行时提醒无parentApplication

原因：建立maven子模块时pom中没有自动写此插件的依赖，所以copy的父级，然后写的父级的主类名字，导致找不到主类。

解决：在pom中引入springboot-maven插件，该插件可设置打包参数。在configuration->mainclass中设置主类的全限定类名。

##### 无法注入mapper

使用Repository加mapper注解，或使用Component注入





