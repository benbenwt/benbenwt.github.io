# 基本

>linux操作系统由4部分组成：内核：kernel，shell，文件系统和应用程序。
>
>linux内核由五个子系统组成：进程调度，内存管理，虚拟文件系统，网络接口，进程间通信。io系统，磁盘等。

# 内存寻址

### 内存地址

##### 逻辑地址：在分段结构中使用，使用一个segment段和一个offset偏移量表示，offset表示断开始到实际地址的距离。

##### 线性地址：32位无符号整数，表示一段地址可高达4GB，将逻辑地址加上基地址就是线性地址。

##### 物理地址：芯片级内存单元寻址，与电信号对应。

cpu控制单元通过分段单元将逻辑地址转为线性地址，通过分页单元将线性地址转为物理地址。

### 硬件分段单元

>一个逻辑地址由一段标识符和一个指定段内相对地址的偏移量。

为了快速检索段选择符：段标识符，处理器用段寄存器存放段选择符。共有6个：cs,ss,ds,es,fs,gs.

其中三个有特殊用途：
cs 代码寄存器

ss 栈段寄存器，指向存放当前程序栈的段。

ds 数据段寄存器，存放静态数据和外部数据的段。

### 段描述符

每个段描述符表示，村塾段的特征，存在全局描述符表中或局部描述符表。

组成：1type域：米哦啊书段的类型2s，表示是否为系统段。3第一个字节的线性地址，用于地址转化。

### 段选择符

当选择符被装入段寄存器，相应的段描述符装入cpu寄存器。这个段的逻辑地址转换不必查找gdt和ldt，直接在寄存器中。

组成：1索引，在gdt或ldt中入口2在ldt还是gdt3段选择符放入cs时优先级

### 段选择符和段描述符

段描述符8字节，段选择符16位长。

段选择符高13位乘以8得到在ldt或gdt中相对地址，因为选择符前13位为索引。

例：gdt起始地址a，段选择符索引号2，则在gdt实际地址=a+相对地址:2*8

### 段单元

逻辑地址转为线性地址

1找到描述符表

2取出索引号，求出描述符在gdt地址。

3根据段描述符域基地址和逻辑地址偏移量求出线性地址。

### 硬件中的分页单元

>将线性地址转为物理地址分为两步，第一步使用的转换表称为页目录，第二种使用的称为页表。
>
>正在使用的页目录表的物理地址放在cr3中，线性地址的目录域决定指向页目录表哪一项，即指向哪个页表。随后，页表域决定页表的一项，此项包含页框所在物理地址。偏移量决定本页框内相对位置。

目录和页表域都10位长，组成:

### 拓展分页

将页框大小设为4kb或4mb，4mb页目录表项的地址低22位无效。4kb低12位无效。使用page size标识大小。

### 硬件保护方案

段有读写执行三种权限，页只有读写。

线性地址有64位。

分页举例执行流程:

1目录域用于选择页目录的对应目录项

2页表域选择页表对应的页框

3偏移量用于在页框中查找位置。

### 三级分页

最高21位不用，13位偏移量：叶匡大小8kb，三个10位的页表域。

### 硬件高速缓存

解决处理器和动态ram或dram之间近10倍的速度差距，用来存放程序和数据结构。

行：由多个连续字节构成，在dram和ram或sram之间进行数据传送。

dram动态随机存取器：电容表示数据，电荷流失需要周期充电

sram静态随机存取器：只要保持通电，数据不消失。

高速缓冲控制器有一个入口数组，每个入口对应高速缓存存储器一个行。

入口：一个tag，描述行的几个flag。

tag：一些位组成，映射对应行。

这块控制器地址构成：tag，控制器子集索引，行内偏移量。

### TLB转换后高速缓存

线性地址转换后，物理地址存储在tlb中。

invlpg使tlb一个单项失效。

### linux三级分页

页全局目录，页中间目录，页表

自动的线性地址转换实现了：1每个进程拥有不同物理地址空间，对应它自己的全局目录和自己的页表集合。进程切换时，将cr3寄存器内容存在tss段中，再把另一个tss段的值装入cr3中。

### 线性地址域

### 页表处理函数

##### PAGE_SHIFT

