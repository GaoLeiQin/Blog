###**一、ArrayList概述**

####**1.实现原理**

ArrayList是基于数组实现的，是一个动态数组，其容量能自动增长，类似于C语言中的动态申请内存，动态增长内存。允许重复，默认第一次插入元素时创建数组的大小为10，超出限制时会增加50%的容量，每次扩容都底层采用System.arrayCopy()复制到新的数组，初始化时最好能给出数组大小的预估值。

#### **2.扩容：**

每个ArrayList实例都有一个容量，该容量是指用来存储列表元素的数组的大小。它总是至少等于列表的大小。随着向ArrayList中不断添加元素，其容量也自动增长。自动增长会带来数据向新数组的重新拷贝，因此，如果可预知数据量的多少，可在构造ArrayList时指定其容量。在添加大量元素前，应用程序也可以使用ensureCapacity操作来增加ArrayList实例的容量，这可以减少递增式再分配的数量。

####**3.同步性：**

　ArrayList的用法和Vector相类似，但是Vector是一个较老的集合，具有很多缺点，不建议使用。它们的区别是：ArrayList是线程不安全的，当多条线程访问同一个ArrayList集合时，程序需要手动保证该集合的同步性，所以只能用在单线程环境下，多线程环境下可以考虑用Collections.synchronizedList(List l)函数返回一个线程安全的ArrayList类，也可以使用concurrent并发包下的CopyOnWriteArrayList类。而Vector则是线程安全的。

####**4.序列化**

ArrayList实现了Serializable接口，因此它支持序列化，能够通过序列化传输，实现了RandomAccess接口，支持快速随机访问，实际上就是通过下标序号进行快速访问，实现了Cloneable接口，能被克隆。


###**二、源码剖析（JDK8）**

####**1.类的继承关系**

```
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

分析：ArrayList支持泛型，它继承自AbstractList类，实现了List、RandomAccess、Cloneable、Java.io.Serializable接口。AbstractList提供了List接口的默认实现（个别方法为抽象方法）。RandomAccess是一个标记接口，接口内没有定义任何内容。对于Cloneable接口，可以调用Object.clone方法返回该对象的浅拷贝。通过实现 java.io.Serializable 接口以启用其序列化功能。未实现此接口的类将无法使其任何状态序列化或反序列化。序列化接口没有方法或字段，仅用于标识可序列化的语义。

类关系图解：

![这里写图片描述](http://img.blog.csdn.net/20170515150629355?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

注：绿色虚线表示实现的接口，黄色实线表示继承的类，绿色实线表示接口间的继承。

####**2.类的属性**

```
	//序列号
    private static final long serialVersionUID = 8683452581122892189L;
    //默认容量
    private static final int DEFAULT_CAPACITY = 10;
    //一个空数组,当指定ArrayList 容量为 0 时，返回该空数组
    private static final Object[] EMPTY_ELEMENTDATA = {};
    
    //一个空数组实例,当没有指定 ArrayList 的容量时(即调用无参构造函数)，返回的是该数组,数据量为 0,
    //当用户第一次添加元素时，该数组将会扩容，变成默认容量为10的一个数组,通过 ensureCapacityInternal() 实现
    
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
    //用该数组保存数据, 该数组的长度就是ArrayList 的容量
    transient Object[] elementData; // non-private to simplify nested class access
    //ArrayList实际存储的数据数量
    private int size;

	//数组缓冲区最大存储容量，若尝试分配这个最大存储容量，可能会导致 OutOfMemoryError(当该值 > VM 的限制时)
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8; 
```

说明：DEFAULTCAPACITY_EMPTY_ELEMENTDATA与 EMPTY_ELEMENTDATA 的区别就是：前者是默认返回的，而后者是在用户指定容量为 0 时返回。

注：ArrayList只定义了两个私有属性：elementData和size，其中elementData用于存储ArrayList内的元素，而size表示它包含的元素的数量。

 

####**3.类的构造器**

（1） ArrayList ( int ) 类型构造器

```
	//带初始容量大小的构造函数
    public ArrayList(int initialCapacity) {
        //若初始容量大于0,实例化数组
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        }
        //若初始量等于0，将空数组赋给elementData
        else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        }
        //若初始容量小于0，则抛出异常
        else {
            throw new IllegalArgumentException("Illegal Capacity: "+ 
                    initialCapacity);
        }
    }
