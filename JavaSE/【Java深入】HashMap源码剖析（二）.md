###**重要方法源码（JDK8）剖析**

####**1.put方法（在映射中关联指定值与指定键）**

```
public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
```

注：HashMap的put方法是通过putVal方法来插入元素的

```
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // table未初始化或者长度为0，进行扩容
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // (n - 1) & hash 确定元素存放在哪个桶中，桶为空，新生成结点放入桶中(此时，这个结点是放在数组中)
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    // 桶中已经存在元素
    else {
        Node<K,V> e; K k;
        // 比较桶中第一个元素(数组中的结点)的hash值相等，key相等
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
                // 将第一个元素赋值给e，用e来记录
                e = p;
        // hash值不相等，即key不相等；为红黑树结点
        else if (p instanceof TreeNode)
            // 放入树中
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        // 为链表结点
        else {
            // 在链表最末插入结点
            for (int binCount = 0; ; ++binCount) {
                // 到达链表的尾部
                if ((e = p.next) == null) {
                    // 在尾部插入新结点
                    p.next = newNode(hash, key, value, null);
                    // 结点数量达到阈值，转化为红黑树
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    // 跳出循环
                    break;
                }
                // 判断链表中结点的key值与插入的元素的key值是否相等
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    // 相等，跳出循环
                    break;
                // 用于遍历桶中的链表，与前面的e = p.next组合，可以遍历链表
                p = e;
            }
        }
        // 表示在桶中找到key值、hash值与插入元素相等的结点
        if (e != null) { 
            // 记录e的value
            V oldValue = e.value;
            // onlyIfAbsent为false或者旧值为null
            if (!onlyIfAbsent || oldValue == null)
                //用新值替换旧值
                e.value = value;
            // 访问后回调
            afterNodeAccess(e);
            // 返回旧值
            return oldValue;
        }
    }
    // 结构性修改
    ++modCount;
    // 实际大小大于阈值则扩容
    if (++size > threshold)
        resize();
    // 插入后回调
    afterNodeInsertion(evict);
    return null;
}
```

####**put方法图解（对照源码理解）**

![这里写图片描述](http://img.blog.csdn.net/20170614211553447?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

put方法步骤：

①.判断键值对数组table[i]是否为空或为null，否则执行resize()进行扩容；

②.根据键值key计算hash值得到插入的数组索引i，如果table[i]==null，直接新建节点添加，转向⑥，如果table[i]不为空， 转向③；

③.判断 table[i]的首个元素是否和key一样，如果相同直接覆盖value，否则转向④，这里的相同指的是hashCode以及equals；

④.判断table[i] 是否为treeNode，即table[i] 是否是红黑树，如果是红黑树，则直接在树中插入键值对，否则转向⑤；

⑤.遍历table[i]，判断链表长度是否大于8，大于8的话把链表转换为红黑树，在红黑树中执行插入操作，否则进行链表的插入操作； 遍历过程中若发现key已经存在直接覆盖value即可；

⑥.插入成功后，判断实际存在的键值对数量size是否超多了最大容量threshold，如果超过，进行扩容。


####**2.resize方法（扩容机制）**

```
final Node<K,V>[] resize() {
    // 当前table保存
    Node<K,V>[] oldTab = table;
    // 保存table大小
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    // 保存当前阈值 
    int oldThr = threshold;
    int newCap, newThr = 0;
    // 之前table大小大于0
    if (oldCap > 0) {
        // 之前table大于最大容量
        if (oldCap >= MAXIMUM_CAPACITY) {
            // 阈值为最大整形
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // 容量翻倍，使用左移，效率更高
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
            oldCap >= DEFAULT_INITIAL_CAPACITY)
            // 阈值翻倍
            newThr = oldThr << 1; // double threshold
    }
    // 之前阈值大于0
    else if (oldThr > 0)
        newCap = oldThr;
    // oldCap = 0并且oldThr = 0，使用缺省值（如使用HashMap()构造函数，之后再插入一个元素会调用resize函数，会进入这一步）
    else {           
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    // 新阈值为0
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
    // 初始化table
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    // 之前的table已经初始化过
    if (oldTab != null) {
        // 复制元素，重新进行hash
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
                    // 将同一桶中的元素根据(e.hash & oldCap)是否为0进行分割，分成两个不同的链表，完成rehash
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

注：构造hash表时，如果不指明初始大小，默认大小为16（即Node数组大小16），如果Node[]数组中的元素达到（填充比*Node.length），就进行扩容，会伴随着一次重新hash分配，并且会遍历hash表中所有的元素，是非常耗时的。在编写程序中，要尽量避免resize。

在resize前和resize后的元素布局如下：
![这里写图片描述](http://img.blog.csdn.net/20170514100648928?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

####**3.hash方法（计算Key）**

JDK8源码：
```
static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
 }
