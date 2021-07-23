### 概念

```
符号执行解决的问题：你拥有一个二进制文件，sb能帮你模拟执行，模拟寄存器，内存的行为。至于操作系统的其他模块，如文件，网络等其他设备，只支持部分。
总的问题：拥有一个恶意二进制文件，需要从中得到情报即信息。
CLE模块载入二进制文件，然后反汇编生成汇编码并进行block的划分。（反汇编为哪一种架构呢看，如何识别的？）
生成VEX中间语言，该语言类似cpu语言，可进行模拟执行。（怎么模拟执行，汇编不行嘛，如何由汇编映射到中间语言的）
进行模拟执行，对不同的函数执行不同的SimProcedure，模拟修改内存和寄存器信息。（使用vex模拟执行，为什么又和Simproducer交互呢，是函数级别的行为嘛）
```



### 点

```
虚假控制流
探索执行策略
路径爆炸
求解器
```



### other

```
自动映射json，合格的全文搜索和聚合统计功能。
1使用、学习、掌握什么技术和知识，2解决什么问题？
例如：符号执行，公共子图，恶意软件分类
题目意思？？？  云关键技术，威胁情报、安全（恶意软件、cve、apt）
添加限制，如精度、速度、资源消耗
威胁情报的存储*、管理、使用（json格式），适用于威胁情报特点的json格式的存储机制。
```

### 数据集

```
准备数据集，确定类别和数量。
```



### 论文

```
#Forecasting Malware Capabilities From Cyber Attack Memory Images
代码提供在此处：https://cyfi.ece.gatech.edu/.
code:https://github.com/CyFI-Lab-Public/Forecast
摘要:
捕获恶意软件内存镜像，通过镜像进行后续符号执行。通过api序列、参数、数据流向确定能力，并根据数据流向计算每个能力出现的可能性。实现了识别7个能力的框架包，以及预测能力可能性。在符号执行时，维护符号执行值和实际值，通过其定义的方法计算DC，进而计算能力分支可能性。相比单独的内存取证，符号执行能获取到api的参数信息，例如strncpy参数中的url。

样例：
1使用DarkHotel样本作为示例，展示算法的功能,DakrHotel通过鱼叉攻击进行apt威胁。DarkHotel感染后删除自身的二进制，并与C&C服务器通信，将线程注入到Windows Explorer中，并过滤侦察数据。

2关于如何确定能力，其记录api调用序列，并记录api之间的数据流向关系，特别是api参数流向。使用AveMaria作为示例，它code injection系统的Sucshost服务，从而窃取Firefox的cookie文件，也窃取屏幕信息。

实验：
在6727多个样本上进行了实验，包含274个家族。

code:
官网下载ubuntu18.04.5系统，clone提供的forecast项目到虚拟机。安装python3.7-dev环境，按照readme安装所需的依赖。
1.Setup the virtual environment python3.7 -m venv venv
2.Activate the virtual environment . ./venv/bin/activate
3.Install custom cle pip install -e ../path/to/Forcast/cle
4.Install custom angr pip install -e ../path/to/Forcast/angr
5.Install simprocedures pip install -e ../path/to/Forcast/simprocedures
6.Install forsee and dependencies pip install -e .[dev]
#运行
python forsee.py 
可以使用相同的方式检测函数的前后关系，但是这种方式只使用特定的函数序列。
```

```
#Technique for malware analysis
```

```
#Optimize symbolic execution  for malware classification
```



```
blogs->网络，文件IO
连续block的相同函数，block划分策略
遍历图
顶会代码
```

### 栈例子

```
void fun(int a) {
	int b;
	char s;
	gets(&s);
	if(a == 0x1234){
		puts(&s);
	}
}
```

