# util

>Collection为顶级的接口，List接口继承自Collection，AbstractCollection实现自Collection。

### Iterator

>Iterator为util包中较高等级的接口。Collection及其子接口，子类，子抽象类等都基于其实现add，get，set，remove等方法。如，AbstractList，ArrayList，LinkedList等

```
#对Iterator中剩下的元素进行操作,default标记的方法为普通方法。该方法会被类继承，若一个类同时继承了冲突的同名default方法，则必须重写。若继承的类和实现的接口中default方法冲突，则覆盖掉interface中的default中方法。
default void forEachRemaining
```

Iterator定义的函数有，next，hasNext，remove。

##### ListIterator

```
继承Iterator，新定义了add,set,previous,previousIndex,hasPrevious,nextIndex
```

previous:返回前一个元素值，并向该方向移动。

next：返回后一个元素，并向该方向移动。

### Collection

##### Collection接口

>Collection接口继承了Iterator，提供了增删改查的抽象方法

collection接口定义了如下：

返回元素个数

```
int size()
```

集合是否为空

```
boolean isEmpty();
```

是否包含此object

```
boolean contains(Object o);
```

返回对应泛型的迭代元素

```
Iterator<E> iterator();
```

以数组形式返回元素

```
Object[] toArray();
```

添加元素

```
boolean add(E e);
```

移除对应object

```
boolean remove(Object o);
```

将collection合并到此对象

```
boolean addAll(Collection<? extends E> c);
```

删除所有在collection参数中元素

```
 boolean removeAll(Collection<?> c);
```

只保持在参数collection中的元素

```
boolean retainAll(Collection<?> c);
```

清除所有元素

```
void clear();
```

是否相等

```
boolean equals(Object o);
```

hash值

```
int hashCode();
```

AbstractCollection



##### List接口

>相比于Collection接口，List接口增添了关于index的操作。

List相较于Collection特有的：

获得在index处的元素

```
E get(int index);
```

替换在index处的元素

```
E set(int index, E element);
```

在index处添加元素

```
void add(int index, E element);
```

在index处插入collection

```
boolean addAll(int index, Collection<? extends E> c);
```

该object在list中的第一个匹配下标

```
int indexOf(Object o);
```

最后一个匹配下标

```
int lastIndexOf(Object o);
```

返回list迭代器

```
ListIterator<E> listIterator();
```

返回从index之后的元素的迭代器

```
ListIterator<E> listIterator(int index);
```

子List

```
List<E> subList(int fromIndex, int toIndex);
```

##### AbstractCollection

>AbstractCollection实现了collection的很多方法

其实现的方法如下:

是否为空

```
public boolean isEmpty() {
    return size() == 0;
}
```

是否包含此object。使用iterator遍历collection，查看是否包含object。

```
public boolean contains(Object o) {
    Iterator<E> it = iterator();
    if (o==null) {
        while (it.hasNext())
            if (it.next()==null)
                return true;
    } else {
        while (it.hasNext())
            if (o.equals(it.next()))
                return true;
    }
    return false;
}
```

数组形式返回collection。size()为collection的元素个数，应该等于元素个数。为什么会有fewer和more的情况。

若比预期的少，Arrays.copyOf(r, i)返回一个包含相同值的新数组。若比预期的多，创建新数组并返回。

```
public Object[] toArray() {
    // Estimate size of array; be prepared to see more or fewer elements
    Object[] r = new Object[size()];
    Iterator<E> it = iterator();
    for (int i = 0; i < r.length; i++) {
        if (! it.hasNext()) // fewer elements than expected
            return Arrays.copyOf(r, i);
        r[i] = it.next();
    }
    return it.hasNext() ? finishToArray(r, it) : r;
}
```

创建扩容的数据，将剩余数据装载进新数组并返回

```
private static <T> T[] finishToArray(T[] r, Iterator<?> it) {
    int i = r.length;
    while (it.hasNext()) {
        int cap = r.length;
        if (i == cap) {
            int newCap = cap + (cap >> 1) + 1;
            // overflow-conscious code
            if (newCap - MAX_ARRAY_SIZE > 0)
                newCap = hugeCapacity(cap + 1);
            r = Arrays.copyOf(r, newCap);//创建新数组
        }
        r[i++] = (T)it.next();//迭代将剩余元素装入新数组
    }
    // trim if overallocated
    return (i == r.length) ? r : Arrays.copyOf(r, i);
}
```

若小于0说明溢出了，抛出异常。若大于数组最大size返回int最大值，否则返回数组最大值。数组最大size比integer最大size小8。由于数组元数据占用一部分空间，故少了8。元数据包括class：描述对象类型的类i信息指针，flag:描述此对象的散列码及形状，lock：是否同步，size：大小。

```
private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError
            ("Required array size too large");
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
        MAX_ARRAY_SIZE;
}
```

AbstractCollection没有实现add方法，其实现了remove,removeAll,containAll,addAll,retainAll,clear，toString等方法。

##### AbstractList

