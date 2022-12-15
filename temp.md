# 0922 理论知识
## hive overwrite 动静态分区
```
#静态分区
insert overwrite table dwt_test partition(dt='2022-09-22', part='2')
select id,dt,part from dws_test

#动态分区
insert overwrite table dwt_test partition(dt=dt, part=part)
select id,dt,part from dws_test

```
```
覆盖健全可度量的理论，测试，总结情况。
```
```
#动静结合
insert overwrite table dwt_test partition(dt='2022-09-22', part=part)
select id,dt,part from dws_test
```
## java 顺序表
```
#优先队列
PriorityQueue<Integer> pqueue=new PriorityQueue<>((o1, o2)->( o1.compareTo(o2)));
#栈
Stack<Integer> stack=new Stack<>();
stack.push();stack.peek();stack.pop()
#队列
Queue<Integer> queue=new Queue<>();
queue.offer();queue.peek();queue.poll();
queue.add();queue.elelment();queue.remove();
```
## 排序算法 时间复杂度 稳定性
![sort](../resources/images/sort.png)
>10种常见排序

>排序算法        平均时间复杂度   最好情况   最坏情况  空间复杂度   排序稳定性   排序方式

>冒泡            O(n^2)          O(n)       O(n^2)   O(1)       稳定          In-place

>快速排序        O(nlogn)        O(nlogn)    O(n^2)  O(logn)    不稳定         In-place
>
>归并            O(nlogn)        O(nlogn)   O(nlogn) O(n)       稳定          Out-place
>
>堆排序          O(nlongn)       O(nlongn)  O(nlongn) O(1)      不稳定        In-place
>
>选择排序        O（n^2)         O(n^2)     O(n^2)    O(1)      不稳定         In-place
>
>插入排序        O（n^2）        O(n)       O(n^2)    O(1)       稳定          In-place
>
>希尔排序        O(nlogn)    O(n(logn)^2) O(n(logn)^2) O(1)     不稳定         In-place
>
>计数排序        O(n+k)        O(n+k)        O(n+k)     O(k)    稳定          Out-place  
>  
>桶排序          O(n+k)        O(n+k)        O(n^2)     O(n+k)  稳定          Out-place 
> 
>基数排序        O（n*k)       O（n*k)        O（n*k)    O(n+k)   稳定          Out-place
>
>选择排序的不稳定性不来自挑选最大最小值的过程，而是由于要原地排序，所以要交换开头的值和查找到的最大最小值，导致改变了数组顺序。
>
>希尔在分组增量变小时，进行插入排序，但是无法区分两组原始数据相同值元素的顺序。比如1357时一组，他们组间是稳定的，但是当与2468合并时，可能1号值与6号值一样，并且1号值原本在6号值后边，插入排序后就乱序了。

## 浮点数存储
>使用正负号、尾数、指数来表示浮点数，即尾数乘以2的指数次方，再取符号位，就得到了具体值。
>对于4字节浮点数，其符号位占据一位，指数占据8位，其余都是尾数位。
>对于8字节浮点数，其符号位占据一位，指数占据11位，其余都是尾数位。
>关于精度损失的问题：
>由于浮点数无法精确表示十进制的数字，例如0.6，其表示为0.10011001......。因为2进制小数点后为：0.5、0.25、0.125、0.0625、0.03125。也就是说当创建一个浮点数为0.6，那么它存储的就是非精确的值了。

## spark cache、persist、checkpoint
>cache：直接将rdd算子的缓存到计算结点的堆内存种，其也是懒执行的，只有action算子触发后，才会将rdd缓存到内存中。
>persist：可以指定缓存的位置，当指定MEMORY_ONLY时，就是cache一样的功能。还可以指定MEMORY_AND_DISK,DIS_ONLY,以及是否序列化和保存的副本数量。
>checkpoint:将快照保存到外部存储，并切断血缘关系，建议使用checkpoint时也要使用cache，这样可以直接读取cache的数据进行checkpoint，否则checkpoint会从头计算一遍。checkpoint相比于cache：1checkpoint会切断血缘关系2checkcpoint一般使用hdfs等容错性高的存储3

## hdfs组件、yarn组件
>NameNode：管理hdfs目录和文件元信息，元信息包括目录名称、权限、副本等，这些原信息保存在内存中用于快速访问。为了容错，在内存中也有对应快照，使用fsimage和edits文件，两者都在namenode.dir的current目录下。通过定时的合并这两个文件获取最新的fsimage。
>SecondarynameNode：负责帮助NameNode和合并fsimage和edits文件。
>Datanode：存储具体的hdfs block，要保证数据块的完整性和正确性，会通过校验和、摘要算法等进行检测，并定时向NameNode汇报block信息。

>ResourceManager：负责响应应用程序的资源请求，接受NodeManager汇报的结点信息，跟踪集群中活动结点和资源的数量。其主要分为调度器和应用程序管理器，调度器可以使用不同的调度策略分配资源和启动任务。
>NodeManager：负责单台机器的资源管理、任务监视等。包括磁盘、内存、cpu等信息。
>ApplicationMaster：用于请求资源创建task，监视任务执行情况，重启启动失败的任务。
>Container：包括内存、cpu等。应用程序必须运行在container种。
>JobHistoryServer：可以读取存储在hdfs上的日志数据，并提供给页面访问。

## ResourceManager调度器
### 先进先出调度器 FIFO
>单队列，根据作业提交的先后顺序，先来先服务。
>优点：逻辑简单
>缺点：不支持多队列，生产环境使用少
### 容量调度器 Capacity Scheduler
>为不同容量预留资源，支持多队列，为每个队列分配资源，队列内部使用FIFO。
>优点：保证了不同容量的资源都可以执行
>缺点：但是浪费了部分资源，当其中一种资源没有人使用时，仍要占用

