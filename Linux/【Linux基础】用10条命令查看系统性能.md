###**前言**

如果你的Linux系统突然很卡，负载很大，这时候你就懵逼了吧，接下来你该怎么办呢？让我来悄悄的告诉你吧，而且只需要10个命令就搞定，是不是很方便呀！


###**10个命令**

一共就这10个命令，让你先大概看一下，心里有个底！

|序号|命令|功能|
|:-----|:----|:-----|
|1|uptime | 快速查看负载情况|
|2|dmesg ]tail|输出系统日志的最后10行|
|3|vmstat 1|输出系统核心指标|
|4|mpstat -P ALL 1|显示每个CPU的占用情况|
|5|pidstat 1|输出进程的CPU占用率|
|6|iostat -xz 1|查看磁盘IO情况|
|7|free -m|查看内存的使用情况|
|8|sar -n DEV 1|网络设备的吞吐率|
|9|sar -n TCP,ETCP 1|查看TCP连接状态|
|10|top|相对全面的查看系统负载的来源|

<br>

下面我来介绍这10个命令，如果想要深入了解，请自行百度。

####**1.uptime**


![这里写图片描述](http://img.blog.csdn.net/20170703181900233?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

分析：这个命令可以快速查看系统的负载情况，输出分别表示1分钟、5分钟、15分钟的平均负载情况。

####**2.dmesg | tail** 

![这里写图片描述](http://img.blog.csdn.net/20170703182048555?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

分析：该命令会输出系统日志的最后10行。

####**3.vmstat 1**

![这里写图片描述](http://img.blog.csdn.net/20170703182149763?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

分析：每行会输出一些系统核心指标，参数1表示每秒输出一次统计信息，相关列的含义如下：

* r：等待在CPU资源的进程数。这个数据比平均负载更加能够体现CPU负载情况，数据中不包含等待IO的进程。如果这个数值大于机器CPU核数，那么机器的CPU资源已经饱和。

* free：系统可用内存数（以千字节为单位），如果剩余内存不足，也会导致系统性能问题。下文介绍到的free命令，可以更详细的了解系统内存的使用情况。

* si, so：交换区写入和读取的数量。如果这个数据不为0，说明系统已经在使用交换区（swap），机器物理内存已经不足。

* us, sy, id, wa, st：这些都代表了CPU时间的消耗，它们分别表示用户时间（user）、系统（内核）时间（sys）、空闲时间（idle）、IO等待时间（wait）和被偷走的时间（stolen，一般被其他虚拟机消耗）。

####**4.mpstat -P ALL 1**

![这里写图片描述](http://img.blog.csdn.net/20170703182348064?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

分析：该命令可以显示每个CPU的占用情况。

####**5.pidstat 1**

![这里写图片描述](http://img.blog.csdn.net/20170703182539690?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

分析：该命令输出进程的CPU占用率，该命令会持续输出，并且不会覆盖之前的数据，可以方便观察系统动态。

####**6.iostat -xz 1**

![这里写图片描述](http://img.blog.csdn.net/20170703182741000?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

分析：该命令命令主要用于查看机器磁盘IO情况。每列的含义如下：

* r/s, w/s, rkB/s, wkB/s：分别表示每秒读写次数和每秒读写数据量（千字节）。读写量过大，可能会引起性能问题。

* await：IO操作的平均等待时间，单位是毫秒。这是应用程序在和磁盘交互时，需要消耗的时间，包括IO等待和实际操作的耗时。如果这个数值过大，可能是硬件设备遇到了瓶颈或者出现故障。

* avgqu-sz：向设备发出的请求平均数量。如果这个数值大于1，可能是硬件设备已经饱和（部分前端硬件设备支持并行写入）。

* %util：设备利用率。这个数值表示设备的繁忙程度，经验值是如果超过60，可能会影响IO性
能（可以参照IO操作平均等待时间）。如果到达100%，说明硬件设备已经饱和。

####**7.free –m**

![这里写图片描述](http://img.blog.csdn.net/20170703182939790?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

分析：

* 该命令可以查看系统内存的使用情况，-m参数表示按照兆字节展示。最后两列分别表示用于IO缓存的内存数，和用于文件系统页缓存的内存数。

* 第二行-/+ buffers/cache，看上去缓存占用了大量内存空间。这是Linux系统的内存使用策略，尽可能的利用内存，如果应用程序需要内存，这部分内存会立即被回收并分配给应用程序。因此，这部分内存一般也被当成是可用内存。

* 如果可用内存非常少，系统可能会动用交换区（如果配置了的话），这样会增加IO开销（可以在iostat命令中提现），降低系统性能。

####**8.sar -n DEV 1**

![这里写图片描述](http://img.blog.csdn.net/20170703183257668?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

分析：该命令在这里可以查看网络设备的吞吐率。在排查性能问题时，可以通过网络设备的吞吐量，判断网络设备是否已经饱和。

####**9.sar -n TCP,ETCP 1**

![这里写图片描述](http://img.blog.csdn.net/20170703183508831?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

分析：sar命令在这里用于查看TCP连接状态，其中包括：

* active/s：每秒本地发起的TCP连接数，既通过connect调用创建的TCP连接；
* passive/s：每秒远程发起的TCP连接数，即通过accept调用创建的TCP连接；
* retrans/s：每秒TCP重传数量；


####**10.top**

![这里写图片描述](http://img.blog.csdn.net/20170703184312938?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

分析：

* 通过该命令可以相对全面的查看系统负载的来源，top命令还支持排序，可以按照不同的列排序，方便查找出诸如内存占用最多的进程、CPU占用率最高的进程等。

* top命令相对于前面一些命令，输出是一个瞬间值，如果不持续盯着，可能会错过一些线索。这时可能需要暂停top命令刷新，来记录和比对数据。

* 可以使用htop 命令来查看系统的使用情况。可以使用键盘的F3 : 搜索进程， F4 : 过滤器，按关键字搜索，F5 : 显示树形结构， F6 : 选择排序方式, F7/ F8 : 减少/ 增加nice值，这样就可以提高/降低对应进程的优先级,F9 : 杀掉选中的进程, F10 : 退出htop

* htop命令的优点:

 * 快速查看关键性能统计数据，如CPU（多核布局）、内存/交换使用；
 * 可以横向或纵向滚动浏览进程列表，以查看所有的进程和完整的命令行；
 * 杀掉进程时可以直接选择而不需要输入进程号；
 *  通过鼠标操作条目；
 * 比top启动得更快；

![这里写图片描述](http://img.blog.csdn.net/20170703184704570?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

上图解析：

从上至下，分别为，cpu、内存、交换分区的使用情况，右边部分为：Tasks为进程总数，当前运行的进程数、Load average为系统1分钟，5分钟，10分钟的平均负载情况、Uptime为系统运行的时间，下面的参数就和top 一样了。

<br>
<br>

本人才疏学浅，若有错，请指出，谢谢！ 

参考资料：[用十条命令在一分钟内检查Linux服务器性能](http://www.infoq.com/cn/news/2015/12/linux-performance/)