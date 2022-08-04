



```
异常页面，权限管理
```





```
pageHelper使用后再在sql中使用LIMIT会报错，。
```



```
空指针异常不指定谁为空指针，这种报错很麻烦。
```



### 常见WEB错误码

```
404
500
405
400
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

### 常见异常

```
java.lang.NullPointerException
```



### 测试类

下边是完整写法，常见的问题就是无法获取spring容器环境，导致其中的mapper报空指针异常，添加RunWith指定spring容器环境，并且使用ComponentScan扫描所需文件夹，避免找不到mapper的注解类。只用在pom中引入springboot-starter-test包，junit直接add 到classpath。

```
package com.ti;

import com.ti.lisa_provider_4243.service.impl.LisaServiceImpl;
import org.junit.Test;

import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.test.context.junit4.SpringRunner;


@RunWith(SpringRunner.class)
@SpringBootTest(classes = com.ti.test.class)
@ComponentScan(value = "com.ti.lisa_provider_4243")
public class test {
    @Autowired
    LisaServiceImpl LisaServiceImpl;
    @Test
    public void test()   {
        System.out.println(LisaServiceImpl.getTaskFinished(1));
    }

}

```

##### 编写测试类

>测试类用于迭代时测试功能是否正常，避免基础性的错误，在每次maven build时，可以开启自动测试。

### 日志管理

pom文件

```
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```



##### 修改application.yml

```
#修改日志级别为debug
logging:
  level:
    root: debug
```

##### logback与log4j

###### logback

>https://www.cnblogs.com/lixuwu/p/5804793.html

```
#配置logging.level.*来具体输出哪些包的日志级别
logging.level.root=INFO
logging.level.org.springframework.web=DEBUG
logging.level.org.hibernate=ERROR
#将日志输出到文件中
logging.path=F:\\demo
logging.file=demo.log
logging.level.root=info
```

```
#完整示例
<?xml version="1.0" encoding="UTF-8"?>
<!--
    Copyright 2010-2011 The myBatis Team
    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at
        http://www.apache.org/licenses/LICENSE-2.0
    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.
-->
<configuration debug="false">
    <!--定义日志文件的存储地址 勿在 LogBack 的配置中使用相对路径-->
    <property name="LOG_HOME" value="/lisa_provider" />
    <!-- 控制台输出 -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
        </encoder>
    </appender>
    <!-- 按照每天生成日志文件 -->
    <appender name="FILE"  class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--日志文件输出的文件名-->
            <FileNamePattern>${LOG_HOME}/TestWeb.log.%d{yyyy-MM-dd}.log</FileNamePattern>
            <!--日志文件保留天数-->
            <MaxHistory>30</MaxHistory>
        </rollingPolicy>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
        </encoder>
        <!--日志文件最大的大小-->
        <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
            <MaxFileSize>10MB</MaxFileSize>
        </triggeringPolicy>
    </appender>

    <!--myibatis log configure-->
    <logger name="com.apache.ibatis" level="TRACE"/>
    <logger name="java.sql.Connection" level="DEBUG"/>
    <logger name="java.sql.Statement" level="DEBUG"/>
    <logger name="java.sql.PreparedStatement" level="DEBUG"/>

    <!-- 日志输出级别 -->
    <root level="INFO">
        <appender-ref ref="STDOUT" />
        <appender-ref ref="FILE" />
    </root>

</configuration>
```

```
private final static Logger logger = LoggerFactory.getLogger(LisaServiceImpl.class);
```



###### 为springboot配置log4j

>https://cloud.tencent.com/developer/article/1782929
>
>https://www.jianshu.com/p/df64dff65015

排除掉starter-web中自带的logback，并引入log4j依赖。

###### 配置xml或properties

```
springboot默认使用的logback进行管理，需要logback.xml文件，示例如下：
<configuration debug="false">
    <logger name="org.apache" level="INFO" />
    <logger name="org.apache.http.wire" level="INFO" />
    <logger name="org.apache.http.headers" level="INFO" />

    <property name="CONSOLE_LOG_PATTERN"
              value="%date{yyyy-MM-dd HH:mm:ss}  %highlight(%-5level) %magenta(%-4relative) --- [%yellow(%15.15thread)] %cyan(%-40.40logger{39}) : %msg%n"/>


    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>${CONSOLE_LOG_PATTERN}</pattern>
        </encoder>
    </appender>

    <root level="ERROR">
        <appender-ref ref="STDOUT"/>
    </root>
</configuration>
```

```
#在application.yml中指定logback.xml文件
logging:
  config: classpath:logback.xml
```



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

##### 读取环境变量

```
配置环境变量后，记得重启IDEA，因为IDEA在启动时才会读取环境变量。可在Run Configuration中查看可用的环境变量。
#使用如下形式配置环境变量,其中bak_url为默认url，跟在环境变量的冒号后边表示当环境变量不存在时，url的值取bak_url。
spring:
  datasource:
    type: org.apache.commons.dbcp2.BasicDataSource
    username: root
    password: root
    bak_url: jdbc:mysql://172.18.65.185:3306/platform?useUnicode=true&characterEncoding=utf-8&useSSL=true&serverTimezone=UTC
    url: ${platform_mysql_url:${spring.datasource.bak_url}}
    driver-class-name: com.mysql.cj.jdbc.Driver
    dbcp2:
      min-idle: 10
      initial-size: 10
      max-total: 300
      max-wait-millis: 10000
      default-auto-commit: true



```

##### 读取application.yml

```
使用@Value(${user.host})即可读取对应的application.yml中的变量。
```



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

```
解决方法：  
<properties>    
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>    
    <maven.compiler.encoding>UTF-8</maven.compiler.encoding>            
    <java.version>11</java.version>  
    <maven.compiler.source>11</maven.compiler.source>    
    <maven.compiler.target>11<maven.compiler.target>
</properties>
```






##### springboot的maven打包出错,java  -jar 运行后显示无主清单


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

##### java.io.EOFException: null

```
控制台搜索sessions.ser，找到该文件备份好并删除，此错误可能是由tomcat异常关闭导致的。
```



