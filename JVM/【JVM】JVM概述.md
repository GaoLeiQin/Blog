**1.JVM定义**

 JVM 是Java Virtual Machine（JVM ）的缩写，Java虚拟机在执行字节码时，把字节码解释成具体平台上的机器指令进行执行，这样实现了Java“一次编译，到处运行”。

**2.JVM组成**

JVM由三大部分组成：类加载器（ClassLoader subsystem），执行引擎（Execution Engine），内存区（Runtime data areas）。 

![这里写图片描述](http://img.blog.csdn.net/20170624095907240?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**3. JVM执行程序的过程**

（1） 加载.class文件
 
（2） 管理并分配内存 

（3） 执行垃圾收集

JRE（java运行时环境）由JVM构造的java程序的运行环境，也是Java程序运行的环境，但是他同时一个操作系统的一个应用程序一个进程，因此他也有他自己的运行的生命周期，也有自己的代码和数据空间。JVM在整个jdk中处于最底层，负责于操作系统的交互，用来屏蔽操作系统环境，提供一个完整的Java运行环境，因此也就虚拟计算机。

操作系统装入JVM是通过jdk中Java.exe来完成，通过下面4步来完成JVM环境

1) 创建JVM装载环境和配置 

2) 装载JVM.dll 

3) 初始化JVM.dll并挂界到JNIENV(JNI调用接口)实例

4) 调用JNIEnv实例装载并处理class类。

**4.JVM运行时数据区**

（1）堆

用来存储对象实例以及数组值的区域，可以认为Java中所有通过new创建的对象的内存都在此分配，Heap中的对象的内存需要等待GC进行回收。

（2）方法区

存放所加载的类的信息（名称、修饰符等）、类中的静态变量、类中定义为final类型的常量、类中的Field信息、类中的方法信息，当开发人员在程序中通过Class对象中的getName、isInterface等方法来获取信息时，这些数据都来源于方法区域，同时方法区域也是全局共享的，在一定的条件下它也会被GC，当方法区域需要使用的内存超过其允许的大小时，会抛出OutOfMemory的错误信息。

（3）栈

栈是线程私有的，每个线程创建的同时都会创建JVM栈，JVM栈中存放的为当前线程中局部基本类型的变量（java中定义的八种基本类型：boolean、char、byte、short、int、long、float、double）、部分的返回结果以及Stack Frame，非基本类型的对象在JVM栈上仅存放一个指向堆上的地址。

（4）程序计数器

用于存储每个线程下一步将执行的JVM指令，如该方法为native的，则程序计数器中不存储任何信息

（5）图解

![这里写图片描述](http://img.blog.csdn.net/20170624101427605?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**5.JVM垃圾回收**

GC (Garbage Collection)的基本原理：

将内存中不再被使用的对象进行回收，GC中用于回收的方法称为收集器，由于GC需要消耗一些资源和时间，Java在对对象的生命周期特征进行分析后，按照新生代、旧生代的方式来对对象进行收集，以尽可能的缩短GC对应用造成的暂停

（1）对新生代的对象的收集称为minor GC；

（2）对旧生代的对象的收集称为Full GC；

（3）程序中主动调用System.gc()强制执行的GC为Full GC。

**6.JVM对象的引用类型：**

（1）强引用：默认情况下，对象采用的均为强引用（这个对象的实例没有其他对象引用，GC时才会被回收）

（2）软引用：软引用是Java中提供的一种比较适合于缓存场景的应用（只有在内存不够用的情况下才会被GC）

（3）弱引用：在GC时一定会被GC回收

（4）虚引用：由于虚引用只是用来得知对象是否被GC

**7.小结**

JVM是独立于Java语言的一套规范以及一个Class解释执行的平台软件，以其性能精良、稳定高效，赢得了大型应用的亲睐，接下来我会详细介绍JVM，敬请期待！

<br>
<br>
本人才疏学浅，若有错误，还请指出
谢谢！