偏移量位数

log 页的大小，x86页大小为4096.

##### PMD_SHIFT

二级页表映射的地址位数

PMD_SIZE页中间目录单独表项映射的区域大小，即一个页表区域。2的22次方，4MB.

##### PGDIR_SHIFT

一级页表区域大小位数

PGDIR_SIZE,全局目录一个表项映射区域大小，即一个页目录大小。4mb.

##### PTRS_PER_PTE,PTRS_PER_PMD,PTRS_PER_PGD

页表，页中间目录，页全局目录表项个数。



### 页表的处理

pte_t，pmd_t和pgd_t为32位分别描述页表，页中间目录和页全局目录的一个表项。

pgprot_t为32位，描述单独表项保护标志。



# 进程

### 进程描述符

>task_struct结构,包含进程相关信息。

##### 进程的状态

可运行状态：TASK_RUNNING,cpu上运行或准备执行

可中断的等待状态：TASK_INTERRUPTIBLE，被挂起，直到一些条件为真。为真后，回到TASK_RUNNING状态。

不可中断状态:TASK_UNINTERRUPTIBLE,与上一个类似，把信号传递到睡眠的过程中，不能改变状态。

暂停状态:TASK_STOPPED,执行被暂停，收到SIGSTOP,SIGTSTP,SIGTTIN,SIGTTOU后进入暂停。当被其他进程监控，如debug时，任何信号都可以让他暂停。

僵死状态:TASK_ZOMBIE，执行被终止，父进程发布wait之前不可丢弃死进程的信息。

##### 标识一个进程

内核对进程描述符的引用大多通过进程描述符进行。

进程描述符中包含pid，pid中导出描述符指针，以便kill进程。

##### 任务数组

大小为NR_TASKS的全局静态数组，其中元素为进程描述符指针。

##### 进程描述符存放

存放在动态内存中。

进程描述符在下即内存开始处，内核态的进程栈在上即内存末端。

esp寄存器时cpu栈指针，存放栈顶位置。

c语言使用联合结构表达stack加描述符。

union task_union{

struc.task_struct task;包含指向进程描述符的指针

unsigned long stack[]2018;

}

##### current 宏

8kb的联合区域，使用13位来确认基地址。

current产生汇编指令:

movl $oxffffe000 , %ecx

andl %esp,%ecx

movl %ecx,p

执行以上命令后，p中位进程描述符指针。



free_task_struct（）

释放8kb的task_union内存区，若高速缓存未满，放入高速缓存中。

alloc_task_struct()

分配8kb的task_union内存区。若至少有一半填满，这个函数从高缓中获得内存。

##### 注

movl  val,   %ebx     ，引用从开始位置的4字节数据，不可传送小于4字节。

movw  %ax,    %bx      ，16位，2字节

 movb   %al,    %lx       ，8位一字节

##### 进程链表

进程描述符包含一个链表指针，指向下一个进程的域。

双向链表把所有进程联系起来。

SET_LINKS,插入进程描述符

REMOVE_LINKS,删除一个进程描述符

#define for_each_task(p)

{

for(p=&ini_task;(p=p->next_task)!=&init_stack;)

}//遍历进程链表

##### TASK_RUNNING状态的进程链表

内核查找一个进程在cpu上运行，只需查找可运行进程。故构造此链表，方便查找，也叫运行队列:renqueue。next_run和prev_run实现可运行队列。

add_to_queue把一个进程进程描述符插入到链表的开始。

del_from_runqueue把第一个进程描述符删除.

move_first_runqueue把描述符移动到队首

move_last_queue把描述符移动到队尾。

wake_up_process使进程可执行，把进程设为TASK_RUNNING。

##### pidhash表及连接表

 内核需要通过进程的pid导出进程描述符指针，比如kill服务，传入pid杀死进程。

顺序扫描进程链表太慢，所以我们构建pidhash，在常数时间内查找指针。

pidhash散列表有PIDHASH_SZ个元素。

pid_hashfn宏把pid转成表的索引：多行宏用\结尾。

#define pid_hashfn(x)    \

(   ( (   (x)>>8)^(x)   )   &      ( PIDHAHS_SZ-1)     )

