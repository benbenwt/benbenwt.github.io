# 待整理

- 1,基本数据类型：char boolean float double  byte short  int long
- 位数                       16     32      32       64      8       16    32   64
- 默认值         ‘\u0000'  false    0.0f    0.0d     0        0     0      0
- 2,面向对象特征：1封装：把描述对象的属性和行为封装进类，用变量和方法表示 2抽象：把生活中的对象抽象为类，数据抽象，过程抽象。
- 3继承：子类继承父类的属性和行为，也可以扩展行为重写方法。  4多态：程序中定义的引用变量所指向的具体类型在编程时不确定，运行时才确定。
- 3,包装类型：为了使基本类型也具有对象的特征，就有了包装类型。  自动装箱：通过构造将基本转为类，底层为调用封装类的valueOf()  自动拆箱：自动将
- 封装类型转为基本类型 ，底层为：intValue()      1声明方式不同：包装类使用new关键字在堆中分配存储空间    2包装类型存储在堆，基本类型存储在栈
- 3默认值 int为0 Integer为null
- 4，==和equals区别： ==比较两个引用是否指向同一对象，即内存地址。 equals：比较某些特征，比如String重写的比较内容。
- 5,String,StringBuffer和StringBuilder  都是常用字符串类，String使用private final char value[]，不可变。StringBuffer和StringBuilder都继承AbstractStringBuilder,使用可变数组。
- StringBuffer线程安全，效率低 StringBuilder线程不安全，效率高。
- 6,进程和线程：1并发执行的程序在执行过程中计算机分配资源的单位，线程包含在进程中，一个进程可以有多个线程。是进程内部的调度单位。
- 7,集合 Collection下有List(有序，允许重复)和Set（无序，不重复）.  set根据equals和hashcode判断，一个对象要存入set，必须重写equals和hashcode。
- Map下有HashMap,线程不同步,TreeMap，线程同步。
- 8，ArrayList基于动态数组，LinkedList基于链表。随机访问set，get时ArrayList优于LinkedList，LinkedList只能移动
- 指针。但插入和删除链表更快。
- 9,ConcurrentModificationExceptin 由于使用iterator访问，但是使用list.remove(),应使用iterator.remove()来删除。
- 10，HashMap,HashTable都实现了Map接口，存储key-value数据。不同点：1HashMap的key和value可以为null。
- 2HashMap线程不安全3迭代器不同
- 11，如何保证线程安全又效率高。currentHashMap替代HashTable
- 12，拷贝文件的工具类使用的是字节流
- 13，线程的创建，1继承thread类，作为线程对象存在。重写run（），start（），sleep（），wait（）。
- 实现Runnable接口，重写run方法。
- 14,static修饰的方法和变量，不属于任何实例对象，属于类，类创建时，就可访问，所以他们称为类方法，类成员。
- 15，

# 源码

### java.util.String

```

public String(String original) {
    this.value = original.value;
    this.hash = original.hash;
}
```

```
equals()
public boolean equals(Object anObject) {
    if (this == anObject) {
        return true;
    }
    if (anObject instanceof String) {
        String anotherString = (String)anObject;
        int n = value.length;
        if (n == anotherString.value.length) {
            char v1[] = value;
          char v2[] = anotherString.value;
            int i = 0;
            while (n-- != 0) {
                if (v1[i] != v2[i])
                    return false;
              i++;
            }
            return true;
        }
    }
    return false;
}
```

比较主串从toffset下标处匹配是否和prefix相等。

```

startWith()
public boolean startsWith(String prefix, int toffset) {
    char ta[] = value;
  int to = toffset;
    char pa[] = prefix.value;
    int po = 0;
  int pc = prefix.value.length;
    // Note: toffset might be near -1>>>1.
    if ((toffset < 0) || (toffset > value.length - pc)) {
      return false;
    }
    while (--pc >= 0) {
        if (ta[to++] != pa[po++]) {
          return false;
        }
    }
    return true;
}

```

