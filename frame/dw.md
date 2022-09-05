[TOC] 

## 理论知识0822
### Java八股
#### equals和==区别，为什么重写equals要重写hashcode
>==是比较两个变量的值是否相等，如果相等返回true否则false。equals则是先比较两个变量的地址，如果相等返回True，否则进一步比较值，如果相等返回true，否则false。equals表示两个变量值是否是相等，hashcode也表示两个变量是否是同一个东西，这两者必须保持一致，也就是equals返回true时，hashcode必须相等，但hashcode一样，不一定equals为true。

>答案：基本类型的变量==直接比较两个值是否相等，引用类型的==是比较的两个变量指向的地址是否相同。这本质上是因为两者在栈中存储的值的意义不同，实际上都是比较其在栈中的值。基本类型的变量在栈中存储的是其真实值，例如1，2.0f等，而引用变量存储的是地址，所以自然比较的也是地址。object对象的equals默认还是比较地址的，继承了object后，需要根据子类的含义重写equals方法，例如String字符串类的equals就是比较两个String的内容是否相等。
>https://blog.csdn.net/weixin_41943637/article/details/105437949
### java集合有哪些
>collection下有List，Set，Map。再细分有ArrayList，LinkedList，HashMap,TreeMap,HashTable,HashSet,TreeSet

### ArrayList和LinkedList区别
>ArrayList底层是基于数组实现的，所有它的随机读写很快，能通过索引快速找到值。但是在删除某一个元素或添加某一个元素到数组中间时，其需要挪动插入位置之后的所有元素，所以复杂度较高。LinkedList底层基于链表，其添加元素和删除元素比较方便，只需要将相邻的链表节点进行修改。但是其在查找元素时，只能顺序遍历链表的节点，所以效率较差。

>答案：补充：ArrayList空间浪费体现在在结尾预留空间，而LinkedList空间花费体现在每一个元素需要花费空间维护信息。
### HashMap默认大小，扩容机制
> HashMap结构是什么样的？什么时候扩容？扩到多少？

>答案：默认capacity为16；loadFactor加载因子，默认是0.75。threshold阈值。阈值=容量*加载因子。默认12，当元素数量超过阈值，触发扩容。扩容到原来的两倍。
### HashMap在哪个版本使用红黑树，之前是使用什么?
> jdk8，之前使用哈希表和链表。

### 线程的创建方式
> 1使用一个类实现Runnable接口，然后调用run方法
> 2直接使用Thread

>答案：1继承Thread类 2实现Runnable接口 3使用线程池例如Executor框架
### 多线程了解
> 线程的状态：NEW,RUNNABLE,BLOCKED,WAITING,TIMED_WAITING,TERMINATED
> 线程状态的转换：当线程试图获取synchronized时，而锁被其他人占用，进入阻塞状态。当线程等待其他线程调度器出现一个条件时，就会计入等待状态。例如wait（），join（）等。以及concurrent中的Lock和Condition锁。当调用带有超时参数的的wait和lock、condition时，就会进入TIME_WAITING状态。
> 多线程的常用编程：线程池的类型、关键参数及使用方法

### MySQL的索引，b树与b+树区别
> MySQL使用b+树构建索引，b+树的每个节点由指向下一层的指针以及值构成，两者互相间隔，指针数比值数目多1。通过与值比较确认所在的子树，从而进入下一层，重复此过程到达叶子节点。b树的查找结果可以分布在非叶子节点，而b+树全部在叶子节点，这样每个查询的时间都相近，保持了查询性能的稳定性。

>答案：b+树相比于b树还使用了链表结构将叶子节点相连，加速遍历的访问速度。b树在有大量热点数据重复访问时，效率更高，因为可以将热点数据放置到离根较近的位置。
### Redis了解

