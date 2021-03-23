```
#清空所有
#network,ports,volumes,depends,devices,sysctls,cap-add
docker-compose up web

COPY --chown=组名:用户名 宿主机目录 容器目录
RUN  脚本命令1\
     &&命令2\
#指定环境变量
ENV <key1>=<value1> <key2>=<value2>...
#开放端口，并不进行宿主机映射.可在run时，通过-P进行映射
EXPOSE <端口1> 
EXPOSE 8080
#容器启动时执行的命令，同entrypoints。可通过--entrypoints在运行时切换
CMD["nginx", "-g", "daemon off;"]
#设定工作目录，随后的命令在此目录执行。若多次使用相对路径，会进行叠加，同cd。
WORDIR
```

# Lisa_docker

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



# docker概述

- docker为什么出现
  开发-上线 两套环境，版本更新，环境不同，无法跨环境等导致运维考验大。环境配置麻烦。项目能否带着环境安装打包。
  ​实现，开发打包部署上线，一套流程做完。

  docker解决方案：项目带上环境放到docker中，需要者下载这个镜像。docker中箱子隔离独立。

- docker的优势
  docker具有更少的抽象层，不需要模拟硬件guest os。
  直接使用宿主机的内核。

# docker安装

https://docs.docker.com/engine/install/ubuntu/，配置依赖及安装ce，cli等。

```
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

https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors

```
vim /etc/docker/daemon.json
#添加
{
  "registry-mirrors": ["https://134lgcv1.mirror.aliyuncs.com"]
}

```



重启服务：
systemctl daemon-reload
systemctl restart docker。

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

##### docker使用

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

### docker 例子

```
docker pull nginx
docker image run nginx:latest -P 80:9870
curl localhost:9870
```

  ### dockerFile

  >构建生成镜像，镜像是一层一层的。
>
  >开发者开发完毕，使用docker封装好环境和服务，使用者使用命令拉取和运行，不用关心运行环境。

  dockerfile1例子：

  FROM centos

  VOLUME ["volume01","volume02"]

  CMD echo "---end---"

  CMD /bin/bash

  

  使用docker build -f dockerfile1 -t   weitao/centos .构建dockerfile文件。

  ##### 基本命令

  FROM，基础镜像

  MAINTAINER， 作者

  RUN，构建需要命令

  ADD，添加镜像，如tomcat压缩包

  WORKDIR，工作路径

  VOLUME，挂载目录

  EXPOSE，保留端口配置

  RUN

  CMD,指定容器启动时的命令

  ENTRYPOINT，会追加命令。

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

  COPPY  *.jar  /app.jar

  CMD ["--server.port=8080"]

  EXPOSE 8080

  ENTRYPOINT [“java","-jar","app.jar"]

  ### 数据卷容器

  > 容器之间数据同步

  

  ### docker网络原理

  ### idea整合docker

  ### docker compose

https://docs.docker.com/compose/install/

```
docker-compose up
docker-compose down
docker-compose --scale servername=5
docker-compose build servername
docker-compose up servername

```

.dockerignore在docker-compose.yml的同级。ignore中的文件会被忽略，dockerfile无法执行对应COPY等操作。dockerfile中的COPY路劲必须是相对路径，以docker-compose的同级目录为基准。

  ### docker swarm

  ### CI/CD jenkins

### problem

##### Error response from daemon: file integrity checksum failed

1.rebuild and then try to save image

2.memory for docker is not enough.

```
docker system prune -a 
#then change the mrmory for docker on your system
```

3try to use save and export  command

##### no such file or directory

打开.dockerignore，注释掉忽略的文件夹。