```
endWith()
可用startsWith实现，相当于从主串长度减去后缀串长度处的下标开始匹配。
public boolean endsWith(String suffix) {
    return startsWith(suffix, value.length - suffix.value.length);
}
```

```
hashcode()
public int hashCode() {
  int h = hash;
    if (h == 0 && value.length > 0) {
        char val[] = value;
        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];
      }
        hash = h;
  }
    return h;
}
```

实际上是调用了构造函数，重新创建了一个String，将value赋予对应的值。

```
subString()
public String substring(int beginIndex, int endIndex) {
    if (beginIndex < 0) {
        throw new StringIndexOutOfBoundsException(beginIndex);
    }
    if (endIndex > value.length) {
        throw new StringIndexOutOfBoundsException(endIndex);
    }
    int subLen = endIndex - beginIndex;
    if (subLen < 0) {
        throw new StringIndexOutOfBoundsException(subLen);
    }
    return ((beginIndex == 0) && (endIndex == value.length)) ? this
            : new String(value, beginIndex, subLen);
}
```

将String.value复制到新的数组中，并返回。

System.arraycopy(value, 0, result, 0, value.length);
将数组value复制到result中

```
toCharArray()
public char[] toCharArray() {
    // Cannot use Arrays.copyOf because of class initialization order issues
    char result[] = new char[value.length];
    System.arraycopy(value, 0, result, 0, value.length);
    return result;
}
```

# String

### 正则表达式

##### 常用函数

mathes,replaceAll,replaceFirst(),split().
java还提供了Pattern类表示正则表达式对象，提供了丰富的API进行各种操作。

##### 元字符

.:匹配除换行符以外任意字符
\s:匹配任意的空白符（空格，指标，换行）
\w:匹配字母和数字或下划线或汉字
\d:匹、配数字
\b：匹配单词的开始或结束
^：匹配字符串的开始
$：匹配字符串的结束

##### 字符转义

元字符等有特殊含义，若要使用，用\转义。
如：.    \.   
\*   \*

##### 限定符

*：重复零次或更多次
+：重复一次或更多次
？：重复零次或一次
{n}：重复n次
{n，}：重复n次和更多次
{n，m}：重复n，m次​​

##### 字符类(对元字符扩展)

[0-9],[a-z0-9A-Z]

##### 分枝条件（对限定符扩展）

|

##### 分组（元字符和限定符连接）

（）进行分组

##### 反义（对元字符扩展）

字母改大写表反义
如\W \S \D \B

##### 后向引用

分组后编号默认从1开始，也可以自定义名字
\b(\w+)\b\s+\1\b
\b(?<Word>\w+)\b\s+\k<Word>\b

##### 零宽断言

(?<=exp1)exp2 匹配exp2前面是exp1的位置
exp2(?=exp1) 匹配exp2后面是exp1的位置
(?<!exp1)exp2   匹配exp2前面不是是exp1的位置
exp2(?!exp1)​    匹配exp2后面不是exp1的位置

##### 负向零宽断言

##### 注释

##### 贪婪与懒惰

当使用*,+,.等时，尽可能匹配多的，称为贪婪。
在限定符后边加？可转换为懒惰模式。
如:*?,.?,+?​,{n,m}?

##### 处理选项

##### 平衡组/递归匹配

other

### Object

### 创建对象5种方式

1.new 2.用类的newinstance 3用constructor的newinstance 4clone 5反序列化

### 序列化

序列化指将java对象数据，通过某种方式转换成二进制流，可以存储和在网络上传输。
反序列化是从流或网络上读取二进制，再恢复成对象的过程。
​为什么使用序列化
1分布式系统需要网络间传输，共享javabean对象。
2​服务器端对于不活跃的对象，将其序列化存储再磁盘。需要时再读出。
实现
需要序列化的​类必须继承Serializable接口
通过ObjectOutputStream。writeObject（）序列化，ObjectInputStream。readObject（）反序列化
    ByteArrayOutputStream bos = new ByteArrayOutputStream();
    ObjectOutputStream oos = new ObjectOutputStream(bos);
    oos.writeObject(this);
    // 反序列化
    ByteArrayInputStream bis = new ByteArrayInputStream(bos.toByteArray());
    ObjectInputStream ois = new ObjectInputStream(bis);

