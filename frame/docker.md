[TOC] 
# Platform_docker

### 使用版本版本

```
docker 20.10.7
docker-compose 1.28.5
```


##### DockerFile例子

```
vim Dockerfile
------------
FROM nginx:latest
RUN useradd -m platform\
&&echo "copying webpages to nginx dir"
RUN rm /etc/nginx/conf.d/default.conf
COPY ./docker/nginx/platform.conf /etc/nginx/conf.d
COPY --chown=platform:platform ./platform/webpages  /home/platform/webpages
------------
docker build -t platform:v1 `pwd`
docker image ls
docker run -p 8344:80 platform:v1
curl localhost:8344
```

platform-mysql

```
#建表脚本
https://blog.csdn.net/10km/article/details/79046864
COPY platform.sql  /docker-entrypoint-initdb.d
```



##### docker-compose例子

```
version: "2"
services:
  nginx:
    image: platform-nginx
    build:
      context: .
      dockerfile: ./docker/nginx/Dockerfile
    ports:
      - 8344:80
    networks:
      platformnet:
        ipv4_address: 172.42.0.12
networks:
  platformnet:
    ipam:
      driver: default
      config:
        - subnet: 172.42.0.0/24#docker-compose例子
vim docker-compose.yml
--------------------
version:"3"

services:
  nginx:
    image:platform-nginx
    build:
      context:.
      dockerfile:./docker/nginx/Dockerfile
    ports:
      -8344:80
    networks:
      platformnet:
        ipv4_address:172.42.0.12
networks:
  platformnet:
    ipam:
      driver:default
      config:
        -subnet:172.42.0.0/24
-----------------------------
docker-compose up
#基础错误，yml格式:后要空格.
```





# Lisa_docker

>使用版本docker 20.10.5,docker-compose 1.28.5

```
#lisa-worker的内容,再将requirements.txt中ripe2改为1.5.3版本。
FROM python:3.6-slim

ARG maxmind_key

RUN apt-get update && apt-get install -y --no-install-recommends \
    curl\
    gcc \
    g++ \
    libpcap-dev \
    make \
    patch \
    git \
    qemu \
    qemu-system \
    openvpn \
    binutils \
    iprange \
    wget \
    tar \
    e2tools 

COPY  ./new  ./radare2

RUN 	./radare2/sys/install.sh \
    && useradd -m lisa \
    && echo "Copying LiSa Linux images ..." 

COPY --chown=lisa:lisa ./images /home/lisa/images

COPY --chown=lisa:lisa ./data /home/lisa/data
COPY --chown=lisa:lisa ./docker /home/lisa/docker
COPY --chown=lisa:lisa ./lisa /home/lisa/lisa
COPY --chown=lisa:lisa ./requirements.txt /home/lisa/requirements.txt

ENV PYTHONPATH /home/lisa

WORKDIR /home/lisa

RUN /usr/local/bin/python -m pip install --upgrade pip

RUN pip install -r requirements.txt --extra-index-url  http://pypi.douban.com/simple/ --trusted-host pypi.douban.com
RUN iprange -j data/blacklists/* > data/ipblacklist \
    && ./docker/worker/maxmind.sh $maxmind_key \
    && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false \
    git \
    gcc \
    g++ \
    make \
    patch \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf /radare2/.git

CMD ["./docker/worker/init.sh"]
```

```
#image
docker save 0fdf2b4c26d3 > hangge_server.tar
docker load < hangge_server.tar
#container
docker export f299f501774c > hangger_server.tar
docker import - new_hangger_server < hangger_server.tar
```



# docker概述

>docker各个容器互相独立，不会互相影响。但是他们都作为宿主机的一个进程存在，使用宿主机的ps指令可查看到很多相关进程并进行管理。


### 常用

```
docker run   --name myhue -p 8888:8888  gethue/hue
docker stop myhue
```



docker为什么出现
开发-上线 两套环境，版本更新，环境不同，无法跨环境等导致运维考验大。环境配置麻烦。项目能否带着环境安装打包。
​实现，开发打包部署上线，一套流程做完。

docker解决方案：项目带上环境放到docker中，需要者下载这个镜像。docker中箱子隔离独立。

docker的优势
docker具有更少的抽象层，不需要模拟硬件guest os。
直接使用宿主机的内核。



# docker安装

进入docker docs，找到安装到linux

doc:https://docs.docker.com/engine/install/ubuntu/。