>Redis是一个基于内存实现的键值型数据库，有高并发，高速度的特点，因为其数据都是维持在内存中的，可以保持较高的访问速度。
>Rdis提供了一些常用的数据结构，如String，List，Set，hash，ZSset（有序set）。Redis可以用于一些临时数据的存储，因为其无法保持较高的可靠性，可能导致数据的丢失。可以用来存储当日平台的用户日活信息等，帮助编写实时处理的逻辑。

### 介绍一下Spark
>Spark是一个并行计算框架，其是在Hadoop之后提出的，相比于Hadoop其执行速度获得了很大的提升。因为其使用内存作为存储媒介，减少了磁盘的交互次数，提升了效率，同时也对设备的内存提出了较高的要求。其提出了弹性分布式数据集RDD，RDD是不可变得分布式对象集合，spark的操作都是基于rdd和算子进行的，算子可以对rdd执行各种操作，例如map，reduce，take，show等，当有action算子调用时，就会进行stage、task的划分，并分发到executor上进行计算。
>Spark的架构：Spark的具体任务由executor执行，executor接受来自driver的任务，并在所在机器执行，并返回结果给driver。
>Spark on yarn：Spark on yarn模式通过将yarn作为资源管理器，提供了集群资源管理功能。需要向yarn服务端请求，并在NM中创建AM，AM用于运行driver程序。AM再向RM申请executor的创建需要的资源，然后才能获得创建的权限。

>Spark的核心组件：Driver，Eexecutor  Yan核心组件：ApplicationMaster，ResourceManager，NodeManager
>SparkContext：main程序创建的sc环境，即driver程序，负责DAG解析，task解析。
>ClusterManager：集群管理器，可以是yarn，mesos，或spark自带的资源管理器（通过master节点，worker节点管理）
>Executor节点：运行在worker机器上的jvm进程，负责处理具体的task，并与driver通信。
>
>spark client和spark cluster区别在于，client模式driver运行在提交的机器，cluster的driver运行在集群中机器上。
### RDD为什么弹性
>弹性就是指他能适应集群这种环境1可以在多个节点上分布，适应集群的计算环境，不同集群节点都可以运行同样的rdd程序。2容错性：当发生故障时，rdd可以进行恢复，通过读取持久化的rdd或从头重新计算rdd来恢复计算任务，不会让任务直接崩溃。

>补充：1内存的弹性：内存与磁盘的自动切换 2容错的弹性：数据丢失可以自动恢复 3Task如果失败会自动进行特定次数的重试 4数据分片的高度弹性
### Spark Stage 划分
>Spark程序在rdd上执行算法来完成任务，当执行算子时我们得到一个新的rdd，这两个rdd构成了前后依赖的关系，通过依赖的宽窄来划分，宽依赖是指同一个父RDD的Partition被多个子RDD的Partition依赖，引起shuffle，宽依赖的两个rdd需要很长时间的等待，宽依赖会被划分为两个stage。窄依赖是指一个父RDD最多被子RDD的一个Partition使用。

>Application:初始化一个sc即为一个Application
>Job：一个action算子生成一个job
>stage：宽依赖划分
>Task：一个stage阶段中，最后一个RDD的分区个数就是Task个数
### spark的cache，persist，checkpoint
>cache 是指缓存到内存中，当发生故障时，被缓存过的rdd不需要从头计算，减少了计算的量，不shuffle缓存在各自节点的内存中。
>persist的语义更为丰富，cache就是persist的默认调用。persist的参数可以指定是否存储内存，磁盘，是否使用堆外内存，是否序列化，是否备份。我们可以自定义存储策略等级，也可以使用设定好的部分策略，例如：MEMORY_ONLY,MEMORY_AND_DISK。cache和persist不是action算子，会懒执行。
>checkpoint是指进行快照，保存当前的rdd状态到磁盘中，其会永久保存，但是会切断血缘关系，当容错恢复时无法使用checkpoint作为代替。相比之下，cache和persist是为容错和rdd复用准备的，保留了血缘记录，但是在application结束后都会销毁。

