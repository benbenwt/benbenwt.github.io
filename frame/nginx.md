# nginx基础

>前后端的分工误区：js是由浏览器进行解析的，nginx是服务端容器，不负责这个。它返回html和js代码，浏览器进行解析和响应。而jsp就是将java代码解析后，生成html页面返回给浏览器。ajax是通过接受数据使用js变更页面。前后端分离，将ajax这部分请求抽离出来，由分布式后端完成。而页面的请求，由nginx负责返回html和js代码，浏览器再解析。

### 安装

linux使用doker或apt-get安装，配置好conf文件。

windws下载好，点击即可启动。

### nginx命令

##### nginx -c conf\my.conf

指定配置文件启动

##### nginx -s reload /reopen/quit/stop

重载配置/重开启/退出/停止

如果未启动就重载，会报告创建nginx.pid失败的错误。

### nginx配置文件

server{

​		listen 80;#监听端口

​		server_name localhost;#本机的ip或域名

​		location /

​	{

​			root    html;#静态网页目录

​			index index.html;

​	}

​	error_page 500 502 503 504 /50x.html#错误页面

​	location  /50x.html

​	{

​			root   html;

​    }

}



### 使用

##### 反向代理

https://blog.csdn.net/lwwl12/article/details/82219288  

找到/etc/nginx/nginx.conf，在http中加上：       

server	{		  
        keepalive_requests 120;  

​	    listen 80; ## nginx监听端口号		  

​       server_name 149.28.231.105; ## nginx监听服务器名称(本机ip或域名)		

location	/	{			  
         proxy_pass [https://benbenwt.github.io](https://benbenwt.github.io/);	  
}	}  

##### 匹配规则

匹配多个，选择匹配的最长的。然后映射改变ip和端口。

##### 用于跨域

使用反向代理替换ip+端口。即使用同意端口反向代理想要访问的跨域ip+端口。


# problem

## 更改设置，页面不变化
清除浏览器缓存

### nginx命令不起作用

如使用关闭命令后，仍能访问。更改设置后，仍未变化。更改页面后，仍未变化。实际上是因为启动了多个nginx，导致跑了几十个nginx容器，之前的内容一直在这些容器中可以访问。

### nginx.pid缺失等

