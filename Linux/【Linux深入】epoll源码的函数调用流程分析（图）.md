###**引入**

* 我在上一篇博文中讲了[epoll源码的剖析](http://blog.csdn.net/baiye_xing/article/details/76352935)，你是不是看的有点懵呢，反正我是有点，接下来我就以流程图的形式梳理一下epoll源码的结构。

* 当然，这篇博文是建立在上一篇博文的基础上，若你还没看过epoll源码，那么我建议你最好还是看一下，请点击[【Linux深入】epoll源码剖析
](http://blog.csdn.net/baiye_xing/article/details/76352935)

* 接下来我就以流程图的形式介绍一下函数的调用过程。

###**整体的数据结构图**

![这里写图片描述](http://img.blog.csdn.net/20170729202604214?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

注：图中黄色和绿色方框表示链表关系，而粉色代表等待队列。

###**主要的函数调用图**

####**1.epoll_create()**

![这里写图片描述](http://img.blog.csdn.net/20170729203023873?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

分析：

* epoll_create()函数没有什么实际作用，它仅有的一个size参数没有，它的主要功能是通过epoll_create1()函数来完成的

* epoll_create1()函数通过ep_alloc生成一个eventpoll对象，并初始化eventpoll的三个等待队列，wait，poll_wait以及rdlist （ready的fd list）。同时还会初始化被监视fs的rbtree 根节点。

* epoll_create1()在调用ep_alloc通过anon_inode_getfd创建一个名字为“[eventpoll]”的eventpollfs文件描述符号并将file->private_data指定为指向前面生成的eventpoll。这样就将eventpoll和文件id关联。最后返回文件描述符id


####**2.epoll_ctl()**

![这里写图片描述](http://img.blog.csdn.net/20170729204359009?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

* epoll_ctl函数
 * 通过epoll_create生成一个eventpoll后，可以通过epoll_ctl提供的相关操作对eventpoll进行ADD，MOD，DEL操作。
 * epoll_ctl首先通过epfd的private_data域获取需要操作的eventpoll，然后通过ep_find确认需要操作的fd是否已经在被监视的红黑树中（eventpoll->rbr）。然后根据op的类型分别作ADD（ep_insert），MOD（ep_modify），DEL（ep_remove）操作。

* ep_insert()
 * 从slub中分配一个epitem的对象epitem。并初始化epitem的三个list头指针，rdllink（指向eventpoll的rdlist），fllist指向（struct file的f_ep_links），pwqlist（指向包含此epitem的所有poll wait queue）。将epitem的ep指针，指向传入的eventpoll，
 * 通过传入参数event对ep内部变量event赋值。然后通过ep_set_ffd将目标文件和epitem关联。这样epitem本身就完成了和eventpoll以及被监视文件的关联。下面还需要做两个动作：将epitem插入目标文件的polllist并注册回调函数；将epitem插入eventpoll的rbtree。

* Poll()函数是系统函数，一般由设备驱动提供。

* poll_wait()函数调用 ep_ptable_queue_proc()函数

* ep_ptable_queue_proc()主要功能是使用等待队列实现epitem和callback函数的关联。然后通过目标文件的poll函数调用callback函数。

* ep_poll_callback()函数
 * 当fd状态改变时（感兴趣事件），调用该函数，
 * 首先会把文件描述符epitem 放到eventpoll 的rdllist中
 * 然后调用wake_up函数唤醒epitem上wq的进程。这样就可以返回到epoll_wait的调用者，将他唤醒。


####**3.epoll_wait()**

![这里写图片描述](http://img.blog.csdn.net/20170729212247508?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


* epoll_wait()函数
 * 首先会检测传入参数的合法性；events指向的空间是否可写；epfd是否合法等。参数合法性检测都通过后，将通过epfd获取锁依赖的struct file，
 * 然后通过file->private_data获取eventpoll。获取epoll后调用ep_poll函数完成真正的epoll_wait工作。

* ep_poll()函数
 * 首先根据timeout的值判断是否是无限等待，如果不是将timeout（ms）转换为jiffs。 
 * 然后判断eventpoll的rdlist是否为空，如果为空，那么将current进程通过一个waitquene entry加入eventpoll的waitlist（wq）。如果rdlist非空，那么通过ep_send_events将event转发到用户空间。

* ep_send_events函数调用ep_scan_ready_list对ready_list进行扫描

* ep_scan_ready_list对ready_list进行扫描。将所有的epitem都转移到了txlist上, 而rdllist被清空了。然后调用ep_send_events_proc()函数。

* ep_send_events_proc扫描rdlist从头上面拿出epitem，然后调用ep_eventpoll_poll函数，完成转发。
先把就绪事件链表转移到中间链表,然后挨个遍历拷贝到用户空间,并且挨个判断其是否为水平触发,是的话再次插入到就绪链表

####**4.全部函数的的调用关系图**


![这里写图片描述](http://img.blog.csdn.net/20170729214509475?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

注：忽略蓝色竖线，手当时抖了一下，多加了一条线。

<br>
<br>
本人才疏学浅，若有错，请指出，谢谢！ 
如果你有更好的建议，可以留言我们一起讨论，共同进步！ 
衷心的感谢您能耐心的读完本篇博文！

> 参考链接：
> 1.[epoll实现——嵌入式Linux](http://www.embeddedlinux.org.cn/html/yingjianqudong/201405/11-2859.html#)
> 2.[EPOLL内核源代码实现原理分析——it165](http://www.it165.net/os/html/201207/2975.html)