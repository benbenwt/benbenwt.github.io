

### jar包管理

idea自带的maven在idea安装目录的plugins->maven3目录下

仓库查看idea的setting，一般在电脑用户目录下的.m2文件夹。

在repository内，以目录结构形式组织jar包。在jar包所在位置，有jar包及验证哈希码用以确认jar包完整性。

powershell自带sha1工具，命令：certutil -hashfile 路径

### 命令
mvn -version

### 模块依赖

若向让a使用b的功能，如b的entity。在项目结构中，选中a模块，点击dependencies，点击+号，点击Module dependence，选择想要的模块。若提示某些模块已包含，在sources中将其删除。

### problem

##### 依赖一直有波浪线

解决：把其删掉再粘上，等重新导入。