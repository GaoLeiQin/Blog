###**在JDK/bin目录下我发现了许多命令行工具**

![这里写图片描述](http://img.blog.csdn.net/20170624205551775?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

这些命令有哪些作用呢，接下来我就来详细介绍一下

###**常用JDK命令行工具**

|命令名称	|全称	|用途|
|:---------|:--------|:-----|
|jstat	|JVM Statistics Monitoring Tool	|用于收集Hotspot虚拟机各方面的运行数据|
|jps	|JVM Process Status Tool	|显示指定系统内所有的HotSpot虚拟机进程|
|jinfo|	Configuration Info for Java	|显示虚拟机配置信息|
|jmap	|JVM Memory Map|	生成虚拟机的内存转储快照，生成heapdump文件|
|jhat|	JVM Heap Dump Browser|	用于分析heapdump文件，它会建立一个HTTP/HTML服务器，让用户在浏览器上查看分析结果|
|jstack	|JVM Stack Trace|	显示虚拟机的线程快照|

####**1. jps** 

（1）	定义：查看虚拟机进程信息

（2）	参数：

 -q 只输出进程ID，不输出类的短名称
-m 输出传递给Java进程的函数
-l  输出主函数的完整路径
-v  显示传递给JVM的参数

（3）实例

![这里写图片描述](http://img.blog.csdn.net/20170624210245001?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


####**2.jinfo**

（1）定义：用于查看和调整虚拟机的配置参数

（2）参数

-flag <name> 打印指定JVM的参数值
-flag [+]-]<name> 设置指定JVM参数的布尔值
-flag <name> = <value> 指定JVM参数的值

（3）实例

![这里写图片描述](http://img.blog.csdn.net/20170624210539134?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

![这里写图片描述](http://img.blog.csdn.net/20170624210703473?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


####**3.jstack**

（1）定义：虚拟机堆栈跟踪

（2）参数

-l 打印锁信息

-m 打印Java和native 的帧信息

-F 强制dump，当jstack没有响应时使用。

（3）实例
![这里写图片描述](http://img.blog.csdn.net/20170624211215995?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)![这里写图片描述](http://img.blog.csdn.net/20170624211223038?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

####**4.jstat**

（1）定义：查看虚拟机统计信息监视工具

（2）参数

-class  监视类装载，卸载数量，总空间以及类装载所耗费的时间

-gc   监视Java堆状况，包括Eden区，两个survivor区，老年代，永久代的容量，已用空间，GC时间合计等信息

-gccapacity  内容与-gc基本相同，但主要输出Java堆各个区域的最大最小空间

-gcutil  内容与-gc基本相同，但主要关注已使用空间占总空间的百分比

-gccause  内容与-gcutil基本相同，但主要关注已使用空间占总空间的百分比,并输出导致上一次GC的原因

-gcnew  监视新生代GC情况

-gcnewcapacity  内容与-gcnew基本相同，但主要输出使用到的最大最小空间

-gcold  监视老年代GC情况

-gcoldcapacity  内容与-gcnew基本相同，但主要输出使用到的最大最小空间

-gcpermcapacity  输出永久代使用到的最大最小空间

-complier   输出JIT 编译器编译过的方法耗时的信息

-printcompliter  输出已经被JIT编译的方法

（3）实例

![这里写图片描述](http://img.blog.csdn.net/20170624211904987?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

####**5.jmap**

（1）定义：生成Java应用程序的堆快照和对象的统计信息

（2）参数

-dump 生成Java堆转储快照，格式为： -dump:[live , ]format=b , file=,其中live子参数 说明只dump出存活的对象

-finalizerinfo 显示在F-Queue中等待Finalizer线程执行finalize方法的对象

-heap 显示Java堆详细信息，如使用哪种回收器，参数配置，分代状况等

-histo  打印每个class的实例数目,内存占用,类全名信息. VM的内部类名字开头会加上前缀”*”

（3）实例

![这里写图片描述](http://img.blog.csdn.net/20170624212413840?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

####**6. jhat**

（1）定义：分析虚拟机转储快照信息

（2）参数：后面接文件路径

（3）实例

![这里写图片描述](http://img.blog.csdn.net/20170624213137117?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

打开浏览器，输入网址：http://127.0.0.1:7000/

然后就可以看到（部分网页）
![这里写图片描述](http://img.blog.csdn.net/20170624213707817?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)![这里写图片描述](http://img.blog.csdn.net/20170624213726800?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

###**其他工具**

当然还有几个其他工具，例如

* 内存dump后的分析工具：MAT

* 看Java进程的native栈：pstack

* 日志相关工具：tail,find,fgrep,awk

* 查看Java堆外的内存消耗：gperf

<br>
<br>
本人才疏学浅，若有错，请指出
谢谢！