### 深拷贝与浅拷贝

基本类型存储再栈中
引用类型将引用放在栈，实际值存储再堆中，栈中的引用指向堆中存放的数据。
​浅拷贝就是直接非静态的属性，即对于基础类型复制其值，而引用类型只复制引用，所以和原对象指向相同的值。
深拷贝则是无论引用还是基础，都复制独立的一份。
how：
对于每个引用类型，让其继承cloneable，并重写其clone方法。​

equals()
public boolean equals(Object obj) {
              return (this == obj);
           }
在java中==是比较两个引用对象的引用或比较基础类型的值。
此处即比较对象的地址。
我们自己如何改写equals：how
1自反性 2对称性 3传递性 4一致性 5对于任何非x，x.equals（null）值为null​
参考String类的equals
若​涉及到父类和子类则instanceof判断类型不合理。可根据情况使用getclass（），获得运行时类。
重写equals必须重写hashcode（）

### hashcode（）

java集合的实现借助哈希表，他可以快速使用equals函数，快速操作。
当集合添加元素时
​1若其位置上无冲突，放置。
2若有冲突，则equals，如不同放置在链表上。
由此可以看出，放置在同一位置的可以是equals不等的元素。
​
故equals同，hash同
不同，不同
​
hash同，equals不一定
不同，不同​​

how：
避免冲突，为int，4字节，避免超出，借鉴String的hashcode。
由于map集合也使用hash进行比对，作为key的对象必须重写equals和hashcode。​​

- toString()
  public String toString() {
      return getClass().getName() + "@" + Integer.toHexString(hashCode());
  }

-  clone()
  返回当前对象的副本，是深拷贝，互不影响。
  调用对象的clone方法，必须让类实现Cloneable接口，覆写clone方法。
  ​

- 关键字

  - native
    被修饰的方法是用C/C++语言实现的，并且被编译成了DLL，由java去调用

- reflect

  - 原理
    使用流程
    method.invoke(instance,args) method.invoke(instance)
    1创建一个输出类
    2创建代理类，自定义函数并在其中用反射调用​方法，添加前置和后置的操作，实现动态代理。
    若使用java的动态代理类，需要接口，则让代理类继承InvocationHandler。在讲操作交给Proxy。
     Subject proxyInstance = (Subject) Proxy.newProxyInstance(subjectProxy.getClass().getClassLoader(), subject.getClass().getInterfaces(), subjectProxy);
    （代理类类加载器，被代理类接口，代理类对象）

  

# String

# GC

### 为什么要gc

当一个对象不可达，或一个对象没有任何引用指向它，他就没有必要存在，此时就可以被gc回收。

### jvm区域

##### java stack

每个jvm线程都有自己私有的java虚拟栈，这个栈和线程同时创建，他的生命周期和线程相同。每个java方法被执行时创建一个栈帧用于存储变量表，操作数，动态链接，方法出口等信息。每个方法被调用直至执行完成对应着再java stack中入栈和出战的过程。

##### native method stack

与虚拟机栈的作用相似，虚拟机栈执行java方法，而本地方法栈为虚拟机使用到的本地方法服务。本地方法：由其他语言编写，和处理器相关的机器代码。本地方法保存在动态链接库中，即dll中。

##### heap

所有线程共享的一块区域，用来存储对象实例和数组值。new的对象。

##### PC regiter

程序计数器保存当前线程执行到哪行（指令地址），以便线程之间切换。每个线程都有自己的PC，以便完成不同线程切换。

##### method area

方法区是线程共享区域，用于存储每个类的结构信息。例如成员变量和方法数据，构造函数和普通函数字节码内容，还包括一些类，实例，接口初始化时用到的特殊方法。当开发人员在程序中通过class对象的getName，isInstance获取消息时，这些数据都来自方法区。在一定条件下也会被gc，这块区域对应Permanent generation持久代。

