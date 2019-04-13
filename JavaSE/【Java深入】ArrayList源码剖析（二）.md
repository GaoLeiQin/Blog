###**类的方法（续）**

（16）add方法 

* add(E )方法 ： 添加新值到 list 末尾
```
	public boolean add(E e) {
        // 确定ArrayList的容量大小(modCount++)
        ensureCapacityInternal(size + 1);
        // 添加 e 到 ArrayList 中，然后 size 自增 1
        elementData[size++] = e;
        return true;
    }
```
注：size + 1是为保证资源空间不被浪费。

* add(int , E)方法 ： 在指定位置插入新元素

```
    public void add(int index, E element) {
        rangeCheckForAdd(index);
        // modCount值会自增1
        ensureCapacityInternal(size + 1); 
        //在原来 index 位置的值往后移动一位
        System.arraycopy(elementData, index, elementData, index + 1,
                size - index);
        elementData[index] = element;
        size++;
    }
```

其中rangeCheckForAdd方法的源码如下：

```
    private void rangeCheckForAdd(int index) {
        if (index > size || index < 0)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
```

其中outOfBoundsMsg方法返回异常消息，用于传给 IndexOutOfBoundsException

源码如下：

```
    private String outOfBoundsMsg(int index) {
        return "Index: "+index+", Size: "+size;
    }

```

（17）remove方法

* remove(int)方法 ： 移除指定索引位置的元素

```
    public E remove(int index) {
        // 越界检查
        rangeCheck(index);
        modCount++;
        // 旧值
        E oldValue = elementData(index);
        // 需要左移的元素个数
        int numMoved = size - index - 1;
        if (numMoved > 0)
            // 将源数组从下标为 index+1 开始的元素，拷贝到目标数组下标为 index的位置，一共拷贝numMoved个
            System.arraycopy(elementData, index+1, elementData, index, numMoved);
        // 将最后一个元素置空
        elementData[--size] = null;  // clear to let GC do its work
        return oldValue;
    }
```
注：该方法利用了 System.arraycopy() 进行左移一位的操作

* remove(Object)方法 ： 删除一个指定元素

```
    public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                // 判断是否存储了 null
                if (elementData[index] == null) {       
                    fastRemove(index);
                    return true;
                }
        } else {
            // 遍历ArrayList，找到o，则删除，并返回true
            for (int index = 0; index < size; index++)
                // 利用 equals 判断两对象值是否相等
                if (o.equals(elementData[index])) {     
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }

```

注：该方法删除的某个元素，若有多个相同值，则删除的是符合条件的结果中索引号最小的那个元素，若不包含要删除的元素，则返回 false，相比 remove(index)，该方法并没有进行越界检查，即调用 rangeCheck()。

其中fastRemove方法的源码如下：

```
    private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        // 左移操作
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                    numMoved);
        //将最后一个元素设为null
        elementData[--size] = null; // clear to let GC do its work
    }
```

（18）clear方法（清空所有存储元素）

```
    public void clear() {
        //记录修改次数
        modCount++;
        // clear to let GC do its work
        for (int i = 0; i < size; i++)
            elementData[i] = null;
        size = 0;
    }
```
注：该方法会将数组缓冲区所以元素置为 null，清空后，若打印 list，却只会看见一个 [], 而不是 [null, null, ....]，这是因为迭代器进行了处理。

（19）addAll方法

* addAll(Collection<? extends E> c)方法 ： 将一个集合的所有元素顺序添加到ArrayList末尾

```
    public boolean addAll(Collection<? extends E> c) {
        // 若 c 为 null，此行将抛出空指针异常
        Object[] a = c.toArray();
        // 要添加的元素个数
        int numNew = a.length;
        //modCount的值会自增1
        ensureCapacityInternal(size + numNew);
        //添加集合元素到list中
        System.arraycopy(a, 0, elementData, size, numNew);
        size += numNew;

        return numNew != 0;
    }

```

