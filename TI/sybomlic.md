### 0b7ab5021e8fcf319d628e52e2121f82

```
fileread : /lib/i386-linux-gnu/libc.so.6, size=512
process : execute a file ,Clone syscall, fork or vfork
printf: .
缓冲区溢出漏洞攻击植入
设置自启动
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