##### 运行时常量池，其空间从方法区分配，存储类中常量，方法，域引用信息。

##### 触发gc的情况

非线程对象不被指向或超出作用域
线程对象线程未启动或停止

##### 改变对象引用，置为null或指向其他对象。

Object x=new Object();//object1 
   Object y=new Object();//object2 
   x=y;//object1 变为垃圾 
   x=y=null;//object2 变为垃圾

##### 超出作用域

if(i==0){ 
      Object x=new Object();//object1 
   }//括号结束后object1将无法被引用，变为垃圾 

##### 类嵌套导致未完全释放

class A{ 
      A a; 
   } 
   A x= new A();//分配一个空间 
   x.a= new A();//又分配了一个空间 
   x=null;//将会产生两个垃圾 

##### 线程中的垃圾

class A implements Runnable{   
     void run(){ 
       //.... 
     } 
   } 
   //main 
   A x=new A();//object1 
   x.start(); 
   x=null;//等线程执行完后object1才被认定为垃圾 
   这样看，确实在代码执行过程中会产生很多垃圾，不过不用担心，java可以有效地处理他们。

##### Java垃圾回收机制算法

 1.标记-清除算法
标记：标记出所有要回收的对象
清除：回收被标记对象
缺点：效率低，这两个操作效率都不高
​​空间问题，形成大量不连续碎片空间。当以后需要分配较大的对象时，无法找到足够大的连续空间，触发gc。

2.复制算法
内存分为相等的两块，需要回收时，把存活的对象拷贝到另一半，清除当前的整个块，以保证空间连续性。
缺点：代价高。
现在一般分为三块，Eden和两块较小的Survivor。当回收时，将存活对象放到另一个Survivor中，清除当前Survivor和Eden.

3.标记-整理算法
标记：标记所有要回收的对象。
整理:将存活对象向一段移动，然后直接清理到边界以外的内存空间

4.分代收集算法
分代：新生代，老生代
新生代：存活少，使用复制算法。
老生代：存活多，使用标记清除或标记整理算法。​

# DB

### JDBC

##### JDBC操作数据库步骤

1注册驱动
2创建连接
3创建语句
4处理结果​

##### Blob和Clob

