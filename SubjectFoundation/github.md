### gitignore语法

##### 写法

```
https://www.cnblogs.com/xkfeng/p/7835055.html

*.h5，忽略h5文件
target/,忽略targe文件夹
```



##### 基础示例

```
/webpages/fontAwesome
HELP.md
target/
!.mvn/wrapper/maven-wrapper.jar
!**/src/main/**
!**/src/test/**

### STS ###
.apt_generated
.classpath
.factorypath
.project
.settings
.springBeans
.sts4-cache

### IntelliJ IDEA ###
.idea
*.iws
*.iml
*.ipr

### NetBeans ###
/nbproject/private/
/nbbuild/
/dist/
/nbdist/
/.nb-gradle/
build/

### VS Code ###
.vscode/

```





# git工作流程

```
分为工作目录，暂存区域，head区域，远程仓库。
checkout创建新的工作目录分支
rollback回归到特定分支的某一版本号，回滚本地部分。
new branch创建新分支
merge合并多个分支，会有冲突代码。
git rebase -i  [startpoint]  [endpoint] ，合并多次的commit，避免push多次和解决多次冲突。将某一分支的变化提交到指定名称分支。
```

```
git log，提交历史
```

### git工作暂存区reset

```
https://www.liaoxuefeng.com/wiki/896043488029600/897889638509536
```



# git命令

>本地仓库由三棵树构成，
>​推送时的顺序：工作目录区域  ->add->  暂存目录区域   ->commit->    head区域  ->push->  远程仓库
>
>拉取时的顺序相反，所有有时可以使用暂存目录区域的内容恢复拉取操作之前的内容。
>
>实际提交改动通过push将head区域中的文件推送到远程仓库。

### 克隆子模块

```
git submodule add <url> <path>
git submodule init
git submodule update
git submodule update --init --recursive
```



### 添加代理

```
git config --globalproxy http.proxy  'socks5://127.0.0.1:1080'
```

### 初始化及配置

```
ssh-keygen -t rsa -C "111111@qq.com"
#进入github网站,将public-key添加ssh-publicKey
ssh -T git@github.com
#出现如下表示ssh配置成功，Hi benbenwt! You've successfully authenticated, but GitHub does not #provide shell access.
git config --global user.name    "mona lisa"
git config --global user.email   "邮箱地址"
#进入项目根路径
git init
#到网页创建test仓库
#添加可选的远程仓库
git remote add test  git@github.com:benbenwt/test.git，创建远程仓库test。
#拉取到本地
git fetch test main
git checkout -b main test/main
git pull test main
#push的远程仓库
git add -A
git commit -m "first commit"
git branch -M main
git push -u test  main
```

### 基本命令

##### git branch

```
git branch #显示本地分支
git branch -r #显示远程分支
git branch  -a #列出远程和本地分支
git branch name #创建本地分支
git branch -m newbranch #重命名分支
git branch -d branchname        #删除分支
git branch -d -r branchname    #删除远程分支
```

##### git add

```
git add <filename>提交到暂存区
```

##### git commit

```
git commit -m “message”提交到head，应用改动。
```

##### git push

```
git push origin master 将head种文件提交到master
 git push -u -f origin main，覆盖远程分支。
 git push --set-upstream origin main，设定默认上传分支。
```

##### git fetch

#### git remote

```
git remote -v #查看已添加远程仓库
git remote add  origin 远程仓库url  #origin为对远程仓库的命名。
```

#### 拉取

```
git config --list
完整流程：
git fetch origin dev（dev为远程仓库的分支名）
git checkout -b dev(本地分支名称) origin/dev(远程分支名称)
git checkout main,切换到本地main分支
git pull origin dev(远程分支名称)
git reset --hard origin/master  强行合并，本地的update覆盖掉。
```

# problem

##### Unable to access ‘https://github.com/**/**/‘: OpenSSL SSL_read: Connection was aborted, errno 10053

```
git config --global http.sslVerify false
```

##### pull，push，fetch等操作卡住

```
GIT_TRACE=2  GIT_CURL_VERBOSE=2 git fetch  #再次执行卡住操作即可
```

##### 云端文件修改后和本地文件不可同步

```
git push -u origin master -f，强制push，多人协作时不可取
git reset --hard origin/master  强行合并，将本地的update覆盖掉。
```

##### github不上传某些文件

```
touch .gitignore
#添加
target/
test/
```

##### The TLS connection was non-properly terminated

需要重新编译git

https://blog.csdn.net/RedKeyer/article/details/96119

# 搭建博客

​		创建名为username.github.io的repository。进入setting，开启pages，选择主题下载它的压缩包，解压后上传放在repository中。[index.md](http://index.md/)为根网页,目录结构会被解析为路径，md文件会被解析为静态网页，也可以不写html后缀访问。使用nginx反向代理国内访问。谷歌收录，使用site语法查询，若没有则进入search console操作。添加网址，加头部，上传html都试一下。添加站点地图，若失败，试试html等格式的sitemap。添加robots.txt。

## 基本语法

为了建立静态页面之间的关系，进行页面跳转，使用规定语法进行。

### 链接
英文方括号加圆弧括号 方括号填写名称，圆弧括号内为网页路径

320