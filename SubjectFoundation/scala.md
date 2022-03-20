# 基于语法

### 基本类型

```
```

### 分支控制及循环

### 函数

### 类

### 文件IO

### 格式化输出

```
```



### 网络编程

### 多线程编程

### 函数编程

### mapPartitionsWithIndex

>mapPartitionsWithIndex详解https://blog.csdn.net/yilulvxing/article/details/98968750
>
>mapPartitionsWithIndex通过额外传递一个int类型的index给mappartion函数，是的在mapPartion内部可以打印分区id。
>
>也可以通过如下函数在mapPartion调用函数内部打印partionId。
>
>```
>TaskContext.getPartitionId()
>```