### Spark数据倾斜处理方法
>找到数据倾斜的任务，以及是哪一个key发生了数据倾斜的现象。对这个key增加前缀，1将数据分布到多个分区上去，减少分区的压力。2可以将该key单独拿出来作为一个任务，其他数据量较少的作为一个key，这样数据量少的执行速度不会受到影响。

>当大多数task执行迅速，但是少数task执行时间很长。或者是大部分task执行迅速，少部分task总是OOM，反复执行都是某个task出现OOM。
>1针对key值分配不均，可以将key值数据量较大的数据sample采样，过滤出来单独计算。
>2针对groupbyKey，reduceByKey，可以添加key值前缀，增加task数量，聚合后再去掉前缀并聚合，通过两次聚合解决倾斜。
>3reduce join 转map join，reduce join通过将相同的key值和value拉取到shuffle read task再join，当其中一个rdd较小时，可以系通过广播小rdd的变量+mapjoin解决此问题。
### Linux查看内存，cpu状态，查看进程的内存消耗和cpu消耗
>  free 查看内存
>  top 查看当前cpu，进程的内存消耗的cpu消耗
>  ps -aux

### clickhouse
>clickhouse 是一个olap数据库，可以实时分析所需要的数据。
>olap场景的特点就是绝大多数是读请求，已添加的数据不能修改，可能取很多行但是很少列，需要行的高吞吐量，事务不必须，数据一致性要求低。从数据变化（不修改），查询什么（读请求，行多列少），特性对比（事务不必须，一致性要求低），压缩算法（数据压缩允许在内存中缓存更多数据）

### 参考
>https://www.nowcoder.com/discuss/842391

## 理论知识0823
### jvm内存结构，堆栈区别
>jvm内存结构分为5块，1虚拟机栈 2本地方法栈 3堆内存 4程序计数器 5方法区  除此之外，还可以通过Native函数库分配堆外内存，其不受java堆大小的控制，
>虚拟机栈和程序计数器都是线程私有的，它们的生命周期和线程相同。虚拟机栈就是java方法的内存模型，每个方法被执行时，java虚拟机都会创建一个栈帧存储局部变量表，操作数栈，动态链接，方法出口。每个方法从调用到执行的过程，对应栈帧在虚拟机栈中从入栈到出栈的过程。 堆和方法区是一个线程共享的区域，堆存储对象实例，堆是垃圾收集器管理的内存区域，故也称为GC堆。用于存储已经被虚拟机加载的信息，例如类型信息，常量，静态变量，即时编译的代码缓存等。
### jvm gc算法
>1标记-复制算法
>标记存活的对象，将其复制到新的内存区域，然后清空原来的内存区域。在分代收集算法中，为了方便复制存活对象，将内存分为Eden，From Survivor，To Survivor，每次将存活对象复制到To Survivor后，清空Eden和From Survivor。
> 2标记-清除
> 标记需要回收的对象，直接清除这些对象，容易导致不连续的碎片空间，浪费空间。当需要分配连续的大内存时，无法提供，触发gc。 
> 3标记-整理 
>为了解决不连续的空间碎片问题，标记需要回收的对象，并移动存活的对象，使存活的对象互相相邻。然后清除边界以外的空间。
>对于分代收集算法：新生代使用标记-复制算法  老年代使用标记清除或标记整理
### objects属性和方法
>getClass：获取class对象
>clone：复制当前对象，在内存地址中创建一个新的独立对象，其属性值和当前对象一样。
>equals：比较两个对象是否相等
>hashcode： 计算对象的hash值
>toString：返回对象的字符串表示方式
>finilize：垃圾回收两次确认执行的方法
### equals()和hashcode()
>==：比较基本类型的数值 比较引用类型的地址
>equals objects是比较地址，string重写为比较字符串内容
>hashcode计算对象的hash值，需要与equasl一致
### 场景题：内存只有256M，有两个10G的文件，从这两个文件中找出相同的数字？（用MapReduce实现，只说思路）
>bitmap用于表示一个集合，用2进制第一位表示数字1是否存在，0表示不存在，1表示存在。本题中，使用bitMap表示文件A，再用文件B去比较，如果bitMap中对应位为1，则说明数字在两个文件中都存在。bitMap的占用空间是n位bit，n是有多少个数字。bitMap也具有随机访问的能力，所以能快速比较出是否存在相同数字。
>将10G文件根据哈希值划分为更小的份，然后文件A的每一份与文件B的对应份遍历比较。
### 场景题：要求说出使用的计算框架和存储框架
>（1）实时计算每天各个种类的热门Top10商品
>（2）实时计算每天一小时内各个种类的热门Top10商品
>Top10热门Flink，存储在kafka：
>对于商品信息，一般是从业务数据库中和后端日志采集而来，为了方便进行实时计算，业务数据库一般使用canal、maxwell采集到kafka消息队列，而后端日志可以借助flume采集到kafka。
```
#获取环境，一般有如下几种方法：1直接new出来，例如new SparkContext() 2通过builder方法 SparkSession.conf().build() 3通过提供的get方法 StreamExecution.getExecutionEnvironment();
StreamExecutionEnvironment env=StreamExecution.getExecutionEnvironment();
Properties properties=new Properties();
properties.setProperties("bootstrap.servers","")
properties.setProperties("group.id","")
properties.setProperties("key.deserializer","")
properties.setProperties("value.deserializer","")
DataStream<String> kafkaStream=env.addSource(new FlinkKafkaConsumer<String>("topicname",new SimpleStringSchema(),properties))
<!-- transfer用于解析string到对应的类 -->
kafkaStream=kafkaStream.map(new Transfer()).assignTimestampsAndWatermaks(WatermakStrategy.<Event>forMonotonousTimestamps().withTimestampAssigner(new SerializableTimestampAssigner<Event>(){
	@Override
	public long extractTimestamp(Event element, long recordTimestamp) 
{
 return element.timestamp;
 }
	}))

kafkaStream.keyBy("category","productId").window(SlidingEventTimeWindow(Time.seconds(24*3600),Time.seconds(24*3600)).aggregate(Count(),new WindowResultFunction()).keyBy("windowEnd").process(new TopItems())
```

