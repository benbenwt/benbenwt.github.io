# 重点章节

存储器（第三章），其中容量的扩展和cache存储器为重点
指令系统（第四章），寻址方式的设计
CPU（第五章），指令周期和微程序控制器

# 第二章 运算方法与运算器

- 数据的表示方法

  - 数据格式

    - 定点数表示方法
      使用n+1位表示纯小数和纯整数，从左向右第一位表符号。
      纯整数：绝对值最小为0，最大为2n-1
      纯小数：绝对值最小为0，最大为1-2-n

    - 浮点数的表示方法
      N=2e*M  M为尾数（定点小数），2为基数，e为指数（整数），常称为阶码​。
      一个机器浮点数由阶码，尾数，符号构成。
      IEEE754标准：
      S E M  ：符号位，移码，尾数。移码隐含阶符，还原用e=E-127.
      1  8  23   32位
      ​​​1  11   52  64位
      尾数最高有效位应为1，称为规格化表示，故为1.M。
      32位：x=(-1)S*（1.M）*2（E-127）
      ​64位：x=(-1)S*（1.M）*2（E-1023）

    - 十进制数的表示方法
      1字符串形式，一个字节存储一个十进制数位或符号位，主要用于非数值计算。
      2压缩的十进制数串形式，一个字节存放两个十进制数位。使用bcd码。

  - 数的机器码表示

    - 原码
      使用最高位作为符号位，1为负。

    - 反码
      正数不变
      负数符号不变 ，数值取反。

    - 补码
      整数不变
      负数符号不变，数值为加一。

    - 移码
      表示范围为：-127-+128
      因为原为：0-255  减去127得-127-+128

- 定点加减法运算
  使用补码加减即可

- 定点运算器的组成
  算术/逻辑运算单元，数据缓冲寄存器，通用寄存器，多路转换器，数据总线

# 第三章 存储器

