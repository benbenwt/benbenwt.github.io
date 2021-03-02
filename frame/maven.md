

### jar包管理

idea自带的maven在idea安装目录的plugins->maven3目录下

仓库查看idea的setting，一般在电脑用户目录下的.m2文件夹。

在repository内，以目录结构形式组织jar包。在jar包所在位置，有jar包及验证哈希码用以确认jar包完整性。

powershell自带sha1工具，命令：certutil -hashfile 路径

##### Dependencymanagement

> 指定子级模块所用版本，子模块只需要写依赖，不用写版本号。
>
> 注意只是指定版本号，自己或子级使用仍然需要在dependencies中导入。

### 命令
mvn -version

### 模块依赖

若向让a使用b的功能，如b的entity。在项目结构中，选中a模块，点击dependencies，点击+号，点击Module dependence，选择想要的模块。若提示某些模块已包含，在sources中将其删除。

### problem

##### 依赖一直有波浪线

解决：把其删掉再粘上，等重新导入。

全部删除，点击Invalidate caches/restart,然后一个一个的添加依赖。

##### 已删除依赖还在右侧栏

点击File->Invalidate caches/restart