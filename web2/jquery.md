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

### 发送请求

get 超链接,location.href

post 表单



### problem

##### ajax请求数据后填充到饼状图失败

由于ajax未设置为同步，执行到填充时，ajax还未请求到数据。填充的为空。调试过程发现了，但忽略了。

##### 引入jquery无效

把引入放在头部即可，尾部加载在后

##### 监听回车无效

谷歌，firefox，opera不适用keycode，用which。

##### window.location.href失效

写在a标签里和button的onclick函数中都有效，但写在script中单独调用无效。

只有放在第一行代码处可执行，其他位置都不行。但是如果使用alert将他夹起来也可以执行,实际上在后边加一个alert可以让他执行。

```
 $(".form-control").keypress(function (e) {
        //window.location.href = "index.html";
        // 兼容写法
        if (e.which == 13 & this.value!="") {
            window.location.href = "demo.html";
        }
    });
```

解决：stackoverflow，除了逻辑的return。在location.href后必须加return false。否则跳转后仍会返回原页面。写成如下。

```
$(".form-control").keypress(function (e) {
    // 兼容写法
    if (e.which == 13 & this.value!="") {
        window.location.href="search.html";
        return false;
    }
});
```

##### 页面json显示失效

解决：jquery代码

```
var pre_var=$("<pre></pre>").append(syntaxHighlight(response));
$("#show_report").append(pre_var);
```

css代码

```
<style >
    #show_port{
    pre {
        outline: 1px solid #ccc;
        padding: 5px;
        .json-string {
            color: #c62628;
        }
        .json-number, .json-boolean {
            color: #1f1bcc;
        }
        .json-null {
            color: #818181;
        }
        .json-key {
            color: #861d8f;
        }
        }
    }
</style>
```

html代码

```
<div id="main" >
    <div id="show_report" style="width: 80%;height:70%;background-color: ghostwhite;">

    </div>

</div>
```

##### 跳转失效

直接执行的代码放在$(function(){})内。调用的代码放在函数体类，不要放在script首层。