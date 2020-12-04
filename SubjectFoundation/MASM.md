# dosbox

>www.dosbox.com下载安装即可

# MASM

>http://www.mediafire.com/file/x13v7gqqmw1pom7/8086_Assembler%2528TechzClub_joyjophin%2529.zip/file
>
>http://techzclub.com/how-to-run-masm-software/
>
>检索MASM dosbox，下载后解压到d：MASM
>
>在dosbox中使用mount c d:MASM挂载目录
>
>c：进入主盘
>
>dir查看可用编译器。环境完成。

### 基本操作

##### debug.exe

debug启动调试模式

debug  1.exe,逐步调试。

r，查看寄存器值

r 寄存器名称，修改寄存器值。例如r ax

d 段地址：偏移地址，查看存放的机器码。

d 段地址：起始地址 偏移地址，查看起始到便宜之间的。例如：1000:0 1，查看0到1之间的。

d 寄存器名称：偏移地址

上一条可以不写数据，以交互式修改。

e 起始地址 数据 数据 ....，改写内存内容。例如：e 1000:0  9 7 6.

e 寄存器：偏移地址

u  段地址：偏移地址，查看机器码含义

u 寄存器：便宜地址

t  ：命令执行CS+IP指向的机器码

>t命令执行修改段栈寄存器ss的指令，会直接随后执行下一条指令。

a   地址：汇编形式写入机器指令。

a 寄存器：偏移地址

q,推出当前模式

##### link.exe

>-link

##### edit.ext

>edit

masm.ext

>masm进入编译
>
>masm>程序名称，执行exe