```

我们可以和JDK7的源码做一下比较

```
static int hash(int h) {
    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
```

* 在JDK8中hash方法已经简化了，使用hash值的高位16位与低16只做一次异或操作，而不是四次，但原理是不变的。

* JDK8源码分析：源码中的key.hashCode()函数调用的是key键值类型自带的哈希函数，返回int型函数。

* hashCode（）源码如下：

```
public int hashCode() {
        int h = 0;
        Iterator<Entry<K,V>> i = entrySet().iterator();
        while (i.hasNext())
            h += i.next().hashCode();
        return h;
    }
```


* hash方法的传入参数从原先的int值的hashCode变成了对象，支持null的传入，所以null传进去得到0的hashCode也能当做普通值一般化处理了，之前的indefFor()方法消失 了，直接用(tab.length-1)&hash，所以看到这个，代表的就是数组的下角标。

* 我认为它的作用就是：高16bit不变，低16bit和高16bit做了一个异或。因为Java中对象的哈希值都32位整数，高位与低位异或一下能保证高低位都能参与到下标计算中，即使在table长度比较小的情况下，也能尽可能的避免碰撞。

hash图解：
![这里写图片描述](http://img.blog.csdn.net/20170514105226345?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

####**4.get方法（返回指定键所映射的值）**

```
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    // table已经初始化，长度大于0，根据hash寻找table中的项也不为空
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        // 桶中第一项(数组元素)相等
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        // 桶中不止一个结点
        if ((e = first.next) != null) {
            // 为红黑树结点
            if (first instanceof TreeNode)
                // 在红黑树中查找
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            // 否则，在链表中查找
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}

final TreeNode<K,V> getTreeNode(int h, Object k) {
        //找到红黑树的根节点并遍历红黑树
        return ((parent != null) ? root() : this).find(h, k, null);
}

//找到从根p开始的节点和给定的散列和键。kc参数在第一次使用比较键时缓存了comparableClassFor。
final TreeNode<K,V> find(int h, Object k, Class<?> kc) {
       TreeNode<K,V> p = this;
       do {
           int ph, dir; K pk;
           TreeNode<K,V> pl = p.left, pr = p.right, q;
           if ((ph = p.hash) > h)
               p = pl;
           else if (ph < h)
               p = pr;
           else if ((pk = p.key) == k || (k != null && k.equals(pk)))
               return p;
           else if (pl == null)
               p = pr;
           else if (pr == null)
               p = pl;
           else if ((kc != null ||
                     (kc = comparableClassFor(k)) != null) &&
                    (dir = compareComparables(kc, k, pk)) != 0)
               p = (dir < 0) ? pl : pr;
           else if ((q = pr.find(h, k, kc)) != null)
               return q;
           else
               p = pl;
       } while (p != null);
       return null;
}

```

注：通过hash值的比较，递归的去遍历红黑树，这里要提的是compareableClassFor(Class k)这个函数的作用，在某些时候如果红黑树节点的元素是 "class C implements Comparable<C>" 类型，则利用他们的compareTo()方法来比较大小，这里需要通过反射机制来check他们到底是不是属于同一个类,是不是具有可比较性.

####**5.treeifyBin方法（将容器中的node变为treeNode）**

```
final void treeifyBin(Node<K,V>[] tab, int hash) {  
        int n, index;   
    Node<K,V> e;  
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)  
            resize();  
        //Node e=tab[该hash对应的角标]，e就是这个角标下的第一个元素。  
        else if ((e = tab[index = (n - 1) & hash]) != null) {  
            TreeNode<K,V> hd = null, tl = null;  
            do {  
        //replacementTreeNode == new TreeNode(),就是包装了一个TreeNode对象  
                TreeNode<K,V> p = replacementTreeNode(e, null);  
                if (tl == null)  
            //遍历链表上的第一个元素的时候，t1==null，将p赋值给hd  
            //也就是先记录一下，方便后面的元素记录pre，next  
                    hd = p;  
                else {  
        //现在p是个tree了，pre记录上一个元素  
                    p.prev = tl;  
            //顺便把自己的引用在上一个元素上做记录  
                    tl.next = p;  
                }  
        //将当前操作的元素的引用传递给t1  
                tl = p;  
        //遍历整个链表，直到没有元素。  
            } while ((e = e.next) != null);  
            if ((tab[index] = hd) != null)  
        //遍历完了，再执行hd.treeify方法  
        //hd=p是在t1==null时执行，也就是只有在第一个元素的时候执行了一次  
        //所以hd代表的是这个树的根。  
                hd.treeify(tab);  
        }  
    }  
