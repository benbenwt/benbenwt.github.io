- 构建ssm项目步骤
- 1.用maven骨架创建web项目，加入archetypeCatalog:internal键值对。
- 2，创建项目结构，java+resource+springmvc.xml+applicationContext.xml+web.xml+pom.xml.完善项目结构.
- 3，dao层mybatis设置较多
- pom:tomcat+jar包
- springmvc :
- 1在web.xml中配置前端控制器，并初始化加载springmvc.xml
- 2在springmvc.xml中配置扫描包，开启注解。配置视图解析器和静态资源映射。
- spring:
- 1:在web.xml中配置监听器，加入spring的配置文件applicationContext.xml
- 2:在applicationContext.xml中配置扫描包。配置datasource，事务管理，mapper,sqlSessionFactory.
- 配置事务通知。
- mybatis：单独写或在applicationContext.xml中
- 不使用spring时：要写datasource和事务通知.       注解方式：配置mappers包或类；          xml方式:配置namespace和sql语句   手动创建获取factory
- 使用spring时:datasource和事务通知。       注解方式：配置mapperScannerConfigurer    xml方式：还没用过                          配置sqlSessionFactoryBean
- 持久层:数据库(datasource,事务)+操作(mapper sql操作)+sqlSessionFactoryBean(框架部分，容器)
- spring和springmvc:框架部分(视图解析器)+controller，service(扫描包，开启注解，静态资源)
- 即controller,service,dao+ssm
- 1事务作用，除了异常？
- 事务是为了保持数据的一致性可完整性。确保一组数据操作都完成或都不做。当一个数据操作异常时，事务会回滚，撤回所有操作，回到事务开始时状态。
- 2springmvc和spring include 和exclude那些文件及效果
- exclude可扫描黑名单之外的所有
- include可扫描白名单和不在黑名单的，为了让其只扫描白名单，即controller.应加上use-default-filters="false".
- 为什么要分开？
- spring容器初始化时，先加载context-param中的配置，再加载init-param中配置，若不分开springmvc会再次扫描，覆盖掉spring的配置，使其失去事务控制。
- DI。三个方面1.注入方式set和construct和anno 2.三种注入数据类型。
- 关于选取。1.bean对象使用方式的选取 xml:外部的包  annotation:自己写的类(当然complex和set type例外）
- 2.注入数据方式的选取:xml一般用set function.
- 即通过注入数据类型和要控制的类是否为外部，来决定方式。
- maven的dependencies作用，把要用的项目集中到repository,整个项目可用，但仍要引用。
- 这样，所有的jar包集中到仓库，需要的类只用引用，不用copy。对于开发者，需要什么，在pom中写好坐标，它会自动下载且放到一起，记得把自动下载设置好。
- 具体的jar包坐标到 https://mvnrepository.com/search?q=springboot查询复制即可。
- compile 参与所有的，编译，测试，打包，部署， 如spring
- test  只参与测试 如junit
- provided 参与编译，测试. 如servlet
- runtime 参与测试，运行时。 如jdbc驱动。
- system 参与compile,test。 如除maven仓库之外的仓库。