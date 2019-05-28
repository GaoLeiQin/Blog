###**引入**


####**1.定义**

* I/O 多路复用技术是为了解决进程或线程阻塞到某个 I/O 系统调用而出现的技术，使进程不阻塞于某个特定的 I/O 系统调用。而select()，poll()，epoll()都是I/O多路复用的机制。

*  I/O多路复用是指内核一旦发现进程指定的一个或者多个IO条件准备读取，它就通知该进程。

####**2.IO多路复用的适用场合**

* 当客户处理多个描述符时（一般是交互式输入和网络套接口），必须使用I/O复用。

* 当一个客户同时处理多个套接口时，而这种情况是可能的，但很少出现。

* 如果一个TCP服务器既要处理监听套接口，又要处理已连接套接口，一般也要用到I/O复用。

* 如果一个服务器即要处理TCP，又要处理UDP，一般要使用I/O复用。

* 如果一个服务器要处理多个服务或多个协议，一般要使用I/O复用。


####**3.与多进程/多线程的比较**

I/O多路复用技术的最大优势是系统开销小，系统不必创建进程/线程，也不必维护这些进程/线程，从而大大减小了系统的开销。

####**4.I/O多路复用的阻塞性**

* I/O多路复用就是通过一种机制，一个进程可以监视多个描述符，一旦某个描述符就绪（一般是读就绪或者写就绪），能够通知程序进行相应的读写操作。

* select，poll，epoll本质上都是同步I/O，因为他们都需要在读写事件就绪后自己负责进行读写，也就是说这个读写过程是阻塞的，而异步I/O则无需自己负责进行读写，异步I/O的实现会负责把数据从内核拷贝到用户空间。




###**select、poll、epoll简介**


####**1.select**

（1）函数

```
int select (int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
```

参数:

* nfds：指定被监听的文件描述符的总数，通常被设置为select监听的所有文件描述符中最大值+1

* readfds：指向可读事件对应的文件描述符集合

* writefds：指向可写事件对应的文件描述符集合

* exceptfds：指向异常事件对应的文件描述符集合

* timeout：设置select函数的超时时间（s、us），若调用失败，timeout的值是不确定的，所以说不能完全信任该值，

注：fd_set结构体仅包含一个整型数组，该数组的每一个元素的每一位(bit)标记一个文件描述符，fd_set能容纳的文件描述符数量由FD_SETSIZE指定，这就限制了select能同时处理文件描述符的总量。

（2）原理

select 函数监视的文件描述符分3类，分别是writefds、readfds、和exceptfds。调用后select函数会阻塞，直到有描述符就绪（有数据 可读、可写、或者有except），或者超时（timeout指定等待时间，如果立即返回设为null即可），函数返回。当select函数返回后，可以通过遍历fdset，来找到就绪的描述符。

（3）流程图