```

注：指定需要转化为树的是哪个数组，对应的哪个下角标所连出来的链表。


####**6.remove方法（移除指定键的映射关系）**

```
public V remove(Object key) {
    Node<K,V> e;
    return (e = removeNode(hash(key), key, null, false, true)) == null ? null : e.value;
}

final Node<K,V> removeNode(int hash, Object key, Object value, boolean matchValue, boolean movable) {
    Node<K,V>[] tab; Node<K,V> p; int n, index;
    if ((tab = table) != null && (n = tab.length) > 0 && (p = tab[index = (n - 1) & hash]) != null) {
        Node<K,V> node = null, e; K k; V v;
        // 直接命中
        if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
            node = p;
        else if ((e = p.next) != null) {
            // 红黑树中查找
            if (p instanceof TreeNode)
                node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
            else {
                // 链表中查找
                do {
                    if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k)))) {
                        node = e;
                        break;
                    }
                    p = e;
                } while ((e = e.next) != null);
            }
        }
        // 命中后删除
        if (node != null && (!matchValue || (v = node.value) == value || (value != null && value.equals(v)))) {
            if (node instanceof TreeNode)
                ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
            else if (node == p)
                tab[index] = node.next;  // 链表首元素删除
            else
                p.next = node.next;  //多元素链表节点删除
            ++modCount;
            --size;
            afterNodeRemoval(node);
            return node;
        }
    }
    return null;
}
```

####**7.小结 **

（1）为什么需要负载因子？

因为要减缓哈希冲突，例如：默认初始桶为16，或等到满16个元素才扩容，某些桶里可能就会有多个元素了。所以加载因子默认为0.75，也就是说大小为16的HashMap，扩容临界值threshold=0.75*16=12,到了第13个元素，就会扩容成32。

（2）什么时候减小负载因子？

在构造函数里，设定小一点的负载因子,比如0.5，甚至0.25。若是一个长期存在的Map,并且key不固定，那可以适当加大初始大小，同时减少加载因子，降低冲突的机率，也能减少寻址的时间。用空间来换时间，这时也是值得的。建议不要轻易修改，除非情况非常特殊。

（3）初始化时是否定义容量？

每次扩容都需要重创建桶数组、链表、数据转换等，扩容成本高，若初始化时能设置准确或预估出需要的容量，即使大一点，用空间来换时间，有时也是值得的。

（4）String型的Key设计优化？

如果无法保证无冲突而且能用==来对比，那就尽量短点，每个字符的equals都是需要花时间的。顺序型的Key,如：k1、k2、k3...k50,这种key的hashCode是数字递增，冲突的可能性很小。

（5）有必要指定HashMap的大小吗？

因为扩容是一个特别耗性能的操作，所以在使用HashMap的时候，最好估算map的大小，初始化的时候给一个大致的数值，从而避免map进行频繁的扩容。

（6）可以在多线程下使用HashMap吗？

因为HashMap是线程不安全的，所以不要在多线程的环境中同时操作HashMap，建议使用ConcurrentHashMap。

（7）频繁扩容会导致异常出现吗？

答：扩容会可能会带来死循环问题，因为Node的next可能不为空。
详情：[不正确地使用HashMap引发死循环及元素丢失](http://www.cnblogs.com/xrq730/p/5037299.html)


<br/>
<br/>
本人才疏学浅，若有错，请指出
谢谢！

参考链接：[Java 8系列之重新认识HashMap](https://tech.meituan.com/java-hashmap.html)
