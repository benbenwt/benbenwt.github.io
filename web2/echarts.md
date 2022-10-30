# 入门示例
## 关系图
```
https://echarts.apache.org/examples/zh/index.html#chart-type-heatmap
https://echarts.apache.org/examples/zh/editor.html?c=graph-force
```
### index.html内容

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>ECharts</title>
    <!-- 引入 echarts.js -->
    <script src="js/echarts/echarts.js"></script>
</head>
<body>
<!-- 为ECharts准备一个具备大小（宽高）的Dom -->
<div id="main" style="width: 600px;height:400px;"></div>
<script type="text/javascript">
    // 基于准备好的dom，初始化echarts实例
    var myChart = echarts.init(document.getElementById('main'));

    // 指定图表的配置项和数据
    var option = {
        title: {
            text: 'ECharts 入门示例'
        },
        tooltip: {},
        legend: {
            data:['销量']
        },
        xAxis: {
            data: ["衬衫","羊毛衫","雪纺衫","裤子","高跟鞋","袜子"]
        },
        yAxis: {},
        series: [{
            name: '销量',
            type: 'bar',
            data: [5, 20, 36, 10, 10, 20]
        }]
    };

    // 使用刚指定的配置项和数据显示图表。
    myChart.setOption(option);
</script>
</body>
</html>
```

下载nginx，修改配置文件root，启动服务即可。

注意echarts.js引入正确

### 配置项属性

https://echarts.apache.org/next/zh/option.html#series-pie.roseType

title.text:标题内容

color:指定各个块的颜色

tooltip.trigger：item：只显示该点 axis:显示该列所有

tootip.formatter:a,b,c,分别表示列名，数据名，数据值等。

series.name：整个饼的名字

​			.type:图的类型

​			.center：图的中心

​    		.roseType:南丁格尔图

​			.data:数据

​			labelline.length;视觉引导线长度。

### 基本语法

>创建好div，使用js操作样式。

##### 初始化

var mychart=echarts.init(document.getElementById('main'));

##### 设置柱状图

如上面代码块

##### visualMap

视觉映射组件

```
visualMap=[                                     //视觉映射组件，用于进行『视觉编码』，也就是将数据映射到视觉元素。视觉元素可以是：symbol: 图元的图形类别。symbolSize: 图元的大小。color: 图元的颜色。
                                                 // colorAlpha: 图元的颜色的透明度。opacity: 图元以及其附属物（如文字标签）的透明度。colorLightness: 颜色的明暗度。colorSaturation: 颜色的饱和度。colorHue: 颜色的色调。
    {
        show:true,                              //是否显示 visualMap-continuous 组件。如果设置为 false，不会显示，但是数据映射的功能还存在
        type: 'continuous',                     // 定义为连续型 viusalMap
        min:10,                                  //指定 visualMapContinuous 组件的允许的最小值
        max:100,                                 //指定 visualMapContinuous 组件的允许的最大值
        range:[15, 40],                          //指定手柄对应数值的位置。range 应在 min max 范围内
        calculable:true,                        //是否显示拖拽用的手柄（手柄能拖拽调整选中范围）
        realtime:true,                          //拖拽时，是否实时更新
        inverse:false,                          //是否反转 visualMap 组件
        precision:0,                            //数据展示的小数精度，默认为0，无小数点
        itemWidth:20,                           //图形的宽度，即长条的宽度。
        itemHeight:140,                         //图形的高度，即长条的高度。
        align:"auto",                           //指定组件中手柄和文字的摆放位置.可选值为：'auto' 自动决定。'left' 手柄和label在右。'right' 手柄和label在左。'top' 手柄和label在下。'bottom' 手柄和label在上。
        text:['High', 'Low'],                   //两端的文本
        textGap:10,                              //两端文字主体之间的距离，单位为px
        dimension:2,                            //指定用数据的『哪个维度』，映射到视觉元素上。『数据』即 series.data。 可以把 series.data 理解成一个二维数组,其中每个列是一个维度,默认取 data 中最后一个维度
        seriesIndex:1,                          //指定取哪个系列的数据，即哪个系列的 series.data,默认取所有系列
        hoverLink:true,                         //鼠标悬浮到 visualMap 组件上时，鼠标位置对应的数值 在 图表中对应的图形元素，会高亮
        inRange:{                               //定义 在选中范围中 的视觉元素
            color: ['#121122', 'rgba(3,4,5,0.4)', 'red'],
            symbolSize: [30, 100]
        },
        outOfRange:{  //定义 在选中范围外 的视觉元素。
            color: ['#121122', 'rgba(3,4,5,0.4)', 'red'],
            symbolSize: [30, 100]
        },
        zlevel:0,                                   //所属图形的Canvas分层，zlevel 大的 Canvas 会放在 zlevel 小的 Canvas 的上面
        z:2,                                         //所属组件的z分层，z值小的图形会被z值大的图形覆盖
        left:"center",                              //组件离容器左侧的距离,'left', 'center', 'right','20%'
        top:"top",                                   //组件离容器上侧的距离,'top', 'middle', 'bottom','20%'
        right:"auto",                               //组件离容器右侧的距离,'20%'
        bottom:"auto",                              //组件离容器下侧的距离,'20%'
        orient:"vertical",                         //图例排列方向
        padding:5,                                   //图例内边距，单位px  5  [5, 10]  [5,10,5,10]
        backgroundColor:"transparent",            //标题背景色
        borderColor:"#ccc",                         //边框颜色
        borderWidth:0,                               //边框线宽
        textStyle:mytextStyle,                      //文本样式
        formatter: function (value) {                 //标签的格式化工具。
            return 'aaaa' + value;                    // 范围标签显示内容。
        }
    },
    {
        show:true,                          //是否显示 visualMap-continuous 组件。如果设置为 false，不会显示，但是数据映射的功能还存在
        type: 'piecewise',                  // 定义为分段型 visualMap
        splitNumber:5,                      //对于连续型数据，自动平均切分成几段。默认为5段
        pieces: [                           //自定义『分段式视觉映射组件（visualMapPiecewise）』的每一段的范围，以及每一段的文字，以及每一段的特别的样式
            {min: 1500},                     // 不指定 max，表示 max 为无限大（Infinity）。
            {min: 900, max: 1500},
            {min: 310, max: 1000},
            {min: 200, max: 300},
            {min: 10, max: 200, label: '10 到 200（自定义label）'},
            {value: 123, label: '123（自定义特殊颜色）', color: 'grey'}, // 表示 value 等于 123 的情况。
            {max: 5}                        // 不指定 min，表示 min 为无限大（-Infinity）。
        ],
        categories:['严重污染', '重度污染', '中度污染', '轻度污染', '良', '优'],  //用于表示离散型数据（或可以称为类别型数据、枚举型数据）的全集
        min:10,                             //指定 visualMapContinuous 组件的允许的最小值
        max:100,                             //指定 visualMapContinuous 组件的允许的最大值
        minOpen:true,                       //界面上会额外多出一个『< min』的选块
        maxOpen:true,                       //界面上会额外多出一个『> max』的选块。
        selectedMode:"multiple",           //选择模式，可以是：'multiple'（多选）。'single'（单选）。
        inverse:false,                      //是否反转 visualMap 组件
        precision:0,                        //数据展示的小数精度，默认为0，无小数点
        itemWidth:20,                       //图形的宽度，即长条的宽度。
        itemHeight:140,                     //图形的高度，即长条的高度。
        align:"auto",                       //指定组件中手柄和文字的摆放位置.可选值为：'auto' 自动决定。'left' 手柄和label在右。'right' 手柄和label在左。'top' 手柄和label在下。'bottom' 手柄和label在上。
        text:['High', 'Low'],               //两端的文本
        textGap:10,                          //两端文字主体之间的距离，单位为px
        showLabel:true,                     //是否显示每项的文本标签
        itemGap:10,                          //每两个图元之间的间隔距离，单位为px
        itemSymbol:"roundRect",             //默认的图形。可选值为： 'circle', 'rect', 'roundRect', 'triangle', 'diamond', 'pin', 'arrow'
        dimension:2,                          //指定用数据的『哪个维度』，映射到视觉元素上。『数据』即 series.data。 可以把 series.data 理解成一个二维数组,其中每个列是一个维度,默认取 data 中最后一个维度
        seriesIndex:1,                        //指定取哪个系列的数据，即哪个系列的 series.data,默认取所有系列
        hoverLink:true,                      //鼠标悬浮到 visualMap 组件上时，鼠标位置对应的数值 在 图表中对应的图形元素，会高亮
        inRange:{                             //定义 在选中范围中 的视觉元素
            : ['#121122', 'rgba(3,4,5,0.4)', 'red'],
            symbolSize: [30, 100]
        },
        outOfRange:{                            //定义 在选中范围外 的视觉元素。
            color: ['#121122', 'rgba(3,4,5,0.4)', 'red'],
            symbolSize: [30, 100]
        },
        zlevel:0,                                   //所属图形的Canvas分层，zlevel 大的 Canvas 会放在 zlevel 小的 Canvas 的上面
        z:2,                                         //所属组件的z分层，z值小的图形会被z值大的图形覆盖
        left:"center",                              //组件离容器左侧的距离,'left', 'center', 'right','20%'
        top:"top",                                   //组件离容器上侧的距离,'top', 'middle', 'bottom','20%'
        right:"auto",                               //组件离容器右侧的距离,'20%'
        bottom:"auto",                              //组件离容器下侧的距离,'20%'
        orient:"vertical",                        //图例排列方向
        padding:5,                                   //图例内边距，单位px  5  [5, 10]  [5,10,5,10]
        backgroundColor:"transparent",            //标题背景色
        borderColor:"#ccc",                         //边框颜色
        borderWidth:0,                               //边框线宽
        textStyle:mytextStyle,                      //文本样式
        formatter: function (value) {                //标签的格式化工具。
            return 'aaaa' + value;                   // 范围标签显示内容。
        }
    }
];