```

（2）ArrayList（）无参构造器

```
//默认容量为10
public ArrayList(){
	this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
```
（3）ArrayList(Collection< ? extends E > c)类型构造器

```
//创建一个包含collection的ArrayList
    public ArrayList(Collection<? extends E> c) {
        //集合转 Object[] 数组
        elementData = c.toArray();
        //将转换后的 Object[] 长度赋值给当前 ArrayList 的 size，并判断是否为 0
        if ((size = elementData.length) != 0) {
            if (elementData.getClass() != Object[].class)
                // 若 c.toArray() 返回的数组类型不是 Object[]，则利用copyof方法创建一个Object[] 数组
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        }
        else {
            //c中没有元素，则换为空数组
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
```

其中Arrays.copyOf()方法的源码如下：

```
public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
        @SuppressWarnings("unchecked")
        T[] copy = ((Object)newType == (Object)Object[].class)
            ? (T[]) new Object[newLength]
            : (T[]) Array.newInstance(newType.getComponentType(), newLength);
        System.arraycopy(original, 0, copy, 0,
                         Math.min(original.length, newLength));
        return copy;
    }
```

分析：如果newType类型为Object[].class的话，则直接创建一个长度为newLength的Object数组，否则使用Array.newInstance(Class< ?> componentType, int length)方法创建一个元素类型为newType.getComponentType() （该方法返回数组中元素的类型）类型的，长度为newLength的数组，这是一个native方法，然后使用System.arraycopy() 这个native方法将original数组中的元素copy到新创建的数组中，并返回该数组。

**4.类的方法**

（1）trimToSize方法（将当前容量值设为当前实际元素大小）

```
	//该方法手动调用，以减少空间资源浪费的目的
    public void trimToSize() {
        // modCount(用于记录结构性变化次数)是 AbstractList 的属性值(protected transient int modCount = 0)
        modCount++;
        // 当实际大小 < 数组缓冲区大小时
        // 如调用默认构造函数后，刚添加一个元素，此时 elementData.length = 10，而 size = 1
        // 通过这一步，可以使得空间得到有效利用，而不会出现资源浪费的情况
        if (size < elementData.length) {
            // 调整数组缓冲区 elementData，变为实际存储大小 Arrays.copyOf(elementData, size)
            elementData = (size == 0) ? EMPTY_ELEMENTDATA : Arrays.copyOf(elementData, size);
        }
    }
```

（2）ensureCapacity方法（指定 ArrayList 的容量）

```
	public void ensureCapacity(int minCapacity) {
        // 最小扩充容量，默认是 10  
        int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA) ? 0 : DEFAULT_CAPACITY;

        // 若用户指定的最小容量 > 最小扩充容量，则以用户指定的为准，否则还是 10  
        if (minCapacity > minExpand) {
            ensureExplicitCapacity(minCapacity);
        }
    }
```

注：ensureCapacity() 是提供给用户使用的方法，在 ArrayList 的实现中并没有使用

（3）ensureCapacityInternal方法（确定ArrarList 的容量）

```
	private void ensureCapacityInternal(int minCapacity) {
        // 若 elementData == {}，则取 minCapacity 为 默认容量和参数 minCapacity 之间的最大值
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        ensureExplicitCapacity(minCapacity);
    }
```
注：该方法用于内部优化，保证空间资源不被浪费：尤其在 add() 方法添加时起效


（4）ensureExplicitCapacity方法（明确 ArrayList 的容量 ）

```
	private void ensureExplicitCapacity(int minCapacity) {
        // 将“修改统计数”+1，该变量主要是用来实现fail-fast机制的
        modCount++;
        // 防止溢出代码：确保指定的最小容量 > 数组缓冲区当前的长度
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
```
注：该方法与ensureCapacityInternal方法类似，用于内部优化，保证空间资源不被浪费：尤其在 add() 方法添加时起效 。

（5）grow方法（用来扩容）

```
	private void grow(int minCapacity) {
        // 防止溢出代码
        int oldCapacity = elementData.length;
        // 扩容为原来的1.5倍
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)                   
            newCapacity = minCapacity;
        // 若 newCapacity 大于最大存储容量，则进行大容量分配
        if (newCapacity - MAX_ARRAY_SIZE > 0)            
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```
注：该方法确保 ArrayList 至少能存储 minCapacity 个元素，扩容计算：newCapacity = oldCapacity + (oldCapacity >> 1)，其中运算符 >> 是带符号右移.

其中大容量分配的hugeCapacity方法源码为：

```
	private static int hugeCapacity(int minCapacity) {  
        if (minCapacity < 0) // overflow  
            throw new OutOfMemoryError();  
        return (minCapacity > MAX_ARRAY_SIZE) ? Integer.MAX_VALUE : MAX_ARRAY_SIZE;  
    }  
```



（6）size方法（获取该 list 所实际存储的元素个数）

```
public int size() {
        return size;
    }

```

（7）isEmpty方法（判断 list 是否为空 ）

```
	public boolean isEmpty() {
        // 利用size 是否为 0来判断list是否为空
        return size == 0;                 
    } 
```

（8）contains方法（判断该 ArrayList 是否包含指定对象(Object 类型)）

```
	public boolean contains(Object o) {
        // 由索引值判断，大于等于 0 就返回true
        return indexOf(o) >= 0;
    }
```
注意：该方法是面对抽象编程，向上转型是安全的，索引号是从 0 开始计数的

（9）indexOf方法（返回元素最先出现的索引位置）

```
	public int indexOf(Object o) {
        if (o == null) {
            // 顺序查找数组缓冲区
            for (int i = 0; i < size; i++)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = 0; i < size; i++)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }
```

注意：元素可以为 null，空值也可以作为元素放入 ArrayList；Arrays 工具类提供了二分搜索，但没有提供顺序查找

（10）lastIndexOf方法（ 返回此列表中最后一次出现的指定元素的索引 ）

```
    public int lastIndexOf(Object o) {
        if (o == null) {
            // 逆序查找
            for (int i = size-1; i >= 0; i--)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = size-1; i >= 0; i--)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }
```

注：此方法为反向查找(从数组末尾向开始查找)

（11）clone方法（深拷贝）

```
public Object clone() {
        try {
            //调用Object 的克隆方法
            ArrayList<?> v = (ArrayList<?>) super.clone();
            // 对需要进行复制的引用变量，进行独立的拷贝：将存储的元素移入新的 ArrayList 中
            v.elementData = Arrays.copyOf(elementData, size);
            v.modCount = 0;
            return v;
        } catch (CloneNotSupportedException e) {
            throw new InternalError(e);
        }
    }
```

注：此克隆方法（深拷贝）对拷贝出来的 ArrayList 对象的操作，不会影响原来的 ArrayList，而Object 的克隆方法（浅拷贝）：会复制本对象及其内所有基本类型成员和 String 类型成员，但不会复制对象成员、引用对象。

若你不了解深拷贝与浅拷贝，请点击 [深拷贝与浅拷贝详解](http://blog.csdn.net/baiye_xing/article/details/71788741)

（12）toArray方法

* 无参（返回 ArrayList 的 Object 数组 ）
```
	public Object[] toArray() {
        return Arrays.copyOf(elementData, size);
    }
```

注：对返回的该数组进行操作，不会影响该 ArrayList（相当于分配了一个新的数组），该操作是安全的，元素存储顺序与 ArrayList 中的一致。

* 带参数（返回 ArrayList 元素组成的数组 ）

```
	@SuppressWarnings("unchecked")
    public <T> T[] toArray(T[] a) {
        // 若数组a的大小 < ArrayList的元素个数,则新建一个T[]数组，数组大小是"ArrayList的元素个数",并将“ArrayList”全部拷贝到新数组中
        if (a.length < size)
            return (T[]) Arrays.copyOf(elementData, size, a.getClass());
        // 若数组a的大小 >= ArrayList的元素个数,则将ArrayList的全部元素都拷贝到数组a中。
        System.arraycopy(elementData, 0, a, 0, size);
        if (a.length > size)
            a[size] = null;
        return a;
    }
```
注：若 a 中本来存储有元素，则 a 会被 list 的元素覆盖，且 a[list.size] = null

（13）elementData方法（返回在索引为 index 的元素）

```
@SuppressWarnings("unchecked")
    E elementData(int index) {
        return (E) elementData[index];
    }
```

（14）get方法（获取指定位置的元素）

```
	public E get(int index) {
        // 检查是否越界
        rangeCheck(index);
        // 随机访问
        return elementData(index);        
    } 
```
其中rangeCheck方法源码如下：

```
private void rangeCheck(int index) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
```
注：rangeCheck方法并没有检查 index 是否合法，例如：当 index< 0
通常由数组类型自己检查。

（15）set方法（设置 index 位置元素的值）

```
public E set(int index, E element) {
        //  数组越界检查
        rangeCheck(index);
        // 取出旧值
        E oldValue = elementData(index);
        // 替换成新值
        elementData[index] = element;           
        return oldValue;
    }
```

。。。。。。
<font color="blue" size="4">
注：由于篇幅有限，其余方法的源码剖析请参考下一篇博文

<br/>
<br/>
本人才疏学浅，若有错，请指出 
谢谢！