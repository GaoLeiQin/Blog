**1.简单的集合分类**

![java类库简化图](http://img.blog.csdn.net/20170413182755953?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**2.List**

（1）ArrayList：一种可以动态增长和缩减的索引序列，基于动态数组的数据结构，

优点：随机访问速度快

（2）LinkedList： 一种可以在任意位置高效的插入和删除的有序序列，基于链表的数据结构。

优点：新增和删除速度快

（3）ArrayList与Vector的区别

*相同点：都是Java的集合类，可以存放Java对象。

*不同点：
Vector是同步的，线程安全的，而ArrayList是异步的，非线程安全的。
Vector在缺省的情况下自动增长为原来一倍的数组长度，ArrayList增长为原来的一半。

**3.Set**

（1）定义：

存放的每个元素都必须是唯一的，因为Set不保证重复元素，加入Set的元素必须定义equals（）方法以确保对象的唯一性。

（2）HashSet： 一种没有重复元素的无序集合。

*为快速查找而设计的Set，存入HashSet的元素必须定义hashCode（）

（3）TreeSet ： 一种有序集。

保持次序的Set，底层为树结构，使用它可以从Set中提取有序的序列，元素必须实现Comparable接口

（4）LinkedHashSet： 一种可以记住元素插入次序的集。
具有HashSet的查询速度，且内部使用链表维护元素的顺序（插入的次序），于是在使用迭代器遍历Set时，结果会按照元素的插入次序显示，元素也必须定义hashCode方法。

（5）enumSet： 一种包含枚举类型的元素

**4.Queue**

（1）PriorityQueue : 一种允许高效删除最小元素的集合。

（2）ArrayDeque : 一种用循环数组实现的双端队列。

（双端队列：可以在任何一端添加或移除元素）

（3）在Java SE 6 中引入了Deque接口，并由ArrayDeque和Linkedlist类实现。

**5.Map**

（1）HahMap： 一种存储键值关联的数据结构，

Map基于散列表的实现（它取代了HashTable），插入和查询键值对的开销是固定的，可以通过构造器设置容量和负载因子，以调整容器的性能。

（2）TreeMap： 一种键值有序排列的映射表。

基于红黑树的实现，查看 键 或 键值对 时，它们会被排序（次序由Comparable或Comparator决定），TreeMap的特点在于，所得到的结果是经过排序的，TreeMap是唯一带有subMap（）方法的Map，它可以返回一个子树。

（3）WeakHashMap：一种其值无用武之地后可以被垃圾回收器回收的映射表

弱键映射：允许释放映射所指向的对象，这是为解决某类特殊问题而设计的，如果映射之外没有引用指向某个键，则此键可以被垃圾回收器回收。

（4）ConcurrrentHashMap：一种线程安全的Map，它不涉及同步加锁。

（5）LinkedHashMap：一种可以记住键值项添加次序的映射表。

类似于HashMap，但是迭代遍历它时，取得键值对的顺序是其插入次序，或者是最近最少使用（LRU）的次序，只比Hash慢一点，而在迭代访问时反而更快，因为它使用链表维护内部次序。

（6）IdentityHashMap：一种使用 == 代替equals( ) 对键进行比较的散列映射。

（7）规则：

*对Map中使用的键的要求与对Set中的元素要求一样，任何键都必须具有一个equals（）方法；
*如果键被用于散列Map，那么它必须还具有恰当的hashCode（）方法；
*如果键被用于TreeMap，则同时覆盖hashCode（）方法；
*若要使用自己的类作为HashMap的键，必须同时重写hashCode（）和equals（）
 
 （8）HashMap和Hashtable的区别

*历史原因：
Hashtable是基于陈旧的Dictionary类；
HashMap是Java1.2引进的Map接口的一个实现。

*同步性：
Hashtable是线程同步的，其中的对象是线程安全的；
HashMap是异步的，不是线程安全的。
由于同步的要求会影响执行的效率，若不需要线程安全，则选择HashMap

*值
HashMap可以让你将空值作为一个表的条目的Key或Value
Hashtable不能放入空值null

**6.Iterator**

（1）定义：
是一种设计模式，它是一个对象，它可以遍历并选择序列中的对象，而开发人员不需要了解该序列的底层结构，通常被称为轻量级对象。

（2）Iterator接口中定义了三个方法
*boolean  hasNext( ) ;    如果仍有元素可以迭代，则返回true
*E  next（）；                返回迭代的下一个元素 
*void remove（）；       从迭代器指向的collection中移除迭代器返回的最后一个元素

（3）ListIterator接口
*它继承了Iterator接口，允许程序员按照任意方向遍历列表，迭代期间修改列表，并获得迭代器在列表中的当前位置

<br/>
<br/>
本人才疏学浅，如有错误，请指出~ 
谢谢！