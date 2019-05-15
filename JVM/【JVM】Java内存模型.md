###**引入**

**硬件的效率与一致性**

* 我们都知道现在计算机的内存与处理器的运算能力有几个数量级的差距，所以现代计算机系统都不得不加入一层读写速度尽可能接近处理器运算速度的高速缓存（cache）来作为内存与处理器之间的缓冲，

*  这引入了一个新的问题：缓存一致性（Cache Coherence）。在多处理器系统中，每个处理器都有自己的高速缓存，而他们又共享同一主存，多个处理器运算任务都涉及同一块主存，需要一种协议可以保障数据的一致性，这类协议有MSI、MESI、MOSI及Dragon Protocol等。

* 为了使得处理器内部的运算单元能竟可能被充分利用，处理器可能会对输入代码进行乱起执行（Out-Of-Order Execution）优化，对结果重组，保证结果准确性。Java虚拟机的即时编译器中也有类似的指令重排序（Instruction Recorder）优化。

图解：

![这里写图片描述](http://img.blog.csdn.net/20170626084010831?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

###**Java内存模型（JMM）**
 
#### **1.定义**

Java内存模型的主要目标是定义程序中各个变量的访问规则，即在虚拟机中将变量存储到内存和从内存中取出变量这样底层细节。

注：变量指包括了实例字段、静态字段和构成数组对象的元素（线程共享），但是不包括局部变量与方法参数（线程私有）。

#### **2.主内存与工作内存**

* Java内存模型中规定了所有的变量都存储在主内存中，每条线程还有自己的工作内存，线程的工作内存中保存了该线程使用到的变量到主内存副本拷贝，线程对变量的所有操作（读取、赋值）都必须在工作内存中进行，而不能直接读写主内存中的变量。

* 不同线程之间无法直接访问对方工作内存中的变量，线程间变量值的传递均需要在主内存来完成，线程、主内存和工作内存的交互关系如下图所示

![这里写图片描述](http://img.blog.csdn.net/20170626085050707?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

####  **3.内存间交互操作**

（1）关于主内存与工作内存之间的具体交互协议，即一个变量如何从主内存拷贝到工作内存、如何从工作内存同步到主内存之间的实现细节，Java内存模型定义了以下八种操作来完成：

* lock（锁定）：作用于主内存的变量，把一个变量标识为一条线程独占状态。

* unlock（解锁）：作用于主内存变量，把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定。

* read（读取）：作用于主内存变量，把一个变量值从主内存传输到线程的工作内存中，以便随后的load动作使用

* load（载入）：作用于工作内存的变量，它把read操作从主内存中得到的变量值放入工作内存的变量副本中。

* use（使用）：作用于工作内存的变量，把工作内存中的一个变量值传递给执行引擎，每当虚拟机遇到一个需要使用变量的值的字节码指令时将会执行这个操作。

* assign（赋值）：作用于工作内存的变量，它把一个从执行引擎接收到的值赋值给工作内存的变量，每当虚拟机遇到一个给变量赋值的字节码指令时执行这个操作。

* store（存储）：作用于工作内存的变量，把工作内存中的一个变量的值传送到主内存中，以便随后的write的操作。

* write（写入）：作用于主内存的变量，它把store操作从工作内存中一个变量的值传送到主内存的变量中。
　　

（2）Java内存模型只要求读写操作必须按顺序执行，而没有保证必须是连续执行。也就是read和load之间，store和write之间是可以插入其他指令的，如：read a，read b，load b， load a。但必须满足如下规则：

* 不允许read和load、store和write操作之一单独出现

* 不允许一个线程丢弃它的最近assign的操作，即变量在工作内存中改变了之后必须同步到主内存中。

* 不允许一个线程无原因地（没有发生过任何assign操作）把数据从工作内存同步回主内存中。

* 一个新的变量只能在主内存中诞生，不允许在工作内存中直接使用一个未被初始化（load或assign）的变量。即就是对一个变量实施use和store操作之前，必须先执行过了assign和load操作。

* 一个变量在同一时刻只允许一条线程对其进行lock操作，lock和unlock必须成对出现

* 如果对一个变量执行lock操作，将会清空工作内存中此变量的值，在执行引擎使用这个变量前需要重新执行load或assign操作初始化变量的值

* 如果一个变量事先没有被lock操作锁定，则不允许对它执行unlock操作；也不允许去unlock一个被其他线程锁定的变量。

* 对一个变量执行unlock操作之前，必须先把此变量同步到主内存中（执行store和write操作）

#### **4.实例**

说了这么多，可能你有点模糊不清了，下面我来举个例子解释一下

![这里写图片描述](http://img.blog.csdn.net/20170626091640835?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

分析：

* 本地内存A和B有主内存中共享变量x的副本。假设初始时，这三个内存中的x值都为0。线程A在执行时，把更新（assign）后的1临时存放在自己的本地内存A中。

* 当线程A和线程B需要通信时，线程A首先会把自己本地内存中修改后的x值存储到主内存（store-write）中，此时主内存中的x值变为了1。

* 线程B到主内存中去读取（read-load）线程A更新后的x值，此时线程B的本地内存的x值也变为了1。


#### **5.重排序**

（1）在执行程序时为了提高性能，编译器和处理器经常会对指令进行重排序。重排序分成三种类型：

* 编译器优化的重排序。编译器在不改变单线程程序语义放入前提下，可以重新安排语句的执行顺序。

* 指令级并行的重排序。现代处理器采用了指令级并行技术来将多条指令重叠执行。如果不存在数据依赖性，处理器可以改变语句对应机器指令的执行顺序。

* 内存系统的重排序。由于处理器使用缓存和读写缓冲区，这使得加载和存储操作看上去可能是在乱序执行。

（2）从Java源代码到最终实际执行的指令序列，会经过下面三种重排序：

![这里写图片描述](http://img.blog.csdn.net/20170626090428225?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

（3）as-if-serial

指不管怎么重排序（编译器和处理器为了提高并行度），（单线程）程序的执行结果不能被改变。编译器，runtime 和处理器都必须遵守as-if-serial语义。

（4）数据依赖性

* 如果两个操作访问同一个变量，且这两个操作中有一个为写操作，此时这两个操作之间就存在数据依赖性，即写后读，写后写，读后写，

* 编译器和处理器在重排序时，会遵守数据依赖性，编译器和处理器不会改变存在数据依赖关系的两个操作的执行顺序。

#### **6.可见性与有序性**

（1）定义

* 可见性：当一个线程修改了共享变量的值，其他线程能够立即得知这个修改

* 有序性：本线程内的操作都是按照顺序执行的，

（2）为了保证内存的可见性，Java编译器在生成指令序列的适当位置会插入内存屏障指令来禁止特定类型的处理器重排序。Java内存模型把内存屏障分为LoadLoad、LoadStore、StoreLoad和StoreStore四种：

![这里写图片描述](http://img.blog.csdn.net/20170626090501609?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


#### **7.synchronized与volatile**

（1）synchronized既保证了多线程的并发有序性，又保证了多线程的内存可见性

（2）volatile可以保证内存可见性，不能保证并发有序性

（3）volatile写-读

例：

```
class VolatileExample {
    int a = 0;
    volatile boolean flag = false;

    public void writer() {
        a = 1;                   //1
        flag = true;               //2
    }

    public void reader() {
        if (flag) {                //3
            int i =  a;           //4
            ……
        }
    }
}
```
例子图解：

![这里写图片描述](http://img.blog.csdn.net/20170626100457837?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

* volatile写的内存语义如下：
当写一个volatile变量时，JMM会把该线程对应的本地内存中的共享变量刷新到主内存

 * 线程A首先执行writer()方法，随后线程B执行reader()方法，初始时两个线程的本地内存中的flag和a都是初始状态。线程A执行volatile写后，共享变量的状态示意图如下：

![这里写图片描述](http://img.blog.csdn.net/20170626100057559?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

* volatile读的内存语义如下：
当读一个volatile变量时，JMM会把该线程对应的本地内存置为无效。线程接下来将从主内存中读取共享变量。
 * 线程B读同一个volatile变量后，共享变量的状态示意图如下：

![这里写图片描述](http://img.blog.csdn.net/20170626100203627?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

#### **8.happens-before原则**

如果一个操作执行的结果需要对另一个操作可见，那么这两个操作之间必须存在happens-before关系，先行发生规则如下（主要是前4个）

* 程序顺序规则：一个线程中的每个操作，happens-before于该线程中任意的后续操作。

* 监视器锁规则：对一个锁的解锁操作，happens-before于随后对这个锁的加锁操作。

* volatile域规则：对一个volatile域的写操作，happens-before于任意线程后续对这个volatile域的读。

* 传递性规则：如果 A happens-before B，且 B happens-before C，那么A happens-before C。

* 线程启动规则：Thread对象的start（）方法happen—before此线程的每一个动作。

* 线程终止规则：线程的所有操作都happen—before对此线程的终止检测，可以通过Thread.join（）方法结束、Thread.isAlive（）的返回值等手段检测到线程已经终止执行。

* 线程中断规则：对线程interrupt（）方法的调用happen—before发生于被中断线程的代码检测到中断时事件的发生。

* 对象终结规则：一个对象的初始化完成（构造函数执行结束）happen—before它的finalize（）方法的开始。

#### **9.顺序一致性模型：**

* 一个线程中的所有操作必须按照程序的顺序来执行。

* 所有线程都只能看到一个单一的操作执行顺序（不管程序是否同步）。

* 每个操作都必须原子执行且立刻对所有线程可见。

还是用上面的例子，比较一下它与JMM的区别，如下图：

![这里写图片描述](http://img.blog.csdn.net/20170626101019640?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

本人才疏学浅，若有错，请指出 
谢谢！

参考资料：[深入理解Java内存模型](http://www.infoq.com/cn/articles/java-memory-model-1?)