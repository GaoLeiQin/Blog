
###**一、LinkedList概述**

####**1.底层实现**

LinkedList底层是基于双向链表实现的，JDK8和JDK7是这样，但在JDK6之前LinkedList的底层却是基于循环链表实现的。

![这里写图片描述](http://img.blog.csdn.net/20170611195559514?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

####**2.线程安全性**

 LinkedList不是线程安全的，如果在多线程中使用，需要在外部作同步处理。

####**3.与ArrayList的异同**

（1）LinkedList插入和删除操作更加高效，但随机访问速度慢；

（2）LinkedList可以作为栈、队列、双端队列数据结构使用；

（3）与ArrayList、Vector相同，它们的内部迭代器都存在“快速失败”

（4）LinkedList支持null元素、有顺序、元素可以重复；


###**二、源码剖析（JDK8）**

####**1.类的继承关系**

```
public class LinkedList<E>
        extends AbstractSequentialList<E>
        implements List<E>, Deque<E>, Cloneable, java.io.Serializable

```

分析：LinkedList支持泛型，继承于AbstractSequentialList类，实现了List、Deque、Cloneable、java.io.Serializable接口，其实Java中只有 LinkedList继承了AbstractSequentialList类，这个抽象类提供了对序列的连续访问的抽象，它实现的Deque接口，这表示是一个双端队列，等价于LinkedList是双端队列的一种实现，即基于双端队列的操作在LinkedList中都有效。

<font color="blue" size="4">类关系图解

![这里写图片描述](http://img.blog.csdn.net/20170714145854215?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

注：绿色虚线表示实现的接口，黄色实线表示继承的类，绿色实线表示接口间的继承。


####**2.内部类**

```
    private static class Node<E> {
        //数据域
        E item;
        //下一个节点（后继）
        Node<E> next;
        //前一个节点（前驱）
        Node<E> prev;

        //构造函数
        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```

####**3.类的属性**

```
    //指定序列号
    private static final long serialVersionUID = 876323262645176354L;
    //链表的大小
    transient int size = 0;
    //头节点
    transient Node<E> first;
    //尾节点
    transient Node<E> last;
```

注：链表的大小、头节点和尾节点都有transient关键字修饰，说明在序列化时它们不会被序列化。

####**4.构造方法**

（1）无参构造方法（默认）

```
public LinkedList() {
}
```

注：此构造方法会构造一个空链表。

（2）带参数构造器

```
    public LinkedList(Collection<? extends E> c) {
	    //继承无参构造器的方法
        this();
        //添加集合中所有的元素
        addAll(c);
    }
```

注：此构造器会创建一个包含指定 collection 中的元素的链表，这些元素按其 collection 的迭代器返回的顺序排列。

####**5.类的方法**

（1）add方法 

* 参数（E） ：将指定元素添加到此链表的结尾

```
    public boolean add(E e) {
        linkLast(e);
        return true;
    }
```
LinkLast方法源码如下:

```
    void linkLast(E e) {
        //暂存尾节点（不可更改）
        final Node<E> l = last;
        //新生成一个前驱为l，后继为null的节点
        final Node<E> newNode = new Node<>(l, e, null);
        //重新赋值尾节点
        last = newNode;
        if (l == null)
            //若尾节点为空，设置新节点为头结点
            first = newNode;
        else
            //若尾节点不为空,则让尾节点的后继指向新节点
            l.next = newNode;
        //链表大小+1
        size++;
        //结构性修改次数+1
        modCount++;
    }
```

* 参数（int ,E） ： 在此链表中指定的位置插入指定的元素

```
    public void add(int index, E element) {
	    //检查下标是否越界
        checkPositionIndex(index);
		
        if (index == size)
	        //若指定位置等于链表大小，则插入到链表最后
            linkLast(element);
        else
	        //否则插入到指定位置之前
            linkBefore(element, node(index));
    }
```
checkPositionIndex方法的源码如下：
```
    private void checkPositionIndex(int index) {
	    //若不符合下标范围就抛出异常
        if (!isPositionIndex(index))
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
```
outOfBoundsMsg方法源码如下：
```
    private String outOfBoundsMsg(int index) {
        return "Index: "+index+", Size: "+size;
    }
```

isPositionIndex方法源码如下：

```
    private boolean isPositionIndex(int index) {
        return index >= 0 && index <= size;
    }
```

linkBefore方法的源码如下：

```
    void linkBefore(E e, Node<E> succ) {
        //获取前一个节点;
        final Node<E> pred = succ.prev;
        //创建一个前驱指向前一个节点，后继指向当前节点的新节点
        final Node<E> newNode = new Node<>(pred, e, succ);
        //令当前节点的前驱指向新节点
        succ.prev = newNode;
        if (pred == null)
            //若前一个节点为空，则新节点就是头节点
            first = newNode;
        else
            //否则前一个节点的后继指向新节点
            pred.next = newNode;
        //链表长度+1
        size++;
        //结构性修改次数+1
        modCount++;
    }
```

（2）addFirst方法 ：将指定元素插入此链表的开头。

```
    public void addFirst(E e) {
        //添加到链表的首部
        linkFirst(e);
    }
```

linkFirst方法源码如下：

```
    private void linkFirst(E e) {
        //暂存头节点（不可更改）
        final Node<E> f = first;
        //创建一个前驱为空，后继指向头节点的新节点
        final Node<E> newNode = new Node<>(null, e, f);
        //设置新节点为头节点
        first = newNode;
        if (f == null)
            //若头节点为空，尾节点就是头结点
            last = newNode;
        else
            //若头结点不为空，则令头结点的前驱指向新节点
            f.prev = newNode;
        //链表大小+1
        size++;
        //结构性修改次数+1
        modCount++;
    }
```

（3）addLast方法：将指定元素添加到此链表的结尾

```
    public void addLast(E e) {
	    //添加到链表末尾
        linkLast(e);
    }
```
（4）addAll方法

* 参数(Collection< ? extends E> c)：添加指定 collection 中的所有元素到此列表的结尾，顺序是指定 collection 的迭代器返回这些元素的顺序。

```
    public boolean addAll(Collection<? extends E> c) {
	    //从最后开始添加指定集合的元素到列表
        return addAll(size, c);
    }
```
* 参数 （int index, Collection< ? extends E> c) ：将指定 collection 中的所有元素从指定位置开始插入此列表

```

    public boolean addAll(int index, Collection<? extends E> c) {
        //检查下标是否越界
        checkPositionIndex(index);

        //将集合转化为数组
        Object[] a = c.toArray();
        //保存集合的大小
        int numNew = a.length;
        //若集合为空，直接返回false
        if (numNew == 0)
            return false;

        //声明前驱，后继
        Node<E> pred, succ;
        if (index == size) {
            //若插入位置为链表末尾，则后继为空，前驱为尾节点
            succ = null;
            pred = last;
        } else {
            //若插入位置为其他位置，寻找到该节点
            succ = node(index);
            //保存该节点的前驱
            pred = succ.prev;
        }

        //遍历数组
        for (Object o : a) {
            //强制转型并忽略异常
            @SuppressWarnings("unchecked") E e = (E) o;
            //创建一个前驱为该节点前驱，后继为空的新节点
            Node<E> newNode = new Node<>(pred, e, null);
            if (pred == null)
                //若该节点前驱为空，则在第一个元素之前插入
                first = newNode;
            else
                //否则令该节点前驱的后继指向新节点
                pred.next = newNode;
            pred = newNode;
        }

        if (succ == null) {
            //在最后一个元素之前插入
            last = pred;
        } else {
            pred.next = succ;
            succ.prev = pred;
        }

        //将链表的大小设置为集合大小+原链表大小
        size += numNew;
        //结构性修改次数+1
        modCount++;
        return true;
    }
```

（5）getFirst方法 ：返回链表的第一个元素

```
    public E getFirst() {
        //暂存头节点
        final Node<E> f = first;
        //若头结点为空，则抛出异常
        if (f == null)
            throw new NoSuchElementException();
        //返回头节点元素
        return f.item;
    }
```

（6）getLast方法 ： 返回链表的最后一个元素

```
    public E getLast() {
        //暂存尾节点
        final Node<E> l = last;
        //若尾节点为空，抛出异常
        if (l == null)
            throw new NoSuchElementException();
        //返回尾节点元素
        return l.item;
    }

```

（7）removeFirst方法 ：移除并返回链表的第一个元素

```
    public E removeFirst() {
        //暂存头结点
        final Node<E> f = first;
        //若头结点为空，抛出异常
        if (f == null)
            throw new NoSuchElementException();
        return unlinkFirst(f);
    }
```
unlinkFirst方法的源码如下：

```
    private E unlinkFirst(Node<E> f) {
        // 获取头结点值
        final E element = f.item;
        //得到头结点的下一个节点
        final Node<E> next = f.next;
        //将头结点值置为空
        f.item = null;
        //将头结点的下一个节点置为空
        f.next = null;
        //将原头结点的下一个节点设置为头节点
        first = next;
        if (next == null)
            //新的头节点为空，则尾节点也置为空
            last = null;
        else
            //否则将新的头节点的前驱置为空
            next.prev = null;
        //链表的大小-1
        size--;
        //结构性修改次数+1
        modCount++;
        //返回原头节点的值
        return element;
    }
```

（8）removeLast方法 ： 移除并返回链表的最后一个元素

```
    public E removeLast() {
        //暂存尾节点
        final Node<E> l = last;
        //若尾节点为空，抛出异常
        if (l == null)
            throw new NoSuchElementException();
        return unlinkLast(l);
    }
```
unlinkLast方法的源码如下：

```
    private E unlinkLast(Node<E> l) {
        // 获取尾节点的值
        final E element = l.item;
        //获取尾节点的前一个节点
        final Node<E> prev = l.prev;
        //将尾节点的值置为空
        l.item = null;
        //将尾节点的前一个节点置为空
        l.prev = null;
        //将原尾节点的前一个节点置为新的尾节点
        last = prev;
        if (prev == null)
            //若新的尾节点为空，则把头结点也置为空
            first = null;
        else
            //否则将新的尾节点的后继置为空
            prev.next = null;
        //链表的大小-1
        size--;
        //结构性修改次数+1
        modCount++;
        //返回原尾节点的值
        return element;
    }
```

（9）contains方法 ：若链表包含指定元素，则返回 true

```
    public boolean contains(Object o) {
        return indexOf(o) != -1;
    }
```
indexOf方法源码如下：

```
    public int indexOf(Object o) {
        int index = 0;
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null)
                    return index;
                index++;
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item))
                    return index;
                index++;
            }
        }
        return -1;
    }
```

（10）size方法 ：返回链表元素的个数

```
    public int size() {
        return size;
    }
```

（11）remove方法

* 无参  ：获取并移除链表的头（第一个元素）

```
    public E remove() {
        return removeFirst();
    }
```
* 参数(int index)  ：移除链表中指定位置处的元素

```
    public E remove(int index) {
	    //检查元素索引是否超出范围
        checkElementIndex(index);
        return unlink(node(index));
    }
```
checkElementIndex方法源码如下：

```
    private void checkElementIndex(int index) {
        //若元素索引超出范围就抛出异常
        if (!isElementIndex(index))
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
```
isElementIndex方法源码如下：

```
    private boolean isElementIndex(int index) {
        return index >= 0 && index < size;
    }
```

unlink方法源码如下：

```
    E unlink(Node<E> x) {
        //获取当前节点值和前驱后继
        final E element = x.item;
        final Node<E> next = x.next;
        final Node<E> prev = x.prev;

        if (prev == null) {
            //若当前节点的前一个节点为空，则后一个节点为头节点
            first = next;
        } else {
            //否则就指向当前节点的下一个节点
            prev.next = next;
            //将当前节点的前驱置为空
            x.prev = null;
        }

        if (next == null) {
            //若当前节点的下一个节点为空，则当前节点为新的尾节点
            last = prev;
        } else {
            //否则后一个节点的指向前一个节点
            next.prev = prev;
            //后一个节点的后继置为空
            x.next = null;
        }

        //将当前节点值置为空
        x.item = null;
        //链表的大小-1
        size--;
        //结构性修改次数+1
        modCount++;
        //返回被删除的值
        return element;
    }
```

* 参数(Object o)：从链表中移除首次出现的指定元素（如果存在）

```
    public boolean remove(Object o) {
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null) {
                    unlink(x);
                    return true;
                }
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item)) {
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }
```

（12）clear方法 ：移除链表的所有元素

```
    public void clear() {
        //清空链表
        for (Node<E> x = first; x != null; ) {
            Node<E> next = x.next;
            x.item = null;
            x.next = null;
            x.prev = null;
            x = next;
        }
        //头节点和尾节点都置为空
        first = last = null;
        //链表大小置为0
        size = 0;
        //结构性修改次数+1
        modCount++;
    }
```

（13）get方法 ：返回此列表中指定位置处的元素

```
    public E get(int index) {
	    //检查索引是否越界
        checkElementIndex(index);
        return node(index).item;
    }
```
node方法源码如下：

```
    Node<E> node(int index) {
        //如果索引小于链表大小的一半，就从前面查找，否则从后面开始查找
        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
```

（14）set方法 ：将此链表中指定位置的元素替换为指定的元素

```
    public E set(int index, E element) {
        //检查索引是否越界
        checkElementIndex(index);
        //新建一个节点
        Node<E> x = node(index);
        //暂存索引处的值为旧值
        E oldVal = x.item;
        //将指定位置处的值设置为指定值
        x.item = element;
        //返回旧值
        return oldVal;
    }
```
（15）lastIndexOf方法：返回此链表中最后出现的指定元素的索引

```
    public int lastIndexOf(Object o) {
        int index = size;
        //从last开始查找
        if (o == null) {
            for (Node<E> x = last; x != null; x = x.prev) {
                index--;
                if (x.item == null)
                    return index;
            }
        } else {
            for (Node<E> x = last; x != null; x = x.prev) {
                index--;
                if (o.equals(x.item))
                    return index;
            }
        }
        return -1;
    }
```
（16）peek方法 ：获取但不移除此链表的头（第一个元素）。

```
    public E peek() {
        //获取头指针（从前面出栈）
        final Node<E> f = first;
        //若头指针为空，就返回null,否则返回头指针的值
        return (f == null) ? null : f.item;
    }
```
（17）peekFirst方法 ：获取但不移除此链表的第一个元素

```
    public E peekFirst() {
        final Node<E> f = first;
        return (f == null) ? null : f.item;
    }
```
（18）peekLast方法 ： 获取但不移除此链表的最后一个元素

```
    public E peekLast() {
	    //获取尾指针（从后面出栈）
        final Node<E> l = last;
        //若尾指针为空，就返回null,否则返回尾指针的值
        return (l == null) ? null : l.item;
    }
```
（19）element方法 ： 获取但不移除此列表的头（第一个元素）。

```
    public E element() {
        //出栈（从前面），不删除元素
        return getFirst();
    }
```
（20）poll方法 ：获取并移除此链表的头（第一个元素）

```
    public E poll() {
        //获取头指针（从前面出栈）
        final Node<E> f = first;
        //若头指针为空，返回null，否则删除头节点
        return (f == null) ? null : unlinkFirst(f);
    }
```
（21）offer方法 ：将指定元素添加到此链表的末尾（最后一个元素）。

```
    public boolean offer(E e) {
        //添加元素到链表尾部
        return add(e);
    }
```
（22）offerFirst方法 ：在此链表的开头插入指定的元素

```
    public boolean offerFirst(E e) {
        //添加到链表头部
        addFirst(e);
        return true;
    }
```

（23）offerLast方法 ： 在此列表末尾插入指定的元素

```
    public boolean offerLast(E e) {
        //添加到链表尾部
        addLast(e);
        return true;
    }
```


（24）poll方法 ：获取并移除此列表的头（第一个元素）

```
    public E pop() {
        return removeFirst();
    }
```
（25）pollFirst方法 ：获取并移除此列表的第一个元素

```
    public E pollFirst() {
        final Node<E> f = first;
        return (f == null) ? null : unlinkFirst(f);
    }
```
（26）pollLast方法 ：获取并移除此列表的最后一个元素

```
    public E pollLast() {
        final Node<E> l = last;
        return (l == null) ? null : unlinkLast(l);
    }
```
（27）push方法 ： 将元素推入此链表的堆栈

```
    public void push(E e) {
	    //添加到头部
        addFirst(e);
    }
```
（28）removeFirstOccurrence方法 ：从此列表中移除第一次出现的指定元素（从头部到尾部遍历列表时）

```
    public boolean removeFirstOccurrence(Object o) {
        return remove(o);
    }
```

（29）removeLastOccurrence方法 ： 从此列表中移除最后一次出现的指定元素（从头部到尾部遍历列表时）

```
    public boolean removeLastOccurrence(Object o) {
        //从last开始遍历
        if (o == null) {
            for (Node<E> x = last; x != null; x = x.prev) {
                if (x.item == null) {
                    unlink(x);
                    return true;
                }
            }
        } else {
            for (Node<E> x = last; x != null; x = x.prev) {
                if (o.equals(x.item)) {
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }
```

（30）clone方法 ：返回此 LinkedList 的浅表副本

```
    public Object clone() {
        //继承Object的clone方法
        LinkedList<E> clone = superClone();

        //将克隆后的副本的头指针和尾指针都置为空
        clone.first = clone.last = null;
        //将副本链表大小和结构性修改次数置为0
        clone.size = 0;
        clone.modCount = 0;

        //初始化副本
        for (Node<E> x = first; x != null; x = x.next)
            clone.add(x.item);

        //返回副本
        return clone;
    }
```
superClone方法源码如下：

```
	//忽略类型转换异常
    @SuppressWarnings("unchecked")
    private LinkedList<E> superClone() {
        try {
            //继承Object的cone方法并强制转型为LinkedList
            return (LinkedList<E>) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new InternalError(e);
        }
    }
```
（31）toArray方法

* 无参 ：返回以适当顺序（从第一个元素到最后一个元素）包含此链表中所有元素的数组

```
    public Object[] toArray() {
        //创建一个大小为size的Object
        Object[] result = new Object[size];
        int i = 0;
        //获取链表中的值
        for (Node<E> x = first; x != null; x = x.next)
            result[i++] = x.item;
        //返回Object
        return result;
    }
```

* 参数 (T[] a)  ：返回以适当顺序（从第一个元素到最后一个元素）包含此链表中所有元素的数组；返回数组的运行时类型为指定数组的类型

```
    @SuppressWarnings("unchecked")
    public <T> T[] toArray(T[] a) {
        //若指定类型的长度小于size，就创建一个size大小的指定类型
        if (a.length < size)
            a = (T[])java.lang.reflect.Array.newInstance(
                    a.getClass().getComponentType(), size);
        int i = 0;
        //将a转为Object类型
        Object[] result = a;
        //将链表的值赋给result
        for (Node<E> x = first; x != null; x = x.next)
            result[i++] = x.item;

        //若a的长度大于size,则把索引为size的值置为null
        if (a.length > size)
            a[size] = null;

        return a;
    }
```

（32）listIterator方法 ： 返回此列表中的元素的列表迭代器（按适当顺序），从链表中指定位置开始

```
    public ListIterator<E> listIterator(int index) {
	    //检查索引是否越界
        checkPositionIndex(index);
        return new ListItr(index);
    }
```
ListItr类源码如下

```
private class ListItr implements ListIterator<E> {
        private Node<E> lastReturned;
        private Node<E> next;
        private int nextIndex;
        private int expectedModCount = modCount;

        ListItr(int index) {
            // assert isPositionIndex(index);
            next = (index == size) ? null : node(index);
            nextIndex = index;
        }

        public boolean hasNext() {
            return nextIndex < size;
        }

        public E next() {
            checkForComodification();
            if (!hasNext())
                throw new NoSuchElementException();

            lastReturned = next;
            next = next.next;
            nextIndex++;
            return lastReturned.item;
        }

        public boolean hasPrevious() {
            return nextIndex > 0;
        }

        public E previous() {
            checkForComodification();
            if (!hasPrevious())
                throw new NoSuchElementException();

            lastReturned = next = (next == null) ? last : next.prev;
            nextIndex--;
            return lastReturned.item;
        }

        public int nextIndex() {
            return nextIndex;
        }

        public int previousIndex() {
            return nextIndex - 1;
        }

        public void remove() {
            checkForComodification();
            if (lastReturned == null)
                throw new IllegalStateException();

            Node<E> lastNext = lastReturned.next;
            unlink(lastReturned);
            if (next == lastReturned)
                next = lastNext;
            else
                nextIndex--;
            lastReturned = null;
            expectedModCount++;
        }

        public void set(E e) {
            if (lastReturned == null)
                throw new IllegalStateException();
            checkForComodification();
            lastReturned.item = e;
        }

        public void add(E e) {
            checkForComodification();
            lastReturned = null;
            if (next == null)
                linkLast(e);
            else
                linkBefore(e, next);
            nextIndex++;
            expectedModCount++;
        }

        public void forEachRemaining(Consumer<? super E> action) {
            Objects.requireNonNull(action);
            while (modCount == expectedModCount && nextIndex < size) {
                action.accept(next.item);
                lastReturned = next;
                next = next.next;
                nextIndex++;
            }
            checkForComodification();
        }

        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }

```

（33）descendingIterator方法 ：递减迭代器

```
    public Iterator<E> descendingIterator() {
        return new DescendingIterator();
    }
```
DescendingIterator类源码如下

```
    private class DescendingIterator implements Iterator<E> {
        private final ListItr itr = new ListItr(size());
        public boolean hasNext() {
            return itr.hasPrevious();
        }
        public E next() {
            return itr.previous();
        }
        public void remove() {
            itr.remove();
        }
    }
```
（34）spliterator方法 ：并行遍历

```
   public Spliterator<E> spliterator() {
        return new LLSpliterator<E>(this, -1, 0);
    }
```
LLSpliterator类的源码如下：

```
static final class LLSpliterator<E> implements Spliterator<E> {
        static final int BATCH_UNIT = 1 << 10;  // batch array size increment
        static final int MAX_BATCH = 1 << 25;  // max batch array size;
        final LinkedList<E> list; // null OK unless traversed
        Node<E> current;      // current node; null until initialized
        int est;              // size estimate; -1 until first needed
        int expectedModCount; // initialized when est set
        int batch;            // batch size for splits

        LLSpliterator(LinkedList<E> list, int est, int expectedModCount) {
            this.list = list;
            this.est = est;
            this.expectedModCount = expectedModCount;
        }

        final int getEst() {
            int s; // force initialization
            final LinkedList<E> lst;
            if ((s = est) < 0) {
                if ((lst = list) == null)
                    s = est = 0;
                else {
                    expectedModCount = lst.modCount;
                    current = lst.first;
                    s = est = lst.size;
                }
            }
            return s;
        }

        public long estimateSize() { return (long) getEst(); }

        public Spliterator<E> trySplit() {
            Node<E> p;
            int s = getEst();
            if (s > 1 && (p = current) != null) {
                int n = batch + BATCH_UNIT;
                if (n > s)
                    n = s;
                if (n > MAX_BATCH)
                    n = MAX_BATCH;
                Object[] a = new Object[n];
                int j = 0;
                do { a[j++] = p.item; } while ((p = p.next) != null && j < n);
                current = p;
                batch = j;
                est = s - j;
                return Spliterators.spliterator(a, 0, j, Spliterator.ORDERED);
            }
            return null;
        }

        public void forEachRemaining(Consumer<? super E> action) {
            Node<E> p; int n;
            if (action == null) throw new NullPointerException();
            if ((n = getEst()) > 0 && (p = current) != null) {
                current = null;
                est = 0;
                do {
                    E e = p.item;
                    p = p.next;
                    action.accept(e);
                } while (p != null && --n > 0);
            }
            if (list.modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }

        public boolean tryAdvance(Consumer<? super E> action) {
            Node<E> p;
            if (action == null) throw new NullPointerException();
            if (getEst() > 0 && (p = current) != null) {
                --est;
                E e = p.item;
                current = p.next;
                action.accept(e);
                if (list.modCount != expectedModCount)
                    throw new ConcurrentModificationException();
                return true;
            }
            return false;
        }

        public int characteristics() {
            return Spliterator.ORDERED | Spliterator.SIZED | Spliterator.SUBSIZED;
        }
    }

```

<font size="3" color="red">注：Spliterator可以理解为Iterator的Split版本（但用途要丰富很多）。使用Iterator的时候，我们可以顺序地遍历容器中的元素，使用Spliterator的时候，我们可以将元素分割成多份，分别交于不于的线程去遍历，以提高效率。使用 Spliterator 每次可以处理某个元素集合中的一个元素 , 不是从 Spliterator 中获取元素，而是使用 tryAdvance() 或 forEachRemaining() 方法对元素应用操作。但Spliterator 还可以用于估计其中保存的元素数量，而且还可以像细胞分裂一样变为一分为二。这些新增加的能力让流并行处理代码可以很方便地将工作分布到多个可用线程上完成。

(35)writeObject方法 ：序列化

```
    private void writeObject(java.io.ObjectOutputStream s)
            throws java.io.IOException {
        // 写出任何的隐藏的“魔法”序列化
        s.defaultWriteObject();

        //写出的大小为size
        s.writeInt(size);

        //写出链表的所有元素值
        for (Node<E> x = first; x != null; x = x.next)
            s.writeObject(x.item);
    }
```

（36）readObject方法 ：反序列化

```
    @SuppressWarnings("unchecked")
    private void readObject(java.io.ObjectInputStream s)
            throws java.io.IOException, ClassNotFoundException {
        // 读入任何隐藏的“魔法”序列化
        s.defaultReadObject();

        //读入大小为链表的大小size
        int size = s.readInt();

        //读入所有的元素并强制转型为LinkLast
        for (int i = 0; i < size; i++)
            linkLast((E)s.readObject());
    }
```

###**总结**

通过分析LinkedList的源码可知：

（1）如果需要经常插入和删除元素就需要LinkedList，ArrayList要拷贝移动数据，但若需要快速访问数据，很少或不插入和删除元素，就应该用ArrayList，LinkedList要移动指针。

（2）LinkedList是基于双链表实现的，而List底层是单链表。

（3）LinkedList是通过节点直接连接来实现的。每一个节点都包含前一个节点的引用，后一个节点的引用和节点存储的值。当一个新节点插入时，只需要修改其中保持先后关系的节点的引用即可，删除相同。



<br/>
<br/>
本人才疏学浅。若有错误，请指出~
谢谢！