注：c不能null，否则会报NPE，由于ArrayList 是线程不安全的，该方法没有加锁，所以当一个线程正在将 c 中的元素加入 list 中，但同时有另一个线程在更改 c 中的元素，就会抛出ConcurrentModificationException（并发修改异常）

* addAll(int index, Collection< ? extends E> c)方法 ： 从 index 位置开始，将集合 c 中的元素添加到ArrayList

```
    public boolean addAll(int index, Collection<? extends E> c) {
        // 越界检查
        rangeCheckForAdd(index);
        // 若c为null，抛出ＮＰＥ
        Object[] a = c.toArray();
        int numNew = a.length;
        //modCount的值会自增1
        ensureCapacityInternal(size + numNew);  // Increments modCount
        // 要移动的元素个数
        int numMoved = size - index;
        //先移动，然后拷贝（以免被覆盖）
        if (numMoved > 0)
            System.arraycopy(elementData, index, elementData, index + numNew,
                    numMoved);
        System.arraycopy(a, 0, elementData, index, numNew);
        size += numNew;
        return numNew != 0;
    }
```

注：该方法并不会覆盖掉在 index 位置原有的值（类似于 insert 方法）

（20）removeRange ( int fromIndex, int toIndex ) 方法  ： 删除fromIndex到toIndex之间的全部元素

```

    protected void removeRange(int fromIndex, int toIndex) {
        modCount++;
        //需要移动元素的个数
        int numMoved = size - toIndex;
        //进行元素拷贝后，需要删除的几个元素就复制到了最后几个位置
        System.arraycopy(elementData, toIndex, elementData, fromIndex,
                numMoved);
        // clear to let GC do its work
        // 删除后新的长度
        int newSize = size - (toIndex-fromIndex);
        // 将需要删除的元素（index在最后几个）置为 null
        for (int i = newSize; i < size; i++) {
            elementData[i] = null;
        }
        size = newSize;
    }

```

注：该方法的区间：fromIndex（包括）和 toIndex（不包括），助记：左闭右开

（21）removeAll方法 （移除 list 中和 c 中共有的元素）

```

    public boolean removeAll(Collection<?> c) {
        // 当 c == null，则抛出NPE
        Objects.requireNonNull(c); 
        //批量删除c             
        return batchRemove(c, false);
    }
```

注：该方法是从类 java.util.AbstractCollection 继承的方法 


批量删除batchRemove方法源码如下：


```
    private boolean batchRemove(Collection<?> c, boolean complement) {
        final Object[] elementData = this.elementData;
        int r = 0, w = 0;
        boolean modified = false;
        try {
            for (; r < size; r++)
                if (c.contains(elementData[r]) == complement)
                    elementData[w++] = elementData[r];
        } finally {
            // Preserve behavioral compatibility with AbstractCollection,
            // even if c.contains() throws.
            if (r != size) {
                System.arraycopy(elementData, r,
                                 elementData, w,
                                 size - r);
                w += size - r;
            }
            if (w != size) {
                // clear to let GC do its work
                for (int i = w; i < size; i++)
                    elementData[i] = null;
                modCount += size - w;
                size = w;
                modified = true;
            }
        }
        return modified;
    }

```


（22）retainAll方法（只保留 list 和 集合 c 中公有的元素）


```
    public boolean retainAll(Collection<?> c) {
        Objects.requireNonNull(c);
        return batchRemove(c, true);
    }
```

注：该方法是从类 java.util.AbstractCollection 继承的方法，与removeAll() 功能相反


（23）writeObject方法（写序列化）

```
    private void writeObject(ObjectOutputStream s) throws IOException{
        int expectedModCount = modCount;
        s.defaultWriteObject();
        // 写入ArrayList大小
        s.writeInt(size);
        // 写入存储的元素
        for (int i=0; i<size; i++) {
            s.writeObject(elementData[i]);
        }
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }
```

注：若在执行该方法的同时，有另一个线程也对elementData对象进行修改，这时候modCount（记录修改次数）的值会改变，所以就抛出ConcurrentModificationException。

（24）readObject方法（读序列化）