## 0901算法题
### 链表指定区间反转
```
import java.util.*;

/*
 * public class ListNode {
 *   int val;
 *   ListNode next = null;
 * }
 */

public class Solution {
    /**
     * 
     * @param head ListNode类 
     * @param m int整型 
     * @param n int整型 
     * @return ListNode类
     */
//     将输入的整个链表反转
    public void ReverseList(ListNode head){
        ListNode pre=null;
        ListNode cur=head;
      
        while(cur!=null){
            ListNode nextNode=cur.next;
            cur.next=pre;        
            pre=cur;
            cur=nextNode;
        }
    }

//     思路基本一致啊。
//     改成前后为null，提前存储4个连接点
    public ListNode reverseBetween (ListNode head, int m, int n) {
        // write code here
        ListNode vHead=new ListNode(-1);
        vHead.next=head;
        ListNode temp=vHead;
//         begin left right end
        for(int i=0;i<=m-2;i++)
        {
            temp=temp.next;
        }
        
        ListNode begin=temp;
        ListNode left=begin.next;
        
        for(int i=m-1;i<=n-1;i++)
        {
            temp=temp.next;
        }
        
        ListNode right=temp;
        ListNode end=right.next;
        
//         System.out.println("begin="+begin.val);
//         System.out.println("left="+left.val);
//         System.out.println("right="+right.val);
//         System.out.println("end="+end.val);
        
        begin.next=null;
        right.next=null;
        //         反转局部        

        ReverseList(left);
        // 连接反转部分的头和尾部
        begin.next=right;
        left.next=end;
        
        return vHead.next;

    }
}
```
## 二分查找-I
```
import java.util.*;


public class Solution {
    /**
     * 代码中的类名、方法名、参数名已经指定，请勿修改，直接返回方法规定的值即可
     *
     * 
     * @param nums int整型一维数组 
     * @param target int整型 
     * @return int整型
     */
    public int search (int[] nums, int target) {
        // write code here
        int left=0;
        int right=nums.length-1;
        int mid=(left+right)/2;
        while(left<=right){
            if(nums[mid]==target)
            {
                return mid;
            }
            if(nums[mid]>target){
//                 左边
                right=mid-1;
            }else{
                left=mid+1;
            }
            mid=(left+right)/2;
        }
        return -1;
    }
}
```

