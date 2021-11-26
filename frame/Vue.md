# vue

# vue-cli

### 安装及helloworld教程

```
https://www.cnblogs.com/lgx5/p/10732016.html
```

安装3版本

```
npm install -g @vue/cli
vue --version
npm i -g @vue/cli-init
vue init webpack runoob-vue3-test
```



安装

```
https://blog.csdn.net/qq_43201542/article/details/90138321
-g表示放在定义的全局目录
npm uninstall vue-cli -g
npm install vue -g
npm install vue-router -g
npm install vue-cli -g
vue --version
vue -V
npm view jquery versions
```



### 创建vue项目

```
https://www.runoob.com/vue3/vue3-install.html
vue init webpack vue01
cd vue01
npm install
npm run dev
npm run build
npm uninstall vue-cli -g
```

##### 目录结构

```
./App.vue 
./main.js:在此处引入需要的文件，然后Vue.use()
router/index.js
componets/，vue模板
package.json，依赖包信息
wepack.base.conf.js：管理文件加载器
config/index.js：控制代理及一些设置，如是否开启eslint
```

##### 添加临时的测试环境代理

```
修改index.js
proxyTable: {
      '/test':{
        target:'http://172.18.65.186:8345',
        changerOrigin:true
      }
```

# vue-cli 4

```
创建：https://www.cnblogs.com/sese/p/11712275.html
```