```
    private void readObject(java.io.ObjectInputStream s)
            throws java.io.IOException, ClassNotFoundException {
        elementData = EMPTY_ELEMENTDATA;
        // 读取s（包括隐藏的值）
        s.defaultReadObject();
        // 从输入流中读取ArrayList的size
        s.readInt(); // ignored
        if (size > 0) {
            ensureCapacityInternal(size);
            Object[] a = elementData;
            // 从输入流中将所有的元素值读出
            for (int i=0; i<size; i++) {
                a[i] = s.readObject();
            }
        }
    }
```

（25）listIterator方法

* listIterator(int index)方法 ：  返回列表中元素的列表迭代器（按适当顺序），从列表的指定位置开始

```
    public ListIterator<E> listIterator(int index) {
        if (index < 0 || index > size)
            throw new IndexOutOfBoundsException("Index: "+index);
        return new ListItr(index);
    }
```

注：该方法中创建的ListItr类是AbstractList.ListItr 的优化版本
源码如下：

```
    private class ListItr extends Itr implements ListIterator<E> {
        ListItr(int index) {
            super();
            cursor = index;                                             // cursor 还是指向下一个返回元素的索引位置
        }
        //是否有上一个元素
        public boolean hasPrevious() {
            return cursor != 0;
        }
        //获取下一个元素的索引
        public int nextIndex() {
            return cursor;
        }
        //获取 cursor 前一个元素的索引(不是当前元素前一个的索引)
        public int previousIndex() {
            return cursor - 1;
        }
        //返回 cursor 的前一元素
        @SuppressWarnings("unchecked")
        public E previous() {
            checkForComodification();
            int i = cursor - 1;
            if (i < 0)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i;
            return (E) elementData[lastRet = i];
        }
        //指定元素(将游标当前指向的索引位置的值设置为指定元素)
        public void set(E e) {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                ArrayList.this.set(lastRet, e);
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }
        //添加元素(在游标当前指向的索引位置插入一个元素)
        public void add(E e) {
            checkForComodification();

            try {
                int i = cursor;
                ArrayList.this.add(i, e);
                cursor = i + 1;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }
    }
```


* listIterator() 方法 ： 返回此列表元素的列表迭代器（按适当顺序）

```
    public ListIterator<E> listIterator() {
        return new ListItr(0);
    }
```

（26）iterator方法 （返回以恰当顺序在此列表的元素上进行迭代的迭代器）

```
    public Iterator<E> iterator() {
        return new Itr();
    }
```

注：该方法返回一个 Iterator 迭代器，该迭代器是 fail-fast 机制的，Itr类是AbstractList.Itr类 的优化版本。

Itr类源码如下：

```
    private class Itr implements Iterator<E> {
        // 下一个返回元素的索引，默认值为 0
        int cursor;
        // 上一个返回元素的索引，若没有上一个元素，则为 -1。每次调用 remove()，lastRet 都会重置为 -1
        int lastRet = -1;                                           
        int expectedModCount = modCount;

        public boolean hasNext() {
            // 判断是否有下一元素
            return cursor != size;                             
        }

        @SuppressWarnings("unchecked")
        public E next() {
            checkForComodification();
            // 临时变量 i，指向游标当前位置。
            int i = cursor;
            //  第一次检查 
            if (i >= size)                                                                  
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            // 第二次检查
            if (i >= elementData.length)                                         
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];                                
        }

        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                ArrayList.this.remove(lastRet); 
                // 移除元素                                
                cursor = lastRet;
                // 指针回移                                                       
                //lastRet 不一定就等于 cursour - 1
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

        @Override
        @SuppressWarnings("unchecked")
        public void forEachRemaining(Consumer<? super E> consumer) {
            // 非空判断
            Objects.requireNonNull(consumer);                               
            final int size = ArrayList.this.size;
            int i = cursor;
            if (i >= size) {
                return;
            }
            final Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length) {
                throw new ConcurrentModificationException();
            }
            while (i != size && modCount == expectedModCount) {
                consumer.accept((E) elementData[i++]);
            }
            // update once at end of iteration to reduce heap write traffic
            cursor = i;
            lastRet = i - 1;
            checkForComodification();
        }

        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }
```