# 数据仓库工具箱
## 第10节 金融服务
>金融服务涉及各行业，如信用卡公司，抵押贷款提供商等，日常接触的零售银行。一家银行提供广泛的产品，包括活期存款，储蓄账户，按揭贷款，个人贷款，信用卡以及银行贵重物品保险箱等。
>主要讨论如下概念：
>银行总线矩阵片段
>对维度进行分类以避免维度太少的陷阱
>家庭维度
>用一个账户关联多个客户的桥接表，以及权重因子。
>报表的动态范围值实时

### 银行案列研究与总线矩阵

>业务用户希望看到每个账户5年的，以月为单位的历史快照数据。
>每个账户有一个基本的余额。业务上希望用同一个分析来分组不同类型的账户并比较余额
>每个类型的账户（银行的产品）有一系列自定义维度属性和数值事实，其内容不同的产品差异大
>每个账户认为属于某一家庭。因为婚姻状况变化和其他不同生命阶段因素，导致账户家庭关系产生大量变化（每天都有人死亡，结婚，离婚）
>除了家庭之外，对人口统计信息关注，以个人或家庭为单位。此外，关注每个账户和家庭的行为积分，如存储行为等特征

### 对维度进行分类以避免维度太少的陷阱
>对于用户月快照事实表这个需求，一般需要在dwd层建立一个周期型快照事实表，ods使用全量数据同步。dwd层再去关联其他事实表的维度，例如家庭维度，账户维度，产品维度，账户状态维度。这里的维度数量很多，将他们放在一起是不行的，因为会造成单个表数据量过大，而且在业务场景中家庭，产品这些都是分块的功能。我们可以按照家庭，产品，账户等建立多个表，然后关联到月份周期型快照事实表，这和一个星型模型建模类似，一个事实表关联多个表。

### 家庭维度
>从银行角度，家庭可能由几个账户和独立的账户拥有人构成。构建商业上的家庭需要设计业务规则和算法以将账户分配到家庭中。
>之所以将家庭和账户维度分开是因为，账户维度的大小和家庭维度不一致，如1000万条账户可能只有300万条家庭数据，导致重复存储家庭数据。而且家庭维度中的账户有很大波动性，家庭维度为与事实表关联提供了更小的入口点，不需要遍历1000万条事实表数据。将家庭与账户分开可以应对两者的不同数据量，以及应对家庭关系中的账户发生频繁的变化。

### 多值维度与权重因子
>在账户维度表中，一个账户可以对应多个客户，多个客户共同拥有此账户，显然这种1对n的关系，无法将客户作为账户维度表的一个属性。我们可以再建立一个桥接表，桥接客户和账户，在该表中描述这种1对多的关系，使用客户表主键和账户表主键关联两个表的数据。
>权重因子，为同一个账户的客户分配权重，其和为1。权重因子是为了在计算以客户为基础的一些事实时，给客户分配可加的事实。

### 报表的动态范围值实时
>假设需要查询范围值报表的能力，如需要余额在0-100,100-500,500-550。可以将这一组分组规则存储在一个单独的范围定义表，使用一个lower value和upper value代表一个范围，同一个group_name的是同一组分组规则。范围定义表属性有 lower value,upper value,group name.

