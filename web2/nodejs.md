# node

# npm

### npm管理命令

```
npm install express@3.21.2
node -v 
查看global安装的所有包：npm list -global
npm config set registry=http://registry.npm.taobao.org
查看所有配置：npm config list
查看源：npm config get registry
查看源是否工作：npm info vue
npm -v
```

##### 查看安装的版本

```
npm list packagename
```



##### npm参数

```
-S --save 写入dependencies，生产环境
-D --save-dev 写入devDependencies，只用于开发环境
-g 全局安装
不带参数，本地安装，放到当前文件夹的node_modules
```



### 安装npm

```
https://www.cnblogs.com/lgx5/p/10732016.html
npm config set registry=http://registry.npm.taobao.org
npm cache clean --force
```

### 安装vue

```
vue-3:https://www.runoob.com/vue3/vue3-install.html
-g表示放在定义的全局目录
npm info vue选择正确版本
npm install vue@next -g
npm install @vue/cli -g
npm install @vue/cli-init  -g
vue --version 查看脚手架版本
npm list vue
npm view jquery versions
```



##### 禁用elint

```
修改.eslintrc.js，在rules中加入如下规则
'no-console':'off',
'no-irregular-whitespace':'off'
```



##### npm安装依赖

```
npm install core-js@3.1.1
```

##### 安装其他

```
element-ui:https://element-plus.gitee.io/zh-CN/guide/installation.html#%E4%BD%BF%E7%94%A8%E5%8C%85%E7%AE%A1%E7%90%86%E5%99%A8
```

