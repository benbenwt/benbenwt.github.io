### function写法的不同

funtion  function_name(param){} ,不会自动执行，通过按键或函数调用。

(function (){})(),自动执行。

$(function (){})，加载页面后自动执行

### each

$.each(list,function(index,item){

}）

### 键值对数组

##### push

var arr1=new Array()

arr1.push({value:value});

##### Map

var map1=new Map()

map1.set("name","zhangsan");

### 获取属性的三个函数差别

attr

prop

valueof,html文件

getAttribute，值

### problem

##### ajax请求数据后填充到饼状图失败

由于ajax未设置为同步，执行到填充时，ajax还未请求到数据。填充的为空。调试过程发现了，但忽略了。

##### 引入jquery无效

把引入放在头部即可，尾部加载在后

##### 监听回车无效

谷歌，firefox，opera不适用keycode，用which。