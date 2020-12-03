# 概览

1自动化批量系统初始化程序（update，软件安装，时区设置，安全策略）

2自动化批量软件部署程序（LAMP/LNMP/Tomcat/）

3管理应用程序（KVM，集群管理扩容，Mysql）

4日志分析处理程序（PV,UV,200,!200,top 100,grep/awk）

5自动化备份恢复程序（Mysql完全备份/增量+Crond）

6自动化管理程序（批量远程修改密码）

7自动化信息采集及监控（收集系统/应用状态信息，CPU,Mem，Disk，Net,Apache,MySQL）

# 其他软件的命令

### 文件

### 网络

# 脚本

>指定shell解析器 #! /bin/bash
>
>注释符号： #
>
>输出：echo $str1
>
>其他服务的命令

### 变量

>字符串，数组，整数

name="bob"

conut=123

str1="${name}hello! "

unset name

创建后变量只读

##### 变量使用范围

本地变量：只能在shell内使用

环境变量：对用户或所有用户可见，如PATH，env。

位置变量：bash pos.sh a b c d e，依次打印脚本名，参数名等。

pos.sh内容：echo $0,echo $1 .......

$*，所有参数

$#，参数的个数

$?,上次的执行结果，0为正确。

##### path变量

用户环境变量在~/.bashrc ，系统环境变量在etc/profile中。

添加path变量，打开对应文件，export PATH=$PATH:添加的目录。

source ~/.bashrc ，source读取并执行文件中命令。

### 文件包含

. include.sh

source include.sh

### 字符串

双引号：可以使用$,`,\,",之外的，可以解析变量。

单引号不会解析变量，所有字符当常量字符。

反引号：`表示当作命令执行的结果。

#str1：长度

echo ${str1:2:3}

### 数组

array=(a b c d)

echo $(array[0])

### 各种运算

expr 1+3

expr 1\*3   

less,grater,than,equal

-lt:小于

-le:小于等于

-gt:大于

-ge:大于等于

-eq:等于

-nq:不等于

test 1 -eq 2,0为成功。



字符串比较：

​	=,!=

​    -n:字符串长度不为0，为真。

​    -z：长度为0时，结果为真。



文件判断：

-f:普通文件

-d：目录文件

-w：文件可写

-x:文件可执行

-s：文件至少有一个字符

-c:字符设备文件

-b：块设备文件



逻辑判断：

  -a:逻辑与

  -o：逻辑或

   ！：逻辑非

### 分支结构

>[   ]格式前后有空格
>
>read -p "input a char:" ch  获取输入存到ch中

##### if

if [  2 -lt 3  ];then

​	echo "2<3"

elif [];then

else

​	echo "2>=3"

fi 



if [ "abc"="abd" ]



##### case



case $ch in 

[a-z])

​	echo "alpha"

;;

[0-9])

​	echo "number"

;;

*)

​	echo "others"

;;

esac



##### for

for x in a b c

do 

​	echo $x

done



for x in /*

do

​	echo $x

done



sum=0

for i in `seq 1 10`

do 

​	let sum+=$i

done

echo $sum        

​        

for(i=0;i<10;i++)

do 

​	echo $i

done



##### while

i=1

while [i -le 10]

do

​		let i+=1

​		i=$[$i+1]

​		#((i++))

done



##### util

i=1

util [$i -gt 10]

do

​	echo $i

​	let i+=1

done

##### break

while :

do

​	read -p "input" $char

​	case $char in

​	[1,5]):

​			echo "ok"

​	;;

​	1|2|3):

​		echo "ok"

​	;;

​	*):

​		echo "a"

​	;;

esac

done



##### continue

i=1

while [$i -le 10]

do

​	if [$()$i%3 ) -eq 0];then

​		echo "a"

​		((i++))

​		continue

​	fi

echo "b"

((i++))

done

##### 函数

func1()

{

​		echo $1

​		echo $2

​		return "hello"

}

func1 123 abc

$?返回值

##### 逻辑运算

if []&&[]

[]||[]

if [c1 -a c2]

[c1 -o c2]

##### /dev/null

echo  /dev/null >1.txt

清空1.txt

##### printf

printf"hello %s %d"  "world" 123

##### set

-n:结合命令p使用，打印内容。

-i:将缓冲区内容插入文件，使其生效。

p:打印

a:增加

d：删除

c：替换整个内容

s：替换特定内容

sed -n '1p' 1.txt，打印第一行

sed -n '1,3p' 1.txt,打印1至3行

sed -n '$p' 1.txt，最后一行

sed '1a xxx' 1.txt，在第一行添加

sed '3c afafa' 1.txt,替换第3整行

sed  '3,7s//jin/xxx/g ' 1.txt