# 流批的差别
>流批的差别：
>1流处理是实时的，但是批处理需要设定具体的时间周期，如一天，12小时等。这也导致流处理时，每到来一个元素就要计算。而批处理是等待全量元素都到达后进行计算。流处理同等的对待所有数据，以单个事件为对象编写逻辑。批处理是以一段时间内的全量数据编写逻辑，将结果存储到数据库。
>2需要达到的结果，就是一段时间内的指标统计，例如截止目前，最近一个月，最近一周，截止目前每周。这些结果实际上是周期性的数据，是对一段时间内的所有数据计算得到的，这与批处理结果相同，但是与流计算相悖。
>如果需要某段时间的指标呢，如1月整个月的销售数量。需要什么就和批处理一样，将其单独处理，例如使用groupby将同一月份拿出来。设计处理任务时，该任务就专门统计每个月的数量。
>假设电商的用kappa架构：多层之间的数据，如果需要业务系统使用，就存储到mysql等。如果需要后续的实时数仓使用就存储到kafka。
>两种需求：1统计每个月的2统计截止目前的。对于需求2不用单独编写代码，可以在需求1计算完成后存储到clickhouse等，再轻度聚合。
>Flink实现了流处理和批处理使用统一的接口，使用流处理思想编写的DataStream也能以批处理方式运行。以sql编写的逻辑，也能以流处理方式运行。
# other
```
计算机基础八股文（算法，网络），八股文(java（jvm，网络，多线程，javase）,框架(hdfs,mr,yarn,hive,spark),javaweb)，项目（数仓（电商场景，TI），web）,笔试（算法题，sql题）
其他：linux环境下的shell及日常使用，git使用，maven使用，nginx，docker
```

```
框架这一块：辅助组件(zookeeper，kafka)，数据采集迁移框架(flume，logtash，sqoop,kettle)，数仓(hadoop,hive)，查询分析框架(spark,kylin,presto,phoenix)，可视化框架(superset)，整体流程控制(azkaban)，数据库（es，mysql）
```

```
常见业务场景：电商，银行证券基金保险，物联网（如温度传感器，网卡设备等，车联网），媒体（短视频，文本等）
```

```
#面经查漏补缺
https://www.nowcoder.com/discuss/792950?channel=-1&source_id=profile_follow_post_nctrack
https://www.nowcoder.com/discuss/719344?source_id=discuss_experience_nctrack&channel=-1
https://www.nowcoder.com/discuss/916657?source_id=discuss_experience_nctrack&channel=-1
https://www.nowcoder.com/discuss/842391?source_id=discuss_experience_nctrack&channel=-1
https://www.nowcoder.com/discuss/868723?source_id=discuss_experience_nctrack&channel=-1
#coursera上的课程，项目
https://www.coursera.org/specializations/big-data
#github iot spark，项目
https://github.com/harsh86/iot-traffic-monitor-flink
```


```
7月提前批，提前批没有笔试，难度较低。
项目一般写相关性强的2-3个项目，项目做了什么，使用了什么技术，要对项目非常熟悉。项目背景，项目挑战，负责内容。
技术栈，项目与实习，奖项，学历

一面：基础知识+算法+项目内容
二面：压力面
基础知识+算法+项目内容，难度加大
表现形式：问的深，连环追击，发散性强，打压
方法：会多少说多少，不会就说不清楚
三面：总监面
开放性问题，
hr面，表现稳定性，合作能力，意愿
反问：团队是做什么，那一个团队

offer选择
流程：意向书->谈薪资（10月中旬开始）->正式offer（10月下旬）
核心业务，薪资，平台

科院内容：根据相关性准备，不相关则不提及。若不相关，但是问及科研内容，主要考察表达能力，只要讲清楚即可。
offershow
银行信息岗：有轮岗
在牛客网搜集信息，查看面经，尝试逐个回答，在官网投递简历。

准备思路清晰的ppt，面试的主动性
```

```
如果需要描述项目的困难和挑战，一般描述技术上的困难、业务场景的困难，不要提逻辑上的困难，没有含量。
如技术上的如何解决倾斜、如何解决并发访问、如何确保精确一次消费、
场景上的如何解决活动流量爆发、地区订单数据倾斜、如何进行订单去重、如何处理UV和PV指标。
```