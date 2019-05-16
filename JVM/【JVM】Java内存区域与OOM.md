###**引入**

Java与C++之间有一堵由内存动态分配和垃圾收集技术所围成的“高墙”，墙外面的人想进去，墙里面的人却想出来。

###**Java虚拟机运行时数据区**

如图所示

![这里写图片描述](http://img.blog.csdn.net/20170624103003282?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

####**1.程序计数器**（线程私有）

* 作用 
记录当前线程所执行到的字节码的行号。字节码解释器工作的时候就是通过改变这个计数器的值来选取下一条需要执行的字节码指令。

* 意义 
JVM的多线程是通过线程轮流切换并分配处理器来实现的，对于我们来说的并行事实上一个处理器也只会执行一条线程中的指令。所以，为了保证各线程指令的安全顺利执行，每条线程都有独立的私有的程序计数器。

* 存储内容 
若线程中执行的是一个Java方法时，程序计数器中记录的是正在执行的线程的虚拟机字节码指令的地址。 如果线程中执行的是一个本地方法时，程序计数器中的值为空。


注：程序计数器区域是唯一一个在JVM上不会发生内存溢出异常（OutOfMemoryError）的区域。

####**2.虚拟机栈**（线程私有）

* 作用 
描述Java方法执行的内存模型。每个方法在执行的同时都会开辟一段内存区域用于存放方法运行时所需的数据，成为栈帧，一个栈帧包含：局部变量表、操作数栈、动态链接、方法出口等信息。

* 意义 
JVM是基于栈的，每个方法从调用到执行结束，就对应着一个栈帧在虚拟机栈中入栈和出栈的整个过程。

* 存储内容 
局部变量表（编译期可知的各种基本数据类型、引用类型和指向一条字节码指令的returnAddress类型）、操作数栈、动态链接、方法出口等信息。 

* 可能出现的异常 
如果线程请求的栈深度大于虚拟机所允许的深度，将抛出StackOverflowError异常。 
如果在动态扩展内存的时候无法申请到足够的内存，就会抛出OutOfMemoryError异常。

注：局部变量表所需的内存空间在编译期间完成分配。在方法运行的阶段是不会改变局部变量表的大小的。

####**3.本地方法栈**（线程私有）

* 作用 
与虚拟机栈类似，但本地方法栈是为JVM所调用到的Nativa（本地方法）服务的。

* 可能出现的异常 
如果线程请求的栈深度大于虚拟机所允许的深度，将抛出StackOverflowError异常。 
如果在动态扩展内存的时候无法申请到足够的内存，就会抛出OutOfMemoryError异常。

####**4.Java堆**（线程共享）

* 作用 
为了更好的回收内存，在虚拟机开启的时候创建。

* 意义 
 * 存储对象实例，更好地分配内存。 
 * 垃圾回收（GC）。堆是垃圾收集器管理的主要区域。更好地回收内存。 

* 为对象分配内存就是把一块大小确定的内存从堆内存中划分出来，有两种方法。
 * 指针碰撞法
已分配的内存和空闲内存分别在不同的一侧，通过一个指针作为分界点，需要分配内存时，仅仅需要把指针往空闲的一端移动与对象大小相等的距离。

 * 空闲列表法
Java堆的内存并不是完整的，已分配的内存和空闲内存相互交错，JVM通过维护一个列表，记录可用的内存块信息，当分配操作发生时，从列表中找到一个足够大的内存块分配给对象实例，并更新列表上的记录。

* 存储内容 
存放对象实例，几乎所有的对象实例都在这里进行分配。堆可以处于物理上不连续的内存空间，只要逻辑上是连续的就可以。 

* 可能出现的异常 
如果堆上没有内存进行实例分配，并无法进行扩展时，将会抛出OutOfMemoryError异常。

注：随着JIT编译器的发展与逃逸分析技术逐渐成熟，所有对象都在堆上进行分配已变得不那么绝对。有些对象实例也可以分配在栈中。

####**5.方法区**（线程共享）

* 作用 
堆的一个逻辑部分，但却是“Non-Heap”

* 意义 
对运行时常量池、常量、静态变量等数据做出了规定。

* 存储内容 
运行时常量池（具有动态性）、已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。

* 可能出现的异常 
当方法区无法满足内存分配需求时，将抛出OutOfMemoryError异常。

####**6.运行时常量池**（线程共享）

* 作用
是方法区的一部分

* 意义
具备动态性，不一定在编译期产生，运行期间也可以将新的常量放入池中，例如String类的intern()方法

* 存储内容
在类加载后进入方法区时，存放编译期生成的各种字面量和符号引用。

* 可能出现的异常
受到方法区内存的限制，当常量池无法再申请到内存时，会抛出OutOfMemoryError异常。

注：直接内存并不是虚拟机运行时数据区的一部分，也不是内存区域，但直接内存也被频繁的使用，会导致OutOfMemoryError异常出现。

###**JDK8中消失的PerGen**

####**1.JDK8之前堆内存的划分**

![这里写图片描述](http://img.blog.csdn.net/20170731192354271?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

分析：从上图可以看出堆的内存区域分为新生代 ( Young )、老年代 ( Old )和永久区（PermGen space）。

（1）新生代（Young ）

* 所有新生成的对象首先都是放在年轻代的。年轻代的目标就是尽可能快速的收集掉那些生命周期短的对象

* 新生代 ( Young ) 被细分为 Eden 和 两个 Survivor 区域，这两个 Survivor 区域分别被命名为 from 和 to。默认Edem : from : to = 8 : 1 : 1 ( 可以通过参数 –XX:SurvivorRatio 来设定 )，

* JVM 每次只会使用 Eden 和其中的一块 Survivor 区域来为对象服务，所以无论什么时候，总是有一块 Survivor 区域是空闲着的。因此，新生代实际可用的内存空间为 9/10 ( 即90% )的新生代空间。

![这里写图片描述](http://img.blog.csdn.net/20170731192814185?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

（2）老年代（Old）

* 在新生代中经历了N次垃圾回收后仍然存活的对象，就会被放到老年代中。因此，可以认为老年代中存放的都是一些生命周期较长的对象。

* 默认的，新生代 ( Young ) 与老年代 ( Old ) 的比例的值为 1:2 ( 该值可以通过参数 –XX:NewRatio 来指定 )，即：新生代 ( Young ) = 1/3 的堆空间大小。老年代 ( Old ) = 2/3 的堆空间大小。

![这里写图片描述](http://img.blog.csdn.net/20170731193511445?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

（3）永久区（PermGen space）

* PermGen space的全称是Permanent Generation space,是指内存的永久保存区域，

* 这部分用于存放Class和Meta的信息,Class在被 Load的时候被放入PermGen space区域，它和存放Instance的Heap区域不同,

* 如果你的APP会LOAD很多CLASS的话,就很可能出现PermGen space错误。这种错误常见在web服务器对JSP进行pre compile的时候。

PermGen space是Oracle-Sun Hotspot才有，JRockit以及J9是没有这个区域。

####**2.JDK8堆内存的划分**

![这里写图片描述](http://img.blog.csdn.net/20170731190438014?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

（1）分析

* 其实，移除永久代的工作从JDK1.7就开始了。JDK1.7中，存储在永久代的部分数据就已经转移到了Java Heap或者是 Native Heap。但永久代仍存在于JDK1.7中，并没完全移除，譬如符号引用(Symbols)转移到了native heap；字面量(interned strings)转移到了java heap；类的静态变量(class statics)转移到了java heap。

* 元空间（MetaSpace）一种新的内存空间诞生JDK8 HotSpot JVM 将移除永久区，使用本地内存来存储类元数据信息并称之为：元空间（Metaspace）；这与Oracle JRockit 和IBM JVM’s很相似，

* 这意味着不会再有java.lang.OutOfMemoryError: PermGen问题，也不再需要进行调优及监控内存空间的使用，

（2）metaspace的小结

* 在JDK8中PermGen 的内存空间将全部移除。JVM的参数：PermSize 和 MaxPermSize 会被忽略并给出警告（如果在启用时设置了这两个参数）。

* Metaspace 内存分配模型：大部分类元数据都在本地内存中分配。用于描述类元数据的“klasses”已经被移除。

* Metaspace 容量
元空间的本质和永久代类似，都是对JVM规范中方法区的实现。不过元空间与永久代之间最大的区别在于：元空间并不在虚拟机中，而是使用本地内存。，理论上取决于32位/64位系统可虚拟的内存大小。可见也不是无限制的，需要配置参数。

* MaxMetaspaceSize参数用于限制本地内存分配给类元数据的大小。如果没有指定这个参数，元空间会在运行时根据需要动态调整。

* Metaspace 垃圾回收
对于僵死的类及类加载器的垃圾回收将在元数据使用达到“MaxMetaspaceSize”参数的设定值时进行。
适时地监控和调整元空间对于减小垃圾回收频率和减少延时是很有必要的。持续的元空间垃圾回收说明，可能存在类、类加载器导致的内存泄漏或是大小设置不合适。

* Java 堆内存的影响
一些杂项数据已经移到Java堆空间中。升级到JDK8之后，会发现Java堆 空间有所增长。

* Metaspace 监控
元空间的使用情况可以从HotSpot1.8的详细GC日志输出中得到。Jstat 和 JVisualVM两个工具，还是能看到PermGen空间出现。

####**3.为什么JDK8中使用metaspace替换了PermGen？**

* 字符串存在永久代中，容易出现性能问题和内存溢出。

* 类及方法的信息等比较难确定其大小，因此对于永久代的大小指定比较困难，太小容易出现永久代溢出，太大则容易导致内存溢出。即永久代内存经常不够用或发生内存泄露，出现异常java.lang.OutOfMemoryError: PermGen

* 永久代会为 GC 带来不必要的复杂度，并且回收效率偏低。

* 移除永久代是为融合HotSpot JVM与 JRockit VM而做出的努力，因为JRockit没有永久代，不需要配置永久代。Oracle 可能会将HotSpot 与 JRockit 合二为一


###**内存溢出异常（OOM）**

####**1.Java堆溢出**
 
 例1：

```
//配置参数： -Xms20m -Xmx20m -XX:+HeapDumpOnOutOfMemoryError
class HeapOOM {

    static class OOMObject {
    }

    public static void main(String[] args) {
        List<OOMObject> list = new ArrayList<OOMObject>();

        while (true) {
            list.add(new OOMObject());
        }
    }
}

```
输出结果：

![这里写图片描述](http://img.blog.csdn.net/20170624130428632?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
 
分析：OOMObject对象数量达到了最大堆的容量限制（20M），产生了内存溢出异常。

####**2.虚拟机栈和本地方法栈溢出**

例2：

```
//配置参数： -Xss128k
class JavaVMStackSOF {

            private int stackLength = 1;

            public void stackLeak() {
                stackLength++;
                stackLeak();
            }

            public static void main(String[] args) throws Throwable {
                JavaVMStackSOF oom = new JavaVMStackSOF();
                try {
                    oom.stackLeak();
                } catch (Throwable e) {
                    System.out.println("stack length:" + oom.stackLength);
                    throw e;
                }
            }
}

```
 输出结果：

![这里写图片描述](http://img.blog.csdn.net/20170624131921350?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

分析：在单线程下，无论是由于栈帧太大还是虚拟机栈容量太小，当内存无法分配时，抛出的都是StackOverflowError异常。

注：一直创建线程，会导致OOM

####**3.方法区和运行时常量池溢出**

例3：

```
class RuntimeConstantPoolOOM {

    public static void main(String[] args) {
        // 使用List保持着常量池引用，避免Full GC回收常量池行为
        List<String> list = new ArrayList<String>();
        // 10MB的PermSize在integer范围内足够产生OOM了
        int i = 0;
        while (true) {
            list.add(String.valueOf(i++).intern());
        }
    }
}
```

分析：常量池已满，也会导致OOM
 
####**4.本机直接内存溢出**

例4：

```
//配置参数： -Xmx20M -XX:MaxDirectMemorySize=10M
class DirectMemoryOOM {

    private static final int _1MB = 1024 * 1024;

    public static void main(String[] args) throws Exception {
        Field unsafeField = Unsafe.class.getDeclaredFields()[0];
        unsafeField.setAccessible(true);
        Unsafe unsafe = (Unsafe) unsafeField.get(null);
        while (true) {
            unsafe.allocateMemory(_1MB);
        }
    }
}

```

输出结果
![这里写图片描述](http://img.blog.csdn.net/20170624134857638?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

分析：抛出异常时并没有真正向操作系统申请分配内存，而是通过计算得知内存无法分配，于是手动抛出异常，真正申请分配内存的方法是unsafe.allocateMemory()

###**内存泄露**

（1）静态集合类像HashMap、Vector等的使用最容易出现内存泄露，这些静态变量的生命周期和应用程序一致，所有的对象Object也不能被释放，因为他们也将一直被Vector等应用着。

例：

```
Static Vector v = new Vector(); 
for (int i = 1; i<100; i++) 
{ 
    Object o = new Object(); 
    v.add(o); 
    o = null; 
}
```

分析：

* 栈中存在Vector 对象的引用 v 和 Object 对象的引用 o 。在 For 循环中，我们不断的生成新的对象，然后将其添加到 Vector 对象中，之后将 o 引用置空。

* 当 o 引用被置空后，若发生 GC，Object 对象不能被 GC 回收，因为 GC 在跟踪代码栈中的引用时，会发现 v 引用，而继续往下跟踪，就会发现 v 引用指向的内存空间中又存在指向 Object 对象的引用。也就是说尽管o 引用已经被置空，但是 Object 对象仍然存在其他的引用，是可以被访问到的，所以 GC 无法将其释放掉。

* 如果在此循环之后， Object 对象对程序已经没有任何作用，那么我们就认为此 Java 程序发生了内存泄漏。

（2）数据库连接，网络连接，IO连接等没有显示调用close关闭，不被GC回收导致内存泄露

（3）监听器的使用，在释放对象的同时没有相应删除监听器的时候也可能导致内存泄露。

<br>
<br>
本人才疏学浅，若有错，请指出，谢谢！ 
如果你有更好的建议，可以留言我们一起讨论，共同进步！ 
衷心的感谢您能耐心的读完本篇博文！

> 
参考链接：
1.[JEP 122: Remove the Permanent Generation——OpenJDK](http://openjdk.java.net/jeps/122)
2.[JVM内存的那些事—简书（占小狼）](http://www.jianshu.com/p/eaef248b5a2c)