```



MapType

https://www.cnblogs.com/20gg-com/p/6697701.html?utm_source=itdadao&utm_medium=referral

```
series : [

        	        {
        	            name: 'iphone3',
        	            type: 'map',
        	            mapType: '杭州市',
        	            roam: true,
        	            itemStyle:{
        	                normal:{label:{show:true}},
        	                emphasis:{label:{show:true}}
        	            },
        	            data:[
        	                //{name: '杭州',value: Math.round(Math.random()*1000)},
        	              //  {name: '卢湾区',value: Math.round(Math.random()*1000)},
        	             
        	            ]
        	        }
        	    ]
  mapType显示地图范围，为china，显示中国地图，为浙江，显示浙江地图板块
```

```
 series : [
                {
                    // 图表类型，必要参数！如为空或不支持类型，则该系列数据不被显示。可选为： 
                    // 'line'（折线图） | 'bar'（柱状图） | 'scatter'（散点图） | 'k'（K线图） 
                    // 'pie'（饼图） | 'radar'（雷达图） | 'chord'（和弦图） | 'force'（力导向布局图） | 'map'（地图）
                    type: 'map',
                    // 系列名称
                    name: 'iphone3',
                    // 地图类型，支持world，china及全国34个省市自治区
                    mapType: 'china',
                    // 是否开启滚轮缩放和拖拽漫游,默认为false（关闭），其他有效输入为true（开启），'scale'（仅开启滚轮缩放），'move'（仅开启拖拽漫游）
                    roam: false,
                    // 图形样式
                    itemStyle:{
                        // 默认状态下地图的文字
                        normal:{label:{show:true}},
                        // 鼠标放到地图上面显示文字
                        emphasis:{label:{show:true}}
                    },
                    data:[
                        {name: '北京',value: Math.round(Math.random()*1000)},
                        {name: '天津',value: Math.round(Math.random()*1000)},
                        {name: '上海',value: Math.round(Math.random()*1000)},
                        {name: '重庆',value: Math.round(Math.random()*1000)},
                        {name: '河北',value: Math.round(Math.random()*1000)},
                        {name: '河南',value: Math.round(Math.random()*1000)},
                        {name: '云南',value: Math.round(Math.random()*1000)},
                        {name: '辽宁',value: Math.round(Math.random()*1000)},
                        {name: '黑龙江',value: Math.round(Math.random()*1000)},
                        {name: '湖南',value: Math.round(Math.random()*1000)},
                        {name: '安徽',value: Math.round(Math.random()*1000)},
                        {name: '山东',value: Math.round(Math.random()*1000)},
                        {name: '新疆',value: Math.round(Math.random()*1000)},
                        {name: '江苏',value: Math.round(Math.random()*1000)},
                        {name: '浙江',value: Math.round(Math.random()*1000)},
                        {name: '江西',value: Math.round(Math.random()*1000)},
                        {name: '湖北',value: Math.round(Math.random()*1000)},
                        {name: '广西',value: Math.round(Math.random()*1000)},
                        {name: '甘肃',value: Math.round(Math.random()*1000)},
                        {name: '山西',value: Math.round(Math.random()*1000)},
                        {name: '内蒙古',value: Math.round(Math.random()*1000)},
                        {name: '陕西',value: Math.round(Math.random()*1000)},
                        {name: '吉林',value: Math.round(Math.random()*1000)},
                        {name: '福建',value: Math.round(Math.random()*1000)},
                        {name: '贵州',value: Math.round(Math.random()*1000)},
                        {name: '广东',value: Math.round(Math.random()*1000)},
                        {name: '青海',value: Math.round(Math.random()*1000)},
                        {name: '西藏',value: Math.round(Math.random()*1000)},
                        {name: '四川',value: Math.round(Math.random()*1000)},
                        {name: '宁夏',value: Math.round(Math.random()*1000)},
                        {name: '海南',value: Math.round(Math.random()*1000)},
                        {name: '台湾',value: Math.round(Math.random()*1000)},
                        {name: '香港',value: Math.round(Math.random()*1000)},
                        {name: '澳门',value: Math.round(Math.random()*1000)}
                    ]
                },
                {
                    name: 'iphone4',
                    type: 'map',
                    mapType: 'china',
                    itemStyle:{
                        normal:{label:{show:true}},
                        emphasis:{label:{show:true}}
                    },
                    data:[
                        {name: '北京',value: Math.round(Math.random()*1000)},
                        {name: '天津',value: Math.round(Math.random()*1000)},
                        {name: '上海',value: Math.round(Math.random()*1000)},
                        {name: '重庆',value: Math.round(Math.random()*1000)},
                        {name: '河北',value: Math.round(Math.random()*1000)},
                        {name: '安徽',value: Math.round(Math.random()*1000)},
                        {name: '新疆',value: Math.round(Math.random()*1000)},
                        {name: '浙江',value: Math.round(Math.random()*1000)},
                        {name: '江西',value: Math.round(Math.random()*1000)},
                        {name: '山西',value: Math.round(Math.random()*1000)},
                        {name: '内蒙古',value: Math.round(Math.random()*1000)},
                        {name: '吉林',value: Math.round(Math.random()*1000)},
                        {name: '福建',value: Math.round(Math.random()*1000)},
                        {name: '广东',value: Math.round(Math.random()*1000)},
                        {name: '西藏',value: Math.round(Math.random()*1000)},
                        {name: '四川',value: Math.round(Math.random()*1000)},
                        {name: '宁夏',value: Math.round(Math.random()*1000)},
                        {name: '香港',value: Math.round(Math.random()*1000)},
                        {name: '澳门',value: Math.round(Math.random()*1000)}
                    ]
                },
                {
                    name: 'iphone5',
                    type: 'map',
                    mapType: 'china',
                    itemStyle:{
                        normal:{label:{show:true}},
                        emphasis:{label:{show:true}}
                    },
                    data:[
                        {name: '北京',value: Math.round(Math.random()*1000)},
                        {name: '天津',value: Math.round(Math.random()*1000)},
                        {name: '上海',value: Math.round(Math.random()*1000)},
                        {name: '广东',value: Math.round(Math.random()*1000)},
                        {name: '台湾',value: Math.round(Math.random()*1000)},
                        {name: '香港',value: Math.round(Math.random()*1000)},
                        {name: '澳门',value: Math.round(Math.random()*1000)}
                    ]
                }
            ]
        };
```



