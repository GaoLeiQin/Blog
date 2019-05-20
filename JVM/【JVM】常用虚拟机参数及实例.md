**常用参数表**

|参数 | 描述|
|:-----|:-------|
|-XX:+PrintGC |启动java虚拟机后，只要遇到gc，就打印日志|
|-XX:+PrintGCDetails| gc发生时，打印更详细的日志|
|-XX:+PrintHeapAtGC| 每一次GC后，都打印堆信息|
|-XX:+PrintGCTimeStamps |gc发生时，额外打印gc时间，该时间为虚拟机启动到现在的时间偏移量|
|-XX:+PrintGCApplicationStoppedTime |gc时打印应用程序由于gc产生停顿的时间|
|-XX:+PrintReferenceGC |跟踪系统的软引用，弱引用，虚引用和Finallize队列|
|-Xloggc |指定gc日志的保存路径。|
|-XX:+TraceClassLoading |跟踪类加载|
|-XX:+TraceClassUnloading| 跟踪类卸载|
|-XX:+PrintVMOptions |程序运行时，打印虚拟机接收到的命令行显示参数|
|-XX:+PrintCommandLineFlags |打印传递给虚拟机的显式和隐式参数|
|-XX:+PrintFlagsFinal |打印所有系统参数的值|
|-XX:MaxHeapSize |指定最大内存|
|-XX:SurvivorRatio |指定新生代中eden区和from/to区的比例关系|
|-XX:NewRatio |设置老年代/新生代的比例|
|-XX:+HeapDumpOnOutOfMemoryError| 内存溢出时，导出整个堆的信息，和下一个参数配合使用|
|-XX:HeapDumpPath |导出的堆信息的保存路径，和上一个参数配合使用|
|-XX:OnOutOfMemoryError |内存溢出发生错误时执行一个脚本文件|
|-XX:PermSize| 配置初始永久区的大小(JDK8中永久区已经被彻底移除，使用了新的元数据区存放类的元数据)|
|-XX:MaxPermSize |配置最大永久区的大小(JDK8中永久区已经被彻底移除，使用了新的元数据区存放类的元数据)|
|-XX:MaxMetaspaceSize| 指定元数据区最大可用值|
|-Xss |指定线程的栈大小|
|-Xms |指定初始堆空间的大小，例如-Xms20m|
|-Xmx |指定最大堆空间的大小，例如-Xmx100m|
|-Xmn| 指定新生代的大小，例如-Xmn1m|
|-XX:MaxDirectMemorySize |指定最大可用直接内存值|
|-XX:+PrintCommandLineFlags |将隐式或者显示传给虚拟机的参数输出|
|-XX:+TraceClassLoading |监控加载的类信息（启动类加载器）|
|-server| 指定虚拟机在server模式下工作|
|-client| 指定虚拟机在client模式下工作|

**注：JDK8删除了永久区，用元数据区替代。**

* 将初始堆与最大堆大小设置相等，可以减少程序运行时的垃圾回收次数，从而提高性能。

* 浅堆：一个对象结构所占用的内存大小
对象大小按照8字节对齐、浅堆大小和对象的内容无关、只和对象的结构有关。

* 深堆：一个对象被GC回收后，可以真实释放的内存大小
只能通过对象访问到的所有对象的浅堆之和（支配树）

注：在命令行输入

* java -verbose:class  ：输出程序运行时类被加载信息
* java –verbose:gc  ：输出虚拟机发生内存回收时在输出设备显示信息
* java -verbose:jni  ：输出native方法调用的相关情况，一般用于诊断jni调用错误信息

<br>
**例1：**

配置参数：
-Xms5m 
-Xmx20m 
-XX:+PrintGCDetails
-XX:+UseSerialGC 
-XX:+PrintCommandLineFlags

```

public class MemoryInfo {
    public static void main(String[] args) {
        //查看GC信息
        System.out.println("max memory : "+Runtime.getRuntime().maxMemory());
        System.out.println("free memory : "+Runtime.getRuntime().freeMemory());
        System.out.println("total memory : "+Runtime.getRuntime().totalMemory());

        byte[] b1 = new byte[1*1024*1024];

        System.out.println("分配1M  ");
        System.out.println("max memory : "+Runtime.getRuntime().maxMemory());
        System.out.println("free memory : "+Runtime.getRuntime().freeMemory());
        System.out.println("total memory : "+Runtime.getRuntime().totalMemory());

        byte[] b2 = new byte[4*1024*1024];

        System.out.println("分配4M  ");
        System.out.println("max memory : "+Runtime.getRuntime().maxMemory());
        System.out.println("free memory : "+Runtime.getRuntime().freeMemory());
        System.out.println("total memory : "+Runtime.getRuntime().totalMemory());

        //GC日志第三个十六进制数减去第一个十六进制数
        int a = 0x00000000fec00000;
        int b = 0x00000000ff2a0000;
        System.out.println("结果为："+(b-a)/1024);

    }
}

```

输出结果（加注释）：

