[TOC]
##### HashMap

>继承和实现AbstractMap,Clone,Serializable。
>
>默认capacity为16
>
>loadFactor加载因子，默认是0.75
>
>threshold阈值。阈值=容量*加载因子。默认12，当元素数量超过阈值，触发扩容
>
>https://blog.csdn.net/qq_45655489/article/details/119563733

###### Table容量
>默认容量：
>默认capacity为16；loadFactor加载因子，默认是0.75。

>扩容机制：
>HashMap使用懒加载机制，首次put的时候，进行容量的设置。我们可以通过构造函数指定默认参数，若没有指定则使用默认参数，即16和0.75。当进行put操作时，会检查是否size大于threshold阈值，若大于则需要扩容为原来尺寸的2倍。

###### resize方法

```
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    //容量超过最大值就不再扩容
    if (oldCap > 0) {
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        //没超过最大值扩容到原来的2倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        //扩容完成后，重新进行hash分配，写入数据
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

###### Table工作原理

> hashmap插入元素的步骤  put方法:
>1如果此hash没有放入值，则直接创建一个node，放入table
>
>2如果已经存在，按照如下步骤处理：
>
>​          1如果插入的hash值与已经存在的元素的hash值相等，且key值equals相等，将元素赋值给节点
>
>​          2如果遍历完红黑树节点没有发现符合条件的，并且是红黑树节点，则将值放入树中。
>          如果小于UNTREEIFY_THRESHOLD 会转换为链表。
>
>​          3如果遍历完链表节点没有发现符合条件的，并且是链表，则将值插入链表末尾。如果大于TREEIFY_THRESHOLD，尝试扩容为红黑树。

```
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    //table未初始化或者长度为0，进行扩容
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    //(n - 1) & hash 确定元素存放在哪个桶中，桶为空，
    //新生成结点放入桶中(此时，这个结点是放在数组中)
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    //桶中已经存在元素
    else {
        Node<K,V> e; K k;
        //key相等，hash相等
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            //将元素赋值给e
            e = p;
        //hash值不相等，即key不相等，并且该节点是红黑树节点
        else if (p instanceof TreeNode)
            //放入树中
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        //链表节点
        else {
            //在链表末尾插入节点
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    //节点数量达到阈值8，执行treeifyBin方法
                    //此方法会根据HashMap的数组来决定是否要转换为红黑树
                    //数组长度大于等于64才会转换为红黑树，否则只会扩容数组
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    //跳出循环
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        //桶中有key值和hash值于插入节点相等的节点
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            //覆盖以后返回旧值
            return oldValue;
        }
    }
    ++modCount;
    //插入完成后实际大小大于阈值，需要扩容
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}

```
>hash计算方式：
>hash根据每个类自己实现的hashCode函数计算，h=key.hashCode()^(h>>>16),主要是让hash高位参与运算，因为hash需要和table.length-1相与，但是length较小时，导致高位全是0，所以最终的index主要取决于length，导致容易发生hash碰撞。
>hash计算完成后需要映射到table的index，使用hash&(table.length-1)映射。
>本质上，hash mod table.length可以通过位运算实现，即 x mod 2^n=x & (1 << n - 1)，所以才要求容量是2的整数次幂，这样方便使用位运算减少计算量。


###### 关于红黑树

>红黑数定义如下：
>
>每个结点为红色或黑色，根和叶子总是为黑色。
>
>相邻结点不可以都为红色
>
>从任意节点到其所有叶子结点的路径黑色结点数目都相同。

当删除或添加后导致颜色不符合规则时，进行recolor或rotaion。

插入结点

```
1根据查找树规则确定插入位置
2 若key等于该节点，update值即可。否则，插入x，将x标记为红色。若其为根，标记为黑色。
3插入后的再平衡
  若x的父节点为黑色：直接插入
  若x的父节点是红色：
     若父节点的兄弟结点为红色，则将父节点和其兄弟结点标记为黑色，将祖父标记为红色。并让x与祖父颜色相同。再从祖父开始继续调整。
     若父亲的兄弟结点为黑色，则进行rotation，recolor操作，具体操作方式参考下边的rotation，recolor操作。
如何理解记忆，当x的父亲是黑色时，说明插入节点没有影响黑色节点数目，无需调整。只有当父亲为红色时，需要重新调整，因为红与红不能相邻。当叔叔节点是红色时，为了解决红红不相邻，可以将父亲改为黑色，但是这样黑色节点数增加了1，所以将祖父改为红色，兄弟改为黑色。当叔叔为黑色时，无法通过上述方式让两个分支黑色节点数相同，导致父亲分支比叔叔分支多一个黑色节点，所以要旋转。将父亲变黑，祖父变红，再向黑色节点少的一侧旋转，让父亲这个新的黑色节点当做根，从而让右侧和左侧都拥有一个黑色节点，让黑色节点数目相同。
当插入节点父亲为红色时，需要继续调整。当插入节点的叔叔节点为黑色时，recolor操作无法处理黑色节点均衡，需要旋转。
```

rotation，recolor

```
rotation，recolor操作分为4种情景，分别为左左，左右，右右，右左。
即x在祖父的左边，但在父亲的右边，即为左右。其他相同道理。
左左：将父亲变为黑色，祖父变为红色，将祖父右旋
左右：插入节点的父亲左旋，转换为左左情景
右右：将父亲变为黑色，祖父变为红色，将祖父左旋
右左：插入节点的父亲右旋，转换为右右情景
```

优点
>借助颜色保持二叉树的平衡

