# 搭建博客

创建名为username.github.io的repository。进入setting，开启pages，选择主题下载它的压缩包，解压后上传放在repository中。[index.md](http://index.md/)为根网页,目录结构会被解析为路径，md文件会被解析为静态网页，也可以不写html后缀访问。使用nginx反向代理国内访问。谷歌收录，使用site语法查询，若没有则进入search console操作。添加网址，加头部，上传html都试一下。添加站点地图，若失败，试试html等格式的sitemap。添加robots.txt。

## 基本语法

为了建立静态页面之间的关系，进行页面跳转，使用规定语法进行。

### 链接
英文方括号加圆弧括号 方括号填写名称，圆弧括号内为网页路径

### cdn



# git命令

## 基本命令

分为远程仓库操作和本地操作
本地仓库由三棵树构成，
​1工作目录  ->add->  2暂存目录   ->commit->    3head  ->push   云端

实际提交改动通过push将head中的文件推送到云端服务器。

#### git branch
git branch 显示本地分支
git branch -r 显示远程分支
-a 列出远程和本地分支
git branch name 创建本地分支
git branch -m   newbranch 重命名分支
-d branchname 删除分支
-d -r  branchname删除远程分支

#### git add,git commit，git push
git add <filename>提交到暂存区
git commit -m “message”提交到head，应用改动。

git push origin master 将head种文件提交到master
 完整流程:
git init
git add [README.md](http://readme.md/)
git commit -m "first commit"
git branch -M main  更改分支名字
git remote add origin  git@github.com:benbenwt/test.git
git push -u origin main

 git push -u -f origin main，覆盖远程分支。

##### git add -A

添加所有修改信息到暂存区

#### git remote

默认显示仓库信息
​-v 仓库信息
​

#### git remote add  origin 远程仓库url
为当前本地仓库设定远程仓库，origin为对远程仓库的命名。

#### git拉取

完整流程：
git fetch origin dev（dev为远程仓库的分支名）
git checkout -b dev(本地分支名称) origin/dev(远程分支名称)

git checkout main,切换到本地main分支

git pull origin dev(远程分支名称)

```
git reset --hard origin/master  强行合并，本地的update覆盖掉。
```

#### git fetch

#### 连接github仓库及仓库
ssh-keygen -t rsa -C "youname@example.com"
进入网站添加key
 ssh -T git@github.com

git config --global [user.name](http://user.name/)"mona lisa"
git config --global [user.email](http://user.email/)

git init
git add [README.md](http://readme.md/)
git commit -m "first commit"
git branch -M main  更改分支名字
git remote add origin  git@github.com:benbenwt/test.git
git push -u origin main
​

# problem

## pull，push，fetch等操作卡住

​		使用GIT_TRACE=2  GIT_CURL_VERBOSE=2 git fetch查看耗时进程，再次执行卡住操作即可