Blob是指二进制大对象，Clob指大字符对象，jdbc两者都支持。
    public static void main(String[] args) {
        Connection con = null;
        try {
            // 1. 加载驱动（Java6以上版本可以省略）
            Class.forName("com.mysql.jdbc.Driver");
            // 2. 建立连接
            con = DriverManager.getConnection("jdbc:[mysql://localhost:3306/test](http://mysql://localhost:3306/test)", "root", "123456");
            // 3. 创建语句对象
            PreparedStatement ps = con.prepareStatement("insert into tb_user values (default, ?, ?)");
            ps.setString(1, "骆昊");              // 将SQL语句中第一个占位符换成字符串
            try (InputStream in = new FileInputStream("test.jpg")) {    // Java 7的TWR
                ps.setBinaryStream(2, in);      // 将SQL语句中第二个占位符换成二进制流
                // 4. 发出SQL语句获得受影响行数
                System.out.println(ps.executeUpdate() == 1 ? "插入成功" : "插入失败");
            } catch(IOException e) {
                System.out.println("读取照片失败!");
            }
        } catch (ClassNotFoundException | SQLException e) {     // Java 7的多异常捕获
            e.printStackTrace();
        } finally { // 释放外部资源的代码都应当放在finally中保证其能够得到执行
            try {
                if(con != null && !con.isClosed()) {
                    con.close();    // 5. 释放数据库连接
                    con = null;     // 指示垃圾回收器可以回收该对象
                }
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }

##### 读和更新性能

使用结果集对象的setFetchSize（）控制抓取记录数，提高读取性能。
​使用PreparedStatement语句构建批处理，提高更新性能。

- PreparedStatement和Statement
  ps：1预编译，减少sql的编译错误，提高sql安全避免sql注射攻击。
  2避免拼接，使用参数。
  3批处理性能高。​

### 事务处理



Connection提供了事务处理的方法，通过调用setAutoCommit(false)可以手动提交事务。当事务完成后用commit（）显式提交。

- 四特性
  原子性：事务要么做完，要么不做。若某一步操作异常，通过回滚到达事务开始或保存点。
  一致性：数据库只能从一种一致的状态转移到另一种一致的状态。不能操作一半。
  隔离性：事务与事务之间独立，事务隔离级别：
  1READ_UNCOMMITTED
  2READ_COMMITTED
  3REPEATABLE​_READ
  4SERIALIZABLE
  持久性：事务提交后，对数据库中数据的改变是永久性的。

- 三大数据问题

  - 脏读
    事务a读取了b未提交的数据，并做了其他操作。

  - 不可重复读
    a读取b已提交的更改数据，两次不一样。隔离后不可修改。

  - 幻读
    a读取b提交的新增数据。隔离后不可插入数据。

- 两类数据更新问题

  - 第一类
    a撤销，导致覆盖b的提交数据。

  - 第二类
    a提交的数据覆盖b提交的数据。

- DAO

  - DAO模式
    为数据库操作提供一个抽象接口，其中抽象了对数据源的操作。定义了事务方法，通过实现该接口来和数据源交互。
    1DAO接口
    ​2DAO实现类
    3实体类
    4数据库连接和关闭工具类​

  - 连接池
    1避免创建和释放连接的消耗，提高性能。典型的用时间换空间，与线程池同理。

# IO

# OOP

- java和javaScript

- 面向对象主要特征

  - 抽象

  - 封装

  - 继承

  - 多态
    声明一个基类，他可以指向任意一个他的派生类。
    调用基类的同一个方法，他可以表现为任意一个派生类的方法。

- 修饰符

  - public
    当前类，同包，子类，其他包

  - protected
    当前类，同包，子类

  - 不写
    当前类，同包

  - private
    当前类

- 关键字

  - assert

  - static

    静态方法不能重写，重写只适用实例方法。

    - 内部类
      直接通过外部类类名创建内部类，不用创建外部类实例。

    - 方法
      类方法，通过类名调用。加载类时初始化。

    - 变量
      类变量，同类方法。

  - final

    - 类
      不可继承

    - 方法
      不可重写

    - 变量
      赋值后不可修改

- 子类父类调用顺序
  new一个类对象，类中各部分执行顺序：静态代码块—非静态代码块—构造函数—一般方法。
  子类继承父类各部分执行顺序为：父静态代码块--子静态代码块--父非静态代码块--父无参构造函数--子非静态代码块--子构造函数--方法。
  如果没有写super()，则默认调用父类的无参构造方法。

# Exception

# JVM

- JVM architecture![img](https://api2.mubu.com/v3/document_image/9be69071-6f20-4f30-97dc-0bbedb078bff-6012434.jpg)
  JVM = 类加载器 classloader + 执行引擎 execution engine + 运行时数据区域 runtime data area

- classloader

  - 两种装载方式
    classloader两种装在class的方式：
    1.隐式：运行过程中，碰到new方式生成对象时，隐式调用classloader到JVM。
    2.显示：通过class.forname()动态加载。​

  - 双亲委派模型（Parent Delegation Model）

    - 工作过程
      1当前ClassLoader首先从自己已经加载的类中查询此类是否已加载。若已经加载就返回加载的类。
      每个类加载器都有自己的加载缓存，当一个类被加载以后就会放入缓存。
      2当前classloader的缓存中没有找到时，委托父类加载器去加载，父类加载器采用同样的策略，直到找到或到达bootstrap classloader.
      3.当所有的父类加载器都没有加载时，再由当前的类加载器加载，并放入自己的缓存。​

    - 优点
      1为了安全性，避免用户的类动态替换java的核心类，比如String。同时避免了重复加载，因为相同的class文件被不同classloader加载是不同的两个类，相互转型会抛出classCaseException。

    - 常用类加载器
      Bootstrap class loader：所有类加载器的父类，当运行jvm时，她负责核心库的加载，如java.lang.*等。例如java.lang.Object就是由根类加载器加载的。此加载器不是由java编写的，而是c/c++写的。
      Extension class loader：这个加载器加载除了基本API之外的拓展类。
      AppClassLoader:加载应用程序和程序员自定义的类。​
      用户也可以自定义自己的类加载器，java提供了java.long.classloader.

- 执行引擎
  执行字节码，或者执行本地方法。

- runtime data area

  jvm运行期间，对内存空间的划分和分配。jvm将内存分为了6个区域来存储。程序员所写的程序都被加载到运行时数据区域中，不同类别存放在heap，java stack,native method stack,PC register ,method area.
  ​

  - java stack
    每个jvm线程都有自己私有的java虚拟栈，这个栈和线程同时创建，他的生命周期和线程相同。
    每个java方法被执行时创建一个栈帧用于存储变量表，操作数，动态链接，方法出口等信息。每个方法被调用直至执行完成对应着再java stack中入栈和出战的过程。

  - native method stack
    与虚拟机栈的作用相似，虚拟机栈执行java方法，而本地方法栈为虚拟机使用到的本地方法服务。
    本地方法：由其他语言编写，和处理器相关的机器代码。本地方法保存在动态链接库中，即dll中。

  - heap
    所有线程共享的一块区域，用来存储对象实例和数组值。new的对象。

  - PC regiter
    程序计数器保存当前线程执行到哪行（指令地址），以便线程之间切换。
    ​每个线程都有自己的PC，以便完成不同线程切换。

  - method area

    方法区是线程共享区域，用于存储每个类的结构信息。例如成员变量和方法数据，构造函数和普通函数字节码内容，还包括一些类，实例，接口初始化时用到的特殊方法。当开发人员在程序中通过class对象的getName，isInstance获取消息时，这些数据都来自方法区。
    在一定条件下也会被gc，这块区域对应Permanent  generation持久代。

    - 运行时常量池，其空间从方法区分配，存储类中常量，方法，域引用信息。

# 集合

- Collection

  - Set

  - List
    - ArrayList和LinkedList
      arrayList是动态数组实现的，linkedList是双向链表实现的。
      随机访问的get和set方法，arrayList更快。
      增删操作，linkedList更快。​
      
    - ArrayList
      
      基于array实现
      
      - ArrayList<>(mincapacity)
        若mincapacity>0,创建对应长度数组。
        为0创建默认空数组。
        否则capacity不合法
      
      - add(element)
        1计算所需大小：若为空数组，使用默认空间大小，否则使用传入的size+1，也就是实际元素个数。
        2查看是否需要扩容，若所需空间大于旧空间的1.5倍，扩充至所需空间大小。若过大，则。。。其次，就扩充至1.5倍即可。
      
      - remove(index)
        1判断合法性，计算需要挪动多少元素
        2使用Array.copy
      
      - remove(object)
        1判断合法性，计算实在前半部分还是后边。
        2开始从头或尾查找，得到index
        3计算并Array.copy
      
    - HashMap
    
    - 

- Map

  - TreeMap

  - HashMap

# Reflection

- 获取反射类
  1.Class stu=Class.forName("com.weitao.domain.Student);
  ​2.Class s=stu.getClass();
  3.Class s=Student.class;
- 获得属性
  getFidlds(),getField(String name)
  getDeclaredFields,getDeclaredField(String name)​
- 获得方法
  getMethods(),getMethod(String name,Class Class<?>..paramTypes)
  getDeclaredMethods(),getMethod(String name,Class Class<?>..paramTypes)
  ​
- 获得构造方法
  getConstructors(),getConstructor(Class<?>..paramsTypes)
  ​getDeclaredConstructors(),getDeclaredConstructor(Class<?>..paramsTypes)
- 获得其他
  getName();
  newInstance();
  getInterFaces();
  getPackage();​
- 实例化对象
  使用获取到的反射类s.
  ​1Student stu=(Student)s.newInstance();
  ​2Constructor<Student>  constructor=s.getConstructor(String.class);
  Student stu=constructor.newInstance("李四");

# Thread

- 线程与锁

  - 进程和线程
    进程：并发执行的程序在执行过程中分配计算机资源的基本单位。
    线程：是进程的一个执行单元，是比进程更小的基本单位。
    一个程序至少一个进程，一个进程至少一个线程。
    ​​区别：进程的地址空间和资源是独立的，一个崩溃互不影响。
    但线程开销更小。
    应用场景：用户接入tomcat，tomcat分配给新线程，再调用servlet。
    ​​                         后台备份，前台不断询问后端进度。

  - 线程的实现
    继承Thread,实现Runnable,重写run方法。

  - 线程的状态

    

    - new:对象被创建后状态

    - runnable:对象start()后，等待cpu资源

    - running:获取到cpu，执行线程，只能由runnable转换而来

    - blocking:放弃cpu，停止

      - 等待阻塞
        - 调用wait(),失去锁，进入等待池。等待自己被唤醒。

      - 同步阻塞
        - 当线程申请锁失败，进入锁池，等待锁。

      - 其他阻塞
        - 通过调用join()，sleep()不会释放锁,IO请求。当以上操作结束或完成，进入就绪状态。

  - 多线程实现同步方法
    sychronized可以修饰方法，代码块。
    ReentrantLock:阻塞所，可重入，操作由程序员编写。可实现非公平锁。
    automaticInteger乐观锁，如原子变量就是乐观锁的应用。
    ​wait(),notify(),sleep()：wait使一个线程处于等待状态，释放所有lock。sleep保留锁。
    volatile(易变的):确保每次都重新取值，而不是使用寄存器的值。

  - 线程池

    - ThreadPool类型
      ThreadPoolExecutor, ScheduledThreadPoolExecutor通常由工厂类Executors创建。
      ​Executors可以创建三种类型的ThreadPoolExecutor:
      SingleThreadPool,FixedThreadPool和CachedThreadPool。
      ​Executors可以创建2种类型的ScheduledThreadPoolExecutor：
      ​SingleScheduledThreadPoo和ScheduledThreadPool。

    - ThreadPoolExecutor的重要参数
      corePoolSize:核心线程数量
      ​ maximumPoolSize:最大线程数量 
      ​workQueue:等待队列，当线程数量​​大于corePoolSize时，任务封装成worker放入队列。 keepAliveTime:当线程数超过corePoolSize时，超过此时间空闲，则回收。
      timeUnit:时间单位
      ​ threadFactory:指定创建的线程池类型
      handler:拒绝策略

    - 等待阻塞队列

      - ArrayBlockingQueue

      - Linked Blocking Queue

      - Synchro nousBlockingQueue

      - PriorityBlockingQueue

    - 拒绝策略
      AbortPolicy抛出RejectedExecutionException
      ​DiscardPolicy什么也不做，直接忽略
      ​DiscardOldestPolicy丢弃执行队列中最老的任务，尝试为当前提交的任务腾出位置
      ​CallerRunsPolicy直接由提交任务者执行这个任务

    - 过程原理
      If fewer than corePoolSize threads are running, the Executor always prefers adding a new thread rather than queuing.
      ​ If corePoolSize or more threads are running, the Executor always prefers queuing a request rather than adding a new thread.
      ​ If a request cannot be queued, a new thread is created unless this would exceed maximumPoolSize, in which case, the task will be rejected.
      即： corePoolSize -> 任务队列 -> maximumPoolSize -> 拒绝策略 

    - 线程池状态

      - running:能接受，能处理

      - shutdown:不接受新任务，能处理已添加

      - stop:不接受新任务，不处理已添加

      - tidying:由shutdown和stop转换而来

      - terminated:线程时彻底终止。由tidying执行terminated()转换而来

    - Executors快捷创建线程
      newFixedThreadPool(int nThreads)创建固定大小的线程池
      ​newSingleThreadExecutor()创建只有一个线程的线程池
      ​newCachedThreadPool()创建一个不限线程数上限的线程池，任何提交的任务都将立即执行

    - 使用ThreadPoolExecutor安全创建线程
      填写ThreadPoolExecutor重要参数：
      ​ExecutorService executorService = new ThreadPoolExecutor(2, 2, 
                      0, TimeUnit.SECONDS, 
                      new ArrayBlockingQueue<>(512), // 使用有界队列，避免OOM
                      new ThreadPoolExecutor.DiscardPolicy());

    - 提交任务
      Future<T> submit(Callable <T> task):
      ​关注返回值,用future.get()获得返回信息
      Future<T> submit(Runnable task)  不关注返回值
      void execute(Runnable command)​   不关注返回值

    - 中断线程

      - shutdown

      - shutdownnow

    - 如何正确使用线程池
      避免使用无界队列
      明确拒绝任务时的行为
      获取处理结果和异常
      ExecutorService executorService = Executors.newFixedThreadPool(4);
      Future<Object> future = executorService.submit(new Callable<Object>() {
              @Override
              public Object call() throws Exception {
                  throw new RuntimeException("exception in call~");// 该异常会在调用Future.get()时传递给调用者
              }
          });
          
      try {
        Object result = future.get();
      } catch (InterruptedException e) {
        // interrupt
      } catch (ExecutionException e) {
        // exception in Callable.call()
        e.printStackTrace();
      }

    - 线程池的优势
      降低资源消耗，创建和销毁线程很占用资源，jvm需要跟踪回收。
      提高响应速度，任务到达，无需等待创建线程。
      方便管理

    - AQS同步器
      AQS就是实现锁的框架，内部实现时FIFO，state状态，定义内部内ConditionObject

  - sleep和yield
    sleep保留锁，进入阻塞状态。他会给予低优先级线程机会。可能出现死锁，和interruptedExceptoion。
    yield进入就绪状态，只会给相同优先级或更高优先级的线程运行机会。

  - stop和suspend
    stop强制中断线程，解除所有锁定，其他线程就可以访问对象。
    suspend()目标线程停止，但仍然持有锁，可能导致死锁。A需要B来苏醒，B需要A的锁。
    应该使用wait(),notify().​

  - cyclicbarrier和countdownlatch
    CountDownLatch用于A等待若干个线程执行完任务后，他才执行。
    CylicBarrier一般用于一组线程互相等待至某个状态，然后同时执行。

  - 公平锁和非公平锁，可重入锁，读写锁，中断锁
    用队列FIFO是公平锁的一个完美方式，能保证每个人都拿到锁。
    公平锁效率低，非公平锁能利用好cpu碎片时间。
    非公平锁需要锁时，直接尝试获取锁，失败则排到队尾。
    ​可重入：多次申请一个对象的锁，如method1调用method2，就会再次申请对象锁。
    读写锁：使多个线程的读操作不冲突。
    中断锁：B在锁池等待，但突然要处理其他事情，就中断自己。​

  - sychronized和Lock
    都是阻塞锁。都可重入，sychronized使用计数器实现。Lock需要程序员自己实现，更加灵活适用复杂的场景，sychronized由系统隐式实现。

  - 悲观锁和乐观锁
    悲观锁：假设数据一定发生冲突，通过阻塞来保证数据安全。
    乐观锁：假设不会发生冲突，到更新时再检查。
    ​CAS就是使用的乐观锁,v:内存值，A：期望的旧值 B：新值，比较v，A,若相等才交换为B。无法处理ABA的情况，处理方法，加上version号。

  - 死锁
    互斥条件：一个资源只能被一个进程使用
    请求与保持条件：一个进程请求资源而阻塞时，对已获得的资源不释放。
    不剥夺条件：进程已获得的资源，在使用完之前，不能强行剥夺
    循环等待条件​：若干进程形成头尾相接的循环等待资源关系。

  - 避免死锁
    1：破环请求和保持条件：请求失败后，释放已有资源。
    2:破坏不可抢占，代价大。
    3：破环循环，规定顺序，避免相互等待。​