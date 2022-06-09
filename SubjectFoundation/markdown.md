# 软件和服务

>使用markdown需要一个markdown编辑器，以及一个云端备份服务，云端备份主要是防止本地md文件丢失，以及跨设备进行查阅。
>
>markdown是一种轻量级标记语言，有很多优点，它允许使用纯文本格式编写文档和控制文档样式，支持公式、表格等，只需要保存和备份md文本文件，就可以保存编写的文本和其格式。借助编辑器，还可以将md文件生成pdf文件，方便阅读。其次，由于使用文本格式存储，不依赖于某个软件，当软件有付费操作后，直接使用其他编辑器替代品。

## 编辑器和云备份

>window平台编辑器编辑器使用Typora的beta版本，因为其最新的版本已经开始收费，大部分情况下beta版本已经足够使用。
>
>下载链接：https://www.typora.io/windows/dev_release.html 
>
>window平台md编写完成后可以同步到github、gitlab、onedriver等平台，其中github同步后可以直接网页查看内容。

>Android平台使用坚果云Markdown编辑器，坚果云支持常见的markdown语法及公式语法，并提供了云端备份功能，只需要注册坚果云账号，就可以将本地编写的md文件上传到坚果云备份。直接应用市场下载
>
>坚果云markdown本地md存储目录：/Android/data/netcloud/files  ，该目录可以看到所有的md文件。

## github同步bat

>如果用github同步，可以写个简单的bat进行提交。这个bat会强制提交，如果有冲突，会用本地的内容覆盖远程仓库。

```
:start
set /p fo=input submit path:
goto other

:other
cd  "%fo%"
d:
goto deal

:deal
git checkout main
git add -A
git commit -m "bat"
git push  -u  -f  origin main
:end
pause>nul
goto  start
```

# 基本语法

## 标题

>使用#表示标题格式，几个#就是几级标题。

## 文本块

>使用大于号表示该行是文本块，即>。

## 代码块

>使用三个反引号包裹代表代码块，即```。

## 公式

>开启内联公式：https://zhuanlan.zhihu.com/p/158156773
>
>常用：https://www.jianshu.com/p/25f0139637b7
>
>https://blog.csdn.net/jyfu2_12/article/details/79207643

### 行内公式

>行内公式与文字一起使用，不会独占一行。在typora中打出$后，按esc键就会出现行内公式块，可以在其中添加公式。


$$
\Gamma(z) = \int_0^\infty t^{z-1}e^{-t}dt\,. \Gamma(z) = \int_0^\infty t^{z-1}e^{-t}dt\,.
$$

$$
z=z_l
$$

$$
\frac{d}{dx}e^{ax}=ae^{ax}\quad \sum_{i=1}^{n}{(X_i - \overline{X})^2}
$$

$$
\sum \frac{1}{i^2}
$$

$\sum$