### 公平调度器 Fair Scheduler
>通过一个权重分配公平的分配资源，每当提交一个新资源时，之前的程序释放部分资源给新程序使用。当新来的执行完后，会返还资源给之前的应用，知道之前的应用执行完成。可以设置多个队列，并按照百分比分配资源给队列，队列再分配给具体的应用。队列内部，根据公平调度的方法分配资源。
>优点：同队列共享资源，在时间尺度上获得公平的资源
>缺点：多个应用共同执行，降低了执行的效率。

```
#指定调度器
vim yarn-defualt.xml
<property>
    <description>The class to use as the resource scheduler.</description>
    <name>yarn.resourcemanager.scheduler.class</name>
<value>org.apache.hadoop.yarn.server.resourcemanager.scheduler.capacity.CapacityScheduler</value>
</property>
```

```
#配置多队列的容量调度器
#capacity-scheduler.xml中都是关于容量调度器的配置信息，相应地公平调度器也有自己的配置文件，fair-scheduler.xml.
vim capacity-scheduler.xml
<property>
    <name>yarn.scheduler.capacity.root.queues</name>
    <value>default,hive</value>
    <description>
      The queues at the this level (root is the root queue).
    </description>
</property>


<!-- 指定hive队列的资源额定容量 -->
<property>
    <name>yarn.scheduler.capacity.root.hive.capacity</name>
    <value>60</value>
</property>

<!-- 用户最多可以使用队列多少资源，1表示 -->
<property>
    <name>yarn.scheduler.capacity.root.hive.user-limit-factor</name>
    <value>1</value>
</property>

<!-- 指定hive队列的资源最大容量 -->
<property>
    <name>yarn.scheduler.capacity.root.hive.maximum-capacity</name>
    <value>80</value>
</property>

<!-- 启动hive队列 -->
<property>
    <name>yarn.scheduler.capacity.root.hive.state</name>
    <value>RUNNING</value>
</property>

<!-- 哪些用户有权向队列提交作业 -->
<property>
    <name>yarn.scheduler.capacity.root.hive.acl_submit_applications</name>
    <value>*</value>
</property>

<!-- 哪些用户有权操作队列，管理员权限（查看/杀死） -->
<property>
    <name>yarn.scheduler.capacity.root.hive.acl_administer_queue</name>
    <value>*</value>
</property>

<!-- 哪些用户有权配置提交任务优先级 -->
<property>
    <name>yarn.scheduler.capacity.root.hive.acl_application_max_priority</name>
    <value>*</value>
</property>

```

## assic，utf，unicode编码
>ASCII使用7位或8位数表示128种或256种可能的字符，主要表示所有大小写字母，数字0到9、标点符号、美式英语中的特殊控制符号等。不支持中文
>GBK编码：中国人定义了当字符小于127时，与ASSIC相同，但是大于127的字符连接在一起时，就表示一个汉字。第一个字节称为高字节，第二个字节称为低字节。
>Unicode字符集：由于世界国家很多，为了统一编码，ISO组织使用统一的编码方案，它是一套字符集。其规定两个字节表示一个字符，从而表示所有的字符。，
>UTF-8：由于unicode存在浪费空间的问题，定义了一种可以解码和编码的规则，降低了占用的空间和带宽。
## jvm内存参数
>1. -Xms 1024m　　//设置堆的最小值
>2. -Xmx 2048m   //设置堆的最大值
>3. -Xmn 512m    //设置新生代大小
>4. -XX:MetaspaceSize=256m //设置初始Metaspace空间的大小
>5. 永久带的初始值-XX:PermSize及最大值-XX:MaxPermSize
>6. -Xss   每个线程的Stack大小，不熟悉最好保留默认值；
>7. 堆中新生代与老年代的比率，该值可以通过参数 –XX:NewRatio 来指定
>8.  新生代中eden与survivor比率，–XX:SurvivorRatio 

## kafka参数
>buffer.size  生产者缓存队列大小
>linger.ms 生产者触发发送的时间间隔
>batch.size 生产者发送的批次大小，凑满一个batch就发送数据。
>maxRequestSize 请求最大大小

## LCS
>找到一个子序列，index单调递增。并不是前缀，不需要index连续。

## tcp三次握手
>为什么不是两次：如果客户端发起的syn请求，在网络延迟了很久才到达服务端，客户端这时很久没有收到服务端的响应报文，就会认为这个链接请求已过时。而服务端并不知道这是一个延迟的消息，它会直接发送ack报文，并认为建立链接，这就是两次握手的问题，三次握手就不会有这个问题。
>tcp的三次握手时，客户端状态从CLOSED->SYN-SENT->ESTAB-LISHED,服务端从LISTEN->SYN-RCVD->ESTAB-LISHED
>tcp的四次挥手时，客户端状态从ESTAB-LISHED->FIN-WAIT-1->FIN-WAIT-2->TIME-WAIT->CLOSED
>服务端状态从ESTAB-LISHED->CLOSED-WAIT->LAST-ACK->CLOSED
>
>为什么TIME_WAIT状态经过2MSL返回CLOSE：为了防止客户端的ACK报文丢失，导致服务端无法从LAST_ACK切换到CLOSED状态，这时服务端会不断发送FIN报文，客户端收到FIN报文后，就意识到ACK报文丢失了，所以需要重发ACK。所以说需要等待2MSL时间，如果这段时间没有收到重发的FIN说明ACK已经抵达了服务端，可以关闭链接了。