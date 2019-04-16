##**一、HashMap的实现原理**

**1.引入**

毫无疑问，有很多人都用过HashMap，它的特点如下：

* HashMap最多只允许一条记录的键为null，允许多条记录的值为null，

* HashMap是非线程安全;

* HashMap储存的是键值对

**2.深入**

例1：HashMap的get()方法的工作原理？

答：

* HashMap通过hash算法，通过put和get存储和获取对象。

* put对象时，我们将K/V传给put方法时，它调用hashCode计算hash从而得到bucket位置，进一步存储，HashMap会根据当前bucket的占用情况自动调整容量(超过Load Factor则resize为原来的2倍)。如果发生碰撞的时候，Hashmap通过链表将产生碰撞冲突的元素组织起来。如果一个bucket中碰撞冲突的元素超过某个限制(默认是8)，则使用红黑树来替换链表，从而提高速度。

* get对象时，我们将K传给get，它调用hashCode计算hash从而得到bucket位置，并进一步调用equals()方法确定键值对。



例2：当两个对象的hashcode相同会发生什么？

答：若两个对象的hashcode相同，但是它们可能并不相等。但它们的bucket位置相同，会发生‘碰撞’，因为HashMap使用LinkedList存储对象，这个Entry(包含有键值对的Map.Entry对象)会存储在LinkedList中。

例3：当两个键的hashcode相同，如何获取值对象？

答：当我们调用get()方法，HashMap会使用键对象的hashcode找到bucket位置，然后会调用keys.equals()方法去找到LinkedList中正确的节点，最终找到要找的值对象。

例4：如果HashMap的大小超过了负载因子(load factor)定义的容量，怎么办？

答：默认的负载因子大小为0.75，即当一个map填满了75%的bucket时候，和其它集合类(如ArrayList等)一样，将会创建原来HashMap大小的两倍的bucket数组，来重新调整map的大小，并将原来的对象放入新的bucket数组中。这个过程叫作rehashing，因为它调用hash方法找到新的bucket位置。


**3.JDK7与JDK８的区别**

JDK7中HashMap采用的是位桶+链表的方式，即我们常说的散列链表的方式，而JDK8的HashMap对之前做了较大的优化，其中最重要的一个优化就是桶中的元素不再唯一按照链表组合，也可以使用红黑树进行存储，所以说，JDK8中采用的是位桶+链表/红黑树的方式，也是非线程安全的。当某个位桶的链表的长度达到某个阀值的时候，这个链表就将转换成红黑树。

