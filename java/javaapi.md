```
FileInputStream  
FileReader,FileWriter
```

```
FileInputStream
从content root开始计算路径
```



##### 关于编译和打包

```
javac进行编译，将源码编译为字节码文件，此时会检查import的文件和代码中使用的类，若不存在也报错。我们通过-cp指定classpath引入jar依赖或class文件依赖，以这种形式指定cp后，不用在文件中写import A也可以直接使用A。编译后的class文件并不包含依赖，运行仍需要依赖的jar或class文件。
jar指令进行打包，他会将编译的class文件（需要自己手动编译为class）打包为jar，还需要提供META-INF等文件夹存储该jar包的清单，这是虚拟机要求的。使用jar指令打包的文件不能自动引入依赖的jar包和class，所以要手动加入jar包和class文件，就能打包为能单独运行的class，因为其包含了需要的class和jar，当然jar包会被解压在jar内融合为一个，按照路径进行存储。maven帮我们完成了这些繁杂的jar包依赖工作，特别是找到jar包依赖的位置。具体参数输入jar指令查看帮助。
如何确定所需方法在哪个class文件或jar，或者说打包时和编译时如何确定的这些依赖？
依赖的class文件直接通过文件夹路径和文件名字确定位置，依赖的jar通过相同的原理。以当前工作路径为根，构建添加新的路径索引，不会真的复制到一个文件夹下。
jar教程：https://www.cnblogs.com/jayworld/p/9765474.html

manifest为清单内容，test为class所在文件夹，.代表所有文件。

mainfest内容:
Manifest-Version: 1.0
Main-Class: B

jar -cvfm main.jar manifest -C test .

所以手动的完整过程为，javac编译文件(注意依赖)。创建mainfest文件，填写内容，jar 打包上一步编译的文件(注意依赖)。
```