注： Iterator 与ListIterator 的区别：ListIterator可以进行双向移动，而Iterator只能单向移动，ListIterator还可以添加元素(用add方法实现)，而Iterator不能添加元素。

（27）subList方法 ： 获取从 fromIndex 到 toIndex 之间的子集合(左闭右开)

```
 public List<E> subList(int fromIndex, int toIndex) {
		// 合法性检查
        subListRangeCheck(fromIndex, toIndex, size);            
        return new SubList(this, 0, fromIndex, toIndex);
    }
```

注：若 fromIndex == toIndex，则返回的空集合，对该子集合的操作，会影响原有集合

* subListRangeCheck方法用来检测输入的下标界限是否合法，源码如下：

```
static void subListRangeCheck(int fromIndex, int toIndex, int size) {
        //越界检查
        if (fromIndex < 0)
            throw new IndexOutOfBoundsException("fromIndex = " + fromIndex);
        if (toIndex > size)
            throw new IndexOutOfBoundsException("toIndex = " + toIndex);
        //非法参数检查
        if (fromIndex > toIndex)
            throw new IllegalArgumentException("fromIndex(" + fromIndex +
                    ") > toIndex(" + toIndex + ")");
    }
```


* 嵌套内部类SubList也实现了 RandomAccess，提供快速随机访问特性，源码如下：