```
#安装步骤
sudo apt-get remove docker docker-engine docker.io containerd runc

sudo yum install -y yum-utils

sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
    
sudo yum install docker-ce docker-ce-cli containerd.io

 yum list docker-ce --showduplicates | sort -r
 
sudo systemctl start docker

sudo docker run hello-world
    

```



```
#源管理
sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
  
sudo yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
sudo yum-config-manager \
    --add-repo \
    https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/docker-ce.repo
 sudo yum install docker-ce docker-ce-cli containerd.io
```



##### 使用阿里云镜像加速下载

阿里云控制台:https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors

```
vim /etc/docker/daemon.json
#添加自己的镜像
{
  "registry-mirrors": ["****"]
}
systemctl daemon-reload。
systemctl restart docker。
```

### docker-compose安装

```
https://github.com/docker/compose
到仓库下载编译好的二进制文件，放到/usr/bin即可。
```



# docker命令

>docker  version,docker  run  hello-world测试是否安装成功

### 镜像命令



```
docker info  系统信息，镜像和容器的数量
docker images 查看镜像信息  -a  all  -q id
docker image ls
```





##### 展示仓库中镜像

```
docker search 
    --filter , -f 		Filter output based on conditions provided
    --format 		Pretty-print search using a Go template
    --limit 	25 	Max number of search results
    --no-trunc 		Don’t truncate output
```

##### 下载镜像

```
docker pull 
docker pull [OPTIONS] NAME[:TAG|@DIGEST]
docker使用分层下载，联合文件系统。
```

##### 删除镜像

```
docker rmi -f
docker image rm image_id
```

### 容器命令

**docker    container/image   start/stop/rm/run/pause**

> run用于初次根据image创建container，之后便可使用start启动，pause暂停，stop停止。rm用于删除此container，以后若要使用就要再次使用镜像run创建container。

##### docker run

使用镜像创建容器并strat容器

--name 		Assign a name to the container
-d 后台运行
-i  使用交互方式，进入容器查看内容
-P   ip：主机端口：容器端口。
-p随机端口
docker run -it centos  /bin/bash启动并进入容器
exit 推出容器
ctrl+p+q 不关闭容器推出
​

若创建的容器未提供服务，自动stop

##### docker rm

 删除创建的容器

docker rm -f $(docker ps -aq)  移除容器，若使用需再次run
​linux管道服务：docker ps -a -q|xargs docker rm
​

##### docker container ls

  (带上-a参数查看所有容器，不带为正运行容器。)
-a   正在运行+历史运行,即已创建的容器
-n=?  显示最近创建的容器。



##### docker container start 容器id

启动创建好的容器

docker container restart id  重启创建好或正在运行的容器
docker  container  kill id 杀死正在运行进程
docker container   stop id  停止正运行进程。

### 常用其他命令

```
sudo docker exec -it $DOCKER_ID /bin/bash -c 'cd /home/lisa && ./docker/worker/init.sh'
#清除部分container和image，腾出空间。
docker system prune -a
```

##### 日志

docker logs  -f -t --tail  容器
执行脚本：docker container  run  -d id   /bin/sh  -c "脚本内容"

##### 容器进程

docker top  id

##### 元数据

docker history创建历史
docker tag将镜像放入仓库并标记
docker inspect id

##### 进入容器命令和拷贝命令

docker container exec -it  id    /bin/bash  "命令内容“  新建一个终端,i表示保持命令行不关闭，t表示创建新的终端。
docker container attach  id  进入正在运行的命令行
容器拷到宿主机：docker  cp  id:path   宿主机path
主机到容器：挂载


### 容器数据卷

>搭建服务环境后，在环境中部署应用。但是删除容器后，数据会丢失。所以将docker目录挂载到外部，实现保存。

docker inspect，查看容器信息。

docker run -it -v 主机目录：容器内目录,挂载目录。

例如：docker run   -it -v /home/ceshi:/home centos  /bin/bash

docker run -d -p  3310:3306  -v  /home/mysql/conf:/home/conf    mysql  /bin/bash

docker attach 容器id，进入运行的容器。

  

应用举例：

将nginx配置文件挂载出来，不用进入容器修改配置

将存储的json文件挂载到外部

将mysql挂载到外部

## docker网络

```
docker network create jenkins
```

## docker管理

```
文件夹复制
COPY  文件 文件，指定的名字
COPY  文件 文件夹，放到文件夹下,文件夹不存在则当作为名字。
COPY  文件夹 文件夹，直接将源文件夹下的零散文件放在目的文件夹
COPY  文件夹/* 文件夹，同上
```