```
//PrintCommandLineFlags命令打印的内容
-XX:InitialHeapSize=5242880 -XX:MaxHeapSize=20971520 -XX:+PrintCommandLineFlags -XX:+PrintGCDetails -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:-UseLargePagesIndividualAllocation -XX:+UseSerialGC 
//分配1M内存之后
[GC (Allocation Failure) [DefNew: 1664K->192K(1856K), 0.0014276 secs] 
//1664k是GC之前该内存已使用容量，GC后该内存使用容量192k，1856k是该内存区域总容量（1M）
1664K->699K(5952K), 0.0014631 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
//方括号之外：1664k表示GC前Java堆已使用量，699k表示GC后Java堆已使用量，5952表示Java堆总容量。
//最大内存
max memory : 20316160
//空闲内存（值不确定）
free memory : 4426464
//初始分配的5M内存大小
total memory : 6094848
[GC (Allocation Failure) [DefNew: 1121K->120K(1856K), 0.0011164 secs] 1629K->818K(5952K), 0.0011364 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
分配1M  
max memory : 20316160
//分配了1M之后，空闲内存变小了1M
free memory : 4178496
//总内存不变
total memory : 6094848
//GC收集之后，新生代收集前1173k，收集后为0，老年代1841k，Metaspace（之前的永久区）
[GC (Allocation Failure) [DefNew: 1173K->0K(1856K), 0.0010111 secs][Tenured: 1841K->1841K(4096K), 0.0021631 secs] 1871K->1841K(5952K), [Metaspace: 3513K->3513K(1056768K)], 0.0032147 secs] [Times: user=0.01 sys=0.00, real=0.00 secs] 
分配4M  
max memory : 20316160
//因为申请4M，空闲内存不够，向最大内存申请
free memory : 4246464
//申请4M之后，总内存增加4M
total memory : 10358784
结果为：7556
//-XX:+PrintGCDetails命令打印的内容
Heap
//新生代区，第三个十六进制数减第一个等于used的值，第二个十六进制值没有实际意义
 def new generation   total 1920K, used 66K [0x00000000fec00000, 0x00000000fee10000, 0x00000000ff2a0000)
  eden space 1728K,   3% used [0x00000000fec00000, 0x00000000fec10930, 0x00000000fedb0000)
  from space 192K,   0% used [0x00000000fedb0000, 0x00000000fedb0000, 0x00000000fede0000)
  to   space 192K,   0% used [0x00000000fede0000, 0x00000000fede0000, 0x00000000fee10000)
//老年代区
 tenured generation   total 8196K, used 5937K [0x00000000ff2a0000, 0x00000000ffaa1000, 0x0000000100000000)
   the space 8196K,  72% used [0x00000000ff2a0000, 0x00000000ff86c6c0, 0x00000000ff86c800, 0x00000000ffaa1000)
//元数据区，committed固定大小，reserved保留区大小
 Metaspace       used 3520K, capacity 4498K, committed 4864K, reserved 1056768K
  class space    used 388K, capacity 390K, committed 512K, reserved 1048576K

```

**例2**

配置参数

-Xms20m 
-Xmx20m 
-Xmn1m 
-XX:SurvivorRatio=2 
-XX:+PrintGCDetails 
-XX:+UseSerialGC

```
public class DefNew {

    public static void main(String[] args) {
        byte[] b = null;
        //连续向系统申请1M空间，一共10M
        for (int i = 0; i < 10; i++) {
            b = new byte[1*1024*1024];
        }
    }
}
```

输出结果：

```
[GC (Allocation Failure) [DefNew: 512K->256K(768K), 0.0011276 secs] 512K->432K(20224K), 0.0011600 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [DefNew: 767K->112K(768K), 0.0010947 secs] 943K->544K(20224K), 0.0011160 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [DefNew: 624K->255K(768K), 0.0007053 secs] 1056K->703K(20224K), 0.0007267 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [DefNew: 767K->56K(768K), 0.0007107 secs] 1215K->728K(20224K), 0.0007333 secs] [Times: user=0.02 sys=0.00, real=0.00 secs] 
Heap
 def new generation   total 768K, used 544K [0x00000000fec00000, 0x00000000fed00000, 0x00000000fed00000)
  eden space 512K,  95% used [0x00000000fec00000, 0x00000000fec79f30, 0x00000000fec80000)
  from space 256K,  21% used [0x00000000fec80000, 0x00000000fec8e128, 0x00000000fecc0000)
  to   space 256K,   0% used [0x00000000fecc0000, 0x00000000fecc0000, 0x00000000fed00000)
 tenured generation   total 19456K, used 10911K [0x00000000fed00000, 0x0000000100000000, 0x0000000100000000)
   the space 19456K,  56% used [0x00000000fed00000, 0x00000000ff7a7f90, 0x00000000ff7a8000, 0x0000000100000000)
 Metaspace       used 3517K, capacity 4498K, committed 4864K, reserved 1056768K
  class space    used 388K, capacity 390K, committed 512K, reserved 1048576K
```

还有两种参数配置，如果有兴趣可以尝试配置，然后输出结果

（1）

