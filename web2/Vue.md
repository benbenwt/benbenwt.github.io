[TOC]
# 理论
## 双向绑定
>  其称为mvvm：即model，view，viewModel
vue实现了model和view之间的双向数据流动，当model发生变化时view会变化，view页面发生变化后也会修改model，减少了数据绑定到视图，以及监听视图的工作量。
从view到model的绑定很好实现，可以通过监听前段组件的事件，触发后修改model的值。
从model到view的绑定，需要监听model object的属性变化，这是通过object的defineProperty实现的，简略的代码如下。

```
function objServer(obj){
let keys = Object.keys(obj);
keys.forEach((item)=>{
definedActive(obj,item,obj[item])
})
return obj;
}



function definedActive(obj,item,val){
Object.defineProperty(obj,item,{
get(){
console.log(`${item}获取了`)
},

set(newVlaue){
val = newVlaue;
console.log(`${item}修改了`)
}
})
}

let obj = objServer({
a:1,
b:2
})

obj.a
obj.b
obj.a = 2;
obj.b = 3;
```
在实际的使用中，需要借助发布者-订阅者模式来管理数量庞大的关系。
### 
## 组件
### 组件通信
## 生命周期
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