![这里写图片描述](http://img.blog.csdn.net/20170704185243649?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

（4）优点

* 良好跨平台性：select目前几乎在所有的平台上都支持


（5）缺点

* 单个进程能够监视的文件描述符的数量存在最大限制，32位机默认是1024，64位机默认是2048，虽然可以通过修改FD_SETSIZE提高限制，但会降低效率。

* 对socket进行线性扫描，即采用轮询的方法，效率较低，时间复杂度为O（N）

* 由于select需要维护一个用来存放大量fd的数据结构，这样会使得用户空间和内核空间在传递该结构时复制开销大。

####**2.poll**

（1）函数

```
int poll (struct pollfd *fds, unsigned int nfds, int timeout);
```

参数：

* fds
select使用三个位图来表示三个fdset，而poll使用一个 pollfd的指针实现。
```
struct pollfd {
    int fd; //文件描述符
    short events; //注册的事件（可读、可写、异常、挂起、fd未打开）
    short revents; //实际发生的事件，由内核填充
};
```

* nfds：指定被监听事件集合fds的大小

* timeout：指定poll的超时值（ms）


（2）原理

poll本质上和select没有区别，它将用户传入的数组拷贝到内核空间，然后查询每个fd对应的设备状态，如果设备就绪则在设备等待队列中加入一项并继续遍历，如果遍历完所有fd后没有发现就绪设备，则挂起当前进程，直到设备就绪或者主动超时，被唤醒后它又要再次遍历fd。它是水平触发。

（3）优点

* 没有最大连接数的限制，因为它是基于链表来存储的，

（4）缺点

* 大量的fd的数组被整体复制于用户态和内核地址空间之间，开销大。

* 对socket进行线性扫描，即采用轮询的方法，效率较低，时间复杂度为O（N）。

####**3.epoll**

（1）三个接口函数

```
int epoll_create(int size)；
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event)；
int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);
```

三个函数的作用：

* epoll_create：创建一个epoll的句柄，size用来告诉内核这个监听的数目一共有多大

* epoll_ctl：对指定描述符fd执行op操作。
 * epfd表示epoll_create()的返回值
 * op用三个宏来表示：
EPOLL_CTL_ADD：注册新的fd到epfd中；
EPOLL_CTL_MOD：修改已经注册的fd的监听事件；
EPOLL_CTL_DEL：从epfd中删除一个fd；
 * fd表示需要监听的fd
 * event表示监听的事件。
struct epoll_event结构如下：

```
struct epoll_event {
__uint32_t events; /* Epoll events */
epoll_data_t data; /* User data variable */
};
```
events可以是以下几个宏的集合：

```
EPOLLIN ：表示对应的文件描述符可以读（包括对端SOCKET正常关闭）；
EPOLLOUT：表示对应的文件描述符可以写；
EPOLLPRI：表示对应的文件描述符有紧急的数据可读（这里应该表示有带外数据到来）；
EPOLLERR：表示对应的文件描述符发生错误；
EPOLLHUP：表示对应的文件描述符被挂断；
EPOLLET： 将EPOLL设为边缘触发(Edge Triggered)模式，（相对于LT）
EPOLLONESHOT：只监听一次事件，当监听完这次事件之后，如果还需要继续监听这个socket的话，需要再次把这个socket加入到EPOLL队列里
```

* epoll_wait：等待epfd上的io事件，最多返回maxevents个事件（不能大于创建epoll_create()时的size）；timeout是超时时间。

（2）原理

* epoll是在2.6内核中提出的，是select和poll的增强版本。

* epoll支持水平触发和边缘触发，最大的特点在于边缘触发，它只告诉进程哪些fd刚刚变为就绪态，并且只会通知一次。

* epoll使用事件的就绪通知方式，通过epoll_ctl注册fd，一旦该fd就绪，内核就会采用类似callback的回调机制来激活该fd，epoll_wait便可以收到通知。


（3）epoll的优点

* 没有最大并发连接的限制：能打开的FD的上限远大于1024（1G的内存上能监听约10万个端口）。

* 效率提升：不是轮询的方式，不会随着FD数目的增加效率下降。只有活跃可用的FD才会调用callback函数；

* 内存拷贝：利用一个文件描述符管理多个描述符，将用户关系的文件描述符的事件存放到内核的一个事件表中，在用户空间和内核空间的copy只需一次。mmap()文件映射内存加速与内核空间的消息传递；即epoll使用mmap减少复制开销，

* EPOLLONESHOT事件：对于注册了该事件的文件描述符，内核最多触发其上注册的一个可读、可写或异常事件，且只触发一次，除非使用epoll_ctl函数重置该事件。

（3）LT与ET

* LT(level triggered)模式（默认）：
 * 当epoll_wait检测到描述符事件（可读或可写）发生并将此事件通知应用程序，可以对这个就绪的fd进行IO操作。如果不作任何操作。下次调用epoll_wait时，会再次响应应用程序并通知此事件。
 * LT同时支持block和no-block socket

* ET(edge-triggered)模式（高速工作方式）：
 * 当epoll_wait检测到描述符事件（可读-->可写或可写-->可读）发生并将此事件通知应用程序，应用程序必须立即处理该事件。如果不处理，下次调用epoll_wait时，不会再次响应应用程序并通知此事件。
 * ET只支持no-block socket。

* ET减少了epoll事件被重复触发的次数，因此效率要比LT模式高。epoll工作在ET模式的时候，必须使用非阻塞套接口，以避免文件描述符的饥饿情况发生。



###**select、poll、epoll的实现机制**

####**1.select/poll实现机制**


![这里写图片描述](http://img.blog.csdn.net/20170704195912889?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**分析：**


* select/poll每次都需要重复传递全部的监听fd进来，涉及用户空间和内核直接的数据拷贝。

* fd事件回调函数是pollwake，只是将本进程唤醒，本进程需要重新遍历全部的fd检查事件，然后保存事件，拷贝到用户空间，函数返回。

* 每次循环都是对全部的监测的fd进行轮询检测，可能发生事件的fd很少，这样效率很低。

* 当有事件发生，需要返回时，也需要将全部fd的事件进行返回，而其中可能只有很少的fd有事件发生。

* select/poll返回时，会将该进程从全部监听的fd的等待队列里移除掉，这样就需select/poll每次都要重新传入全部监听的fd，然后重新将本进程挂载到全部的监测fd的等待队列，大量重复工作，效率极低。


####**2.epoll实现机制**

![这里写图片描述](http://img.blog.csdn.net/20170704200108738?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


**分析：**

* 每次累加添加，不需要每次传入全部的监测fd。

* 每个fd只将本进程挂载到自己的等待队列一次，直到该fd被从epoll移除，不需要重复挂载。

* fd事件回调函数是ep_epoll_callback，该函数将发生事件的fd加入到epoll专门的就绪队列rdllist中，同时唤醒本进程。

* 进程不需要遍历每一个fd去监测事件是否发生，而只需要判断epoll中的就绪队列rdllist是否为空即可。而rdllist是eventpoll的链表节点, 所有已经ready的epitem都会被链到rdllist中.

* epoll返回时，只返回就绪队列rdllist中的项，避免了无关项的操作，应用层也就不需要再次重复遍历。

* epoll内部使用红黑树存储监测fd，支持大量fd的快速查询、修改和删除操作。


####**3.两种机制的异同**

* 主要监测流程一样：都需要将当前进程挂载到对应fd的队列中去。如果fd有事件发生，调用挂载的回调函数，该回调函数基本的作用是唤醒本进程。

* 主事件检测循环一样：循环检测是否有事件发生，有则处理事件后返回；没有则调用schedule_timeout睡眠一会。

* 不同点：select/poll直接检测每个fd，而epoll只需检测就绪队列rdllist是否有数据即可。

###**select、poll、epoll的区别与适用场景**

####**1.三个函数的区别**

注：在并发连接数（socket）较多的情况下，

|描述|select|poll|epoll|
|:----|:-----|:----|:-----|
|事件集合|通过三个fd_set参数传入感兴趣事件，每次调用select需重置参数来反馈就绪事件|通过参数pollfd.events传入感兴趣事件，每次调用只需修改参数来反馈就绪事件|通过一个事件表管理所有感兴趣事件，每次调用epoll_wait的events参数反馈就绪事件|
|最大连接数|1024（x86） or 2048（x64）|无上限|有上限，但很大|
|IO效率|采用轮询方式检测就绪事件|采用轮询方式检测就绪事件|采用回调方式检测就绪事件|
|fd拷贝|将消息由内核态拷贝到用户态|将消息由内核态拷贝到用户态|通过内核和用户空间由共享内存实现|
|时间复杂度|O（n）|O（n）|O（1）|
|工作模式|LT|LT|LT或ET|


<br>
####**2.适用场景**

* 若同时需要保持大量长连接，而且连接的关闭很频繁时，即在长连接服务器的场景中，虽然同时可能需要维持数十万条长连接，但其中只有少数的连接是活跃的就能够发挥epoll最大的优势了。

* 若有大量的idle-connection或者dead-connection，使用select/poll，但是当遇到大量的idle-connection，使用epoll的效率大大高于select/poll。

* 若在连接数少并且连接都十分活跃的情况下，select和poll的性能可能比epoll好，毕竟epoll的通知机制需要很多函数回调。




<br>
<br>
本人才疏学浅，若有错，请指出，谢谢！ 
如果你有更好的建议，可以留言我们一起讨论，共同进步！ 
衷心的感谢您能耐心的读完本篇博文！

> 参考资料：
> 1.[epoll源码剖析](http://blog.csdn.net/baiye_xing/article/details/76352935)
> 2. [epoll源码的函数调用流程分析图](http://blog.csdn.net/baiye_xing/article/details/76360290)
> 3.[Linux select/poll和epoll实现机制对比](http://www.cnblogs.com/NerdWill/p/4996476.html)
> 4.[ Linux下select, poll和epoll IO模型的详解](http://blog.csdn.net/tianmohust/article/details/6677985/)
5.[聊聊IO多路复用之select、poll、epoll详解](http://www.jianshu.com/p/dfd940e7fca2)
6. [I/O多路复用select、poll、epoll的区别使用](http://blog.csdn.net/tennysonsky/article/details/45745887)


