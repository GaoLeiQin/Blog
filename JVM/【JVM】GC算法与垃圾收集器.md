###**引入**

* java  语言中一个显著的特点就是引入了java回收机制，是c++程序员最头疼的内存管理的问题迎刃而解，它使得java程序员在编写程序的时候不在考虑内存管理。由于有个垃圾回收机制，可以有效的防止内存泄露，有效的使用空闲的内存；

* 内存泄露：指该内存空间使用完毕后未回收，在不涉及复杂数据结构的一般情况下，java的内存泄露表现为一个内存对象的生命周期超出了程序需要它的时间长度

* Java虚拟机的内存区域中，程序计数器、虚拟机栈和本地方法栈三个区域是线程私有的，随线程生而生，随线程灭而灭；栈中的栈帧随着方法的进入和退出而进行入栈和出栈操作，每个栈帧中分配多少内存基本上是在类结构确定下来时就已知的，因此这三个区域的内存分配和回收都具有确定性。垃圾回收重点关注的是堆和方法区部分的内存。

###**常用的垃圾回收算法**

####**1.引用计数算法：**

* 给对象中添加一个引用计数器，每当有一个地方引用它时，计数器值就加1；当引用失效时，计数器值就减1；任何时刻计数器都为0的对象就是不再被使用的，垃圾收集器将回收该对象使用的内存。

* 引用计数算法实现简单，效率很高，但是引用计数算法对于对象之间相互循环引用问题难以解决，因此java并没有使用引用计数算法。

