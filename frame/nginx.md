# 反向代理

https://blog.csdn.net/lwwl12/article/details/82219288
找到/etc/nginx/nginx.conf，在http中加上：


server	{		
        keepalive_requests 120;
	    listen 80; ## nginx监听端口号		
        server_name 149.28.231.105; ## nginx监听服务器名称(本机ip或域名)		
location	/	{			
         proxy_pass [https://benbenwt.github.io](https://benbenwt.github.io/);	
}	}


# problem

## 更改设置，页面不变化
清除浏览器缓存