>AbstractList实现了List接口，继承了AbstractCollection类。但是，它没有实现List接口中的add,remove,set,get等关于index的操作。其实现了List中的Indexof,LastIndexOf,Sublist，addAll，equals等方法。自己创建了rangeCheckForAdd,removeRange,outofBoundMsg方法等。
>
>在Abstract类中首次实现了Iterator接口，其使用Itr实现了Iterator。又使用ListItr继承和实现了ListIterator，Itr类。随后的子类多使用它定义的ListItr内部类。

31 * i == (i << 5）- i,31是奇素数，且可用移位和减法代替。

```
public int hashCode() {
    int hashCode = 1;
    for (E e : this)
        hashCode = 31*hashCode + (e==null ? 0 : e.hashCode());
    return hashCode;
}
```

###### Itr内部类

```
其实现了Iterator接口，实现了next,remove,hasNext三个方法，并重写了checkForComodification方法。
```

###### ListItr内部类

```
实现和继承了Itr和ListIterator接口,其实现了previous,nextIndex,previousIndex,add,set这几个函数
```

##### ArrayList

>ArrayList继承AbstractList接口，serialize,cloned,randomAccess接口

构造函数：

指定初始容量，若使用无参构造方法，创建为DEFAULT_EMPTY_ELEMENT.

```
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    }
}
```

ArrayList添加的方法有如下：

将容量缩小到现有元素大小

```
public void trimToSize() {
    modCount++;
    if (size < elementData.length) {
        elementData = (size == 0)
          ? EMPTY_ELEMENTDATA
          : Arrays.copyOf(elementData, size);
    }
}
```

确保容量安全函数,通过calculate函数计算newcapacity，并更新minicapacity。若数组length不满足minicapacity则使用grow函数进行扩容。

```
private void ensureCapacityInternal(int minCapacity) {
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}

```

```
private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}
```

```
private static int calculateCapacity(Object[] elementData, int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    return minCapacity;
}
```

```
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

hugeCapacity用来获取一个极大数，其值为Integer.MAX_VALUE或MAX_ARRAY_SIZE=Integer.MAX_VALUE-8

其必须实现AbstractList接口的所有方法，增删改查如下：

add，检查下标，检查容量，移动数组位置，添加元素。

remove，检查下标，移动数组位置，将空位置null方便gc。

set,get流程较少，使用rangeCheck检查下标后，即可取值。

##### Queue

```
继承自collection接口，Queue只定义了五个方法，分别为add,remove,element,offer,poll
```

remove获取并移除队首元素，为空返回null。

offer获取并移除队首元素，为空抛出异常。

element用于获取队首元素，但不删除。为空返回null

poll获取队首元素但不移除，队列为空抛出异常

##### Deque

```
继承自Queue,基于remove,offer,element,poll添加了很多方法。同时定义了size,peek,push,pop,contain,iterator,get*等方法。
```

##### AbstractSequentialList

```
继承自AbstractList，需要自己实现set,get,add,remove方法
```

```
#获取从index处起始的iterator，取出oldValue返回并set新值。
public E set(int index, E element) {
        try {
            ListIterator<E> e = listIterator(index);
            E oldVal = e.next();
            e.set(element);
            return oldVal;
        } catch (NoSuchElementException exc) {
            throw new IndexOutOfBoundsException("Index: "+index);
        }
    }
```

```
get函数：使用AbstractList实现的listIterator迭代器取得index处的值。
remove调用iterator的remove移除index处的值
add调用iterator的add函数添加值
```



##### LinkedList

>LinkedList继承和实现了AbstractSequentialList,Serializable,Cloneable,Deque

LinkedList相比于ArrayList:

```
都继承了AbstractList，所以都支持index相关操作。都实现了了Serializable和cloned接口,并实现了Clone，readObject,writeObject方法。但其实现了了Deque接口，故实现了peek,offer,poll,element等方法。而ArrayList接口实现了RandomAccess接口，该接口无实际功能是一个标记接口，被此接口标记的List进行binarySearch时，使用普通的for循环，否则使用迭代器进行循环。
```

##### Set

>继承了Collection，和List及Queue的区别主要为,其元素值不可重复。主要有add,remove,clear,iterator,equals,hahsCode系列的函数。特有default spliterator()，

default spliterator():与Stream Api一起使用，用于并行处理。需要自行实现其内容。暂时不细看。

##### AbstractSet

>实现了Set接口,实现了三个方法，hashCode,equals,removeAll。

##### HashSet

>继承了AbstractSet,实现了iterator,add,remove,clear等参数。hashset使用onlyIfAbsent替换equals相等的元素，保证值的唯一性。

HashSet会调用HashMap进行存储，HashMap的putValue参数构造如下.

```
final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict)  public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }
hash由输入的key计算得来，key就是set.put(e)插入的e值。value是一个static final变量，内容为Object。onlyIfAbsent由map.put设定，其总为false，也就是若插入equals相等的元素，即覆盖它。
```

##### Vector

>继承和实现了AbstractList，RandomAccess,Cloneable,Serialiable.Vector操作是线程安全的。



### Map

>由上而下为Map接口，abstractMap和HashTable，HashMap。

##### Map接口

map中的内部接口Entry。静态内部类只可调用类成员，非静态内部类可以调用所有。但是由于内部接口无法实例化，故只能调用类成员，内部接口默认是静态接口，无需指定static关键字。

###### 内部接口Entry

>其定义了getkey,getvalue,setvalue等。用于Set<Entry<k,v>> entrySet进行迭代访问。

##### AbstractMap

>AbstractMap实现了Map接口的containValue,containKey,size,isEmpty,get,putAll,remove,values,clear,keyset,entrySet。还有hashcode,equals,toString等基础方法。但是需要其子类自己实现put等方法。

##### HashMap

>继承和实现AbstractMap,Clone,Serializable。

###### 内部类Node

>其为具有4个属性的静态类，next指向下一个对象，依次相连。
>
>```
>final int hash;
>final K key;
>V value;
>Node<K,V> next;
>```

其有三个函数，hashCode,equals,setValue。

```
transient表示此属性不进行序列化，只可存活在内存中。无法序列化进行存储或传输。
```

调用旋转函数调整平衡二叉树结构的流程：put/remove,treeifyBin,treeify,balanceInsertion,RotateLeft,RotateRight

balance*函数是调整的具体内容。

##### TreeMap

>实现NavigableMap->SortedMap->Map。TreeMap使用红黑树实现，SortedMap指定了部分函数，TreeMap是有序的结构。

###### 关于红黑树

>红黑数定义如下：
>
>每个结点为红色或黑色，根和叶子总是为黑色。
>
>相邻结点不可以都为红色
>
>从任意节点到其所有叶子结点的路径上红色结点数目都相同。

当删除或添加后导致颜色不符合规则时，进行recolor或rotaion。

插入结点

```
1插入x，将x标记为红色。若其为根，标记为黑色。
2若x的父节点是红色
	2.1若父节点的兄弟结点为红色，则将父节点和其兄弟结点标记为黑色，将祖父标记为红色。并让x与祖父颜色相同。
	2.2若父亲的兄弟结点为黑色，则进行rotation操作。