JDK8图解：
![这里写图片描述](http://img.blog.csdn.net/20170513210158943?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

<br/>
##**二、HashMap源码（JDK8）剖析**

**1.类的继承关系**

```
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable
```

分析：HashMap继承自父类（AbstractMap），实现了Map、Cloneable、Serializable接口。其中，Map接口定义了一组通用的操作；Cloneable接口则表示可以进行拷贝，在HashMap中，实现的是浅层次拷贝，即对拷贝对象的改变会影响被拷贝的对象；Serializable接口表示HashMap实现了序列化，即可以将HashMap对象保存至本地，之后可以恢复状态

类关系图解：

![这里写图片描述](http://img.blog.csdn.net/20170513213512928?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

注：绿色虚线代表实现的接口，黄色实线代表继承的类。

**2.类的属性**

```
	// 序列号
    private static final long serialVersionUID = 362498820763181265L;  
    // 默认的初始容量是16
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16 
    // 最大容量
    static final int MAXIMUM_CAPACITY = 1 << 30;
    // 默认的负载因子
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    // 当桶(bucket)上的冲突节点数>=(8-1)时，转换成红黑树
    static final int TREEIFY_THRESHOLD = 8; 
    // 当桶(bucket)上的结点数小于阀值时树转链表
    static final int UNTREEIFY_THRESHOLD = 6;
    // 桶中结构转化为红黑树对应的table的最小值
    static final int MIN_TREEIFY_CAPACITY = 64;
    // 存储（位桶）数组，总是2的幂次倍
    transient Node<k,v>[] table;
    // 存放具体元素的集
    transient Set<map.entry<k,v>> entrySet;
    // 存放元素的个数，这并不等于数组的长度。
    transient int size;
    // 每次扩容和更改map结构的计数器
    transient int modCount;   
    // 临界值 当实际大小(容量*加载因子)超过临界值时，会进行扩容
    int threshold;
    // 负载因子
    final float loadFactor;
```

注：  HashMap类的一个非常重要的属性 Node[] table，即哈希桶数组，

**3.基本的数据结构**

（1）单向链表Node

```
static class Node implements Map.Entry {  
    final int hash;  
    final K key;  
    V value;  
    Node next;  
    //构造函数Hash值 键 值 下一个节点  
    Node(int hash, K key, V value, Node next) {  
        this.hash = hash;  
        this.key = key;  
        this.value = value;  
        this.next = next;  
    }  
  
    public final K getKey()        { return key; }  
    public final V getValue()      { return value; }  
    public final String toString() { return key + = + value; }  
  
    public final int hashCode() {  
        return Objects.hashCode(key) ^ Objects.hashCode(value);  
    }  
  
    public final V setValue(V newValue) {  
        V oldValue = value;  
        value = newValue;  
        return oldValue;  
    }  
    //判断两个node是否相等,若key和value都相等，返回true。可以与自身比较为true  
    public final boolean equals(Object o) {  
        if (o == this)  
            return true;  
        if (o instanceof Map.Entry) {  
            Map.Entry e = (Map.Entry)o;  
            if (Objects.equals(key, e.getKey()) &&  
                Objects.equals(value, e.getValue()))  
                return true;  
        }  
        return false;  
    }  
} 
```

（2）红黑树

```
static final class TreeNode extends LinkedHashMap.Entry {  
    TreeNode parent;  // 父节点  
    TreeNode left;  //左子树  
    TreeNode right;//右子树  
    TreeNode prev;    // needed to unlink next upon deletion  
    boolean red;    //颜色属性  
    TreeNode(int hash, K key, V val, Node next) {  
        super(hash, key, val, next);  
    }  
  
    //返回当前节点的根节点  
    final TreeNode root() {  
        for (TreeNode r = this, p;;) {  
            if ((p = r.parent) == null)  
                return r;  
            r = p;  
        }  
    }
    //略......  
}  
```

（3）数组

```
transient Node[] table;//存储（位桶）的数组
```

小结：有一个每个元素都是链表的数组，当添加一个元素（key-value）时，就首先计算元素key的hash值，以此确定插入数组中的位置，但是可能存在同一hash值的元素已经被放在数组同一位置了，这时就添加到同一hash值的元素的后面，他们在数组的同一位置，但是形成了链表，所以说数组存放的是链表。而当链表长度超过8时，链表就转换为红黑树。 

<br/>
**4.类的构造函数**

（1）HashMap(int, float)型构造函数

```
public HashMap(int initialCapacity, float loadFactor) {
    // 初始容量不能小于0，否则报错
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                            initialCapacity);
    // 初始容量不能大于最大值，否则为最大值
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    // 负载因子不能小于或等于0，不能为非数字
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                            loadFactor);
    // 初始化负载因子                                        
    this.loadFactor = loadFactor;
    // 初始化threshold（临界值）
    this.threshold = tableSizeFor(initialCapacity);    
}
```

注：loadFactor越小，占用的空间就越多，查询检索的效率就越高；反之loadFactor越大，占用空间就小，而查询检索效率就低。由于haspMap的定义就是空间换时间，loadFactor负载因子值设置非常重要。 

默认情况下，loadFactor为0.75，threshold 为12；如果loadFactor为1，threshold 就是16，扩容后，位桶table也会增大，为旧的两倍，对比不扩容，位桶上hash冲突会减少很多，检索查询效率自然会高些。 

tableSizeFor方法返回大于initialCapacity的最小的二次幂数值。

```
static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```

注：>>> 操作符表示无符号右移，高位取0。

（2）HashMap(int)型构造函数。

```
public HashMap(int initialCapacity) {
    // 调用HashMap(int, float)型构造函数
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}
```

（3）HashMap()无参型构造函数

```
public HashMap() {
    // 初始化负载因子（0.75）
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}
```

（4）HashMap( Map< ? extends K >)型构造函数。

```
public HashMap(Map<? extends K, ? extends V> m) {
    // 初始化填充因子
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    // 将m中的所有元素添加至HashMap中
    putMapEntries(m, false);
}
```

注：putMapEntries方法将m的所有元素存入HashMap实例中

```
final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
    int s = m.size();
    if (s > 0) {
        // 判断table是否已经初始化
        if (table == null) { 
            // 未初始化，s为m的实际元素个数
            float ft = ((float)s / loadFactor) + 1.0F;
            int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                    (int)ft : MAXIMUM_CAPACITY);
            // 计算得到的t大于阈值，则初始化阈值
            if (t > threshold)
                threshold = tableSizeFor(t);
        }
        // 已初始化，并且m元素个数大于阈值，进行扩容处理
        else if (s > threshold)
            resize();
        // 将m中的所有元素添加至HashMap中
        for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
            K key = e.getKey();
            V value = e.getValue();
            putVal(hash(key), key, value, false, evict);
        }
    }
}
```

注：由于篇幅有限，putVal方法和resize方法的源码请参考下一篇博文。

<br/>
<br/>
本人才疏学浅，若有错，请指出
谢谢！