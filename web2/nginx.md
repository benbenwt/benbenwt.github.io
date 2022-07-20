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

```
server{
		listen 80;#监听端口
		server_name localhost;#本机的ip或域名
		location /
	{
			root    html;#静态网页目录
			index index.html;
	}
	error_page 500 502 503 504 /50x.html#错误页面
	location  /50x.html
	{
			root   html;
    }

}
```

### 使用

##### rewrite改写路径

>取出需要的值，然后用$访问，拼接为想要的路径

```
https://blog.csdn.net/AdminPwd/article/details/102680565
location / {
            if ( $request_uri  ~*  \.asp$){ #截取后缀asp的
			rewrite ^/(.*)\.asp$ /$1.html last; #后缀为asp的替换成html
			break;
			}
        }

```



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

##### try_files

https://www.cnblogs.com/boundless-sky/p/9459775.html

```

location /api { try_files $uri @app; }
uri为除去ip和端口的url后缀部分，app为指定的配置项，用于控制转发。当配置多个选项时，前面的失败会尝试后边的选项。
location @app {
    include uwsgi_params;
    uwsgi_pass api:5000;
  }
该配置项指定转发对象，
```

# nginx理论

>nginx是一款轻量级的web服务器、反向代理服务器。经常用来处理前端资源，并代理后端程序。正向代理代理的是请求者、客户端，反向代理代理的是服务器、服务端，代理了谁，隐藏了谁。
>
>优点：它占用内存少、启动快、高并发能力强。
>
>如何做到高并发的，nginx的work数目与cpu绑定,可以启动多个worker。

## Server

>server组件用于配置一个服务，需要为该服务指定监听的ip和端口，配置后，nginx会启动一个服务处理发送到该端口的请求。
>
>server需要配置如下属性：
>
>listen：监听的端口号
>
>server_name：监听的ip
>
>location: 如何映射发送到该端口的请求，可以使用正则表示，匹配顺序由上至下。
>
>root：用于指定静态资源目录
>
>index： 用于指定默认加载的页面
>
>proxy_pass: 用于指定该location请求转发到何处，本质是反向代理。

## upstream

>借助upstream我们可以将多个服务归为一组，然后使用一个location反向代理这个组，一般组内的多个服务是同质的，是分布在多个机器上的一样的服务，主要通过upstream实现负载均衡，或者灵活的机器上下线。
>
>负载均衡有如下的可选策略：
>
>轮询：时序轮流分配给每个服务
>
>权重：指定分配的比重
>
>公平：公平的按照响应时间分配，响应时间短的优先分配
>
>url_hash：按照url的内容硬性分配机器
>
>ip_hash: 按照前段ip分配机器，保证同一个ip总是分配至相同的后端server

## worker

>nginx使用master管理多个worker，一般worker的数量不能大于核心数目。master负责管理配置文件，服务的启动销毁等。worker负责处理客户端发送的请求。

## Location

>Location用于指定路径映射的服务，控制反向代理的具体规则。
>
>Location内部可以配置很多属性，如：
>
>```
>rewrite "^/testtest/(.*)$"  /cvelistnew/$1 break;//用于改写发送到后端服务后的访问接口路径
>proxy_pass http://up1;
>```

## 配置示例

```nginx
 server {
        listen       8344;
        server_name  localhost;
        client_max_body_size 100m;
        location /{
            root   /home/platform/dist20220616;
            index  index.html;
        	proxy_pass http://up1;
            rewrite "^/testtest/(.*)$"  /cvelistnew/$1 break;
    	}
        upstream up1 {
            server 192.168.1.22；
            server 192.168.1.33；
            server 192.168.1.44；
        }
}    
```

# nginx用法

### 常见错误码

>502 bad gateway

### shell常用命令

```nginx
nginx -s reload/stop
ngix -c 绝对路径/nginx.conf
```

### 配置文件常用语法

##### root

>root用于配置静态资源的目录，index用来配置默认的web主页。

```nginx
location /{
    root   D:/webpages;
    index  index.html;
}  
```

##### proxy_pass

>将后缀为test开头的所有请求发送到 172.18.66.66机器10000的端口上的后端服务

```nginx
location /test{
    proxy_pass 172.18.66.66:10000
}
```

##### rewrite

>rewrite用于改写转发的后缀，此例子将testtest替换为cvelistnew，使用正则取值。

```
location  /testtest{
    proxy_pass 172.18.66.66:10000
	rewrite "^/testtest/(.*)$"  /cvelistnew/$1 break;  
}
```






# problem

## 更改设置，页面不变化
清除浏览器缓存

### nginx命令不起作用

如使用关闭命令后，仍能访问。更改设置后，仍未变化。更改页面后，仍未变化。实际上是因为启动了多个nginx，导致跑了几十个nginx容器，之前的内容一直在这些容器中可以访问。

### nginx.pid缺失等

#### nginx.conf不生效

```
要写nginx.conf的绝对路径，不然没用
```