- 存储器容量的扩展

  https://blog.csdn.net/xiaoxi_hahaha/article/details/84837178

  - 概念
    1k*4表示容量为1k=1024（10位二进制），数据线位数为4的存储器。他需要10位二进制表示地址。

  - 位扩展![img](https://api2.mubu.com/v3/document_image/34d9c993-b951-43d9-84af-41b98cfe3617-6012434.jpg)
    增加存储字长，如用1k*4位的存储芯片构成1k*8位的存储器，扩充数据线位数，控制线和地址线共用。D位数据线，A为地址线。
    ​

  - 字扩展![img](https://api2.mubu.com/v3/document_image/d39ebad6-4318-4d9e-aa22-f15179862034-6012434.jpg)
    增加存储字的数量，如1k*8位的存储芯片若干，构成2k*8位的存储器，扩充容量，数据线共用，地址线变为2k，即由2的10次方到了2的11次方，由于每个块容量只有10次方，故同一块上地址第11次方位一定相同。故可用A11表示数据传给哪个块。
    ​

  - 位字扩展![img](https://api2.mubu.com/v3/document_image/e7a103e1-e5ff-4a60-972f-b7bc5e00a3c1-6012434.jpg)
    4K的空间我们分配到了4个存储器当中，每个存储器包含了两片1K*4bit的存储芯片，
    第一个存储器的范围为 00 0...0(10个0)~00 1...1(10个1)
    第二个存储器的范围为 01 0...0(10个0)~01 1...1(10个1)
    第三个存储器的范围为 10 0...0(10个0)~10 1...1(10个1)
    第四个存储器的范围为 11 0...0(10个0)~11 1...1(10个1)
    由A11和A10来判断要访问的地址在哪一个存储器当中，我们采用译码器进行译码，当
    A11 =0    A10=0  选择第一个存储器
    A11 =0    A10=1  选择第二个存储器
    A11 =1    A10=0  选择第三个存储器
    A11 =1    A10=1  选择第四个存储器

    

- cache存储器

  - 功能
    cache是一种高速缓冲存储器，解决CPU和主存之间速度不匹配而采用的技术，基于程序运行中的空间局部性和时间局部性。cache速度比主存快，快速向cpu提供指令和数据，它由高速的SRAM组成。

  - 原理
    若在cpu外，其控制逻辑与主存在一起，称为主存/cache控制器；若在cpu内，由cpu提供。
    cache与主存用块为单位传输，cpu和cache之间以字为单位。一个块由若干字构成。当cpu请求一个数据时发出数据的地址到cache和主存中，若cache中有，则传给cput。否则由主存传给cpu，同时把含有该数据的块传入cache。块的大小为cache的一行，取数据前后相继的数据替换最少使用的一行，LRU策略。

  - 命中率
    使用cache就是为了让主存的平均读出时间和cache的读出时间尽可能接近。即，由cache完成读取的比例越高越好。
    h为命中率，Nc为cache存取次数，Nm为主存存取次数。h=Nc/(Nc+Nm);
    tc为cache访问花费，tm为主存访问花费。cache/主存平均访问时间为：ta=htc+(1-h)tm
    r=tm/tc ,e=tc/ta=1/(r+(1-r)h)；

- 主存与cache的地址映射

  cpu请求数据时会给出内存地址，访问cache时需要得到cache地址，这两个地址要有映射关系

  - 直接映射
    多对一映射，一个主存块只能拷贝到特定的cache行。cache的行号i和主存的块号j有如下关系。i=j mod m;m为cache的总行数。

  - 全相联映射
    将主存的一个块的地址与块的内容一起存于cache的行中，其中块的地址存于cache行的标记部分中。可以将主存的块保存至cache的任意一行，非常灵活。

  - 组相联映射
    将cache分成u组，每组v行。主存的块号为j，组好q=j mod u。主存必须放到q组，但是在组内哪一行时灵活的。他的主存地址信息和数据存在一行，在该行的标记位上。

- 虚拟存储器

  - FIFO
    先进先出

  - FIFO+LRU
    与FIFO相同，但发生命中时，将该块移动到第一行，使其存活更久。

  - LRU
    最近最少使用。cache每命中一次，被命中的计数清零，其他的计数加一。替换时，换出计数最大的。这种方法保护了刚拷贝到cache新数据行。

  - LFU
    每访问一次，被访问计数器加一。替换时，将计数最小的换出。

# 第四章 指令系统

- 9中寻址方式

  - 隐含寻址
    未明显给出操作数地址。例如，单地址的指令格式未直接指出第二操作数的地址，而是规定累加寄存器AC作为第二操作数地址。因此，累计寄存器AC对单地址指令格式来说就是隐含地址

  - 立即寻址
    指令的地址字段直接给出操作数

  - 直接寻址
    直接给出操作数地址。

  - 间接寻址
    指令字段中形式地址A不是操作数地址，而是操作数地址的指示器。寻址特征位未0，则直接寻址。否则间接寻址。
    ​

  - 寄存器寻址
    操作数不在内存中，而在通用寄存器中。指令字段给出的未寄存器编号。

  - 寄存器间接寻址
    寄存器内容中不为操作数，而是操作数内存地址。

  - 偏移寻址

    - 相对寻址
      EA=A+(PC),A为地址字段中A的值，有效地址时对当前地址PC的上下范围的一定偏移。

    - 基址寻址
      将CPU中基址寄存器的内容（偏移量），加上指令格式中的形式地址而形成操作数的有效地址。

    - 变址寻址
      把变址寄存器的内容（通常是首地址）与指令地址码部分给出的地址（通常是位移量）之和作为操作数的地址来获得所需要的操作数就称为变址寻址。

  - 段寻址方式
    1MB分为多块，每块为64KB（即2的16次方）。确定一个内存位置时，通过基地址+偏移量（16位），来确定。这里基地址就是段寄存器。

  - 堆栈寻址
    分为寄存器堆栈和存储器堆栈，先进后出。PUSH时，堆栈指示器减一，反之POP加一。

# 第五章 中央处理器

### cpu功能和组成

##### cpu功能

- 指令控制
  程序的顺序控制。

- 操作控制
  一条指令由若干操作信号实现，因此cpu负责产生和取出操作信号，并送往相应的部件。

- 时间控制
  每个操作都有实施时间的限制。

- 数据加工
  算数运算和逻辑运算处理，时cpu的根本任务。

##### CPU组成

- 控制器：由程序计数器，指令寄存器，指令译码器，时序产生器，操作控制器组成。
  控制器主要功能有：1从cache取出指令并指出下一条指令位置
  2对指令译码或测试，产生操作信号。
  3指挥并控制cpu，数据cache和输入输出设备之间数据流动。​

- 运算器：由算数逻辑单元（ALU），通用寄存器，数据缓冲寄存器DR和状态条件寄存器PSW组成。
  主要功能：1执行所有算数运算
  2，执行逻辑运算和逻辑测试，如0值测试和两个值的比较。

- cpu中的主要寄存器

  - 数据缓冲寄存器（DR)
    1作为ALU运算结果和通用寄存器之间信息传送的缓冲。
    ​2补偿cpu和内存，外围设备之间再操作速度上的差别。

  - 指令寄存器（IR）
    保存当前正在执行的指令，指令从cache取出后存入IR。指令分为操作码和地址码，IR的操作码输出就是指令译码器的输入，译码后就可发出操作信号。

  - 程序计数器（PC）
    保证程序顺序执行下去，确定下一条指令的地址，又称指令计数器。

  - 数据地址寄存器（AR）
    cpu当前访问数据在cache章地址，保存直到一次读写操作完成。

  - 通用寄存器（R0-R3）
    当ALU执行算术或逻辑运算时，为ALU提供工作区。如加法结果存在两个寄存器中，结果再送回其中一个寄存器中。

  - 状态字寄存器（PSW）
    保存由算术指令和逻辑指令运算或测试结果建立的各种条件代码。如进位标志C，一处表示V，结果为0标志Z，为负标志N，通常由一位触发器保存。

### 五种指令周期理解与运用

1、寄存器-寄存器（RR）型指令：需要多个通用寄存器或个别专用寄存器，从寄存器中取操作 数，把操作结果放到另一寄存器中。机器执行这种指令的速度很快，不需要访e799bee5baa6e59b9ee7ad9431333431353430问内存。 
2、寄存器-存储器（RS）型指令：执行此类指令，既要访问内存单元，又要访问寄存器。 
3、存储器-存储器（SS）型指令：参与操作的数都放在内存里，从内存某单元中取操作数，操 作结果存放至内存另一单元中。因此机器执行这种指令需要多次访问内存。

- MOV指令周期
  两个cpu周期
  取指：1.pc装入第一条指令地址  2.译码 3.将指令装入IR 4​.pc+1 5.操作码译码 6识别出MOV
  执行：1，确定寄存器 2，指定ALU操作 3，打开ALU输出数据 4，送出操作信号打入DR。5，将DR数据打入目标寄存器。

- LAD指令周期
  三个CPU周期

- ADD指令周期

- STO指令周期

- JMP指令周期

- 微程序理解与设计

  - 微程序控制原理

    - 什么是微程序：使用一种语言编写微程序，实现机器指令的功能。将完成的微程序写入控制寄存器，cpu来控制寄存器调用。
      具体就是将一条机器指令编写成一段微程序，一个机器指令对应一个微程序。每一个微程序包含若干条微指令，每一条微指令对应一条或多条微操作。在有微程序的系统中，CPU内部有一个控制存储器,用于存放各种机器指令对应的微程序段。当CPU执行机器指令时，会在控制存储器里寻找与该机器指令对应的微程序，取出相应的微指令来控制执行各个微操作，从而完成该程序语句的功能。微程序设计技术，指的是利用软件技术来实现硬件设计的一门技术。

    - 微命令和微操作

      - 控制部件通过控制总线发送各种控制命令称为微命令，而执行部件接收后进行的操作称为微操作。微操作分为相容性和相斥性。

      - 通过开关门来控制数据流动。

    - 微指令和微程序
      在机器的一个cpu周期中，一组实现一定操作功能的微命令的组合，构成一条微指令。微指令字长23位，他又操作控制和顺序控制组成。

  - 微程序设计

### 冯诺依曼机特点