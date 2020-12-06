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
使用阿里云镜像加速下载，
重启服务：
systemctl daemon-reload
systemctl restart docker。

# docker命令

docker  version
docker  run  hello-world

### 镜像命令

docker info  系统信息，镜像和容器的数量
docker images 查看镜像信息  -a  all  -q id

- docker search 展示仓库中镜像
                  --filter , -f 		Filter output based on conditions provided
                  --format 		Pretty-print search using a Go template
                  --limit 	25 	Max number of search results
                  --no-trunc 		Don’t truncate output

- docker pull 下载镜像
  docker pull [OPTIONS] NAME[:TAG|@DIGEST]
  docker使用分层下载，联合文件系统。

- docker rmi -f删除镜像
  -f force

### 容器命令

容器start后在运行队列中，stop后退出运行队列。(带上-a参数查看所有容器，不带为正运行容器。)仍在容器列表内，可使用rm移除。
标准格式命令：
docker    container/images  start/stop/rm/run/pause
​

- docker run使用镜像创建容器并strat容器

  --name 		Assign a name to the container
  -d 后台运行
  -i  使用交互方式，进入容器查看内容
  -P   ip：主机端口：容器端口。
  -p随机端口
  docker run -it centos  /bin/bash启动并进入容器
  exit 推出容器
  ctrl+p+q 不关闭容器推出
  ​

  - 若创建的容器未提供服务，自动stop

- docker rm 删除创建的容器
  docker rm -f $(docker ps -aq)  移除容器，若使用需再次run
  ​linux管道服务：docker ps -a -q|xargs docker rm
  ​

- docker container ls  
  或docker container ls 
  ​-a   正在运行+历史运行,即已创建的容器
  -n=?  显示最近创建的容器。
  ​
  ​

- docker container start 容器id，启动创建好的容器
  docker container restart id  重启创建好或正在运行的容器
  docker  container  kill id 杀死正在运行进程
  docker container   stop id  停止正运行进程。

### 常用其他命令

- 日志
  docker logs  -f -t --tail  容器
  执行脚本：docker container  run  -d id   /bin/sh  -c "脚本内容"

- 容器进程
  docker top  id

- 元数据
  docker history创建历史
  docker tag将镜像放入仓库并标记
  docker inspect id

- 进入容器命令和拷贝命令
  docker container exec -it  id    /bin/bash  "命令内容“  新建一个终端,i表示保持命令行不关闭，t表示创建新的终端。
  docker container attach  id  进入正在运行的命令行
  容器拷到宿主机：docker  cp  id:path   宿主机path
  主机到容器：挂载

- docker使用

  - docker安装nginx
    docker image pull nginx
    docker run -p 3344:80 nginx
    curl [localhost:3344](http://localhost:3344/)

  - tomcat
    docker  image pull tomcat=9.0
    docker container run -d -p 3355:8080 --name="tomcat01" tomcat
    docker  container exec -it tomcat01 /bin/bash
    ​cd   /usr/local/tomcat/webapps
    cp webapps-dist/* webapps

  ### docker镜像

  ### docker基础例子

  >docker pull nginx
>
  >docker run -d --name nginx01 -p 3344:80  nginx
>
  >若使用80：80无法访问，可能被占用，请使用其他端口。
>
  >curl localhost:3344
>
  >若使用容器卷，挂载目录使用sudo才可修改。

  ### 容器数据卷

  >搭建服务环境后，在环境中部署应用。但是删除容器后，数据会丢失。所以将docker目录挂载到外部，实现保存。

  docker inspect，查看容器信息。

  docker run -it -v 主机目录：容器内目录,挂载目录。

  例如：docker run   -it -v /home/ceshi:/home centos  /bin/bash

  docker run -d -p  3310:3306  -v  /home/mysql/conf:/home/conf    mysql  /bin/bash
  
  docker attach 容器id，进入运行的容器。
  
  
  
  应用举例：
  
  将nginx配置文件挂载出来，不用进入容器修改配置。
  
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

  ### docker swarm

  ### CI/CD jenkins