![函数栈&EIP、EBP、ESP寄存器的作用](https://www.k2zone.cn/wp-content/uploads/2018/09/TIM%E6%88%AA%E5%9B%BE20180914001704.jpg)



### 0b7ab5021e8fcf319d628e52e2121f82

```
fileread : /lib/i386-linux-gnu/libc.so.6, size=512
process : execute a file ,Clone syscall, fork or vfork
printf: .
缓冲区溢出漏洞攻击植入
设置自启动

获取execl函数的argv，argc得到sbin/linuxconf字符串
```

### HOOK

```
hook会替换原函数的功能，影响state或simgr进行模拟
```

### CLE

```
https://github.com/angr/cle/tree/master/cle/backends
```



### simproducers

```
angr/angr/procedures/
https://docs.angr.io/extending-angr/simprocedures
state.inspect.simprocedure
```

### Block

```
proj.factory.block(state.addr).capstone

```



### 基本元素

```
address
block,node
function
state
cfg
```

### 函数

```
functions=cfg.kb.functions
addresses=[(addr,func.name) for addr,func in functions.items()]
addresses
#addr->func对象
entry_func=cfg.kb.functions[4195789]
```

### NODE

```
cfg.model.nodes()
G.nodes()
```

### 图

```
https://blog.csdn.net/wspba/article/details/75671573
gspan

连续的相同功函数合并
能修改和展示
如何收集内存数据
那个库

遍历它们，复制一个图出来。
```



### 数据集

```
microsoft 在kaggle举办的2015 malware classification，包含二进制文件。
Microsoft Malware Classification Challenge (BIG 2015)
ember数据集，不包含二进制文件，只有提取的信息。
天池的新人赛，提供了二进制文件。
```

### Temp

```
寄存器，内存（函数堆栈），函数参数

除了api调用外额=外获得寄存器、内存信息，函数的传递参数,行为造成的后果，如改变注册表，防火墙，文件等。

求解，路径剪枝，外部函数库
生成图的策略，合并，丢弃
图

使用了wget如何获取到？寄存器字符串？
api顺序是否重要？需要顺序判断api组合是否恶意，或者说顺序不重要，只会混淆结果。

抛弃部分不达标路径，函数过少，循环过多等。
合并相同函数-地址，合并多个图为一个
每个路径一个tid，给一个index。
```

### 输入与输出

```
state.posix.dumps(0)
state.posix.dumps(1)
#求解标准输入，与state.posix.dumps(0)同义
input_data = state1.posix.stdin.load(0, state.posix.stdin.size)
state1.solver.eval(input_data, cast_to=bytes)
```

### 求解器

```
#solver
state1.solver.eval(input_data, cast_to=bytes)
#
state.solver.add(x > y)
state.solver.eval(x)
```



### keyword

```
自动化
angr
```



### 相关组织网站

```
def con
darpa
```

### Solver

```
manx
min
```

### Simulation_manager

```
simgr通过stash管理状态，它将state分为如下几类。
active
deadended
unsat
unconstrained
pruned

state.active
state.unsat

#state 迭代
while True:
 	if len(rsb.successors)==2:
        break
    addr=state.block().addr
    print('addr :',hex(addr),end=" ")
    rsb=state.step()
    state=rsb.successors[0]
    
#simgr 迭代
while len(sim.active) == 2:
    print('0 :',hex(sim.active[0].addr))
    print('1 :',hex(sim.active[1].addr))
    sim.step()
```

```
simgr的exploration策略
深度优先:将其他路径放入deferred stash中
explorer:允许使用find和avoid规则
MemoryWatcher:控制内存不超过阈值
Loopseer:循环丢弃等待
veritesting:合并关键点
exploration策略可以用简单的address表示，或者用自定义函数控制返回true或false。通过find和avoid策略进行路径探测并求解值。

simgr.use_technique(tech)
angr.exploration_techniques
```



```
#Techniques for Malware Analysis based
To implement Symba ,a custom exploration technique based on constraint   creation has been  developed and applied to the manager.
开发了自定义的路径探索策略（也可以是外部函数库策略，求解策略）
给定关键函数名，揭示所有相关的路径和约束。通过过去祖先block地址，创建状态并执行。输出输入信息，路径信息，函数信息。
寄存器，api序列，（网络，文件IO，内存，注册表）,
```



### Angr基本API

>见自己的angr_learn.ipython

```
node.block.capstone.insns
```



```
#CFGENode __x86.get_pc_thunk.bx
mov %eip,%eax
```



```
[149, 86, 225, 23, 99, 202, 101, 240, 139, 166, 177, 204, 114, 141, 51, 78, 89, 116, 15, 154, 53, 156, 232, 30, 169, 106, 43, 198, 209, 108, 135, 45, 184, 83, 58, 85, 150, 60, 226, 163, 201, 228, 165, 75, 102, 113, 39, 178, 77, 216, 142, 153, 180, 90, 27, 54, 92, 29, 195, 105, 170, 197, 172, 71, 210, 120, 147, 46, 57, 212]
[32, 187, 124, 162, 254, 137, 74, 229, 49, 24, 179, 91, 246, 129, 66, 221, 41, 196, 16, 171, 7, 146, 238, 213, 33, 188, 125, 8, 100, 255, 138, 230, 50, 205, 25, 117, 0, 155, 247, 130, 67, 222, 42, 17, 109, 248, 84, 239, 59, 214, 34, 189, 126, 9, 164, 76, 231, 206, 26, 181, 118, 1, 93, 131, 68, 223, 18, 173, 110, 249, 148, 224, 215, 35, 190, 127, 10, 241, 140, 52, 207, 182, 119, 2, 157, 94, 233, 132, 69, 208, 44, 199, 19, 174, 111, 250, 61, 200, 36, 191, 112, 11, 103, 242, 217, 192, 28, 183, 104, 3, 158, 95, 234, 133, 70, 121, 20, 175, 96, 251, 87, 62, 37, 176, 12, 167, 88, 243, 79, 218, 193, 168, 4, 159, 80, 235, 134, 185, 122, 21, 160, 97, 252, 151, 72, 227, 63, 38, 13, 152, 244, 143, 64, 219, 55, 194, 5, 144, 81, 236, 56, 211, 47, 186, 123, 22, 161, 98, 253, 136, 73, 48, 203, 115, 14, 245, 128, 65, 220, 40, 31, 107, 6, 145, 82, 237]
```



```
pydot :File not found安装Graphviz库https://blog.csdn.net/somecare/article/details/88311874
```



```
#静态分析控制流分析，数据流分析，符号执行#动态分析Hook api调用分析#插装工具 pintools,#SMT约束求解器#符号执行平台KLEE,S2E,QSYM，ANGR
```

| Name | Object      | IR     | Program Language | concolic execution |
| ---- | ----------- | ------ | ---------------- | ------------------ |
| KLEE | source code | LLVM   | C++              |                    |
| S2E  | binary      | LLVM   | C++              | QEMU with KVM      |
| angr | binary      | VEX IR | Python           | Unicore            |
| QSYM | binary      |        | C++              | directly on CPU    |



```
#graphembedding,entity linking,class,#生成embedding algrothimNode2Vec,Metapath2Vec,HAN,DEEPWALK,TransE#应用link prediction(graph completion),correlation,community#keyworld#数据集Freebase,yago
```