```
 private class SubList extends AbstractList<E> implements RandomAccess {
        private final AbstractList<E> parent;
        // 相对于父集合的偏移量(即fromIndex)
        private final int parentOffset;
        // 偏移量，默认是 0
        private final int offset;
        // SubList 中存储元素的个数
        int size;

        //因为属性是final的，所以在取子集合后，父集合不能删除 SubList 中的首个元素（offset不会变）
        SubList(AbstractList<E> parent,
                int offset, int fromIndex, int toIndex) {
            // 子集合的处理，仅给出了一个映射到父集合相应区间的引用
            this.parent = parent;
            this.parentOffset = fromIndex;
            this.offset = offset + fromIndex;
            this.size = toIndex - fromIndex;
            this.modCount = ArrayList.this.modCount;
        }

        // 设置新值，返回旧值
        public E set(int index, E e) {
            rangeCheck(index);
            // 越界检查
            checkForComodification();
            E oldValue = ArrayList.this.elementData(offset + index);
            ArrayList.this.elementData[offset + index] = e;
            return oldValue;
        }

        // 取值
        public E get(int index) {
            rangeCheck(index);
            // 越界检查
            checkForComodification();
            return ArrayList.this.elementData(offset + index);
        }
        //获取SubList大小
        public int size() {
            checkForComodification();
            return this.size;
        }
        // 添加元素
        public void add(int index, E e) {
            rangeCheckForAdd(index);
            checkForComodification();
            // 对子类添加元素，是直接操作父类添加的
            parent.add(parentOffset + index, e);
            this.modCount = parent.modCount;
            this.size++;
        }
        // 删除元素
        public E remove(int index) {
            rangeCheck(index);
            checkForComodification();
            // 对子类删除元素，是直接操作父类删除的
            E result = parent.remove(parentOffset + index);
            this.modCount = parent.modCount;
            this.size--;
            return result;
        }

        // 范围删除
        protected void removeRange(int fromIndex, int toIndex) {
            checkForComodification();
            parent.removeRange(parentOffset + fromIndex,
                    parentOffset + toIndex);
            this.modCount = parent.modCount;
            this.size -= toIndex - fromIndex;
        }
        //将一个集合的所有元素顺序添加到 lits 末尾
        public boolean addAll(Collection<? extends E> c) {
            return addAll(this.size, c);
        }
        //从 index 位置开始，将集合 c 中的元素添加到ArrayList
        public boolean addAll(int index, Collection<? extends E> c) {
            rangeCheckForAdd(index);
            int cSize = c.size();
            if (cSize==0)
                return false;

            checkForComodification();
            parent.addAll(parentOffset + index, c);
            this.modCount = parent.modCount;
            this.size += cSize;
            return true;
        }
        //返回一个Iterator迭代器，但底层实现的是ListIterator
        public Iterator<E> iterator() {
            return listIterator();
        }
        // 返回一个 ListIterator迭代器
        public ListIterator<E> listIterator(final int index) {
            checkForComodification();
            // 越界检查（使用了add方法专用的越界检查方法）
            rangeCheckForAdd(index);
            final int offset = this.offset;

            // 匿名内部类
            return new ListIterator<E>() {
                int cursor = index;
                int lastRet = -1;
                int expectedModCount = ArrayList.this.modCount;

                public boolean hasNext() {
                    return cursor != SubList.this.size;
                }

                @SuppressWarnings("unchecked")
                public E next() {
                    checkForComodification();
                    int i = cursor;
                    if (i >= SubList.this.size)
                        throw new NoSuchElementException();
                    Object[] elementData = ArrayList.this.elementData;
                    if (offset + i >= elementData.length)
                        throw new ConcurrentModificationException();
                    cursor = i + 1;
                    return (E) elementData[offset + (lastRet = i)];
                }

                public boolean hasPrevious() {
                    return cursor != 0;
                }

                @SuppressWarnings("unchecked")
                public E previous() {
                    checkForComodification();
                    int i = cursor - 1;
                    if (i < 0)
                        throw new NoSuchElementException();
                    Object[] elementData = ArrayList.this.elementData;
                    if (offset + i >= elementData.length)
                        throw new ConcurrentModificationException();
                    cursor = i;
                    return (E) elementData[offset + (lastRet = i)];
                }

                @SuppressWarnings("unchecked")
                public void forEachRemaining(Consumer<? super E> consumer) {
                    Objects.requireNonNull(consumer);
                    final int size = SubList.this.size;
                    int i = cursor;
                    if (i >= size) {
                        return;
                    }
                    final Object[] elementData = ArrayList.this.elementData;
                    if (offset + i >= elementData.length) {
                        throw new ConcurrentModificationException();
                    }
                    while (i != size && modCount == expectedModCount) {
                        consumer.accept((E) elementData[offset + (i++)]);
                    }
                    // update once at end of iteration to reduce heap write traffic
                    lastRet = cursor = i;
                    checkForComodification();
                }

                public int nextIndex() {
                    return cursor;
                }

                public int previousIndex() {
                    return cursor - 1;
                }

                public void remove() {
                    if (lastRet < 0)
                        throw new IllegalStateException();
                    checkForComodification();

                    try {
                        SubList.this.remove(lastRet);
                        cursor = lastRet;
                        lastRet = -1;
                        expectedModCount = ArrayList.this.modCount;
                    } catch (IndexOutOfBoundsException ex) {
                        throw new ConcurrentModificationException();
                    }
                }

                public void set(E e) {
                    if (lastRet < 0)
                        throw new IllegalStateException();
                    checkForComodification();

                    try {
                        ArrayList.this.set(offset + lastRet, e);
                    } catch (IndexOutOfBoundsException ex) {
                        throw new ConcurrentModificationException();
                    }
                }

                public void add(E e) {
                    checkForComodification();

                    try {
                        int i = cursor;
                        SubList.this.add(i, e);
                        cursor = i + 1;
                        lastRet = -1;
                        expectedModCount = ArrayList.this.modCount;
                    } catch (IndexOutOfBoundsException ex) {
                        throw new ConcurrentModificationException();
                    }
                }

                final void checkForComodification() {
                    if (expectedModCount != ArrayList.this.modCount)
                        throw new ConcurrentModificationException();
                }
            };
        }

        public List<E> subList(int fromIndex, int toIndex) {
            subListRangeCheck(fromIndex, toIndex, size);
            return new SubList(this, offset, fromIndex, toIndex);
        }

        private void rangeCheck(int index) {
            if (index < 0 || index >= this.size)
                throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
        }

        private void rangeCheckForAdd(int index) {
            if (index < 0 || index > this.size)
                throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
        }

        private String outOfBoundsMsg(int index) {
            return "Index: "+index+", Size: "+this.size;
        }

        private void checkForComodification() {
            if (ArrayList.this.modCount != this.modCount)
                throw new ConcurrentModificationException();
        }

        public Spliterator<E> spliterator() {
            checkForComodification();
            return new ArrayListSpliterator<E>(ArrayList.this, offset,
                    offset + this.size, this.modCount);
        }
    }
```