```
-Xms20m -Xmx20m -Xmn1m -XX:SurvivorRatio=2 -XX:PrintGCDetails 
-XX:+UseSerialGC
```

（2）

```
-Xms20m -Xmx20m -XX:NewRatio=2 -XX:PrintGCDetails -XX:+UseSerialGC
```

**例3**

配置参数
-Xms64m 
-Xmx64m 
-XX:+PrintGCDetails

```
public class EnterEdenSurvivor {
    public static void main(String[] args) {
        //初始化的对象在eden区
        for (int i=0; i < 5; i++) {
            byte[] b = new byte[1*1024*1024];
        }
    }
}
```

输出结果

```
Heap
 PSYoungGen      total 18944K, used 8403K [0x00000000feb00000, 0x0000000100000000, 0x0000000100000000)
  eden space 16384K, 51% used [0x00000000feb00000,0x00000000ff334f80,0x00000000ffb00000)
  from space 2560K, 0% used [0x00000000ffd80000,0x00000000ffd80000,0x0000000100000000)
  to   space 2560K, 0% used [0x00000000ffb00000,0x00000000ffb00000,0x00000000ffd80000)
 ParOldGen       total 44032K, used 0K [0x00000000fc000000, 0x00000000feb00000, 0x00000000feb00000)
  object space 44032K, 0% used [0x00000000fc000000,0x00000000fc000000,0x00000000feb00000)
 Metaspace       used 3517K, capacity 4498K, committed 4864K, reserved 1056768K
  class space    used 388K, capacity 390K, committed 512K, reserved 1048576K
```

**例4**
配置参数
-Xmx1024m 
-Xms1024m 
-XX:+UseSerialGC 
-XX:MaxTenuringThreshold=15 
-XX:+PrintGCDetails

```
    public static void maxTenure() {
        Map<Integer,byte[]> m = new HashMap<Integer,byte[]>();
        for (int i = 0; i < 5; i++) {
            byte[] b = new byte[1024*1024];
            m.put(i,b);
        }

        for (int k = 0; k < 20; k++) {
            for (int j = 0; j < 300; j++) {
                byte[] b = new byte[1024*1024];
            }
        }
    }
```

输出结果

```
[GC (Allocation Failure) [DefNew: 278925K->5937K(314560K), 0.0035484 secs] 278925K->5937K(1013632K), 0.0035893 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
//中间省略类似的13行
[GC (Allocation Failure) [DefNew: 284897K->5903K(314560K), 0.0023524 secs] 284897K->5903K(1013632K), 0.0023898 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [DefNew: 284897K->0K(314560K), 0.0033498 secs] 284897K->5903K(1013632K), 0.0033751 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [DefNew: 278994K->0K(314560K), 0.0002507 secs] 284897K->5903K(1013632K), 0.0002911 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [DefNew: 278994K->0K(314560K), 0.0001911 secs] 284898K->5903K(1013632K), 0.0002124 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [DefNew: 278995K->0K(314560K), 0.0002929 secs] 284898K->5903K(1013632K), 0.0003164 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [DefNew: 278995K->0K(314560K), 0.0002338 secs] 284898K->5903K(1013632K), 0.0002560 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [DefNew: 278995K->0K(314560K), 0.0001902 secs] 284898K->5903K(1013632K), 0.0002102 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [DefNew: 278995K->0K(314560K), 0.0001920 secs] 284898K->5903K(1013632K), 0.0002120 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
Heap
 def new generation   total 314560K, used 45992K [0x00000000c0000000, 0x00000000d5550000, 0x00000000d5550000)
  eden space 279616K,  16% used [0x00000000c0000000, 0x00000000c2cea0e8, 0x00000000d1110000)
  from space 34944K,   0% used [0x00000000d1110000, 0x00000000d1110000, 0x00000000d3330000)
  to   space 34944K,   0% used [0x00000000d3330000, 0x00000000d3330000, 0x00000000d5550000)
 tenured generation   total 699072K, used 5903K [0x00000000d5550000, 0x0000000100000000, 0x0000000100000000)
   the space 699072K,   0% used [0x00000000d5550000, 0x00000000d5b13c50, 0x00000000d5b13e00, 0x0000000100000000)
 Metaspace       used 3519K, capacity 4498K, committed 4864K, reserved 1056768K
  class space    used 388K, capacity 390K, committed 512K, reserved 1048576K
```

**例5**

配置参数
-Xss1m

```
public class StackOverFlow {

    //栈调用深度
    private static int count;

    public static void recursion() {
        count++;
        recursion();
    }

    public static void main(String[] args) {
        try {
            recursion();
        }catch(Throwable e){
            System.out.println("调用最大栈深度："+count);
            e.printStackTrace();
        }
    }
}
```

输出结果

```
调用最大栈深度：23533
java.lang.StackOverflowError
	at jvm.StackOverFlow.recursion(StackOverFlow.java:19)
	at jvm.StackOverFlow.recursion(StackOverFlow.java:19)
	at jvm.StackOverFlow.recursion(StackOverFlow.java:19)
	//省略一样的许多行
```


<br>
<br>
本人才疏学浅，若有错，请指出
谢谢！