```

rotation

```
rotation操作分为4种，分别为左左，左右，右右，右左。
即x在祖父的左边但在父亲的右边，即为左右。其他相同道理。
```



### String

##### String类

equals

```
#String属性有hash和value值。value值实际上是字符数组，用来存储内容。hash为重写的方法，用于equals和Mapkey使用。
public String(String original) {
    this.value = original.value;
    this.hash = original.hash;
}
```

```
#String重写了equals()，流程为先比较是否为同一栈对象，如果是同一栈对象，那么肯定内容也相等了，返回true。否则，字符串再比较长度，进而逐个比较字符数组内容，如果内容全部一样，则true。equals本质是在比较两个字符串内容是否相等。
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

##### StringBuilder类

>继承AbstractStringBuilder,定义了value,count等属性，和append,容量相关,delete,deleteCharAt,insert,reverse等属性.

其线程不安全，使用一个可变字符数组。

##### StringBuffer类

>线程安全，在StringBuilder的基础上添加了synchronized。

#### 正则表达式

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

# lang

```
#判断该类是否为某个类的父类
isAssignableFrom()
#判断该示例是否为某个类的子类的实例
instanceof()
```

##### Integer

```
intValue()是把Integer对象类型变成int的基础数据类型；
parseInt()是把String 变成int的基础数据类型；
Valueof()是把String 转化成Integer对象类型；（现在JDK版本支持自动装箱拆箱了。）
```



##### Clone

###### 深拷贝与浅拷贝

基本类型存储在栈中
引用类型将引用放在栈，实际值存储在堆中，栈中的引用指向堆中存放的数据。
​浅拷贝就是，对于基础类型复制其值，而引用类型只复制引用，所以和原对象指向相同的值。如果拷贝之后，更改原对象将会影响拷贝的值。
深拷贝则是无论引用类型还是基础类型，都复制独立的一份，两个对象不会互相影响。
**如何实现深拷贝**
对于每个引用类型，让其继承Cloneable接口，并重写其clone方法。手动调用引用属性的clone方法创建新堆对象，然后传递给拷贝对象的属性。

```
#引用拷贝,创建一个新的栈对象指向相同的堆对象。
Stu s2=s1;
#对象拷贝,创建新的栈对象指向新的堆对象。对象拷贝包括浅拷贝和深拷贝。
Stu s2=(Stu)s1.clone();
#浅拷贝，虽然创建新的堆对象，但是对象所包含的引用属性不会真的拷贝。而是和原来的对象共享相同的引用属性。
深拷贝，创建新的堆对象，并且其包含的基础类型和引用类型的属性都独立复制一份。
```



##### Object

###### 创建对象5种方式

1.new 2.用类的newinstance 3用constructor的newinstance 4clone 5反序列化

###### 序列化

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

##### Reflect

>可以分析类的能力，动态的操作java代码称为反射。多用于开发供他人使用的构建工具。

###### Class

>关系:继承和实现了Serializable,GenericDeclaration,Type,AnnotatedElement。该类描述了一个类的信息，如属性，方法。