### volume local dirver

```
在/var/lib/docker 目录下存储
```

### 查看磁盘占用

```
docker system df -v
```



```
docker network ls
docker network rm 
```

```
查看latest具体版本
docker image inspect    rabbitmq:latest|grep -i version
```

```
tail -f /dev/null
```

### 删除none镜像

```
清空
docker rmi $(docker images -a|grep none|awk '{print $3}')
```

## docker 例子

```
docker pull nginx
docker image run nginx:latest -P 80:9870
curl localhost:9870
```

##### **docker**安装nginx

docker image pull nginx
docker run -p 3344:80 nginx
curl [localhost:3344](http://localhost:3344/)

##### **docker安装tomcat**

docker  image pull tomcat=9.0
docker container run -d -p 3355:8080 --name="tomcat01" tomcat
docker  container exec -it tomcat01 /bin/bash
​cd   /usr/local/tomcat/webapps
cp webapps-dist/* webapps


# dockerFile
>构建生成镜像，镜像是一层一层的。
>开发者开发完毕，使用docker封装好环境和服务，使用者使用命令拉取和运行，不用关心运行环境。
  
```
DockerFile编写时分层应该按照功能，基础的功能放在前边，需要修改的放到后边。比如构建一个nginx的基础镜像，然后在这个基础上再编写导入前端代码的dockerFile。
```

```
如果直接携带docker镜像，他只是作为一个固定镜像存在，如果需要更改，需要另外编写一个DockerFile，在它的基础上更改。
```


```
#关于编译环境和发布环境
ARG webhost=locahost:4242，定义的ARG在build必须通过**--build-arg a_name=a_value**形式指定。
--from=build 从build复制，build为基础环境的别名.
如FROM  centos as build.多个from是为了编译环境和发布环境分离，使得最终的镜像只包含需要的部分。
或者在docker-compose中指定。
  dockerfile1例子：
  FROM centos
  VOLUME ["volume01","volume02"]
  CMD echo "---end---"
  CMD /bin/bash
```

  使用docker build -f dockerfile1 -t   weitao/centos .构建dockerfile文件。

```
docker build -t myjenkins-blueocean:1.1 .
```

##### 基本命令

  FROM，基础镜像

  MAINTAINER， 作者

  RUN，build时执行的命令

  ADD，添加镜像，如tomcat压缩包

  WORKDIR，工作路径

  VOLUME，挂载目录

  EXPOSE，保留端口配置

  CMD,容器run时执行的命令

  ENTRYPOINT，会追加之前输入的命令参数。

  ONBUILD,当构建一个被继承DockerFile时，运行ONBUILD命令。

  COPY,类似ADD，将我们的文件拷贝到镜像中。

  ENV，环境变量。

  

##### 构建镜像例子

  FROM centos

  MAINTAINER weitao

  ENV MYPATH /use/local

  WORKDIR $MYPATH

  RUN yum -y  install vim

  EXPOSE 80

  CMD echo $MYPATH

  CMD /bin/bash

  

  docker history 镜像id，查看构建步骤

##### 构建tomcat镜像例子

  FROM centos

  MAINTAINER weitao

  COPY readme.txt  /usr/local/readme.txt

  ADD jdk  /usr/local

  ADD apache   /usr/local

  ENV MYPATH /usr/local

  WORKDIR $MYPATH

  ENV JAVA_HOME  /usr/local/jdk

  ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

  ENV CATALINA_HOME /usr/local/apache-tomcat

  ENV CATALINA_BASH  /usr/local/apache-tomcat

  ENV PATH $PATH:$JVA_HOME/BIN:$CATALINE_HOME/LIB:$CATALINA_HOME/bin

  

  EXPOSE 8080

  CMD /usr/local/apache-tomcat/bin/startup.sh && tail -F  /usr/local/apache/bin/logs/catalina.out

##### 整合springboot，打包docker发布

  FROM java:8

  COPY  *.jar  /app.jar

  CMD ["--server.port=8080"]

  EXPOSE 8080

  ENTRYPOINT [“java","-jar","app.jar"]



# docker compose

## docker-compose.yml 文件
```
#通过docker-compose.yml管理多个容器,需要指定镜像名，镜像构建路径，进行挂载目录和网络的设置等:
version: "2"
services:
  mysql:
    image: vs-mysql
    build:
      context: ./docker/mysql
      dockerfile: ./Dockerfile
    environment:
      - MYSQL_ROOT_PASSWORD=root
      - MYSQL_USER=vsserver
      - MYSQL_PASSWORD=vsserver
      - MYSQL_DATABASE=gmall_report
    volumes:
      - "./data/mysqldb:/var/lib/mysql"
    ports:
      - 3307:3306
    networks:
      vs_server_net:
        ipv4_address: 172.42.1.10

```
##### depends_on

```
被依赖的镜像必须先启动，而
```

https://docs.docker.com/compose/install/

```
docker-compose up
docker-compose down
docker-compose --scale servername=5
docker-compose build servername
docker-compose up servername

```

.dockerignore在docker-compose.yml的同级。ignore中的文件会被忽略，dockerfile无法执行对应COPY等操作。dockerfile中的COPY路劲必须是相对路径，以docker-compose的同级目录为基准。

### 网络管理

```
docker network ls
docker network inspect  <网卡id> 
docker network rm <网卡id> 
```

```
docker -p 4242:80 会将容器内端口映射到主机所有ip的4242端口上，也可以通过指定ip，限制其映射。如下：
docker -p 192.168.0.5:4242:80,此时只有192.168.0.5:4242能访问容器的80端口服务。
```

### problem

##### File "docker/transport/unixconn.py", line 43, in connect FileNotFoundError: [Errno 2] No such file or directory

```
只要说urllib相关的错误，极有可能是没启动docker服务端，导致docker客户端发发送请求失败。
```



##### Error response from daemon: file integrity checksum failed

1.rebuild and then try to save image

2.memory for docker is not enough.

```
docker system prune -a 
#then change the mrmory for docker on your system
```

3.try to use save and export  command

##### no such file or directory

打开.dockerignore，注释掉忽略的文件夹。

##### 挂载目录没有权限访问

```
使用sudo chmod -R 777 ./data，允许所有人访问
```

# Docker离线环境使用

>由于某些情况，在部署环境无法连接到互联网，那么docker就无法在线拉取镜像，并且DockerFile中编写的apt命令、pip命令等网络请求都会无法工作，导致镜像无法构建。
>
>可用的解决方案有：1自己针对需要的apt、python包下载好离线版本，然后让apt和pip离线安装，这样在离线环境也可以成功构建，但这种方式工作量很大，费时费力。
>
>2将原本的dockerfile拆分为两部分，有网络请求下载依赖的作为第一部分，其他的容易变动的代码部分作为第二部分。第一部分，仍然在dockerfile中使用apt和pip命令请求网络资源，然后构建镜像，将构建好的镜像导出为tar。然后在第二部分的Dockerfile中引入此tar镜像(FROM part1-image:1.0)，就得到了拥有完整运行依赖的环境。当在离线环境中部署时，需要携带第一部分的tar文件和第二部分的Dockerfile文件，然后就可以对第二部分进行构建了。

## 第一部分镜像

>首先编写es_search_base第一部分镜像的Dockerfile

```
FROM python:3.6-slim
WORKDIR /usr/local

RUN cp /etc/apt/sources.list /etc/apt/sources.list.bak  && \
echo 'deb http://mirrors.ustc.edu.cn/debian stable main contrib non-free' >>/etc/apt/sources.list  && \
echo 'deb http://mirrors.ustc.edu.cn/debian stable-updates main contrib non-free' >>/etc/apt/sources.list

RUN useradd -m platform \
&&pip install flask_sqlalchemy==2.5.1\
 sqlalchemy==1.4.17\
 pymysql==1.0.2\
 flask==1.1.2\
 elasticsearch==7.11.0\
 tornado\
 urllib3  -i https://pypi.douban.com/simple    \
&&echo "finished downloading python libs"
```

>然后构建第一部分镜像并导出为tar

```
docker build -t es_search_base:0.1 .
docker save  es_search_base:0.1  > es_search_base.tar
```

## 第二部分镜像

>首先编写第二部分镜像es_search的Dockerfile，引入第一部分镜像

```
FROM es_search_base:0.1
RUN echo "Copying  code"
COPY  --chown=platform:platform  ./entrypoint.sh /usr/local/entrypoint.sh
COPY  --chown=platform:platform  ./es_search /usr/local/es_search
WORKDIR /usr/local/es_search
CMD ["/usr/local/entrypoint.sh"]
```

>然后load第一部分镜像的tar文件，这样才能构建第二部分的dockerfile

```
docker load <  es_search_base.tar
#进入第二部分Dockerfile所在目录，构建镜像
docker build  es_search:1.0  .
```



