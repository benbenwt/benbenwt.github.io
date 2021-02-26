### idat

idat.exe可以在命令行中调用python脚本。idag使用图形界面调用，不能自动化批处理。
我们可以编写程序，使用subprocess执行命令行程序，在其中批量调用处理py脚本。
相关博客：https://bbs.pediy.com/thread-254386.htm

##### 环境配置

安装好python27，首先应保证idag.exe图形界面能正常运行py脚本。
随后根据idat.exe所在路径调用命令。

##### 命令行语法

例子：C:\Users\guo\Desktop\run_idat-master\run_idat-master\here\IDA_Pro_v7.0_Portable\idat.exe -A -L"C:\Users\guo\Desktop\run_idat-master\run_idat-master\here\log\wordpad___learn01.txt" -S"C:\Users\guo\Desktop\run_idat-master\run_idat-master\here\script\[learn01.py](http://learn01.py/) C:\Users\guo\Desktop\run_idat-master\run_idat-master\here\output\wordpad___learn01.json" C:\Users\guo\Desktop\run_idat-master\run_idat-master\here\input\wordpad.exe

##### 处理脚本

处理脚本用idaapi.autoWait()  idc.exit(0) 包裹

##### 调用方法

处理得到所有对应路径后,按照语法生成cmd命令后，使用[subprocess.call](http://subprocess.call/)(cmd命令)执行命令。idc.ARGV接受命令行参数。

##### IDC

基本语法
语法类似c
变量类型有整型，浮点，字符串。
例子：
auto addr,name;
for(addr=NextFunction(addr);addr!=BADADDR;addr=NextFunction(addr))
{
     name=Name(addr);
     Message("function name:%s",name);
​}

#### idapython

基本语法

idautils.Funtions()获取所有函数
Name(func)获取函数名

##### problem

can not load python64.dll
配置python27