取反射类
1.Class stu=Class.forName("com.weitao.domain.Student);  由字符串常量取得
​2.Class s=stu.getClass();             由类的实列取得
3.Class s=Student.class;            由类对象取得

获得属性
getFidlds(),getField(String name)                        可以取得该类所有方法，但无法取得父类方法
getDeclaredFields,getDeclaredField(String name)   

获得方法
getMethods(),getMethod(String name,Class Class<?>..paramTypes)         可以取得该类所有方法，但无法取得父类方法
getDeclaredMethods(),getMethod(String name,Class Class<?>..paramTypes) 无法取得静态方法，但能父类方法

获得构造方法
getConstructors(),getConstructor(Class<?>..paramsTypes)
​getDeclaredConstructors(),getDeclaredConstructor(Class<?>..paramsTypes)

获得其他
getName();
newInstance();
getInterFaces();
getPackage();​

实例化对象
使用获取到的反射类s.
​1Student stu=(Student)s.newInstance();
​2Constructor<Student>  constructor=s.getConstructor(String.class);
Student stu=constructor.newInstance("李四");



###### equals

```
equals()
public boolean equals(Object obj) {
              return (this == obj);
           }
```

在java中==是比较两个引用对象的引用或比较基础类型的值。
此处即比较对象的地址。
重写equals的要求:
1自反性 2对称性 3传递性 4一致性 5对于任何非x，x.equals（null）值为null​
参考String类的equals
重写equals必须重写hashcode()，因为必须保证值相同的对象必须有相同的hashcode。若两者equals结果为true，hashcode一定相同。若hashcode不同，equals一定为false。hashcode相同时，equals不一定为true。

当map插入时，先比较hashcode，确定存储位置。再使用equals与存储的值比较，如果为true则不存储该值，为false就插入到散列结构中。散列结构一般为二叉树和红黑树。

如果重写equals但不重写hashcode，会出现equals为true，但hashcode不一样的情况。

##### Thread

###### 线程与锁

**进程和线程**
进程：并发执行的程序在执行过程中分配计算机资源的基本单位。
线程：是进程的一个执行单元，是比进程更小的基本单位。
一个程序至少一个进程，一个进程至少一个线程。
​​区别：进程的地址空间和资源是独立的，一个崩溃互不影响。
但线程开销更小。
应用场景：用户接入tomcat，tomcat分配给新线程，再调用servlet。
​​                         后台备份，前台不断询问后端进度。

**线程的实现**
继承Thread,实现Runnable,重写run方法。

###### 线程的状态



-new:对象被创建后状态

-runnable:对象start()后，等待cpu资源

-running:获取到cpu，执行线程，只能由runnable转换而来

-blocking:放弃cpu，停止

  等待阻塞

  调用wait(),失去锁，进入等待池。等待自己被唤醒。

  同步阻塞

  当线程申请锁失败，进入锁池，等待锁。

 其他阻塞

   	通过调用join()，sleep()不会释放锁,IO请求。当以上操作结束或完成，进入就绪状态。
   	
   		多线程实现同步方法

sychronized可以修饰方法，代码块。
ReentrantLock:阻塞所，可重入，操作由程序员编写。可实现非公平锁。
automaticInteger乐观锁，如原子变量就是乐观锁的应用。
wait(),notify(),sleep()：wait使一个线程处于等待状态，释放所有lock。sleep保留锁。
3(易变的):确保每次都重新取值，而不是使用寄存器的值。

###### 线程池

**ThreadPool类型**

ThreadPoolExecutor, ScheduledThreadPoolExecutor通常由工厂类Executors创建。
​Executors可以创建三种类型的ThreadPoolExecutor:
SingleThreadPool,FixedThreadPool和CachedThreadPool。
​Executors可以创建2种类型的ScheduledThreadPoolExecutor：
​SingleScheduledThreadPoo和ScheduledThreadPool。

**ThreadPoolExecutor**的重要参数
corePoolSize:核心线程数量
​ maximumPoolSize:最大线程数量 
​workQueue:等待队列，当线程数量​​大于corePoolSize时，任务封装成worker放入队列。 keepAliveTime:当线程数超过corePoolSize时，超过此时间空闲，则回收。
timeUnit:时间单位
​ threadFactory:指定创建的线程池类型
handler:拒绝策略

**等待阻塞队列**

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

##### Annotation



# IO

>![img](http://uploadfiles.nowcoder.com/images/20150328/138512_1427527478646_1.png)

```
IO流用于向内存写入数据或从内存读出数据。
IO流有四个顶级抽象父类，关于java的框架都基于这些类进行IO流的扩展。
这四个类分别是字节流:InputStream,OutputStream,字符流:Reader，Writer。
对于压缩文件的输入输出流，从磁盘等待压缩创建的输入流按序读取数据到内存，再从内存中将数据写入压缩文件磁盘中，完成一次压缩。
BufferedReader和BufferWriter使用了自定义的缓冲区，一次读取8192字节，减少内存读写磁盘次数从而提升速度，和减少mysql网络请求次数原理相同。每次IO或网络请求会耗费时间建立连接、维护协议信息、磁盘寻址和读写
```

```
按照流是否直接与特定的地方（如磁盘、内存、设备等）相连，分为节点流和处理流两类。

节点流：可以从或向一个特定的地方（节点）读写数据。如FileReader.
处理流：是对一个已存在的流的连接和封装，通过所封装的流的功能调用实现数据读写。如BufferedReader.处理流的构造方法总是要带一个其他的流对象做参数。一个流对象经过其他流的多次包装，称为流的链接。
JAVA常用的节点流：

文 件 FileInputStream FileOutputStrean FileReader FileWriter 文件进行处理的节点流。
字符串 StringReader StringWriter 对字符串进行处理的节点流。
数 组 ByteArrayInputStream ByteArrayOutputStreamCharArrayReader CharArrayWriter 对数组进行处理的节点流（对应的不再是文件，而是内存中的一个数组）。
管 道 PipedInputStream PipedOutputStream PipedReaderPipedWriter对管道进行处理的节点流。
常用处理流（关闭处理流使用关闭里面的节点流）

缓冲流：BufferedInputStrean BufferedOutputStream BufferedReader BufferedWriter  增加缓冲功能，避免频繁读写硬盘。
转换流：InputStreamReader OutputStreamReader 实现字节流和字符流之间的转换。
数据流 DataInputStream DataOutputStream  等-提供将基础数据类型写入到文件中，或者读取出来.
流的关闭顺序
一般情况下是：先打开的后关闭，后打开的先关闭
另一种情况：看依赖关系，如果流a依赖流b，应该先关闭流a，再关闭流b。例如，处理流a依赖节点流b，应该先关闭处理流a，再关闭节点流b
可以只关闭处理流，不用关闭节点流。处理流关闭的时候，会调用其处理的节点流的关闭方法。
```



##### File

>用来描述文件并进行基础性操作的类，方法有renameTo,delete(),exists,isFile,isDirectory,getName,getPath



##### InputStreamReader类

>他是字节流到字符流的桥梁，读取字节并用特定字符集节码，字符集默认使用平台字符集。要保证效率使用BufferReader。其定义的属性有StreamDecoder，方法有getEncode,read,ready,close。

##### BufferedReader

>BufferedReader继承自Reader，提供高效字符读取,其定义了Reader,char,nChars,nextChar,ensureOpen(),read(),readline()。缓冲区的大小可以指定，否则使用默认大小。大多数情况下默认大小就够用的。 每次Reader的读取请求都会产生相应的对字符或字节流的读取请求，所以最好用BufferedReader包装那些read操作影响效率的Reader，比如FileReader和InputStreamReader。





# NET



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



# NIO



# 待整理

### 转义字符串

```
kafka乱码
来源此处中文乱码：LOC-['u7231u5c14u5170', 'u7f8eu56fd']
python的producer加入,json.dump(result,ensure_ascii=false)
java的consumer用utf解码，并用StringEscapeUtil解码字符串.
```

```
\t \n 分别代表缩进与空格，添加\进行转义。
区分\t \n到底代表\t \n还是代表转义后的缩进与空格，取决于该字符串是否进行转义。在python中，在字符串前添加r前缀，表示此字符串不进行字符串转义。
```



### 类型

| 类型       | byte | short | int  | long | float | double | char | boolean |
| ---------- | ---- | ----- | ---- | ---- | ----- | ------ | ---- | ------- |
| 长度(字节) | 1    | 2     | 4    | 8    | 4     | 8      | 4    | 8       |

### 路径拼接

```
#使用Paths.get拼接，或使用separatorChar。separatorChar可以实现不同平台的分隔符。
Path currentRelativePath= Paths.get("");
Path currentDir=currentRelativePath.toAbsolutePath();
System.out.println(currentDir);

String filename="data"+ File.separatorChar+"test.log";
Path filepath=currentDir.resolve(filename);
System.out.println(filepath);
```



##### 路径问题

```
new File("").getAbsolutePath（）得到jar包放置的目录。使用idea运行得到项目根目录。

打包后无src->main等路径，如果访问jar包内文件，要通过如下方式访问。
以main和resources为合并的根目录，再加上META-INF文件夹组成根目录。输入相对路径访问即可。
如果访问外部，通过绝对路径访问。
#访问jar包内
this.getClass().getResource("/library.properties").getPath();
this.getClass().getClassLoader().getResource()；
this.getClass().getClassLoader().getResourceAsStream("logback.xml");
```

##### 自定义外部配置文件

```
一般使用源码发布，所以不使用所谓的外部配置文件。
1使用类加载器加载配置文件，可以加载jar内部文件。
2使用file直接读取jar包外部文件
3使用Properties类保存属性集
4使用ResourceBundle
```



### 安装

```
下载压缩包，配置环境变量
安装和管理：https://blog.csdn.net/u012707739/article/details/78489833
```



# 刷题的遗忘的点

##### 异常类

```
JAVA异常类,Throwable,Error,Exception，RuntimeException,IOException的继承关系
```

##### 访问修饰符

```

```



# JVM理论知识

## 自动内存管理

### java内存区域与内存溢出异常

#### 运行时数据区域

>jvm将内存区域划分为若干个不同的数据区域，这些区域有各自的用途，以及创建和销毁的时间。有些随着虚拟机进程的启动一致存在，有些区域则是依赖用户线程的启动和结束而建立和销毁。
>
>包括以下几个运行时数据区域：VM Stack、Native Method Stack、Program Counter Register、Method Area、Heap。

##### Program Counter Register

>程序计数器可看作当前线程所执行的字节码的行号指示器，通过程序计数器选择下一条需要执行的字节码指令，程序的分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖这个计数器完成。
>
>线程恢复：由于一个cpu核心同时只会执行一条线程中的指令，因此为了线程切换后能回到正确的位置，每条线程都需要一个独立的程序计数器，各条线程之间计数器互不影响。

##### Java Virtual Machine Stack

>与程序计数器一样，vm stack也是线程私有的，它的生命周期和线程相同。虚拟机栈是java方法执行的线程内存模型：每个方法被执行的时候，java虚拟机都会创建一个栈帧用于存储局部变量表、操作数帧、动态链接、方法出口等信息。每个方法被调用直至执行完毕的过程，对应着栈帧在虚拟机栈中从入栈到出栈的过程

##### Native method Stack

>本地方法栈与虚拟机栈作用类似，区别在于java虚拟机栈为java方法（字节码）服务，而本地方法栈则是为虚拟机使用到的本地方法服务。

##### Java 堆

>java堆是虚拟机所管理的内存中最大的一块，是被所有线程共享的一块区域。此内存区域唯一目的就是存放对象实例，java世界里几乎所有对象实例都在这里分配内存。
>
>java堆是垃圾收集器管理的内存区域，因此也被称为GC堆。

##### Method area

>方法区与java堆一样，是多个线程的共享区域，它用于存储已经被虚拟机加载的类型信息、常量、静态变量、即时编译后的代码缓存等数据。

>运行时常量池
>
>运行时常量池是方法区的一部分，class除了有类的版本、字段、方法等描述信息，还有一项常量池表Constant Pool Table，用于存放编译器生成的各种字面量与符号引用，这部分内容在类加载后放入运行时常量池。例如你在代码中写了一个String str=“hello”，这个“hello”就是一个字面量，需要放入此处。

##### 直接内存

>通过Native函数库直接分配堆外的内存，然后通过java堆内的DirectByteBuffer对象为这块内存的引用进行操作，这样避免了java堆和native堆来回复制数据。但是其不受java堆大小的控制，也可能超过机器的物理内存，导致OutOfMemoryError异常。

### 垃圾收集器和内存分配策略

>那些内存需要回收、什么时候回收、如何回收
>
>由于程序计数器、虚拟机栈、本地方法栈三个区域随线程而成，随线程而灭，栈中的栈帧随着方法的进入和退出而变化。每个栈帧中分配多少内存基本上在类结构确定下来时就已知，所以这几个区域的内分分配与回收具有确定性，当方法结束或线程结束时自然就回收了。
>
>但是Java堆和方法区这两个区域有显著的不确定性，一个接口的多个实现类内存要求不一样，一个方法的不同分支需要的内存也不一样，也就是说只有在运行时才能确定到底需要多少空间，这部分内存的分配和回收是动态的。垃圾收集器主要关系这一部分内存如何分配和回收。

#### 那些对象需要回收？

>为了确定那些对象死亡了，需要回收，可以通过引用计数法、可达性分析。

##### 引用计数法

>当有一个地方引用时，计数器值加1；当引用失效时，计数器值减1.任何时刻计数器值为零的对象不可能再被使用。
>
>优点：逻辑简单、判定效率高。
>
>缺点：需要处理很多额外情况，如对象之间循环引用。

##### 可达性分析

>通过GC Roots的根节点作为起始节点集，从这些结点往下搜索，搜索所走过的路径称为引用链，如果从某个对象到GC Roots之间没有任何路径，引用链，那么不可达，此对象可以回收。
>
>即根据GC Roots对象，遍历出他们（引用）关联的所有对象（Heap中的对象），没有遍历到的就是非存活对象。
>
>可作为GC Roots的对象包括以下几种：
>
>1虚拟机栈中的引用的对象，如局部变量、临时变量
>
>2方法区中的字符串常量池里的引用
>
>3本地方法栈引用的对象
>
>4jvm内部引用，如常驻的异常对象。

##### 引用分类

>强引用，最传统的定义，即代码中普遍存在的引用赋值
>
>软引用，描述一些还有用，但是非必须的对象。
>
>弱引用，非必须
>
>虚引用，最弱的引用关系

##### 两次标记

>当发现不可达后，进行一次标记。
>
>然后判断是否需要finalize方法，如果没有，就是直接回收。否则，让该对象放入一个F-Queue队列，等待执行finalize方法后再次检测，如果还是不可达，则回收。

##### 回收方法区

>主要回收废弃的常量和不再使用的类型
>
>废弃的常量和堆的回收类似，但是回收不再使用的类型比较苛刻。
>
>1该类所有实例回收了2该类类加载器回收了3该类java.lang.Class对象没有被引用。

#### 什么时候回收？

>Eden区满了，触发young gc
>
>如果整个堆在young gc时，发现晋升到老年代的剩余空间过大，就直接full gc。
>
>手动调用system.gc（）

#### 如何回收？

>分代收集理论，建立在两个假说上：
>
>1弱分代假说：绝大多数对象是朝生夕灭的
>
>2强分代假说：熬过越多次垃圾回收过程的对象越难以消亡
>
>所以其原则为，将内存回收的java堆划分为不同区域，根据其年龄分配到不同区域存储。对于不同区域采取不同的策略，提高回收效率。
>
>实际上分代假说需要面临很多问题，比如跨代引用问题，这里引入第三条法则
>
>3跨代引用假说：跨代引用相对于同代引用来说占比较少。

##### heap的内存划分

>为了实现标记-复制算法、标记-清除算法、标记-整理算法。
>
>堆被划分成两个不同的区域：新生代 ( Young )、老年代 ( Old )。新生代 ( Young ) 又被划分为三个区域：Eden、From Survivor、To Survivor。
>这样划分的目的是为了使 JVM 能够更好的管理堆内存中的对象，包括内存的分配以及回收。
>
>默认的，新生代 ( Young ) 与老年代 ( Old ) 的比例的值为 1:2 ( 该值可以通过参数 –XX:NewRatio 来指定 )，即：新生代 ( Young ) = 1/3 的堆空间大小。
>老年代 ( Old ) = 2/3 的堆空间大小。其中，新生代 ( Young ) 被细分为 Eden 和 两个 Survivor 区域，这两个 Survivor 区域分别被命名为 from 和 to，以示区分。
>默认的，Eden : from : to = 8 : 1 : 1 ( 可以通过参数 –XX:SurvivorRatio 来设定 )，即： Eden = 8/10 的新生代空间大小，from = to = 1/10 的新生代空间大小。
>JVM 每次只会使用 Eden 和其中的一块 Survivor 区域来为对象服务，所以无论什么时候，总是有一块 Survivor 区域是空闲着的。
>因此，新生代实际可用的内存空间为 9/10 ( 即90% )的新生代空间。

##### 1.标记-清除算法

标记：标记出所有要回收的对象
清除：回收被标记对象
缺点：效率低，这两个操作效率都不高
​​空间问题，形成大量不连续碎片空间。当以后需要分配较大的对象时，无法找到足够大的连续空间，触发gc。

##### 2.标记-复制算法

内存分为相等的两块，需要回收时，把存活的对象拷贝到另一半，清除当前的整个块，以保证空间连续性。
缺点：代价高。
现在一般分为三块，Eden和两块较小的Survivor。当回收时，将存活对象放到另一个Survivor中，清除当前Survivor和Eden.

##### 3.标记-整理算法

标记：标记所有要回收的对象。
整理:将存活对象向一段移动，然后直接清理到边界以外的内存空间

##### 4.分代收集算法

分代：新生代，老生代
新生代：存活少，使标记-复制算法。
老生代：存活多，使用标记清除或标记整理算法。

#### 代码如何产生不可达变量

非线程对象不被指向或超出作用域
线程对象线程未启动或停止

###### 改变对象引用，置为null或指向其他对象。

```
Object x=new Object();//object1 
Object y=new Object();//object2 
x=y;//object1 变为垃圾 
x=y=null;//object2 变为垃圾
```

###### 超出作用域

```
if(i==0){ 
      Object x=new Object();//object1 
   }//括号结束后object1将无法被引用，变为垃圾 
```

###### 类嵌套导致未完全释放

```
class A{ 
      A a; 
   } 
   A x= new A();//分配一个空间 
   x.a= new A();//又分配了一个空间 
   x=null;//将会产生两个垃圾 
```

###### 线程中的垃圾

```
class A implements Runnable{   
     void run(){ 
       //.... 
     } 
   } 
   //main 
   A x=new A();//object1 
   x.start(); 
   x=null;//等线程执行完后object1才被认定为垃圾 
```

## 虚拟机执行子系统

### 虚拟机类加载机制

>一个类从被加载到虚拟机内存开始，到卸载内存为止，它经历了加载、验证、准备、解析、初始化、使用和卸载七个阶段，简称为加载、连接、初始化、使用、卸载。

#### 触发初始化的时机

>1new实例对象，读取或设置一个静态字段
>
>2使用java.lang.reflect包的方法进行反射调用的时候，如果没有进行过初始化，则需要先初始化
>
>3当初始化类的时候，其父类还没有初始化
>
>4挡虚拟机启动时，需要指定执行main方法的那个类

#### 双亲委派模型（Parent Delegation Model）

##### **工作过程**

>1当前ClassLoader首先从自己已经加载的类中查询此类是否已加载。若已经加载就返回加载的类。
>每个类加载器都有自己的加载缓存，当一个类被加载以后就会放入缓存。
>2当前classloader的缓存中没有找到时，委托父类加载器去加载，父类加载器采用同样的策略，直到找到或到达bootstrap classloader.
>3.当所有的父类加载器都没有加载时，再由当前的类加载器加载，并放入自己的缓存。

##### **优点**

>为了安全性，避免用户的类动态替换java的核心类，比如String。.
>
>同时避免了重复加载，因为相同的class文件被不同classloader加载是不同的两个类，相互转型会抛出classCaseException。

##### **常用类加载器**

>Bootstrap class loader：所有类加载器的父类，当运行jvm时，她负责核心库的加载，如java.lang.*等。例如java.lang.Object就是由根类加载器加载的。此加载器不是由java编写的，而是c/c++写的。
>Extension class loader：这个加载器加载除了基本API之外的拓展类。
>AppClassLoader:加载应用程序和程序员自定义的类。
>用户也可以自定义自己的类加载器，java提供了java.long.classloader.

## 相关链接

>深入理解jvm虚拟机

# JAVA理论知识

# JAVA用法

## Lambda表达式

>函数是指一个集合到另一个集合的映射关系，在流处理中是指将输入流转换为正确的输出流。在java中函数式编程主要用在匿名函数和集合的stream操作.
>
>lambda只可以访问final标记的局部变量，这说明lambda不剋改变定义在域外的局部变量，否则编译错误。
>
>lambda表达式的局部变量不用声明为final，但是必须不可被后边的代码修改。

```
#语法格式如下
(parameters)->expression
(parameters)->{statements;}
```

# 并发编程基础

## 线程状态

>java中的线程状态分为6种。
>
>1初始NEW:新创建一个对象，还没有start（）
>
>2RUNNABLE:java线程中将ready和running统称为可运行，这表示这个线程具备运行的条件，但是可能在ready状态，也可能在running状态。
>
>3BLOCKED：表示线程阻塞于锁
>
>4WAITING：进入此状态的线程需要等待其他线程做出一些特定动作
>
>5TIMED_WAITING：不同于WAITING，它可以在指定时间后自行返回。
>
>6TERMINATED：表示线程执行完毕。

### NEW

>当一个线程被使用new Thread（）创建时，其就处于NEW状态，等待使用start（）方法启动

### RUNNABLE

>当调用start（）方法后，线程计入RUNNABLE状态，等待CPU分配时间片和资源。

### BLOCKED

>当一个线程试图获取一个内部的对象锁，即synchronized锁，而锁被其他线程占用时，进入阻塞状态。

### WAITING

>当线程等待另一个线程通知调度器出现一个条件时，这个线程会进入等待状态。例如调用object.wait(),object.join(),以及等待java.util.concurrent中的Lock和Condition时，会出现这种状况。

### TIMED_WAITING

>调用带有超时参数的方法，在等待一定时间后自动返回。例如object.wait(timeout),object.join(timeout),Lock.tryLock(timeout),Condition.await(timeout)，Thread.sleep(timeout)等方法。

### TERMINATED

>run方法自动正常退出，或者因为未捕获的异常终止了run方法。

## 中断线程

>使用interupt方法时，会将线程设置为中断状态，中断不意味着终止，如何响应中断状态可以由程序定义。可以使用isInterrupted（）检查线程的中断状态，并采取相应的措施。

## 守护线程

>调用t.setDaemon(true)将线程设置为守护线程，守护线程为其他线程提供服务。当只剩守护线程时，程序会直接退出。

## 未捕获异常的处理器

>在平时编写程序时，由于编写错误程序也会停止运行，并报异常，然后我们根据异常情况修复代码。这些未捕获的异常是由谁处理的呢，默认情况下会调用线程组的UncaughtExceptionHandler接口，他会将Throwable的栈轨迹输出到控制台。
>
>我们可以通过setUncaughtExceptionHandler指定自己的异常处理器，可以在处理器中使用日子API将异常存储到日志文件。

## 锁对象

>当多线程并发访问对象时，会导致数据的不一致性。java提供了sychronized关键字、ReentrantLocak类

```java
#ReentrantLock方法可以保护代码块临界区，它确保任何时候只有一个线程进入临界区。当一个线程成功调用Lock方法后，其他线程只能等待。
#可以通过传入true，false指定是否采用公平锁。公平锁是指让每个线程都有机会执行，其倾向于等待过久的线程。
var bankLock=new ReentrantLock(false);
bankLock.lock();
try
{}
finally{
    bank.unlock();
}
```

## sychronized关键字

>Lock和condition可以允许灵活的控制锁定，但大多数情况下，我们只需要确保互斥的访问即可。java语言内置了对象的内部锁。如果一个方法带有sychronized关键字，那么对象的锁将保护整个方法。

```java
#public sychronized void method()
{
	method body
}
#sychronized 代码块
sychronized {
    //do something
}
```

## 条件对象

>有时候，线程获取到了资源，并进入临界区。却发现只有满足某个条件后其才能执行，例如转账时自己的账户金额必须足够支付。这时，可以使用一个条件对象进行管理，其目的是管理那些已经获得了锁但是却无法做有用工作的线程。这种情况，使用if语句检查是错误的，因为线程可能在if之后中断，然后账户金额发生变动。如果使用ReentrantLock加锁，当账户金额不够时，会导致其他线程无法访问账户对象，更无法增加账户金额，形成死锁。使用条件对象可以让当前线程进入await状态，并允许其他线程访问对象变更金额条件，当金额变动后调用singnalAll（）唤醒所有被await的线程尝试工作。

```
var bankLock=new ReentrantLock(false);
private Condition sufficientFunds=bankLock.newCondition()
bankLock.lock()
try{
whiel(accounts[from]<amount)
    sufficientFunds.await();
sufficientFunds.singnalAll()
}finally
{
  bankLock.unlock()
}
```

## volatile字段

>有时如果只是为了读写一两个实例字段而使用同步，所带来的开销过大，可以使用volatile可以确保读取的值是最新的值。例如, return done，如果使用volatile修饰，则可以保证done得到的值是正确的a值。但是，注意其无法确保原子性。如done=！done就无法保证读取、反转、写入时不被中断。