注：SubList仅给出了一个映射到父集合相应区间的引用。

（28）forEach方法（用于函数式编程）

```
    @Override
    public void forEach(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        final int expectedModCount = modCount;
        @SuppressWarnings("unchecked")
        final E[] elementData = (E[]) this.elementData;
        final int size = this.size;
        for (int i=0; modCount == expectedModCount && i < size; i++) {
            action.accept(elementData[i]);
        }
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }
```

（29）spliterator方法 （获取一个分割器）

```
    @Override
    public Spliterator<E> spliterator() {
        return new ArrayListSpliterator<>(this, 0, -1, 0);
    }
```

分割器类ArrayListSpliterator的源码如下：

```
	// 基于索引的、二分的、懒加载的分割器
    static final class ArrayListSpliterator<E> implements Spliterator<E> {
        private final ArrayList<E> list;
        private int index; // current index, modified on advance/split
        private int fence; // -1 until used; then one past last index
        private int expectedModCount; // initialized when fence set

        // Create new spliterator covering the given  range
        ArrayListSpliterator(ArrayList<E> list, int origin, int fence,
                             int expectedModCount) {
            this.list = list; // OK if null unless traversed
            this.index = origin;
            this.fence = fence;
            this.expectedModCount = expectedModCount;
        }

        private int getFence() { // initialize fence to size on first use
            int hi; // (a specialized variant appears in method forEach)
            ArrayList<E> lst;
            if ((hi = fence) < 0) {
                if ((lst = list) == null)
                    hi = fence = 0;
                else {
                    expectedModCount = lst.modCount;
                    hi = fence = lst.size;
                }
            }
            return hi;
        }

        public ArrayListSpliterator<E> trySplit() {
            int hi = getFence(), lo = index, mid = (lo + hi) >>> 1;
            return (lo >= mid) ? null : // divide range in half unless too small
                    new ArrayListSpliterator<E>(list, lo, index = mid,
                            expectedModCount);
        }

        public boolean tryAdvance(Consumer<? super E> action) {
            if (action == null)
                throw new NullPointerException();
            int hi = getFence(), i = index;
            if (i < hi) {
                index = i + 1;
                @SuppressWarnings("unchecked") E e = (E)list.elementData[i];
                action.accept(e);
                if (list.modCount != expectedModCount)
                    throw new ConcurrentModificationException();
                return true;
            }
            return false;
        }

        public void forEachRemaining(Consumer<? super E> action) {
            int i, hi, mc; // hoist accesses and checks from loop
            ArrayList<E> lst; Object[] a;
            if (action == null)
                throw new NullPointerException();
            if ((lst = list) != null && (a = lst.elementData) != null) {
                if ((hi = fence) < 0) {
                    mc = lst.modCount;
                    hi = lst.size;
                }
                else
                    mc = expectedModCount;
                if ((i = index) >= 0 && (index = hi) <= a.length) {
                    for (; i < hi; ++i) {
                        @SuppressWarnings("unchecked") E e = (E) a[i];
                        action.accept(e);
                    }
                    if (lst.modCount == mc)
                        return;
                }
            }
            throw new ConcurrentModificationException();
        }

        public long estimateSize() {
            return (long) (getFence() - index);
        }

        public int characteristics() {
            return Spliterator.ORDERED | Spliterator.SIZED | Spliterator.SUBSIZED;
        }
    }
```

（30）removeIf方法