Linux使用连接表chained list处理hash冲突。连接表是一个双向链表，由冲突的表象的进程描述符构成。使用pidhash_next和pidhash_pprev域实现。

这种方法比线性映射更好，因为更节省空间。

hash_pid：插入一个进程。

unhash_pid:删除一个进程。

find_task_by_pid：查找pid对应的进程描述符指针。

##### task空闲表项的链表

为了快速增加和删除，每当进程被撤销，将空闲向加入链表。他是一个非循环双向链表。

##### 进程的亲属关系

进程具有父子关系，同一进程的子进程又有兄弟关系。

进程0，1是由内核创建的，1是所有进程的祖先。

进程描述符中包含描述亲属关系的域，其构成如下：

p_opptr 祖先

指向创建此进程的进程描述符，若父进程不存在，指向1进程。

例如shell启动后台进程后推出，后台进程该域指向init。

p_pptr  父进程

当前父进程，一般与p_opptr相同，但有些情况不同。

例如：当另一个进程发布ptrace系统调用请求监控p时。

p_cptr  子进程

指向年龄最小的子进程的描述符。

p_ysptr  弟进程

指向它父进程的下一个创建进程。

p_osptr 兄进程

指向它父进程的上一个创建进程。

##### 等待队列

TASK_RUNNING组成运行队列

TASK_STOPPED和TASK_ZOMBIE不用创建独立的链表。因为，可以通过父进程查找pid，或进程间的亲属关系检索子进程。

TASK_INTERRUPTIBLE和TASK_UNINTERRUPTIBLE创建独立的链表。而且对于不同的等待事件，创建独立的链表，称为等待队列。

等待队列对中断处理，进程同步及定时用处大。等待队列由循环链表实现，元素指向进程描述符。

c语言定义如下：

struct wait_queue{

struct task_struct * task;

struct wait_queue *next;

}

每一个等待队列由一个等待队列指针标识，指向第一个元素地址，或为空。



intel指针的大小为4字节。

init_waitqueue()函数初始化一个空的等待队列，它接受等待队列指针的地址q作为参数，然后把指针设为q-4.

add_wait_queue(q,entry)函数把地址为entry的元素插入等待队列。



等待队列关中断执行插入操作：

if(*q!=NULL)

 		entry->next=*q;  指向队首元素

else 

​		entry->next=(struct wait_queue *)(q-1);  指向空指针

*q=entry;    插入到队首



remove_wait_queue移除由entry指向的元素，执行此操作必须关中断。

next=entry->next;

head=next;

while((tmp=head->next)!=entry)

​	head=tmp;

head->next=next;

此函数查找到循环链表中位于entry前一个的元素，让其next指向entry->next，删除entry。



希望等待事件可以调用一下函数：

sleep_on，设置为TASK_INTERRUPTIBLE,插入等待队列。调用调度恢复程序，当当前进程被唤醒，从等待队列中删除。

interruptible_sleep_on，设为TASK_UNINTERRUPTIBLE

sleep_on_timeout和interruptible_sleep_on_timeout,定时唤醒，使用schedule_timeout()实现。使用wake_up或wake_up_interruptible宏。

##### 进程的使用限制

进程与一组使用限制相关联，决定进程能使用的系统资源数量。

例举部分如下：

RLIMIT_CPU,cpu使用最长时间

RLIMIT_FSIZE，文件最大值

RLIMIE_DATA,堆最大值

RLIMIT_STACK，栈大小的最大值

RLIMIT_CCRE，内存信息转储文件的大小

RLIMIT_RSS，页框最大数

RLIMIT_NPROC，拥有进程最大数

RLIMIT_NOFILE，打开文件最大数

RLIMIT_MEMLOCK，非交换内存最大尺寸

RLIMIT_AS，进程地址空间最大尺寸

使用限制存放在进程描述符的rlim域，通常大多数限制值为RLIMIT_INFINITY,无限制。



##### 进程切换

主要内容：硬件上下文，硬件支持，linux代码，保存浮点寄存器

每个进程都可以拥有自己的地址空间，但进程必须共享cpu寄存器。因此，恢复执行前，内核必须确认每个寄存器装入了挂起进程时的值。




