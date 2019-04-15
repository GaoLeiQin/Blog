##**一、ConcurrentHashMap概述**

**1.底层实现**

ConcurrentHashMap与HashMap类似，基于数组+（链表/红黑树），但是为了实现并发，链表/红黑树增加了很多辅助的类，例如TreeBin，Traverser等对象内部类。

**2.为什么使用ConcurrentHashMap**

在多线程的情况下，使用HashMap可能会导致死循环，因为HashMap是非线程安全的，而使用线程安全的HashTable效率又低下，因为HashTable只有一个锁，当有其他线程访问时，会进入阻塞或者轮询状态。

**3.线程安全的原理**

（1）相比于HashTable（只有一个锁），ConcurrentHashMap在并发的情况下就显得很高效了，因为它使用了分段锁，即将数据分成一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一段数据的时候，其他段的数据也能被其他线程访问。

<font size="4" color="blue">分段锁图解：

![这里写图片描述](http://img.blog.csdn.net/20170612101754274?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

（2）jdk7针对ConcurrentHashMap的改进是增加了分段锁Segment对HashEntity的控制，与之前版本不同的是，在JDK8中ConcurrentHashMap 不再采用 Segment 实现，而是改用 Node，Node 是一个链表的结构，每个节点可以引用到下一个节点(next)。

![这里写图片描述](http://img.blog.csdn.net/20170612102731303?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

注：Segment虽然在JDK8中保留了，但已经简化了属性，只是为了兼容旧版本。

(3)在JDK8中还利用了CAS算法。

**4.与HashMap的区别**

（1）ConcurrentHashMap不允许key或value为null值，而HashMap的键值可以为空

（2）ConcurrentHashMap是线程安全的，而HashMap不是线程安全的

**5.预备知识**

（1） 读过HashMap的源码

（２）了解volatile关键字的作用（保证可见性和禁止指令重排序）

（３）了解ＣＡＳ算法

（４）了解内置锁和显示锁

##**二、源码剖析（JDK8）**

**1.类的继承关系**

```
public class ConcurrentHashMap<K,V> extends AbstractMap<K,V>
        implements ConcurrentMap<K,V>, Serializable 
```

分析：ConcurrentHashMap支持泛型，继承于AbstractMap类，实现了ConcurrentMap接口和Serializable 接口。

<font size="4" color="blue">类关系图解

![这里写图片描述](http://img.blog.csdn.net/20170612103725363?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

注：绿色虚线表示实现的接口，黄色实线表示继承的类，绿色实线表示接口间的继承。

**2.内部类Node**

```
    static class Node<K,V> implements Map.Entry<K,V> {
        //定义属性
        final int hash;
        final K key;
        //带有同步锁的val和next
        volatile V val;
        volatile Node<K,V> next;

        Node(int hash, K key, V val, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.val = val;
            this.next = next;
        }

        public final K getKey()       { return key; }
        public final V getValue()     { return val; }
        public final int hashCode()   { return key.hashCode() ^ val.hashCode(); }
        public final String toString(){ return key + "=" + val; }
        
        //通过setValue方法改变value的值，但不能直接改变
        public final V setValue(V value) {
            throw new UnsupportedOperationException();
        }

        public final boolean equals(Object o) {
            Object k, v, u; Map.Entry<?,?> e;
            return ((o instanceof Map.Entry) &&
                    (k = (e = (Map.Entry<?,?>)o).getKey()) != null &&
                    (v = e.getValue()) != null &&
                    (k == key || k.equals(key)) &&
                    (v == (u = val) || v.equals(u)));
        }

        //find方法用于辅助map.get()方法
        Node<K,V> find(int h, Object k) {
            Node<K,V> e = this;
            if (k != null) {
                do {
                    K ek;
                    if (e.hash == h &&
                            ((ek = e.key) == k || (ek != null && k.equals(ek))))
                        return e;
                } while ((e = e.next) != null);
            }
            return null;
        }
    }
```

**3.类的属性**

```
    //指定序列号
    private static final long serialVersionUID = 7249069246763182397L;
    //最大容量为1左移30位，即（2的30次方=1073741824）
    private static final int MAXIMUM_CAPACITY = 1 << 30;
    //默认容量为16
    private static final int DEFAULT_CAPACITY = 16;
    //数组的最大值为整型最大值-8
    static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
    //table并发层默认大小为16
    private static final int DEFAULT_CONCURRENCY_LEVEL = 16;
    //负载因子为0.75
    private static final float LOAD_FACTOR = 0.75f;
    //链表转换为红黑树的阈值为8
    static final int TREEIFY_THRESHOLD = 8;
    //红黑树转链表的阈值为6
    static final int UNTREEIFY_THRESHOLD = 6;
    //TREEIFY的最小容量为64
    static final int MIN_TREEIFY_CAPACITY = 64;
    //TRANSFER的最小幅度为16
    private static final int MIN_TRANSFER_STRIDE = 16;
    //sizeCtl的扩容比特数为16
    private static int RESIZE_STAMP_BITS = 16;
    // resize的最大线程数
    private static final int MAX_RESIZERS = (1 << (32 - RESIZE_STAMP_BITS)) - 1;
    //sizeCtl中记录size大小的偏移量
    private static final int RESIZE_STAMP_SHIFT = 32 - RESIZE_STAMP_BITS;
    //hash值是-1，表示这是一个forwardNode节点
    static final int MOVED     = -1;
    //hash值是-2  表示这是一个TreeBin节点
    static final int TREEBIN   = -2;
    //hash值是-3，表示这是一个保留节点（不能被序列化）
    static final int RESERVED  = -3;
    //标准的节点hash值
    static final int HASH_BITS = 0x7fffffff;
    //可用处理器数量 
    static final int NCPU = Runtime.getRuntime().availableProcessors();
    //放Node元素的数组
    transient volatile Node<K,V>[] table;
    //下一个table
    private transient volatile Node<K,V>[] nextTable;
    //记录容器的容量大小，通过CAS更新
    private transient volatile long baseCount;
    //hash表初始化或扩容时的一个控制位标识量,
    //负数代表正在进行初始化或扩容.正数或0代表还没有被初始化
    private transient volatile int sizeCtl;
    //在需要扩容时下一个table的索引
    private transient volatile int transferIndex;
    //创建CounterCells时使用
    private transient volatile int cellsBusy;
    //对table进行计数
    private transient volatile CounterCell[] counterCells;
    //键值对
    private transient KeySetView<K,V> keySet;
    private transient ValuesView<K,V> values;
    private transient EntrySetView<K,V> entrySet;

```

**4.构造方法**

（1）无参构造器

```
    public ConcurrentHashMap() {
    }
```
注：无参构造器会创建一个空ConcurrentHashMap，默认大小为16

（2）参数 （int） ：创建指定大小的ConcurrentHashMap

```
    public ConcurrentHashMap(int initialCapacity) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException();
        int cap = ((initialCapacity >= (MAXIMUM_CAPACITY >>> 1)) ?
                MAXIMUM_CAPACITY :
                tableSizeFor(initialCapacity + (initialCapacity >>> 1) + 1));
        this.sizeCtl = cap;
    }
```

（3）参数 （Map） ：创建一个指定集合的ConcurrentHashMap

```
    public ConcurrentHashMap(Map<? extends K, ? extends V> m) {
        this.sizeCtl = DEFAULT_CAPACITY;
        putAll(m);
    }
```

（4）参数 （int，float） ：创建一个指定大小和负载因子的ConcurrentHashMap

```
    public ConcurrentHashMap(int initialCapacity, float loadFactor) {
        this(initialCapacity, loadFactor, 1);
    }
```
（5）参数 （int，float，int） ：创建一个指定大小、负载因子和并发级别的ConcurrentHashMap

```
    public ConcurrentHashMap(int initialCapacity,
                             float loadFactor, int concurrencyLevel) {
        if (!(loadFactor > 0.0f) || initialCapacity < 0 || concurrencyLevel <= 0)
            throw new IllegalArgumentException();
        if (initialCapacity < concurrencyLevel)   // Use at least as many bins
            initialCapacity = concurrencyLevel;   // as estimated threads
        long size = (long)(1.0 + (long)initialCapacity / loadFactor);
        int cap = (size >= (long)MAXIMUM_CAPACITY) ?
                MAXIMUM_CAPACITY : tableSizeFor((int)size);
        this.sizeCtl = cap;
    }
```

**5.重要方法**

（1）clear方法 ：从该映射中移除所有映射关系

```
   public void clear() {
        long delta = 0L;
        int i = 0;
        ConcurrentHashMap.Node<K,V>[] tab = table;
        while (tab != null && i < tab.length) {
            int fh;
            ConcurrentHashMap.Node<K,V> f = tabAt(tab, i);
            //若f为空，就直接跳过
            if (f == null)
                ++i;
            //若其他线程正对其扩容，则协助其扩容，然后重置计数器
            else if ((fh = f.hash) == MOVED) {
                tab = helpTransfer(tab, f);
                i = 0;
            }
            else {
                //上锁
                synchronized (f) {
                    //若其他线程没有在此期间操作f
                    if (tabAt(tab, i) == f) {
                        //首先删除链表、树的末尾元素，避免产生大量垃圾
                        ConcurrentHashMap.Node<K,V> p = (fh >= 0 ? f :
                                (f instanceof ConcurrentHashMap.TreeBin) ?
                                        ((ConcurrentHashMap.TreeBin<K,V>)f).first : null);
                        while (p != null) {
                            --delta;
                            p = p.next;
                        }
                        //利用CAS无锁置nul
                        setTabAt(tab, i++, null);
                    }
                }
            }
        }
        //若delta被修改
        if (delta != 0L)
            //直接return
            addCount(delta, -1);
    }
```

（2）contains方法 ：（遗留方法）测试此表中是否有一些与指定值存在映射关系的键

```
    public boolean contains(Object value) {
        return containsValue(value);
    }
```

（3）containsValue方法 ：如果此映射将一个或多个键映射到指定值，则返回 true

```
    public boolean containsValue(Object value) {
        //若值为空，抛出异常
        if (value == null)
            throw new NullPointerException();
        ConcurrentHashMap.Node<K,V>[] t;
        if ((t = table) != null) {
            ConcurrentHashMap.Traverser<K,V> it = new ConcurrentHashMap.Traverser<K,V>(t, t.length, 0, t.length);
            for (ConcurrentHashMap.Node<K,V> p; (p = it.advance()) != null; ) {
                V v;
                if ((v = p.val) == value || (v != null && value.equals(v)))
                    return true;
            }
        }
        return false;
    }
```
（4）containsKey方法 ： 测试指定对象是否为此表中的键

```
    public boolean containsKey(Object key) {
        return get(key) != null;
    }
```
（5）get方法 ：返回指定键所映射到的值

```
    public V get(Object key) {
        ConcurrentHashMap.Node<K,V>[] tab; ConcurrentHashMap.Node<K,V> e, p; int n, eh; K ek;
        //计算hash值
        int h = spread(key.hashCode());
        //确定节点位置
        if ((tab = table) != null && (n = tab.length) > 0 &&
                (e = tabAt(tab, (n - 1) & h)) != null) {
            //如果搜索到的节点key与传入的key相同且不为null,直接返回这个节点
            if ((eh = e.hash) == h) {
                if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                    return e.val;
            }
            //如果eh<0 说明这个节点在树上 直接寻找
            else if (eh < 0)
                //TreeBin操作
                return (p = e.find(h, key)) != null ? p.val : null;
            //否则遍历链表 找到对应的值并返回
            while ((e = e.next) != null) {
                if (e.hash == h &&
                        ((ek = e.key) == key || (ek != null && key.equals(ek))))
                    return e.val;
            }
        }
        return null;
    }
```
（6）put方法 ：将指定键映射到此表中的指定值

```
    public V put(K key, V value) {
        return putVal(key, value, false);
    }
```
putVal方法源码如下：

```
    final V putVal(K key, V value, boolean onlyIfAbsent) {
        //不允许 key或value为null
        if (key == null || value == null) throw new NullPointerException();
        //计算hash值(h^(h>>>16)
        int hash = spread(key.hashCode());
        int binCount = 0;
        //死循环 何时插入成功 何时跳出
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh;
            //如果table为空的话，初始化table
            if (tab == null || (n = tab.length) == 0)
                tab = initTable();
                //根据hash值计算出在table里面的位置
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                //如果这个位置没有值 ，直接放进去，不需要加锁
                if (casTabAt(tab, i, null,
                        new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
            //若遇到表连接点
            else if ((fh = f.hash) == MOVED)
                //检测到正在扩容，帮助其扩容
                tab = helpTransfer(tab, f);
            else {
                V oldVal = null;
                //结点上锁（hash值相同的链表的头节点）
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        //这是链表节点，不是树的节点
                        if (fh >= 0) {
                            binCount = 1;
                            //在这里遍历链表所有的结点
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                //如果hash值和key值相同  则修改对应结点的value值
                                if (e.hash == hash &&
                                        ((ek = e.key) == key ||
                                                (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    //仅putIfAbsent()方法中onlyIfAbsent为true
                                    if (!onlyIfAbsent)
                                        //putIfAbsent()包含key则返回get，否则put并返回
                                        e.val = value;
                                    break;
                                }
                                Node<K,V> pred = e;
                                //已遍历到链表尾部，直接插入
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key,
                                            value, null);
                                    break;
                                }
                            }
                        }
                        //如果这个节点是树节点，就按照树的方式插入值
                        else if (f instanceof TreeBin) {
                            Node<K,V> p;
                            binCount = 2;
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                    value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                if (binCount != 0) {
                    //如果链表长度已经达到临界值8 就需要把链表转换为树结构
                    if (binCount >= TREEIFY_THRESHOLD)
                        //转化为红黑树
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        //将当前ConcurrentHashMap的元素数量+1
        addCount(1L, binCount);
        return null;
    }
```

注：

* ConcurrentHashMap不允许key或value为null

* addCount方法会利用CAS快速更新baseCount的值，然后判断check>=0.是否需要扩容

* hash值计算：利用spread方法对key的hashcode进行hash计算

（7）remove方法 ：

* 参数（Object key） ：从此映射中移除键（及其相应的值）

```
    public V remove(Object key) {   
        return replaceNode(key, null, null);
    }
```
注：若key为null，将在计算hashCode时报空指针异常。

* 参数（Object key, Object value） :只有目前将键的条目映射到给定值时，才移除该键的条目

```
public boolean remove(Object key, Object value) {
	//若key为空，抛出异常  
    if (key == null)  
        throw new NullPointerException();  
    returnvalue != null && replaceNode(key, null, value) != null;  
} 
```

replaceNode方法源码如下：

```
    //注：cv是key-value中的value
    final V replaceNode(Object key, V value, Object cv) {
        inthash = spread(key.hashCode());
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; intn, i, fh;
            // 该桶位第一个元素为空，直接跳过
            if (tab == null || (n = tab.length) == 0 ||
                    (f = tabAt(tab, i = (n - 1) & hash)) == null)
                break;   
            elseif ((fh = f.hash) == MOVED)
            // 协助扩容 
            tab = helpTransfer(tab, f);
        else {
                V oldVal = null;
                booleanvalidated = false;
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        if (fh >= 0) {
                            validated = true;
                            for (Node<K,V> e = f, pred = null;;) {
                                K ek;
                                if (e.hash == hash &&((ek = e.key) == key ||
                                        (ek != null && key.equals(ek)))){  
                                    V ev = e.val;
                                    // value为null或和查到的值相等  
                                    if (cv == null || cv == ev ||
                                            (ev != null && cv.equals(ev))) {
                                        oldVal = ev;
                                        if (value != null)
                                            e.val = value;
                                        elseif (pred != null)
                                        pred.next = e.next;  
                                    else
                                        setTabAt(tab, i, e.next);
                                    }
                                    break;
                                }
                                pred = e;
                                if ((e = e.next) == null)
                                    break;
                            }
                        }
                        // 以树的方式find、remove
                        elseif (finstanceof TreeBin) {  
                            validated = true;
                            TreeBin<K,V> t = (TreeBin<K,V>)f;
                            TreeNode<K,V> r, p;
                            if ((r = t.root) != null &&
                                    (p = r.findTreeNode(hash, key, null)) != null) {
                                V pv = p.val;
                                if (cv == null || cv == pv ||
                                        (pv != null && cv.equals(pv))) {
                                    oldVal = pv;
                                    if (value != null)
                                        p.val = value;
                                    elseif (t.removeTreeNode(p))
                                    setTabAt(tab, i, untreeify(t.first));
                                }
                            }
                        }
                    }
                }
                if (validated) {
                    if (oldVal != null) {
                        if (value == null)
                            addCount(-1L, -1);
                        returnoldVal;
                    }
                    break;
                }
            }
        }
        return null;
    }
```

###**源码内部使用的方法**

（8）3个原子操作

* tabAt方法

```
@SuppressWarnings("unchecked")  
	// 获取索引i处Node  
	static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) { 
	    return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);  
    }  
   
```

* casTabAt方法

```
	 // 利用CAS算法设置i位置上的Node节点（将c和table[i]比较，相同则插入v）。  
    static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,  
                                        Node<K,V> c, Node<K,V> v) {  
        return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);  
    }  
```

* setTabAt方法

```
	// 设置节点位置的值，仅在上锁区被调用  
    static final <K,V> void setTabAt(Node<K,V>[] tab, int i, Node<K,V> v) {  
        U.putObjectVolatile(tab, ((long)i << ASHIFT) + ABASE, v);  
    } 
```

（9）transfer方法 ：扩容

```
    private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
        int n = tab.length, stride;
        // 每核处理的量小于16，则强制赋值16
        if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
            stride = MIN_TRANSFER_STRIDE;
        if (nextTab == null) {
            try {
                @SuppressWarnings("unchecked")
                Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];
                nextTab = nt;
            } catch (Throwable ex) {
                sizeCtl = Integer.MAX_VALUE;
                return;
            }
            nextTable = nextTab;
            transferIndex = n;
        }
        int nextn = nextTab.length;
        //连节点指针,标志位，fwd的hash值为-1，fwd.nextTable=nextTab。
        ForwardingNode<K,V> fwd= new ForwardingNode<K,V>(nextTab);
        //并发扩容的关键属性,若等于true,说明此节点已经处理过
        boolean advance= true;
        boolean finishing = false;
        // 死循环
        for (int i = 0, bound = 0;;) {
            Node<K,V> f; int fh;
            //通过循环控制i-- ，再通过i--依次遍历原hash表中的节点
            while (advance) {
                int nextIndex, nextBound;
                if (--i >= bound || finishing)
                    advance = false;
                else if ((nextIndex = transferIndex) <= 0) {
                    i = -1;
                    advance = false;
                }//TRANSFERINDEX 即用CAS计算得到的transferIndex
                else if (U.compareAndSwapInt
                        (this, TRANSFERINDEX, nextIndex,
                                nextBound = (nextIndex > stride ?
                                        nextIndex - stride : 0))) {
                    bound = nextBound;
                    i = nextIndex - 1;
                    advance = false;
                }
            }
            if (i < 0 || i >= n || i + n >= nextn) {
                int sc;
                // 若所有节点复制完毕
                if (finishing) {
                    //清空临时对象
                    nextTable = null;
                    table = nextTab;
                    //扩容阀值设为原来的1.5倍，即现在的0.75倍
                    sizeCtl = (n << 1) - (n >>> 1);
                    // 仅有的2个跳出死循环出口之一
                    return;
                }
                //CAS更新扩容阈值,sc-1表明新加入一个线程参与扩容
                if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                    if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                        // 仅有的2个跳出死循环出口之二
                        return;
                    finishing = advance = true;
                    i = n;
                }
            }
            //如果遍历到的节点为空 则放入ForwardingNode指针
            else if ((f = tabAt(tab, i)) == null)
                advance = casTabAt(tab, i, null, fwd);
                //遍历到ForwardingNode节点，说明此节点被处理过了，直接跳过。这是控制并发扩容的核心
            else if ((fh = f.hash) == MOVED)
                advance = true;
            else {
                //上锁
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        //ln原位置节点，hn新位置节点
                        Node<K,V> ln, hn;
                        // 这是一个链表
                        if (fh >= 0) {
                            int runBit = fh & n; // f.hash & n
                            // 构造lastRun和p两个链表，一个是原链表，一个是其逆序链表
                            Node<K,V> lastRun = f;
                            for (Node<K,V> p = f.next; p != null; p = p.next) {
                                int b = p.hash & n; // f.next.hash & n
                                if (b != runBit) {
                                    runBit = b;
                                    lastRun = p;
                                }
                            }
                            if (runBit == 0) {
                                ln = lastRun;
                                hn = null;
                            }
                            else {
                                hn = lastRun;
                                ln = null;
                            }
                            for (Node<K,V> p = f; p != lastRun; p = p.next) {
                                int ph = p.hash; K pk = p.key; V pv = p.val;
                                if ((ph & n) == 0)
                                    // 和HashMap确定扩容后的节点位置一样
                                    ln = new Node<K,V>(ph, pk, pv, ln);
                                else
                                    //新位置节点
                                    hn = new Node<K,V>(ph, pk, pv, hn);
                            }
                            //在nextTable[i]插入原节点
                            setTabAt(nextTab, i, ln);
                            //在nextTable[i+n]插入新节点
                            setTabAt(nextTab, i + n, hn);
                            //在nextTable[i]插入forwardNode节点，表示已经处理过该节点
                            setTabAt(tab, i, fwd);
                            //设置advance为true 返回到上面的while循环中 就可以执行--i操作
                            advance = true;
                        }
                        //对TreeBin对象进行处理，类似于链表过程
                        else if (f instanceof TreeBin) {
                            TreeBin<K,V> t = (TreeBin<K,V>)f;
                            TreeNode<K,V> lo = null, loTail = null;
                            TreeNode<K,V> hi = null, hiTail = null;
                            //lc、hc=0两计数器分别++记录原、新bin中TreeNode数量
                            int lc = 0, hc = 0;
                            //构造正序和反序两个链表
                            for (Node<K,V> e = t.first; e != null; e = e.next) {
                                int h = e.hash;
                                TreeNode<K,V> p = new TreeNode<K,V>
                                        (h, e.key, e.val, null, null);
                                if ((h & n) == 0) {
                                    if ((p.prev = loTail) == null)
                                        lo = p;
                                    else
                                        loTail.next = p;
                                    loTail = p;
                                    ++lc;
                                }
                                else {
                                    if ((p.prev = hiTail) == null)
                                        hi = p;
                                    else
                                        hiTail.next = p;
                                    hiTail = p;
                                    ++hc;
                                }
                            }
                            //扩容后树节点个数若<=6，将树转链表
                            ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                                    (hc != 0) ? new TreeBin<K,V>(lo) : t;
                            hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                                    (lc != 0) ? new TreeBin<K,V>(hi) : t;
                            //在nextTable的i位置上插入一个链表
                            setTabAt(nextTab, i, ln);
                            //在nextTable的i+n的位置上插入另一个链表
                            setTabAt(nextTab, i + n, hn);
                            //在table的i位置上插入forwardNode节点  表示已经处理过该节点
                            setTabAt(tab, i, fwd);
                            //设置advance为true 返回到上面的while循环中 就可以执行i--操作
                            advance = true;
                        }
                    }
                }
            }
        }
    }
```

（10）initTable方法 ：初始化

```
    private final Node<K,V>[] initTable() {
        Node<K,V>[] tab; int sc;
        while ((tab = table) == null || tab.length == 0) {
            //sizeCtl表示有其他线程正在进行初始化操作，把线程挂起。对于table的初始化工作，只能有一个线程在进行。
            if ((sc = sizeCtl) < 0)
                Thread.yield();
            //利用CAS方法把sizectl的值置为-1 表示本线程正在进行初始化
            else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
                try {
                    if ((tab = table) == null || tab.length == 0) {
                        int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                        @SuppressWarnings("unchecked")
                        Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                        table = tab = nt;
                        //设置一个扩容的阈值(0.75*n)
                        sc = n - (n >>> 2);
                    }
                } finally {
                    sizeCtl = sc;
                }
                break;
            }
        }
        return tab;
    }
```
（11）helpTransfer方法 ：协助扩容

```
    final Node<K,V>[] helpTransfer(Node<K,V>[] tab, Node<K,V> f) {
        Node<K,V>[] nextTab; intsc;
        if (tab != null && (finstanceof ForwardingNode) &&
        (nextTab = ((ForwardingNode<K,V>)f).nextTable) != null) {
            //计算一个操作校验码
            intrs = resizeStamp(tab.length);
            while (nextTab == nextTable && table == tab &&
                    (sc = sizeCtl) < 0) {
                if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                        sc == rs + MAX_RESIZERS || transferIndex <= 0)
                    break;
                if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1)) {
                    transfer(tab, nextTab);//调用扩容方法，直接进入复制阶段
                    break;
                }
            }
            return nextTab;
        }
        return table;
    }
```

注：   若当前线程检测到其他线程正进行扩容操作，则协助其扩容；（只有这种情况会被调用），只要检测到扩容，就会优先去扩容。但调用之前，nextTable就已存在。

（12）addCount方法 ：,先更新baseCount的值，然后检测是否进行扩容。

```
    private final void addCount(long x, int check) {
        CounterCell[] as; long b, s;

        //利用CAS方法更新baseCount的值
        if ((as = counterCells) != null ||
                !U.compareAndSwapLong(this, BASECOUNT, b = baseCount, s = b + x)) {//1
            CounterCell a; long v; int m;
            boolean uncontended = true;
            if (as == null || (m = as.length - 1) < 0 ||
                    (a = as[ThreadLocalRandom.getProbe() & m]) == null ||
                    !(uncontended =
                            U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))) {
                //多线程CAS发生失败的时候执行
                fullAddCount(x, uncontended);//2
                return;
            }
            if (check <= 1)
                return;
            s = sumCount();
        }
        //如果check值大于等于0 则需要检验是否需要进行扩容操作
        if (check >= 0) {
            Node<K,V>[] tab, nt; int n, sc;
            //当条件满足开始扩容
            while (s >= (long)(sc = sizeCtl) && (tab = table) != null &&
                    (n = tab.length) < MAXIMUM_CAPACITY) {
                int rs = resizeStamp(n);
                if (sc < 0) {//如果小于0说明已经有线程在进行扩容操作了
                    //已经有在扩容或者多线程进行了扩容，其他线程直接break不要进入扩容操作
                    if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                            sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                            transferIndex <= 0)
                        break;
                    //如果相等说明扩容已经完成，可以继续扩容
                    if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                        transfer(tab, nt);
                }
                //若当前线程是唯一的或是第一个发起扩容的线程  此时nextTable=null
                else if (U.compareAndSwapInt(this, SIZECTL, sc,
                        (rs << RESIZE_STAMP_SHIFT) + 2))
                    transfer(tab, null);
                s = sumCount();
            }
        }
    }
```

**小结：**

（1）JDK8中不采用segment而采用node，锁住node来实现减小锁粒度。

（2）设计了MOVED状态 当resize的中过程中 线程2还在put数据，线程2会帮助resize。

（3）使用3个CAS操作来确保node的一些操作的原子性，这种方式代替了锁。

（4）sizeCtl的不同值来代表不同含义，起到了控制的作用。


<br/>
<br/>
本人才疏学浅。若有错误，请指出~
谢谢！

> 参考链接：[深入分析ConcurrentHashMap1.8的扩容实现](http://www.jianshu.com/p/f6730d5784ad)