![这里写图片描述](http://img.blog.csdn.net/20170624154019558?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

####**2.可达性分析算法：**

* 通过一系列的名为“GC Root”的对象作为起点，从这些节点向下搜索，搜索所走过的路径称为引用链(Reference Chain)，当一个对象到GC Root没有任何引用链相连时，则该对象不可达，该对象是不可使用的，垃圾收集器将回收其所占的内存。

* 在java语言中，可作为GC Root的对象包括以下几种对象：

 * java虚拟机栈(栈帧中的本地变量表)中的引用的对象。

 * 方法区中的类静态属性引用的对象。

 * 方法区中的常量引用的对象。

 * 本地方法栈中JNI本地方法的引用对象。

![这里写图片描述](http://img.blog.csdn.net/20170624154527672?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

####**3.回收方法区**

* 方法区在Sun HotSpot虚拟机中被称为永久代，很多人认为该部分的内存是不用回收的，但是方法区中的废弃常量和无用的类还是需要回收以保证永久代不会发生内存溢出。

* 判断废弃常量的方法：如果常量池中的某个常量没有被任何引用所引用，则该常量是废弃常量。

* 判断无用的类：

 * 该类的所有实例都已经被回收，即java堆中不存在该类的实例对象。

 * 加载该类的类加载器已经被回收。

 * 该类所对应的java.lang.Class对象没有任何地方被引用，无法在任何地方通过反射机制访问该类的方法。

####**4.四种引用**

* 强引用

　　使用最普遍的引用。如果一个对象具有强引用，那就 类似于必不可少的生活用品，垃圾回收器绝不会回收它。当内存空 间不足，Java虚拟机宁愿抛出OutOfMemoryError错误，使程序异常终止，也不会靠随意回收具有强引用的对象来解决内存不足问题。

* 软引用(SoftReference)

　　如果一个对象只具有软引用，那就类似于可有可物的生活用品。如果内存空间足够，垃圾回收器就不会回收它，如果内存空间不足了，就会回收这些对象的内存。只要垃圾回收器没有回收它，该对象就可以被程序使用。软引用可用来实现内存敏感的高速缓存。

　　软引用可以和一个引用队列(ReferenceQueue)联合使用，如果软引用所引用的对象被垃圾回收，Java虚拟机就会把这个软引用加入到与之关联的引用队列中。

* 弱引用(WeakReference)

　　如果一个对象只具有弱引用，那就类似于可有可物的生活用品。 弱引用与软引用的区别在于：只具有弱引用的对象拥有更短暂的生命周期。在垃圾回收器线程扫描它 所管辖的内存区域的过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。不过，由于垃圾回收器是一个优先级很低的线程， 因此不一定会很快发现那些只具有弱引用的对象。

* 虚引用

　　又称为幽灵引用或幻影引用，，虚引用既不会影响对象的生命周期，也无法通过虚引用来获取对象实例，而且它的优先级低，仅用于在发生GC时接收一个系统通知，不知道什么时候调用，因此尽量避免使用finalize方法，可以使用try/catc/finallyh语句块代替。


####**5.finalize（）方法**

因为任何一个对象的finalize方法只会被系统自动调用一次，如果面临下一次回收，它的finalize方法不会被执行，因此尽量避免使用finalize方法。


###**Java中常用的垃圾收集算法**

####**1.标记-清除算法：**

（1）定义：

分为“标记”和“清除”两个阶段：首先标记出所有需要回收的对象，在标记完成之后统一回收掉所有被标记的对象。

（2）缺点

* 效率问题，标记和清除效率都不高。

* 标记清除之后会产生大量的不连续的内存碎片，空间碎片太多会导致当程序需要为较大对象分配内存时无法找到足够的连续内存而不得不提前触发另一次垃圾收集动作。

（3）图解

![这里写图片描述](http://img.blog.csdn.net/20170624155457346?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

####**2.复制算法：**

（1）定义：

将可用内存按容量分成大小相等的两块，每次只使用其中一块，当这块内存使用完了，就将还存活的对象复制到另一块内存上去，然后把使用过的内存空间一次清理掉。这样使得每次都是对其中一块内存进行回收，内存分配时不用考虑内存碎片等复杂情况，只需要移动堆顶指针，按顺序分配内存即可，实现简单，运行高效。

（2）缺点

* 可使用的内存缩减到原来的一半

* 当对象存活率高时，就要执行较多的复制操作，效率将会变低

（3）图解

![这里写图片描述](http://img.blog.csdn.net/20170624155913984?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

####**3.标记-整理算法：**

（1）定义

改进了标记-清除算法，标记阶段是相同的标记出所有需要回收的对象，在标记完成之后不是直接对可回收对象进行清理，而是让所有存活的对象都向一端移动，在移动过程中清理掉可回收的对象，这个过程叫做整理。

（2）特点

* 内存被整理以后不会产生大量不连续内存碎片问题。

* 当对象存活率高时，使用标记-整理算法效率会大大提高。

（3）图解

![这里写图片描述](http://img.blog.csdn.net/20170624160254920?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

####**4.分代收集算法：**

（1）定义

根据内存中对象的存活周期不同，将内存划分为几块，java的虚拟机中一般把内存划分为新生代和年老代，当新创建对象时一般在新生代中分配内存空间，当新生代垃圾收集器回收几次之后仍然存活的对象会被移动到年老代内存中，当大对象在新生代中无法找到足够的连续内存时也直接在年老代中创建。

（2）分类

堆内存被分成新生代和年老代两个部分，整个堆内存使用分代复制垃圾收集算法。

（3）新生代：

* 新生代使用复制垃圾回收算法，新生代分为Eden区，Survivor from和Survivor to三部分，其占新生代内存容量默认比例分别为8：1：1，其中Survivor from和Survivor to总有一个区域是空白，当新生代内存空间不足需要进行垃圾回收时，仍然存活的对象被复制到空白的Survivor内存区域中，Eden和非空白的Survivor进行标记-清理回收，两个Survivor区域是轮换的。

* 对于创建大对象时，如果新生代中无足够的连续内存时，也直接在年老代中分配内存空间。

* Java虚拟机对新生代的垃圾回收称为Minor GC，次数比较频繁，每次回收时间也比较短。

* 使用java虚拟机-Xmn参数可以指定新生代内存大小。

（4）老年代：

* 老年代中的对象一般都是长生命周期对象，对象的存活率比较高，因此在老年代中使用标记-整理垃圾回收算法。

* Java虚拟机对老年代的垃圾回收称为MajorGC/Full GC，次数相对比较少，每次回收的时间也比较长。

* 当新生代中无足够空间为对象创建分配内存，年老代中内存回收也无法回收到足够的内存空间，并且新生代和年老代空间无法在扩展时，堆就会产生OutOfMemoryError异常。

* java虚拟机-Xms参数可以指定最小内存大小，-Xmx参数可以指定最大内存大小，这两个参数分别减去Xmn参数指定的新生代内存大小，可以计算出年老代最小和最大内存容量。

（5）永久代（JDK8之前）

* 虚拟机内存中的方法区被称为永久代，是线程共享的内存区域，它用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译后的代码等数据。

* 永久代垃圾回收比较少，效率也比较低，但是也必须进行垃圾回收，否则会永久代内存不够用时仍然会抛出OutOfMemoryError异常。

* 永久代也使用标记-整理算法进行垃圾回收，java虚拟机参数-XX:PermSize和-XX:MaxPermSize可以设置永久代的初始大小和最大容量。（JDK8的这两个参数已经失效，JDK8用Metaspace替代了永久区）

（6）图解

![这里写图片描述](http://img.blog.csdn.net/20170624161606815?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

####**（7）Minor GC与Full GC**

* Minor GC
 
 * 触发条件：当新对象生成，并且在Eden申请空间失败时，就会触发Minor GC，对Eden区域进行GC，清除非存活对象，并且把尚且存活的对象移动到Survivor区。然后整理Survivor的两个区。

 * 主要包含两个步骤：查找GC Roots，拷贝所引用的对象到 to 区；递归遍历前一步中对象，并拷贝其所引用的对象到 to 区，当然可能会存在自然晋升，或者因为 to 区空间不足引起的提前晋升的情况 

 * 然后清理所使用过的 Eden 以及 Survivor 区域 ( 即 from 区域 )，并且将这些对象的年龄设置为1，以后对象在 Survivor 区每熬过一次 Minor GC，就将对象的年龄 + 1，当对象的年龄达到某个值时 ( 默认是 15 岁，可以通过参数 -XX:MaxTenuringThreshold 来设定 )，这些对象就会成为老年代。

 * 但这也不是一定的，对于一些较大的对象 ( 即需要分配一块较大的连续内存空间 ) 则是直接进入到老年代。

 * Minor GC是对新生代的Eden区进行，不会影响到老年代。因为大部分对象都是从Eden区开始的，同时Eden区不会分配的很大，所以Eden区的GC会频繁进行。一般使用复制算法，使Eden去能尽快空闲出来。

![这里写图片描述](http://img.blog.csdn.net/20170801083448144?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

* Full GC
 * 对整个堆进行整理，包括Young、Tenured和Perm。Full GC因为需要对整个对进行回收，所以比Minor  GC要慢，因此应该尽可能减少Full GC的次数。在对JVM调优的过程中，很大一部分工作就是对于FullGC的调节。

 * 因为FullGC采用的是标记-清除算法，所以在收集垃圾的时候会产生许多的内存碎片 ( 即不连续的内存空间 )，此后需要为较大的对象分配内存空间时，若无法找到足够的连续的内存空间，就会提前触发一次 GC 的收集动作。

* 可能导致Full GC的原因
 * 老年代（Tenured）被写满，或空间不足。
 * 永久代（Perm）被写满
 *  System.gc()被显示调用，底层调用Runtime.getRuntime().gc()这个本地方法。或jmap -histo:live < pid>、jmap -dump ...
 * Minor GC晋升到旧生代的平均大小大于老年代的剩余空间
 * 堆中分配很大的对象
 * CMS GC时出现promotion failed和concurrent mode failure

**注：**

* promotion failed是在进行Minor GC时，survivor space放不下、对象只能放入老年代，而此时老年代也放不下造成的；
* concurrent mode failure是在执行CMS GC的过程中同时有对象要放入老年代，而此时老年代空间不足造成的（有时候“空间不足”是CMS GC时当前的浮动垃圾过多导致暂时性的空间不足触发Full GC）。
* 解决方法：增大survivor space、老年代空间或调低触发并发GC的比率

* TLAB：JVM在内存新生代Eden Space中开辟了一小块线程私有的区域，称作TLAB（Thread-local allocation buffer）。默认设定为占用Eden Space的1%。小对象通常JVM会优先分配在TLAB上，并且TLAB上的分配由于是线程私有所以没有锁开销。每个TLAB都只有一个线程可以操作，TLAB结合bump-the-pointer技术可以实现快速的对象分配，而不需要任何的锁进行同步，也就是说，在对象分配的时候不用锁住整个堆，而只需要在自己的缓冲区分配即可。（主要就是三个指针：start，top 和 end ），TLAB满后，再分配一块TLAB，不管之前的。

####**5.分区算法**

将整个内存分为N个小的独立空间，每个独立空间都可以独立使用，并不是对整个空间进行GC，从而提升性能，并减少GC的停顿时间。

###**几种垃圾收集器的比较**

####**1.垃圾收集器分类图解**

![这里写图片描述](http://img.blog.csdn.net/20170624161855566?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

####**2.新生代收集器**

（1）Serial收集器

* Serial是一个单线程收集器，并且在它进行垃圾收集时，必须暂停所有用户线程。采用复制算法，

* Serial 收集器尽管显得“简陋”并且用户体验不佳，但到1.6为止它仍然是虚拟机运行在Client模式下的默认新生代收集器

* Serial 收集器实现简单高效，但是会给用户带来停顿。

Serial收集器工作原理图如下

![这里写图片描述](http://img.blog.csdn.net/20170624163543330?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

（2）ParNew收集器

* ParNew收集器是Serial收集器的多线程版本，使用多个线程进行垃圾收集，其他与serial类似。

* 只有ParNew收集器和Serial收集器能与CMS收集器配合工作

ParNew收集器工作原理图如下

![这里写图片描述](http://img.blog.csdn.net/20170624162343660?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

（3）Parallel Scavenge收集器

* 并行与并发
 * 并行（Parallel）：指多个垃圾收集器线程并行工作，但此时用户线程仍处于等待状态。
 * 并发（Concurrent）：指用户线程与回收器线程同时工作

* 一个新生代的多线程收集器（并行收集器），它在回收期间不需要暂停其他用户线程，其采用的是复制算法，

* 为了达到一个可控的吞吐量

Parallel Scavenge收集器工作原理图如下

![这里写图片描述](http://img.blog.csdn.net/20170624162835452?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

####**3.老年代收集器**

（1）Serial Old收集器

* 一个单线程收集器，采用标记-整理算法

Serial Old收集器工作原理图如下

![这里写图片描述](http://img.blog.csdn.net/20170624163543330?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

（2）Parallel Old收集器

* 一个Parallel Scavenge收集器的老年代版本（并行收集器），使用多线程和标记-整理算法。

Parallel Old收集器工作原理图如下

![这里写图片描述](http://img.blog.csdn.net/20170624162835452?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

（3）CMS（Current Mark Sweep）收集器

* 一种以获取最短回收停顿时间为目标的收集器，它是一种并发收集器，采用的是标记-清除算法。

* 它的运行过程有四个步骤
①初始标记(CMS initial mark)
②并发标记(CMS concurrenr mark)
③重新标记(CMS remark)
④并发清除(CMS concurrent sweep)
其中初始标记、重新标记这两个步骤任然需要停顿其他用户线程（STW）
　
* 缺点：对CPU资源敏感（影响系统性能）、无法处理浮动垃圾、会产生大量空间碎片

CMS收集器工作原理图如下

![这里写图片描述](http://img.blog.csdn.net/20170624170019429?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

####**4. G1收集器**

* G1(Garbage First)收集器是JDK1.7提供的一个新收集器

* 并行与并发：充分利用多CPU、多核的硬件优势，缩短STW停顿的时间

* 空间整合：采用“标记 - 整理”算法，避免产生碎片

* 可以预测的停顿：通过让使用指定一个参数来控制在一个长度为M的时间片内垃圾回收的时间N

* 分代收集：之前的收集器是进行收集的范围是整个新生代或 老年代，而G1将整个Java堆（包括新生代、老年代）划分为多个大小固定的独立区域，并且跟踪这些区域的堆积成都，在后台维护一个优先列表，每次根据优 先级从列表中挑选区域进行收集。

* 它的运行过程有四个步骤
①初始标记(CMS initial mark)
②并发标记(CMS concurrenr mark)
③最终标记(CMS Marking)
④筛选回收(CMS Data Counting and Evacuation)

* G1收集器工作原理图如下

![这里写图片描述](http://img.blog.csdn.net/20170624170256501?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

* G1收集器的对象分配在某个Region中，可以和Java堆上任意的对象有引用关系，为避免全堆扫描，将G1收集器中Region之间的对象引用关系和其他收集器中新生代与老年代之间的对象引用关系被保存在Remenbered Set数据结构中。G1中每个Region都有一个对应的Remenbered Set，当虚拟机发现程序对Reference类型的数据进行写操作时，会产生一个Write Barrier暂时中断写操作，检查Reference引用的对象是否处于相同的Region中，如果不是，则通过CardTable把相关引用信息记录到被引用对象所属Region的Remenbered Set中。


###**垃圾收集器参数**

![这里写图片描述](http://img.blog.csdn.net/20170624165833180?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)![这里写图片描述](http://img.blog.csdn.net/20170624165839032?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


<br>
<br>
本人才疏学浅，若有错，请指出，谢谢！ 
如果你有更好的建议，可以留言我们一起讨论，共同进步！ 
衷心的感谢您能耐心的读完本篇博文。

参考链接：[Java GC、新生代、老年代](http://www.cnblogs.com/yydcdut/p/3959711.html)