```
    @Override
    public boolean removeIf(Predicate<? super E> filter) {
        Objects.requireNonNull(filter);
        // figure out which elements are to be removed
        // any exception thrown from the filter predicate at this stage
        // will leave the collection unmodified
        int removeCount = 0;
        final BitSet removeSet = new BitSet(size);
        final int expectedModCount = modCount;
        final int size = this.size;
        for (int i=0; modCount == expectedModCount && i < size; i++) {
            @SuppressWarnings("unchecked")
            final E element = (E) elementData[i];
            if (filter.test(element)) {
                removeSet.set(i);
                removeCount++;
            }
        }
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }

        // shift surviving elements left over the spaces left by removed elements
        final boolean anyToRemove = removeCount > 0;
        if (anyToRemove) {
            final int newSize = size - removeCount;
            for (int i=0, j=0; (i < size) && (j < newSize); i++, j++) {
                i = removeSet.nextClearBit(i);
                elementData[j] = elementData[i];
            }
            for (int k=newSize; k < size; k++) {
                elementData[k] = null;  // Let gc do its work
            }
            this.size = newSize;
            if (modCount != expectedModCount) {
                throw new ConcurrentModificationException();
            }
            modCount++;
        }

        return anyToRemove;
    }
```


（31）replaceAll方法

```
    @Override
    @SuppressWarnings("unchecked")
    public void replaceAll(UnaryOperator<E> operator) {
        Objects.requireNonNull(operator);
        final int expectedModCount = modCount;
        final int size = this.size;
        for (int i=0; modCount == expectedModCount && i < size; i++) {
            elementData[i] = operator.apply((E) elementData[i]);
        }
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
        modCount++;
    }

```



（32）sort方法

```
    @Override
    @SuppressWarnings("unchecked")
    public void sort(Comparator<? super E> c) {
        final int expectedModCount = modCount;
        Arrays.sort((E[]) elementData, 0, size, c);
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
        modCount++;
    }
```

###**和LinkedList、Vector的区别**

####**1.和LinkedList的区别**

* ArrayList是实现了基于动态数组的数据结构，LinkedList基于链表的数据结构。

* 对于随机访问get和set，ArrayList优于LinkedList，因为LinkedList要移动指针。

* 对于新增和删除操作add和remove（不是在尾部添加删除），LinkedList比较占优势，因为ArrayList要移动数据。


####**2.和Vector的区别**

* Vector和ArrayList几乎是完全相同的,唯一的区别在于Vector是同步类(synchronized)，属于强同步类。因此开销就比ArrayList要大，访问要慢。

* Vector每次扩容请求其大小的2倍空间，而ArrayList是1.5倍。

* Vector还有一个子类Stack.


###**总结**：

(1) ArrayList 基于数组实现，其内存储元素的数组为 elementData，声明为：transient Object[] elementData;

(2) ArrayList 中EMPTY_ELEMENTDATA 和 DEFAULTCAPACITY_EMPTY._ELEMENTDATA 的使用
 这两个常量，使用场景不同。前者是用在用户通过 ArrayList(int initialCapacity) 该构造方法直接指定初试容量为 0 时，后者是用户直接使用无参构造创建 ArrayList 时。

(3) ArrayList 默认容量为 10。

(4) ArrayList 的扩容计算为 newCapacity = oldCapacity + (oldCapacity >> 1); 且扩容并非是无限制的，有内存限制、虚拟机限制。

(5) ArrayList 的 toArray() 方法和 subList() 方法，在源数据和子数据之间的区别

 * toArray()：对该方法返回的数组，进行操作（增删改查）都不会影响源数据(ArrayList中elementData)。二者之间是不会相互影响的

 * subList()：对返回的子集合，进行操作(增删改查)都会影响父集合。而且若是对父集合中进行删除操作（仅仅在删除子集合的首个元素）时，会抛出异常 java.util.ConcurrentModificationException

(6)  ArrayList 中涉及到用 index 进行访问，就需要进行越界检查。

（7）System.arrayCopy调用，这是一个native方法，浅拷贝

<br/>
<br/>

本人才疏学浅，若有错，请指出
谢谢！
